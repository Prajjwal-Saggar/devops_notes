# Docker 02 — Distroless, Hub, Volumes, Networking, Compose

## Distroless Images

> Your understanding is close — let's sharpen it slightly.

**What distroless images actually are:** images that contain **only your application and its runtime dependencies** — no shell, no package manager, no OS utilities, no `/bin/bash`, not even a `useradd`/`adduser` binary. A regular minimal image (like `alpine`) still has a tiny OS with a shell and basic utilities; a **distroless** image strips even that away.

**Why this makes them more secure:**
* No shell (`bash`/`sh`) inside the container → if an attacker somehow gets code execution, they can't just "drop into a shell" and poke around, install tools, or move laterally — there's no shell to drop into.
* No package manager → nothing to `apt install` malicious tools with, even if they wanted to.
* No user-management binaries → correct, you can't run `useradd`/`adduser` inside a distroless container simply because those programs don't exist in the image at all — it's not about permissions, the tools themselves are absent.
* Smaller attack surface overall — fewer binaries = fewer known CVEs/vulnerabilities to worry about.

**Trade-off:** you lose the ability to `docker exec -it container bash` in to debug — since there's no shell. Debugging distroless containers usually relies on external tools/logs rather than jumping inside.

---

## Docker Hub — Push / Pull

**Steps to store (push) an image to Docker Hub:**

1. Log in from your terminal:
   ```
   docker login
   ```
