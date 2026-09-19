# Linux02

# User & Group Management in Linux

## User Management

| Task | Command |
|---|---|
| Create a user (with home directory) | `sudo useradd -m babitaji` |
| Create a user with a specific shell | `sudo useradd -m -s /bin/bash tappu` |
| Set/change a user's password | `sudo passwd babitaji` |
| Switch to another user | `su babitaji` (or `su - babitaji` to also load their environment/home dir) |
| Exit back to your previous user | `exit` |
| Delete a user (keep home directory) | `sudo userdel username` |
| Delete a user AND their home directory | `sudo userdel -r username` (`--remove-home` also works) |
| Modify an existing user (shell, home, etc.) | `sudo usermod -s /bin/zsh username` |
| Lock / unlock a user's password | `sudo passwd -l username` / `sudo passwd -u username` |
| Check password expiry / aging info | `sudo chage -l username` |

## Verifying Users

* Check if a user exists in the system: `ls /home/` (only shows users with a home dir — not fully reliable)
* Better — check the actual user database:
  ```
  cat /etc/passwd
  getent passwd username
  ```
* `getent passwd username` returns nothing if the user was fully deleted; a line of output means the user still exists.
* Check which user you currently are: `whoami`
* See who's currently logged in: `who` or `w`
* See a user's login history: `last username`

## Group Management

| Task | Command |
|---|---|
| Create a group | `sudo groupadd devops` |
| Delete a group | `sudo groupdel devops` |
| Rename a group | `sudo groupmod -n newname oldname` |
| Add an existing user to a group (secondary group) | `sudo usermod -aG devops babitaji` (⚠️ always use `-aG` together — `-G` alone *overwrites* all existing groups) |
| Remove a user from a group | `sudo gpasswd -d babitaji devops` |
| Change a user's primary group | `sudo usermod -g newgroup username` |
| Set a group password (rare, for `newgrp`) | `sudo gpasswd devops` |

## Verifying Groups

* See all groups on the system:
  ```
  cat /etc/group
  getent group
  ```
* See which groups a specific user belongs to:
  ```
  groups babitaji
  id babitaji
  ```
  (`id` also shows UID, GID, and all group memberships in one shot — most commonly used by DevOps engineers for a quick check)
* Check group membership from `/etc/group` directly: `getent group devops`

## Notes

* `/etc/passwd` → user account info (username, UID, GID, home dir, shell)
* `/etc/shadow` → encrypted passwords + password aging info (root-only readable)
* `/etc/group` → group definitions and their members
* `useradd`/`groupadd` are the low-level standard commands (used in scripts/automation). Debian/Ubuntu also has `adduser`/`addgroup`, which are more interactive/friendly wrappers — good for manual use, less common in automation/DevOps scripting since `useradd` is more predictable and portable across distros.

## File Ownership & Permissions (chmod, chown, chgrp)

> Every file/directory in Linux has an **owner (user)**, a **group**, and a set of **permissions** for owner / group / others. These three commands are how you manage that.

### chown — change owner (and/or group)

| Task | Command |
|---|---|
| Change owner of a file | `sudo chown babitaji file.txt` |
| Change owner AND group in one go | `sudo chown babitaji:devops file.txt` |
| Change owner recursively (whole directory) | `sudo chown -R babitaji:devops /var/www/html` |
| Change only the group via chown | `sudo chown :devops file.txt` |

### chgrp — change group only

| Task | Command |
|---|---|
| Change the group of a file | `sudo chgrp devops file.txt` |
| Change group recursively | `sudo chgrp -R devops /var/www/html` |

(In practice, most people just use `chown user:group` instead of a separate `chgrp` — but `chgrp` still shows up, especially in older scripts.)

### chmod — change permissions

> Permissions are Read (r=4), Write (w=2), Execute (x=1), for three groups: **owner, group, others**.

| Task | Command |
|---|---|
| Give owner full, group+others read/execute (common for scripts) | `chmod 755 script.sh` |
| Give owner+group read/write, others nothing | `chmod 660 file.txt` |
| Make a file executable | `chmod +x script.sh` |
| Remove write access for group | `chmod g-w file.txt` |
| Add read access for others | `chmod o+r file.txt` |
| Apply recursively to a directory | `chmod -R 755 /var/www/html` |

**Numeric (octal) cheat sheet:**
```
7 = rwx (read+write+execute)
6 = rw- (read+write)
5 = r-x (read+execute)
4 = r-- (read only)
0 = --- (no access)
```
So `chmod 755` = owner: rwx, group: r-x, others: r-x — very common for scripts/binaries.
`chmod 644` = owner: rw-, group: r--, others: r-- — very common for regular files.

### Checking current ownership/permissions

```
ls -l file.txt
```
Output like `-rwxr-xr-- 1 babitaji devops file.txt` reads as: permissions, owner, group, filename.
