# GitHub 02 — PRs, Ahead/Behind, Revert vs Reset, Stash, Rebase

## Pull Request vs Merge Request

> They're the **same concept**, just named differently depending on the platform:
* **GitHub** calls it a **Pull Request (PR)**
* **GitLab** calls it a **Merge Request (MR)**

Both mean the exact same thing: *"here's a branch with changes — please review and merge it into the target branch."* No functional difference, just naming convention per platform.

### Example: `git merge` vs Pull Request (GUI)

**`git merge` — done locally, on the command line, no review step:**
```
git checkout main
git merge dev
git push origin main
```
You're directly combining `dev` into `main` yourself, then pushing — no one reviews it before it lands.

**Pull Request — done through the platform (GitHub/GitLab), with review:**
1. You push your branch: `git push origin feature/login`
2. You open a PR on GitHub: *"Merge `feature/login` into `main`."*
3. A teammate reviews the code, comments, approves (or requests changes).
4. Only then does someone click **"Merge"** on GitHub — which runs the merge on the server side.

**Key difference:** `git merge` is a raw, unsupervised action you run yourself. A PR is a *process* — same underlying merge operation, but wrapped with visibility, discussion, and approval before it actually happens.

---

## Ahead / Behind (seen in the branch list on the GUI)

* **Ahead** → your branch has commits that the target branch (e.g. `main`) **doesn't have yet**. Meaning: you've done new work that hasn't been merged in yet.
* **Behind** → the target branch (e.g. `main`) has commits that **your branch doesn't have**. Meaning: other people have merged stuff into `main` since you branched off, and your branch is now outdated/missing those updates.

**Example reading:** "3 ahead, 2 behind" on your branch means:
- You have 3 commits ready to be merged into `main` that main doesn't have.
- `main` has 2 commits (from others) that your branch is missing.

If your branch is **ahead**, you can go ahead and merge it into `main`/`master` — it has new work ready to contribute. If it's also **behind**, it's usually good practice to pull/merge the latest `main` into your branch first, to avoid conflicts, before merging your branch back in.

---

## Creating a Pull Request from Your Branch to Main (GUI Steps)

1. Push your branch to the remote: `git push origin your-branch-name`
2. On GitHub, go to your repo — you'll usually see a **"Compare & pull request"** prompt appear automatically for your recently pushed branch.
3. Click it (or go to **Pull Requests → New Pull Request** manually).
4. Set:
   * **Base branch** → `main` (where you want it merged *into*)
   * **Compare branch** → `your-branch-name` (what you want merged)
5. Add a title and description explaining what the PR does.
6. **Add a reviewer** — on the right sidebar, under "Reviewers," pick a teammate. This requests their explicit review/approval before the PR can (or should) be merged — many teams enforce this so nothing gets merged without a second pair of eyes.
7. Click **"Create Pull Request."**
8. Once the reviewer approves (and any CI checks pass), click **"Merge Pull Request."**

---

## `git revert` vs `git restore`

| | `git restore` | `git revert` |
|---|---|---|
| What it undoes | **Uncommitted** changes in your working directory | An already-**committed** change, in history |
| How | Throws away local edits, goes back to last commit | Creates a **new commit** that reverses an old one |
| History impact | No history involved — nothing committed yet | Adds a new commit; original commit stays visible too |

**Real-life example:**
- You're editing `app.py`, mess it up, haven't committed yet → `git restore app.py` throws away your edits, back to the last committed version.
- You committed a bug last week (already pushed, others have pulled it) → you can't just delete that commit safely since others already have it. Instead:
  ```
  git revert <commit-hash>
  ```
  This creates a brand new commit that undoes the bad commit's changes — history shows both "the bug" and "the fix for the bug," nothing is erased.

### If `git revert` causes a conflict

Sometimes the commit you're reverting overlaps with later changes, and Git can't auto-resolve it. When that happens:
1. Git pauses mid-revert and marks the conflicting file(s).
2. Open the file — you'll see conflict markers:
   ```
   <<<<<<< HEAD
   current code
   =======
   code from the commit being reverted
   >>>>>>> parent of <commit-hash>
   ```
