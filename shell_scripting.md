# Shell Scripting — Full Notes

## 1. Variables — Predefined vs User-defined

* **Predefined (environment) variables** — already exist, set by the system. Examples: `$HOME`, `$USER`, `$PATH`, `$PWD`, `$SHELL`.
  ```
  echo $HOME
  echo $USER
  ```
* **User-defined variables** — you create them yourself:
  ```
  name="prajjwal"
  echo $name
  ```
  (No spaces around `=` — `name = "prajjwal"` will break.)

## 2. `read` and `echo`

* **`echo`** → prints/outputs text.
  ```
  echo "Hello World"
  ```
* **`read`** → takes input from the user and stores it in a variable.
  ```
  read name
  echo "Hello $name"
  ```
  With a prompt message built in:
  ```
  read -p "Enter your name: " name
  ```

## 3. `mkdir -p` vs without `-p`

* **`mkdir foldername`** → creates a single folder. Fails with an error if the parent directory doesn't exist, or if the folder already exists.
* **`mkdir -p a/b/c`** → creates the **entire nested path** in one shot, even if `a` and `b` don't exist yet — it creates whatever's missing along the way. Also won't throw an error if the folder already exists (safe to re-run).

**Rule of thumb:** in scripts, prefer `mkdir -p` — it's safer since it won't crash your script if the folder already exists or if parent folders are missing.

## 4. User Input vs Arguments

* **User input** → the script asks for input *while running*, using `read` (interactive).
* **Arguments** → values passed **when calling the script**, without needing to pause and ask:
  ```
  ./script.sh prajjwal mypassword
  ```
  Here `prajjwal` and `mypassword` are arguments passed straight in — no `read` needed, no interruption. Much better for automation since the script doesn't have to wait for a human to type something.

### Argument variables — `$0`, `$1`, `$@`, `$#`

| Variable | Meaning |
|---|---|
| `$0` | The name of the script itself |
| `$1` | The 1st argument passed |
| `$2` | The 2nd argument passed (and so on: `$3`, `$4`...) |
| `$@` | **All** arguments passed, individually — this skips `$0` and gives you just the arguments (`$1 $2 $3...`) |
| `$#` | The **count** of how many arguments were passed |

Example:
```
./script.sh alice bob
```
* `$0` → `./script.sh`
* `$1` → `alice`
* `$2` → `bob`
* `$@` → `alice bob`
* `$#` → `2`

## 5. Multi-line Comments — `<<HEREDOC`

Bash doesn't have a native multi-line comment symbol, so people fake one using a heredoc that goes nowhere:
```
<<comment
This is line 1 of my comment
This is line 2 of my comment
Anything in here is ignored
comment
```
The word after `<<` (here `comment`, could be `help`, `END`, anything) just needs to match on both the opening and closing line. Since it's not being piped into a command, it effectively acts as a "block comment."

## 6. Script: Create a New User Automatically

```bash
#!/bin/bash

<<help
This Script is to create the user automatically
help

echo "============ User Creation Initiated ============"
password="$2"
sudo useradd -m "$1"
echo -e "$password\n$password" | sudo passwd "$1"
echo "============ User Creation Completed ============"
```
* `$1` → username, `$2` → password (passed as arguments).
* `echo -e "$password\n$password"` sends the password twice (since `passwd` normally asks you to type it and confirm it), piped into `sudo passwd` so it doesn't need interactive input.

**Modern/cleaner alternative** — `chpasswd`:
```bash
echo "$1:$password" | sudo chpasswd
```
This sets the password in one line, `username:password` format, no need to send it twice — simpler and considered the more modern approach.

## 7. Escape Sequences — `\n` and `\t`

* `\n` → newline
* `\t` → tab space

These only work when `echo` is told to interpret them — which is where `-e` comes in.

### `-e` with `echo`

By default, `echo` prints `\n`/`\t` as literal characters, not as an actual newline/tab. `-e` tells `echo` to **interpret** escape sequences.
```
echo "Hello\nWorld"        # prints literally: Hello\nWorld
echo -e "Hello\nWorld"     # prints:
                            # Hello
                            # World
```
This is exactly why the user-creation script above uses `echo -e "$password\n$password"` — it needs an actual newline between the two password entries, not the literal text `\n`.

## 8. `wc` (word count)

> Counts lines, words, and characters/bytes in a file or input.
```
wc file.txt          # lines, words, characters — all three
wc -l file.txt        # only line count
wc -w file.txt        # only word count
wc -c file.txt        # only byte count
```
**Common real use:** `ls | wc -l` → count how many files/folders are in a directory.

## 9. `-y` flag (e.g. `apt install xyz -y`)

> Auto-confirms any "Do you want to continue? [Y/n]" prompt that would normally require you to manually type `y` and press enter.
```
sudo apt install nginx -y
```
Essential for scripts — a script can't respond to an interactive prompt on its own, so without `-y` the script would just hang waiting for input that never comes.

