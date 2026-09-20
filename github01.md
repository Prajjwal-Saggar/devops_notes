# Source Code Management — Git & GitHub

## What is Git?

> **Git** = "Global Information Tracker" — a version control system that tracks changes to your code over time, so you can see history, collaborate, and undo mistakes.

**Git vs GitHub/GitLab/etc. — important distinction:**
Git is just the underlying *technology*. The platforms below are all just hosting services built **on top of** Git — they all use the same Git engine underneath, just with different UI/features/hosting:
1. GitHub
2. GitLab
3. Bitbucket
4. AWS CodeCommit
5. Azure Repos

---

## File System vs Version Control System (5 quick points)

1. **History** — a plain file system only shows the current state of a file; a VCS keeps the *entire history* of every change ever made.
2. **Collaboration** — a file system has no real concept of multiple people editing safely; a VCS is built for many people working on the same code without overwriting each other.
3. **Undo/rollback** — file systems have no built-in "go back to yesterday's version"; a VCS lets you revert to any previous point instantly.
4. **Who changed what** — a file system doesn't track authorship; a VCS records exactly who made each change, and when.
5. **Branching** — a file system has one single state; a VCS lets you create parallel versions (branches) to work on different features/fixes without affecting the main code.

---

## Core Concepts

* **Local** → the Git repository sitting on *your* machine.
* **Remote** → the Git repository hosted elsewhere (e.g. on GitHub) — the shared/central copy.
* **Author** → the person who made a particular change/commit (recorded via `git config`).
* **Directory** → just a regular folder — no version tracking at all.
* **Repository ("repo")** → a directory that Git is tracking — same folder, but now Git is watching every change inside it.

---

## Getting Started

**`git init`** → *initializes a new Git repository* (turns a normal folder into a tracked repo).

**Check hidden files (see the `.git` folder Git just created):**
```
ls -a
```
(`-a` shows hidden files/folders — anything starting with a dot, like `.git`.)

---

## The Three Stages — Untracked, Staged, Tracked

| Stage | Meaning | Color in `git status` |
|---|---|---|
| **Untracked** | A new file Git has noticed but isn't tracking/monitoring changes for yet | Red |
| **Staged** | File has been marked (via `git add`) as "ready to be included in the next commit" | Green |
| **Tracked / Committed** | File has been saved into Git's history via a commit | (no longer shown as changed — clean state) |

**Check current stage of all files:**
```
git status
```

**Move a file to staged:**
```
git add .              # stage everything
git add filename.txt   # stage just one file
```

**Unstage a file (remove from staging, but keep the actual file):**
```
git rm --cached filename.txt
```

**Commit (save staged changes permanently to history):**
```
git commit -m "your message here"
```

**`git restore`** — how and when to use:
> Used to **discard changes** in your working directory and go back to the last committed version — i.e. "undo my uncommitted edits."
```
git restore filename.txt        # discard uncommitted changes to a file
git restore --staged filename.txt  # unstage a file (alternative to git rm --cached)
```
Use this when you've messed something up locally and just want to throw away your edits and start fresh from the last commit.

---

## `git config` — Why Set Username/Email?

```
git config --global user.name "Prajjwal"
git config --global user.email "you@example.com"
```
> Every commit needs an **author** attached to it — Git uses this name/email to stamp *who* made each commit. `--global` means this applies to every repo on your machine (not just one project), so you only need to set it once.

---

## Terminology — Plain-English Glossary (so nothing on your terminal scares you)

