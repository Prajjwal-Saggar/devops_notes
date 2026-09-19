# Linux 01

1. Operating System
2. Client OS vs Server OS
3. Linux OS
4. How Linux works
5. Commands Practice
6. NGINX Server

## Why Linux?

* It's open source.
* Linux runs the vast majority of the world's servers and powers nearly all top supercomputers.
* Linux machines are generally considered more secure than Windows, largely due to permission structure and smaller attack surface.
* Multiuser, multitasking.
* A lot of flavours (distros) -> Ubuntu, Red Hat (RHEL), CentOS, Kali Linux.
* Linux is a Unix-like OS — inspired by UNIX (the original, all-black-screen, shell-based system) but built independently by Linus Torvalds, not literally "started from" UNIX.
* Ubuntu (beginner-friendly) -> RHEL (more advanced, enterprise-focused).
* Everything in Linux is either a file or a directory.
* Commands in **sbin** are meant to be run by the **root user** (system administration), whereas commands in **bin** can be run by a **normal user**.

> Application -> Shell -> Kernel -> Hardware

* Vim is heavily used in the industry, but this is a matter of preference — plenty of engineers still use nano, especially for quick edits.

## Sudo

> `sudo` is a command that lets a permitted user run commands with root (superuser) privileges. On many distros, membership in the **sudo** (or **wheel**) group is what grants this permission.
> Example: `sudo systemctl start nginx`

## NGINX

> NGINX is a web server that can also be configured as a reverse proxy (as well as a load balancer). As a web server, it's commonly used to serve web pages and static content.
> By default, the pages served by NGINX are stored in `/var/www/html`.


# SSH, Bastion Host & SCP

## SSH (Secure Shell)

> Keys
* Public key
* Private key

* EC2 holds the **public key**, and we hold the **private key**. The reason: anyone can technically try to connect to the AWS instance, but only someone holding the matching private key is granted access.
* The command to SSH into an AWS instance:

```
ssh -i /path/to/key.pem ec2-user@<public-dns-or-ip>
```
(Replace `/path/to/key.pem` with your private key file, and use the public DNS / public IP shown in the AWS console.)

## What is a Jump Server?

> A Jump Server (also called a "jump box") is an intermediate server used as a single, controlled entry point to access other servers in a private network. Instead of exposing every internal server to the internet, you expose just the jump server — all traffic hops through it.

## What is a Bastion Host / SSH Jump / SSH Pivoting?

> A Bastion Host is essentially a hardened jump server, usually sitting in a public subnet, whose sole purpose is to let admins securely SSH into private servers that have no direct internet access. "SSH Jump" / "SSH Pivoting" refers to the technique of hopping from the bastion into those private instances.

> [!Note]
> The machine you're SSH-ing **from** should have the **private key**, and the target server you're SSH-ing **into** should have the corresponding **public key** in its `~/.ssh/authorized_keys` file. (In a bastion setup, you typically use SSH agent forwarding so your private key never has to sit on the bastion itself — it stays on your local machine and is forwarded through.)

* `ssh-keygen` is the command used to generate a new public/private key pair:
```
ssh-keygen -t rsa -b 4096
```
This creates a private key (kept secret) and a matching public key (`.pub`), which gets placed in the target server's `authorized_keys` file to allow access.

## SCP (Secure Copy)

### How to transfer a file from one server to another using SCP, and its alternatives

> `scp` copies files/directories between hosts over SSH, using the same key-based authentication.

```
scp -i /path/to/key.pem localfile.txt ec2-user@<public-dns>:/remote/path/
```

* To copy a whole directory, add the `-r` (recursive) flag.
* To copy server-to-server (not through your local machine), you can run `scp` directly between two remote hosts if they can reach each other.

**Alternatives to SCP:**
* **rsync** — like scp but supports incremental transfers (only copies changed parts of files), resuming interrupted transfers, and is generally faster/more efficient for large or repeated syncs.
  ```
  rsync -avz -e "ssh -i /path/to/key.pem" localfile.txt ec2-user@<public-dns>:/remote/path/
  ```
* **sftp** — an interactive, FTP-like session over SSH; useful when you need to browse/navigate the remote filesystem rather than run one-off copy commands.





