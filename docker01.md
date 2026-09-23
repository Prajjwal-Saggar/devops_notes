# Docker 01

> Docker is often described as a **package management tool** for applications — it packages an app *plus everything it needs to run* (code, dependencies, runtime, system libraries) into one portable unit called a **container**, so it runs the same way everywhere.

---

## Virtualization vs Docker

| | Virtualization (VMs) | Docker (Containers) |
|---|---|---|
| What it virtualizes | The **entire hardware** — each VM runs its own full OS (kernel included) | Just the **application layer** — containers share the host machine's OS kernel |
| Size | Heavy — several GBs per VM (full OS included) | Lightweight — usually MBs, since there's no duplicate OS |
| Boot time | Slow — minutes (booting a full OS) | Fast — seconds (just starting a process) |
| Isolation | Very strong (separate kernels entirely) | Process-level isolation (shared kernel, isolated via namespaces/cgroups) |
| Resource usage | Higher — each VM reserves its own OS overhead | Lower — many containers can share host resources efficiently |
| Managed by | A **Hypervisor** (e.g. VMware, VirtualBox) | The **Docker Engine** |

**In short:** a VM virtualizes the whole computer, a container virtualizes just the app — which is why containers are so much faster and lighter.

---

## How Docker Does What It Does (underlying process, in short)

1. You write a **Dockerfile** describing how to build your app's environment.
2. Docker builds an **image** from it — a read-only template/blueprint.
3. When you run that image, Docker creates a **container** — a running instance of that image.
4. The **Docker Engine** talks to the host machine's **OS kernel** directly (not a separate virtual OS) to create an isolated process — using Linux kernel features like **namespaces** (isolation — the container thinks it's alone on the machine) and **cgroups** (resource limits — CPU/memory caps).
5. So a container is really just a regular process on the host, but boxed in and made to *feel* isolated — that's the whole trick, and why it's so much lighter than a VM.

**Your note is correct:** the Docker Engine accesses the host OS's kernel directly and gives us that isolated process — which is what we call a "container."

---

## Docker's Internal Architecture

```
Docker CLI (docker run, docker build, etc.)
        │
        ▼
   dockerd (Docker Daemon)
        │
        ▼
   containerd
        │
        ▼
     runc  →  actually creates the container (kernel-level)
```

### `dockerd` (Docker Daemon)

> The background service that does the heavy lifting when you type any `docker` command — it manages images, networks, volumes, and delegates the actual container creation/running down to `containerd`. It's the "brain" you're talking to whenever you run a Docker CLI command.

### `containerd`

> A separate, lower-level daemon that `dockerd` relies on — its job is specifically **container lifecycle management**: pulling images, creating containers, starting/stopping them, managing storage, and passing the actual low-level creation work further down to `runc` (which does the real kernel-level work of spinning up the isolated process). `containerd` is actually its own independent project (donated to the CNCF) — other tools like Kubernetes use it directly too, not just Docker.

**Quick summary chain:** you run a command → `dockerd` receives it → hands the container work to `containerd` → `containerd` hands it to `runc` → `runc` talks to the Linux kernel to actually create the isolated process.

---

## Image → Container

* **Docker Image** → a read-only, static **blueprint/template** — contains the app code, dependencies, and instructions on how to run it. Doesn't run by itself.
* **Container** → a **running instance** of an image — the live, executing version. You can spin up multiple containers from the same image.

```
docker image → docker run → container
```

---

## `docker run` and Its Flags

**Basic:**
```
docker run image-name
```
Runs a container from the given image.

**`docker run -it`**
```
docker run -it ubuntu bash
```
* `-i` → interactive (keeps STDIN open, so you can type input)
* `-t` → allocates a pseudo-terminal (TTY), so it *feels* like a normal terminal session
* Together (`-it`), this lets you get an actual interactive shell **inside** the container — commonly used to jump into a container and poke around manually.

**`-itd` meaning:**
* Combines `-it` (interactive terminal) **with** `-d` (detached — runs in the background instead of tying up your terminal).
* In practice, `-itd` gives you a container that runs in the background but still has a terminal allocated, so you can attach to it later (`docker attach`) and get an interactive session if needed.

### Breaking Down: `docker run -d -p 80:80 nginx`

```
docker run -d -p 80:80 nginx
```

| Part | Meaning |
|---|---|
| `docker run` | Create and start a new container |
| `-d` | **Detached mode** — run the container in the background, don't block your terminal |
| `-p 80:80` | **Port mapping** — `hostPort:containerPort`. Maps port 80 on your actual machine to port 80 inside the container, so traffic hitting your machine on 80 gets forwarded into the container's port 80 |
| `nginx` | The **image** to run — pulls it from Docker Hub if not already present locally, then starts a container from it |