## 10. `/dev/null` — use case

> A special "black hole" file — anything written to it is discarded/thrown away. Used to suppress output you don't want to see or log.
```
command > /dev/null           # discard normal output (stdout)
command 2> /dev/null           # discard error output (stderr)
command > /dev/null 2>&1       # discard both output and errors
```
**Common real use:** cron jobs — you often don't want routine script output cluttering logs/emails, so you redirect it to `/dev/null` and only keep real errors (or vice versa).

## 11. If-Else and Comparison Operators

```bash
if [ "$1" -eq 10 ]; then
    echo "Equal to 10"
elif [ "$1" -gt 10 ]; then
    echo "Greater than 10"
else
    echo "Less than 10"
fi
```

**Numeric comparisons:**
| Operator | Meaning |
|---|---|
| `-eq` | equal to |
| `-ne` | not equal to |
| `-gt` | greater than |
| `-lt` | less than |
| `-ge` | greater than or equal to |
| `-le` | less than or equal to |

**String comparisons:**
| Operator | Meaning |
|---|---|
| `=` | strings are equal |
| `!=` | strings are not equal |
| `-z` | string is empty |
| `-n` | string is NOT empty |

## 12. Functions in Shell Scripts

```bash
greet() {
    echo "Hello, $1"
}

greet "Prajjwal"
```
* Functions let you reuse blocks of logic instead of repeating code.
* Arguments passed to a function work the same way as script arguments (`$1`, `$2`, etc.) — but scoped to that function call.

## 13. For Loop — Creating Multiple Folders