| Term | Plain-English meaning |
|---|---|
| **Repository (repo)** | A project folder that Git is tracking |
| **Commit** | A saved snapshot of your code at a point in time |
| **Branch** | A separate, parallel line of work — lets you experiment without touching the main code |
| **Tree** | The structure of files/folders Git is tracking at a given commit (think: a snapshot of your whole folder structure) |
| **HEAD** | A pointer to whatever commit/branch you're *currently* on — "you are here" |
| **Working directory** | Your actual files on disk, as you see/edit them |
| **Staging area (index)** | The "waiting room" before a commit — files you've `git add`-ed but not yet committed |
| **Clean** | Means `git status` shows no pending changes — everything is committed, nothing left to save |
| **Remote** | A version of your repo hosted elsewhere (e.g. GitHub) |
| **Origin** | The default nickname Git gives to your main remote repo |
| **Merge** | Combining changes from one branch into another |
| **Clone** | Copying an entire remote repo down to your local machine |

---

## Branches

**Create and switch to a new branch in one step:**
```
git branch dev          # just create the branch
git checkout -b dev      # create AND switch to it in one command
```

**Check status/current branch:**
```
git status
```

**See commit history (compact form):**
```
git log --oneline
```
* **`HEAD`** in this output → points to your **latest commit** — literally means "where you currently are."

**Switch between branches:**
```
git switch master
git switch main
git switch dev
```
(`git switch` is the modern replacement for `git checkout` when just changing branches — `checkout` still works but does more things, so `switch` is clearer/safer for this specific job.)

### Difference between `master` and `main`

> They're functionally identical — both are just the conventional name for the default/primary branch. `master` was the old default name; GitHub (and the wider Git community) shifted to `main` as the new default a few years back for more inclusive terminology. Which one you have just depends on when the repo was created / what the platform defaults to — there's no technical difference.