**Put together:** *"Start an NGINX container in the background, and make it reachable on my machine's port 80."* If you visit `http://localhost` afterward, you'd see NGINX's default page — because port 80 on your host is now forwarded straight into the container's port 80, where NGINX is listening.

(Note: the two 80s don't have to match — e.g. `-p 8080:80` would let you access it via `localhost:8080` while NGINX still listens on 80 *inside* the container.)

---

## Single-stage vs Multi-stage Builds (and "DHI")

* **Single-stage build** → the whole Dockerfile runs in **one stage** — build tools, source code, and the final runtime all end up in the same final image. Simple, but the resulting image is often bloated (compilers, build caches, dev dependencies all left inside).

* **Multi-stage build** → uses **multiple `FROM` stages** in one Dockerfile. You build/compile your app in an early "builder" stage (with all the heavy build tools), then copy **only the final built artifacts** into a clean, minimal final stage. The build tools/intermediate junk never make it into your final image — much smaller, more secure image.

* **DHI (Docker Hardened Images)** → Docker's own line of minimal, security-hardened base images (much smaller attack surface, fewer vulnerabilities, regularly patched) — designed to be used as the base for the *final* stage of a multi-stage build, instead of a generic full OS image. Using a hardened/minimal base image (DHI, or alternatives like `alpine`/`distroless`) for your final stage is best practice for production images.

**Rule of thumb:** always prefer multi-stage builds + a minimal final base image (like a DHI, `alpine`, or `distroless` image) for anything going to production — keeps images small, fast to pull/deploy, and reduces your security attack surface.

---

## Optimal Dockerfiles — Python, Node.js, Java

All three below use **multi-stage builds** and a **minimal final image** — the current best-practice approach.

### Python (Flask/FastAPI-style app)

```dockerfile
# ---------- Stage 1: Build ----------
FROM python:3.12-slim AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

COPY . .

# ---------- Stage 2: Final (minimal runtime) ----------
FROM python:3.12-slim

WORKDIR /app

# Copy only installed packages + app code from builder — no build cache/tools carried over
COPY --from=builder /root/.local /root/.local
COPY --from=builder /app .

ENV PATH=/root/.local/bin:$PATH

EXPOSE 8000
CMD ["python", "app.py"]
```
**Why this way:** `pip install --user` keeps installed packages isolated and easy to copy across stages; `--no-cache-dir` avoids bloating the image with pip's download cache; `slim` base keeps things small without going full `alpine` (which can sometimes cause native-dependency build issues with Python).

### Node.js

```dockerfile
# ---------- Stage 1: Build ----------
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .
RUN npm run build   # if your app has a build step (e.g. TypeScript, React)

# ---------- Stage 2: Final (minimal runtime) ----------
FROM node:20-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist    # adjust to wherever your build output is
COPY package*.json ./

EXPOSE 3000
CMD ["node", "dist/index.js"]
```
**Why this way:** `npm ci` (not `npm install`) gives faster, reproducible installs from `package-lock.json`; `--omit=dev` skips dev-only dependencies you don't need at runtime; `alpine` base keeps Node images (which can otherwise get large) small; only the built output + `node_modules` get copied to the final stage, not your raw source/build tooling.

### Java (Spring Boot-style app, Maven)

```dockerfile
# ---------- Stage 1: Build ----------
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline    # cache dependencies separately (speeds up rebuilds)

COPY src ./src
RUN mvn clean package -DskipTests

# ---------- Stage 2: Final (minimal runtime) ----------
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```
**Why this way:** the build stage uses a full JDK + Maven image (needed to compile), but the final image only needs a **JRE** (not the full JDK/Maven toolchain) to actually *run* the compiled `.jar` — hugely reducing final image size; copying `pom.xml` and running `dependency:go-offline` before copying source code lets Docker cache the dependency layer, so rebuilds are much faster when only your code changes, not your dependencies.

### Common theme across all three
* **Separate build and runtime stages** — heavy tools stay in the builder, never ship to production.
* **Copy dependency manifests first, install, THEN copy source code** — maximizes Docker's layer caching so you're not reinstalling dependencies on every code change.
* **Use minimal final base images** (`slim`, `alpine`, hardened images) to reduce size and attack surface.
* **Skip dev dependencies / tests / unneeded tooling** in the final image.