2. Docker Hub image names must follow the format: **`username/imagename`** — so you first tag your local image to match that naming convention:
   ```
   docker image tag oldname username/imagename:tag
   ```
   (`docker image tag` is how you **rename/re-tag** an image — it doesn't move or copy the actual image data, just adds a new name pointing to the same image.)
3. Push it:
   ```
   docker push username/imagename:tag
   ```

**Pull an image (from Docker Hub or elsewhere):**
```
docker pull username/imagename:tag
```

---

## `docker compose` (quick mention — full section below)

> A tool to define and run **multi-container** applications using a single YAML file (`docker-compose.yml`), instead of manually running multiple long `docker run` commands. Covered in full detail near the end of this doc.

---

## Docker Scout

> A built-in Docker feature for **image vulnerability scanning** — analyzes your image's layers and dependencies against known CVE databases, flags security issues, and suggests fixes (e.g. "upgrade base image X to fix Y known vulnerabilities").
```
docker scout cves image-name
docker scout quickview image-name
```
Useful for catching security problems in your image *before* deploying it, especially as part of a CI/CD pipeline.

---

## Cleanup Commands

```
docker rm $(docker ps -aq)         # remove ALL containers (running + stopped)
docker rmi $(docker images -aq)     # remove ALL images  (note: it's "rmi", not "rm i")
docker system prune                  # remove all stopped containers, unused networks, dangling images, and build cache
docker system prune -a                # more aggressive — also removes ALL unused images, not just dangling ones
```
⚠️ These are destructive — always double-check before running on a server that has stuff you still need.

---

## Docker Volumes

> Containers are **ephemeral** — when a container is removed, any data written inside it is lost. **Volumes** solve this by storing data **outside** the container's writable layer, on the host (managed by Docker), so data survives even if the container is deleted/recreated.

**Full workflow — MySQL example:**

```
docker volume create mysql-data          # create a named volume
docker volume ls                          # list all volumes
docker volume inspect mysql-data           # see details: where it lives on the host, etc.
```

**Run MySQL with an environment variable (no volume yet — data would be lost if container dies):**
```
docker run -d -e MYSQL_ROOT_PASSWORD=root mysql:latest
```
* `-e` → sets an **environment variable** inside the container (here, MySQL's root password).

**Get a shell inside the running container:**
```
docker exec -it container_id bash
mysql -u root -p
```

**Now run it properly, WITH the volume attached:**
```
docker run -d -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:latest
```
* `-v mysql-data:/var/lib/mysql` → **volume mapping**, format is `volume-name:path-inside-container`.
* `mysql-data` = the volume you created (lives on the host, managed by Docker).
* `/var/lib/mysql` = where MySQL stores its actual data *inside* the container's filesystem (this exact path is specific to each image/app — you'd check the image's docs, or Google "where does X store its data").
* Now, even if this container is deleted, the actual database files persist inside the `mysql-data` volume — you can attach a new container to the same volume and pick up right where you left off.

### General pattern for volume-mapping ANY app

1. Figure out **where the app stores its important/persistent data** inside the container (check the image's documentation — e.g. Postgres → `/var/lib/postgresql/data`, MongoDB → `/data/db`, WordPress → `/var/www/html`).
2. Create a volume (or just let Docker auto-create one by naming it inline).
3. Run the container with `-v your-volume-name:/that/path/inside/container`.
4. Now that specific directory's data is safely stored outside the container's lifecycle.

---

## Docker Networking

### The "Tiers" (application architecture context, not Docker-specific)
* **1 Tier — Presentation Tier** → the frontend (UI the user interacts with)
* **2 Tier — Logical Tier** → the backend (business logic/application server)
* **3 Tier — Database Tier** → where data is stored

(This is general software architecture terminology — Docker networking becomes relevant when these tiers run as *separate containers* that need to talk to each other.)

### Inspecting & Managing Networks
```
docker inspect container-name-or-id       # full details on a container, including its network settings
docker network ls                          # list all networks
docker network rm network-name-or-id        # remove a network
```

### Docker Network Types

| Type | What it does |
|---|---|
| **Bridge** (default) | Creates an isolated internal network on the host; containers on it can talk to each other, and reach the outside world via NAT. This is the default network new containers join if you don't specify one. |
| **None** | No networking at all — the container is completely isolated, no network access. |
| **Host** | Container shares the host machine's network directly — no port mapping needed/possible, since the container just uses the host's actual network stack. |
| **User-defined bridge** | A custom bridge network *you* create — preferred over the default bridge because containers on it can resolve each other **by container name** automatically (built-in DNS), which the default bridge doesn't support well. |
| **MACVLAN** | Gives a container its own MAC address, making it appear as a physical device directly on your network. |
| **IPVLAN** | Similar to MACVLAN but shares the host's MAC address, using IP-level routing instead. |
| **Overlay** | Connects containers **across multiple different Docker hosts/machines** — used for multi-host container communication. |

**Your note is accurate:** MACVLAN, IPVLAN, and Overlay are less commonly hand-configured today in real-world setups because **Kubernetes** (with its own networking model/CNI plugins) has largely taken over that "multi-host, complex networking" use case — Docker's own networking is mostly used for simpler single-host or docker-compose setups now.

---

## Real-Life Scenario — Two-Tier App (Java Backend + MySQL Database)

**Goal:** run a Java app and a MySQL database as separate containers, on the same custom network, so the Java app can reach MySQL by container name.

**Step 1 — Create a user-defined bridge network:**
```
docker network create twotier
```

**Step 2 — Create a volume for MySQL's persistent data:**
```
docker volume create mysql_data
```

**Step 3 — Run the MySQL container on that network:**
```
docker run -d \
  --name mysql-db \
  --network=twotier \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=appdb \
  mysql:latest
```

**Step 4 — Run the Java app container on the SAME network, connecting to MySQL by container name:**
```
docker run -d \
  -p 8080:8080 \
  -v mysql_data:/var/lib/mysql \
  --network=twotier \
  -e DB_HOST=mysql-db \
  -e DB_USER=root \
  -e DB_PASSWORD=root \
  --name=java-app \
  java-app-image
```
* Since both containers are on the custom `twotier` network, the Java app can connect to MySQL using the hostname `mysql-db` (the container's `--name`) — no need for IP addresses, Docker's built-in DNS resolves it automatically.
* `-p 8080:8080` exposes the Java app itself to the outside world; MySQL usually does **not** need a `-p` here since only the Java container needs to reach it internally (not the outside world).

### If Something Fails — Debugging Steps

1. **Check if both containers are actually running:**
   ```
   docker ps
   ```
   If one isn't listed, check `docker ps -a` to see if it exited/crashed.

2. **Check the container's logs for errors:**
   ```
   docker logs mysql-db
   docker logs java-app
   ```
   This is usually the first and most useful step — most connection errors show up clearly here (e.g. "Access denied," "Connection refused," wrong password, etc.)

3. **Confirm they're actually on the same network:**
   ```
   docker network inspect twotier
   ```
   Should list both containers under that network.

4. **Get inside the Java container and manually test connectivity to MySQL:**
   ```
   docker exec -it java-app bash
   ping mysql-db
   # or, if a MySQL client is available inside:
   mysql -h mysql-db -u root -p
   ```
   If `ping`/connection fails here, it's a networking issue, not an app-config issue.

5. **Double-check environment variables match what the app expects:**
   ```
   docker exec -it java-app env
   ```
   Compare `DB_HOST`, `DB_USER`, `DB_PASSWORD` against what your app's code actually reads.

6. **Check if MySQL is actually ready/finished initializing** — MySQL containers can take a few seconds to fully start on first run; if the Java app starts too fast and tries to connect before MySQL is ready, it'll fail. (This is exactly what a **healthcheck** — covered below in Docker Compose — is meant to solve.)

7. **Inspect the specific container in detail if still stuck:**
   ```
   docker inspect java-app
   ```
   Shows full config — network settings, mounted volumes, env vars, everything — good for spotting a typo/misconfiguration.

---

## Docker Compose

> Lets you define your entire multi-container setup (services, networks, volumes, env vars, ports) in one `docker-compose.yml` file, and spin it all up/down with a single command — instead of typing out multiple long `docker run` commands by hand.

### Your `docker-compose.yml`, Corrected

(A few syntax issues in the original: needs a space after every `:`, YAML is indentation-sensitive, and several keys were misspelled — `environement` → `environment`, `voulmes` → `volumes`, `heatlthcheck` → `healthcheck`, and `depends_on` needs to be a list.)

```yaml
services:
  mysql:
    image: mysql:5.7
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: admin
    volumes:
      - ./mysql_data_new:/var/lib/mysql
    networks:
      - twotier
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  flask-app:
    build:
      context: .
    container_name: flask_app_container
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - twotier
    restart: always

volumes:
  mysql_data_new:

networks:
  twotier:
```

### `healthcheck` Fields Explained

> A healthcheck tells Docker how to actively check whether a container is *actually* ready/working — not just "running," but genuinely healthy and able to serve requests. This is exactly the fix for the "app starts before the database is ready" problem from the debugging section above.

| Field | Meaning |
|---|---|
| **`test`** | The actual command Docker runs to check health. If it exits with code 0, the container is considered healthy. Here, `mysqladmin ping` checks if MySQL is actually accepting connections. |
| **`interval`** | How often Docker runs the health check (e.g. every `10s`). |
| **`timeout`** | How long to wait for the health check command itself to respond before considering that individual check a failure (e.g. `5s`). |
| **`retries`** | How many consecutive failures are allowed before Docker marks the container as **unhealthy** (e.g. `5` failed checks in a row = unhealthy). |
| **`start_period`** | A grace period after the container first starts, during which failed checks **don't count** against the retry limit — gives slow-starting apps (like MySQL on first boot) time to actually get ready before being judged. |

**Why `depends_on: mysql: condition: service_healthy` matters:** plain `depends_on: mysql` (like in your original) only waits for the MySQL **container to start** — not for MySQL to actually be *ready* to accept connections. Using the healthcheck + `condition: service_healthy` makes `flask-app` wait until MySQL is genuinely ready, directly solving the "app crashes because DB wasn't ready yet" problem.

### `docker compose up` / `down` and Useful Options

```
docker compose up              # create + start all services (attached, shows logs in terminal)
docker compose up -d            # same, but detached (runs in background)
docker compose up --build        # force rebuild of images before starting
docker compose up --force-recreate  # recreate containers even if config hasn't changed

docker compose down              # stop and remove containers + networks (keeps volumes by default)
docker compose down -v            # ALSO remove volumes (⚠️ deletes your persistent data — use carefully)
docker compose down --rmi all       # also remove the images built/used by the compose file

docker compose ps                 # list running services in this compose project
docker compose logs                # view logs for all services
docker compose logs -f service-name  # follow (live-tail) logs for one specific service
docker compose restart            # restart all services
docker compose stop / start        # stop/start without removing containers
```