**Merge a branch into your current branch:**
```
git merge dev
```
(Run this *while on* the branch you want to merge **into** — e.g. while on `main`, run `git merge dev` to bring `dev`'s changes into `main`.)

---

## Local Repo vs Remote Repo

* **Local repo** (`git init`) → lives only on your machine.
* **Remote repo** → hosted elsewhere (GitHub, GitLab, etc.) — the shared copy others can also access.

### Connecting Local → Remote

You need a way to authenticate. Two common options:
1. **SSH** — set up an SSH key pair, add the public key to your GitHub account, then connect via an SSH URL (`git@github.com:user/repo.git`). No password typing after setup — very common for regular day-to-day use.
2. **Personal Access Token (PAT)** — used instead of a password over HTTPS. GitHub generates a token for you (from Settings → Developer Settings), and you use it in place of a password when prompted.

**Check what remote(s) your repo is linked to:**
```
git remote -v
```
Shows the remote's nickname (usually `origin`) and its URL, for both fetch and push.

**Link your local repo to a remote for the first time:**
```
git remote add origin https://github.com/username/repo.git
```
> Registers the remote repo's URL under the nickname `origin`, so future push/pull commands know where to send/receive from.

**Push your local commits up to the remote:**
```
git push origin master
```

**If using a PAT and need to update the stored remote URL (e.g. to embed auth or switch protocol):**
```
git remote set-url origin https://<PAT>@github.com/username/repo.git
```
> This updates the URL Git uses for `origin`, in this case embedding your Personal Access Token directly so you're not prompted for credentials on every push.

**Pull changes from remote down to local:**
```
git pull origin master
```

**Clone an entire remote repo to your local machine (fresh copy):**
```
git clone repo-url
```

**See exact line-by-line differences between versions:**
```
git diff
```

---

## Fork vs Clone

* **Fork** → copies a repo from **one remote account to another remote account** (e.g. someone else's GitHub repo → your own GitHub account). Remote-to-remote.
* **Clone** → copies a repo from a **remote to your local machine**. Remote-to-local.

**Common real use of fork:** contributing to open-source — you fork someone else's project to your own GitHub, make changes there, then submit a Pull Request back to the original.

---

## `.gitignore`

> A file listing patterns/filenames Git should **never track** — e.g. `.env`, `node_modules/`, `*.log`. Keeps secrets, build artifacts, and junk files out of your commits entirely.

---

## More Commands (the "more you learn, the more there is" list)

| Command | What it does |
|---|---|
| `git cherry-pick <commit-hash>` | Grabs one specific commit from another branch and applies it to your current branch, without merging everything else |
| `git rebase` | Replays your branch's commits on top of another branch — rewrites history to look like a clean, linear line instead of a merge |
| `git reset` | Moves your branch pointer backward — can undo commits (with options to keep or discard the changes) |
| `git revert` | Creates a **new** commit that undoes a previous commit — safer than reset since it doesn't rewrite history |
| `git stash` | Temporarily "shelves" your uncommitted changes so you can switch branches cleanly, then bring them back later with `git stash pop` |
| `git fetch` | Downloads new commits/branches from the remote, but does **NOT** merge them into your working branch (unlike `git pull`, which fetches AND merges) |

---

## Git Branching Strategy — The Most Common One (Real-World)

> **Feature Branch Workflow (a.k.a. "GitHub Flow")** — by far the most commonly used in real companies, especially for web/app teams shipping continuously.

**How it works:**
1. `main` branch is always kept **stable and deployable**.
2. For any new feature/fix, a developer creates a new branch off `main` (e.g. `feature/login-page`, `fix/payment-bug`).
3. They commit work to that branch, push it, and open a **Pull Request (PR)** back into `main`.
4. Teammates **review** the PR, leave comments, request changes if needed.
5. Once approved (and CI/tests pass), it gets **merged into `main`**.
6. `main` is deployed — often automatically via CI/CD.

**Real-life company scenario:**
A team building a SaaS product has 5 engineers. One dev is asked to add a "forgot password" feature.
- They run `git checkout -b feature/forgot-password` off the latest `main`.
- They write the code, commit as they go, push to the remote.
- They open a PR: *"Add forgot password flow."*
- A senior engineer reviews it, asks for one small change, dev pushes a fix commit to the same branch (PR auto-updates).
- Once approved and automated tests pass, it's merged into `main`.
- CI/CD picks up the merge to `main` and deploys it to production automatically.

This keeps `main` always shippable, avoids one dev's unfinished work blocking others, and gives a clean review/approval trail for every change — which is exactly why it's the industry default for most teams.

(Note: larger, release-cycle-driven companies — e.g. shipping versioned software on a schedule — sometimes use the more complex **Git Flow** instead, with separate `develop`/`release`/`hotfix` branches. But for most modern, continuously-deployed products, Feature Branch/GitHub Flow above is what you'll actually encounter.)

---

## Using Jira with GitHub (Real-Life Workflow)

> Jira is a project/ticket management tool. Teams connect it to GitHub so that code changes are automatically linked back to the Jira ticket they relate to — full traceability from "task assigned" to "code merged."

**How it typically works in a real company:**

1. A ticket is created in Jira, e.g. **`PROJ-123`: "Fix login button not working on mobile."**
2. The developer creates a Git branch **using the Jira ticket ID in the branch name** — this is the convention that makes the integration work:
   ```
   git checkout -b PROJ-123-fix-login-button
   ```
3. When committing, they often include the ticket ID in the commit message too:
   ```
   git commit -m "PROJ-123: fix login button click handler on mobile"
   ```
4. With the **GitHub–Jira plugin/integration** installed (via the GitHub Marketplace or Jira's "GitHub for Jira" app), Jira **automatically detects** the ticket ID in branch names/commit messages/PR titles, and:
   * Shows the linked branch, commits, and PR directly inside the Jira ticket.
   * Can auto-transition the ticket's status (e.g. moves it from "In Progress" → "In Review" when a PR is opened, and → "Done" when the PR is merged) — this is often called a **"smart commit"** or workflow automation rule.
5. Project managers/QA can now open the Jira ticket and instantly see exactly which code change addressed it — no manual updating needed.

**Why companies do this:** it removes the manual busywork of updating ticket status and cross-referencing "which PR fixed which bug" — everything links automatically just by following the branch/commit naming convention.