Your example (creates 10 folders — but note: this version just prints numbers, doesn't create folders yet):
```bash
for (( num=1; num<=10; num++ ))
do
    echo "$num"
done
```

**To actually create `fold1` through `fold10`:**
```bash
for (( num=1; num<=10; num++ ))
do
    mkdir -p "fold$num"
done
```
This loops `num` from 1 to 10, and each time creates a folder named `fold` + the current number (`fold1`, `fold2`, ... `fold10`).

---

## 14. Cron — Scheduling Automated Tasks

> **Cron** is a Linux job scheduler — it runs scripts/commands automatically at fixed times/intervals, without you needing to run them manually.

**Edit your cron jobs:**
```
crontab -e
```
(Not `cron-tab` — it's one word, `crontab`.) This opens your personal crontab file for editing.

**Cron syntax (the 5 asterisks):**
```
*  *  *  *  *   command-to-run
│  │  │  │  │
│  │  │  │  └── day of week (0-6, Sunday=0)
│  │  │  └───── month (1-12)
│  │  └──────── day of month (1-31)
│  └─────────── hour (0-23)
└────────────── minute (0-59)
```
Each `*` means "every" for that field. Examples:
```
0 2 * * *     → runs every day at 2:00 AM
*/15 * * * *  → runs every 15 minutes
0 0 1 * *     → runs at midnight on the 1st of every month
0 9 * * 1     → runs every Monday at 9:00 AM
```

**Tip:** use **"crontab guru"** (crontab.guru) — a free website where you type/build your cron schedule and it tells you in plain English exactly when it'll run. Great for double-checking your syntax before saving.

---

## 15. Automated Backup Script (Local, Zipped)

```bash
#!/bin/bash

SOURCE_DIR="/home/ubuntu/data"
BACKUP_DIR="/home/ubuntu/backups"
DATE=$(date +%F_%H-%M-%S)
BACKUP_NAME="backup_$DATE.tar.gz"

mkdir -p "$BACKUP_DIR"

echo "============ Backup Started ============"
tar -czf "$BACKUP_DIR/$BACKUP_NAME" "$SOURCE_DIR"
echo "Backup created at: $BACKUP_DIR/$BACKUP_NAME"
echo "============ Backup Completed ============"
```
* `tar -czf` → `c` = create archive, `z` = gzip compress it, `f` = filename to save as. This zips your folder into one compressed `.tar.gz` file.
* `$(date +%F_%H-%M-%S)` → generates a timestamp so every backup has a unique name (e.g. `backup_2026-09-18_14-30-00.tar.gz`) instead of overwriting the previous one.

**Schedule it with cron (run every day at 2 AM):**
```
0 2 * * * /home/ubuntu/backup.sh >> /home/ubuntu/backup.log 2>&1
```

---

## 16. `watch` command

> Repeatedly runs a command at a set interval and shows the live-updating output on screen — useful for monitoring something in real time without manually re-running a command.
```
watch -n 5 df -h     # re-run "df -h" every 5 seconds, refresh the screen
watch -n 2 "ls -la /home/ubuntu/backups"
```
Press `Ctrl+C` to stop.

---

## 17. Local Server Storage vs Artifactory (S3)

* **Local server storage** → files live on the same disk as your server (or an attached EBS volume). Fast to access, but:
  * Limited by disk size.
  * If the server/instance is lost or terminated, your backups could be lost too (single point of failure).
  * Not easily shareable/accessible from elsewhere.
* **S3 (used as an artifact/backup store)** → cloud object storage, completely separate from your server's disk.
  * Virtually unlimited storage, pay for what you use.
  * Durable — AWS replicates data across multiple facilities, far safer than a single disk.
  * Accessible from anywhere (with permissions), independent of any one server's lifecycle.
  * Ideal for backups specifically *because* it doesn't die when your instance does.

**In short:** local storage = fast and simple but risky as a *sole* backup location. S3 = the actual safe, durable place backups should end up.

---

## 18. Saving Backups to S3

Once you have a zipped backup locally, you upload it to an S3 bucket:
```
aws s3 cp /home/ubuntu/backups/backup_2026-09-18.tar.gz s3://your-bucket-name/backups/
```

---

## 19. AWS CLI — Installation, Connection, and IAM User Setup

**Install AWS CLI (on Ubuntu):**
```
sudo apt update
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version   # confirm installation
```

**Create an IAM user (AWS Console):**
1. Go to **IAM → Users → Create User**.
2. Give it a name (e.g. `backup-user`).
3. Attach a policy — for S3 access, something like `AmazonS3FullAccess` (or a scoped-down custom policy for just your bucket, which is better practice).
4. Go to that user → **Security credentials → Create access key**.
5. Save the **Access Key ID** and **Secret Access Key** — you'll need both, and the secret key is only shown once.

**Connect AWS CLI to your account:**
```
aws configure
```
It'll prompt for:
* AWS Access Key ID
* AWS Secret Access Key
* Default region (e.g. `ap-south-1`)
* Default output format (e.g. `json`)

---

## 20. Useful AWS S3 CLI Commands

```
aws s3 ls                              # list all your S3 buckets
aws s3 ls s3://your-bucket-name/       # list contents inside a specific bucket
aws s3 cp file.txt s3://your-bucket/    # copy/upload one file
aws s3 sync /local/folder s3://your-bucket/folder   # sync a whole folder — only uploads new/changed files, very efficient for repeated backups
aws s3 rm s3://your-bucket/file.txt      # delete a file from the bucket
```
`sync` is generally preferred over `cp` for backup scripts, since re-running it won't re-upload files that haven't changed — much faster on repeat runs.

---

## 21. Putting It All Together — Backup Script Evolution

**Step 1 — Backup to a local folder only:**
```bash
#!/bin/bash
SOURCE_DIR="/home/ubuntu/data"
BACKUP_DIR="/home/ubuntu/backups"
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="backup_$DATE.tar.gz"

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"
echo "Local backup done: $BACKUP_DIR/$BACKUP_FILE"
```

**Step 2 — Extend it to also upload to S3, with status printed:**
```bash
#!/bin/bash

SOURCE_DIR="/home/ubuntu/data"
BACKUP_DIR="/home/ubuntu/backups"
S3_BUCKET="s3://your-bucket-name/backups"
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="backup_$DATE.tar.gz"

mkdir -p "$BACKUP_DIR"

echo "============ Local Backup Started ============"
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"
echo "Local backup created: $BACKUP_DIR/$BACKUP_FILE"

echo "============ Uploading to S3 ============"
aws s3 cp "$BACKUP_DIR/$BACKUP_FILE" "$S3_BUCKET/"

if [ $? -eq 0 ]; then
    echo "Upload to S3 successful!"
else
    echo "Upload to S3 FAILED."
fi

echo "============ Backup Process Completed ============"
```
* `$?` → holds the exit code of the last command (`0` = success, anything else = failure) — this is how the script checks whether the S3 upload actually worked before printing success/failure.
* Schedule this whole thing with `crontab -e` the same way as before, for a fully automated local + cloud backup.

---

## 22. Good Practices for Writing Shell Scripts (in general)

* Always start with `#!/bin/bash` (the shebang) so the script knows which interpreter to run it with.
* **Quote your variables**: use `"$variable"` not `$variable` — protects against issues with spaces or empty values.
* Use `mkdir -p` instead of `mkdir` in scripts so it doesn't fail if the folder already exists.
* Check command success with `$?` (or `&&`/`||`) for anything that could fail — especially network/cloud commands like S3 uploads.
* Add clear `echo` status messages at each major step — makes debugging and reading logs much easier.
* Use meaningful variable names (`BACKUP_DIR`, not `x`) — future-you will thank present-you.
* Add comments (or a heredoc block comment) explaining what the script does, especially near the top.
* Redirect noisy/unwanted output to `/dev/null`, but never blindly discard *error* output you might actually need to debug later.
* Test scripts manually before putting them on a cron schedule — a broken script running silently at 2 AM is hard to notice.
* Give scripts execute permission before running: `chmod +x script.sh`.
* When scheduling with cron, always log output (`>> logfile.log 2>&1`) so you can check later whether it actually ran successfully.