3. Manually edit the file to keep what you actually want, and delete the conflict markers.
4. Stage the resolved file: `git add filename`
5. Continue the revert: `git revert --continue`
   (or `git revert --abort` if you want to cancel the whole thing)

---

## `git reset` — Why It's Risky (and how it differs from revert)

* **`git revert`** → keeps history intact, adds a new "undo" commit. Safe for shared/pushed branches since it doesn't rewrite anything others already have.
* **`git reset`** → actually **moves your branch pointer backward**, and depending on the mode, can permanently discard commits/changes. If you've already pushed those commits and then reset + force-push, you can rewrite shared history and cause real problems for teammates who already pulled the old commits.

**Rule of thumb:** if the commit is only local (never pushed), `reset` is fine. If it's already shared with others, prefer `revert`.

### `git reset --soft` vs `--mixed` vs `--hard`

| Mode | What happens to commits | What happens to staged changes | What happens to your working files |
|---|---|---|---|
| `--soft` | Removed from history | **Kept staged** (ready to re-commit) | Untouched — files still show the changes |
| `--mixed` (default) | Removed from history | **Unstaged** (moved back to working directory) | Untouched — files still show the changes |
| `--hard` | Removed from history | Discarded | **Discarded — actual file changes are wiped out** ⚠️ |

```
git reset --soft HEAD~1     # undo last commit, keep changes staged
git reset --mixed HEAD~1    # undo last commit, keep changes but unstaged (default)
git reset --hard HEAD~1     # undo last commit AND permanently delete the changes
```

**Your intuition was right:** `git revert` creates a visible trace in history (a new "revert" commit everyone can see). `git reset` can make it look like the commit **never happened at all** — no trace — which is exactly why it's the one to be careful with, especially `--hard`.

---

## `git stash`

> Temporarily shelves your **uncommitted** changes (both staged and unstaged) so you can switch branches with a clean working directory, then bring them back later.

**Does it work the same whether staged or not?**
By default, `git stash` grabs **both staged and unstaged changes** together into the same stash — it doesn't separate them. When you `git stash pop` later, staged changes come back staged, and unstaged changes come back unstaged, exactly as they were. So yes — it preserves the staged/unstaged state, it just doesn't stash them as two separate things by default.

```
git stash              # stash all current changes (staged + unstaged)
git stash list          # see all your stashed entries
git stash pop            # apply the most recent stash AND remove it from the stash list
git stash apply           # apply the most recent stash but KEEP it in the stash list
```

**To pop/apply a specific (not the most recent) stash:**
```
git stash list
# example output:
# stash@{0}: WIP on main: ...
# stash@{1}: WIP on dev: ...

git stash pop stash@{1}     # pop that specific one
git stash apply stash@{1}    # or apply without removing it
```

---

## `git rebase`

> Takes your branch's commits and **replays them on top of** another branch's latest commit, instead of creating a merge commit — resulting in a clean, straight-line history rather than a branching/merging graph.

```
git checkout feature-branch
git rebase main
```
This moves `feature-branch`'s commits to sit *after* the latest commit on `main`, as if you'd started your work from the newest point all along.

**Merge vs rebase, quickly:** `merge` preserves the true branching history (shows exactly when/where branches diverged and reunited). `rebase` rewrites your branch's commits to look linear — cleaner to read, but changes commit hashes, so ⚠️ never rebase commits that have already been pushed and shared with others (same danger as `reset --hard` — it rewrites history).

---

## What is Non-linear History?

> Normally, if everyone just committed one after another with no branching, your Git history would be one straight line. **Non-linear history** happens once you start branching and merging — the commit graph now has splits (where branches diverged) and joins (merge commits where branches came back together), instead of one single line.

This is completely normal and expected in real teams — multiple people working on multiple branches simultaneously naturally creates a non-linear graph. `git rebase` is one way people flatten a *specific branch's* history back into a straight line before merging, if they prefer a cleaner-looking log — but the overall project history (across all branches/merges) is still typically non-linear in any real, multi-person repo.
