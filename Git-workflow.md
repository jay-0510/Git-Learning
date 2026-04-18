# Git Workflow and Branching Basics

3. [Basic Daily Workflow](#3-basic-daily-workflow)
4. [Branching Basics](#4-branching-basics)

# 3. Basic Daily Workflow

## 💾 3.1 Commit — Saving Work the Right Way

### How it works (backend)

When you commit, Git creates three types of objects in `.git/objects/`:

```
1. BLOB      → stores file content (one per file version)
2. TREE      → stores directory structure (which blobs are in which folder)
3. COMMIT    → stores: tree pointer + parent commit + author + message

Commit chain:

  [commit C1]──▶[tree]──▶[blob: app.js v1]
       │
       ▼ (parent)
  [commit C2]──▶[tree]──▶[blob: app.js v2]
       │                 └▶[blob: utils.js v1]
       ▼
  [commit C3] ← HEAD (latest)
```

Each commit has a **SHA-1 hash** (40-char hex string like `a1b2c3d4...`) that uniquely identifies it.

### Commands

```bash
git add .                          # stage all changes
git add src/app.js                 # stage specific file
git add src/                       # stage entire folder
git add -p                         # interactive: stage in chunks (hunks)

git commit -m "feat: add login API"        # commit with message
git commit                                  # opens editor for longer message
git commit -am "fix: typo in README"       # add tracked files + commit in one step
```

### ✍️ Commit Message Discipline (Industry Standard)

Format: `<type>(<scope>): <short description>`

```
feat:     new feature
fix:      bug fix
docs:     documentation only
style:    formatting, no logic change
refactor: code restructure, no feature/bug
test:     adding tests
chore:    build process, dependencies
perf:     performance improvement

Examples:
  feat(auth): add JWT token refresh
  fix(cart): prevent double-click purchase
  docs(api): update endpoint documentation
  chore(deps): bump axios from 0.21 to 1.0
```

### 📍 Scenario: End of a coding session

```
FLOW:
  1. You wrote code all day
  2. git status -s       → see what changed
  3. git diff            → review exactly what changed (before staging)
  4. git add -p          → stage only meaningful changes (not debug logs)
  5. git commit -m "feat(payment): integrate Razorpay webhook"
  6. git log --oneline   → confirm commit looks right

```

### `git add -p` — The Professional Way to Stage

```bash
$ git add -p src/app.js
# Git shows each "hunk" of changes:
# y = stage this hunk
# n = skip this hunk
# s = split into smaller hunks
# q = quit

# WHY: You might have made 3 unrelated changes in one file.
# Stage them as separate commits for clean history.
```

---

## ✏️ 3.2 `git commit --amend` — Fix Last Commit

### How it works (backend)

Amend does NOT edit the old commit. It creates a **brand new commit** with a new SHA replacing the old one. The old commit becomes unreachable (until garbage collected).

```
BEFORE amend:
  C1 ← C2 ← C3 (HEAD, main)
              ↑
         "fix typo" (bad message, missing file)

AFTER amend:
  C1 ← C2 ← C3' (HEAD, main)    ← new commit with different SHA!
              ↑
         "fix: correct login redirect URL" + added missing file

  (C3 is now unreachable, will be garbage collected)
```

> ⚠️ **Never amend a commit that's already been pushed to shared remote.** It rewrites history and forces everyone else to reconcile.

### Commands

```bash
# Fix just the message
git commit --amend -m "fix: correct login redirect URL"

# Add a forgotten file to last commit
git add src/forgot-this.js
git commit --amend --no-edit    # keeps same message

# Amend message in editor
git commit --amend
```

### 📍 Scenario: You committed but forgot to include a file

```
BEFORE:
  git log --oneline
  a1b2c3d  feat: add user profile page   ← missing profile.css!

  git status -s
  ?? src/profile.css    ← sitting untracked

STEPS:
  git add src/profile.css
  git commit --amend --no-edit

AFTER:
  git log --oneline
  f9e8d7c  feat: add user profile page   ← same message, new SHA, file included!

  git show HEAD --stat
  src/profile.js    ← ✓
  src/profile.css   ← ✓ now included
```

---

## 🔎 3.3 `git diff` — See Exact Changes

### How it works (backend)

`git diff` compares snapshots from different areas:

```
Working Dir ←──── git diff ────▶ Staging Area
Staging Area ←── git diff --cached ──▶ HEAD
Working Dir ←──── git diff HEAD ──▶ HEAD (both above combined)
Branch A ←──── git diff branchA..branchB ──▶ Branch B
```

### Commands

```bash
git diff                        # working dir vs staging (unstaged changes)
git diff --staged               # staging vs HEAD (what will be committed)
git diff HEAD                   # all changes since last commit
git diff HEAD~2                 # changes since 2 commits ago
git diff main feature-branch    # differences between branches
git diff a1b2c3 f9e8d7          # between two specific commits
git diff --stat                 # summary: which files, how many lines
git diff src/app.js             # diff only one file
```

### 📍 Scenario: Before committing — review what you changed

```
$ git diff --staged

diff --git a/src/auth.js b/src/auth.js
index 3a4b5c6..7d8e9f0 100644
--- a/src/auth.js          ← old version
+++ b/src/auth.js          ← new version
@@ -12,7 +12,9 @@
 function login(user, pass) {
-  return db.find(user);
+  const result = await db.find(user);
+  if (!result) throw new Error('User not found');
+  return result;
 }

GREEN (+) = lines added
RED   (-) = lines removed
```

### 📍 Scenario: Compare your branch to main before PR

```bash
git diff main..feature/payment --stat
# Shows which files you touched, how many lines added/removed
# Good sanity check before creating pull request
```

---

## 👁️ 3.4 `git blame` — Who Wrote This Line?

### How it works (backend)

`git blame` annotates each line of a file with the **commit hash**, **author**, and **date** of the last change to that line. Git walks through the object history to find this.

### Commands

```bash
git blame src/app.js                    # annotate every line
git blame -L 20,35 src/app.js           # only lines 20–35
git blame --since="3 weeks ago" src/app.js
git blame -w src/app.js                 # ignore whitespace changes
git blame -C src/app.js                 # detect code moved from other files
```

### 📍 Scenario: Production bug — who wrote this?

```
A bug is on line 47 of payment.js.

```

$ git blame -L 45,50 src/payment.js

^a1b2c3d (Rahul Kumar 2024-01-15 14:32:11) function processPayment(amount) {
^a1b2c3d (Rahul Kumar 2024-01-15 14:32:11) if (amount > 0) {
f9e8d7c2 (Priya Singh 2024-03-02 09:14:55) return gateway.charge(amount \* 100); ← BUG HERE
^a1b2c3d (Rahul Kumar 2024-01-15 14:32:11) }
f9e8d7c2 (Priya Singh 2024-03-02 09:14:55) }

→ You can see Priya changed line 47 on March 2.
→ git show f9e8d7c2 → see the full commit context for why that change was made.

```

> 💡 **Note**: Don't use `blame` to point fingers. Use it to understand context — what was the original intent of that code?

---

## 🔍 3.5 `git grep` — Search Inside Repo

### How it works (backend)

```

`git grep` searches the **tracked content** of your repo (Git's object store), not just your filesystem. It's faster than regular grep because Git already knows what files to look in.

### Commands

```bash
git grep "TODO"                           # find "TODO" in all tracked files
git grep -n "processPayment"              # with line numbers
git grep -l "useState"                    # only list filenames
git grep -i "error"                       # case-insensitive
git grep "apiKey" HEAD~10                 # search in a specific commit's state
git grep --all-match -e "login" -e "auth" # lines matching BOTH patterns
```

### 📍 Scenario: Find all places where an API key might be hardcoded

```bash
git grep -n "api_key\|API_KEY\|apiKey"

src/config.js:14:  apiKey: "sk-1234abcd"     ← found it! Remove before commit.
src/utils/http.js:8:  headers: { API_KEY: key }  ← this one is fine (uses variable)
```

### 📍 Scenario: Find all usages of a function before refactoring it

```bash
git grep -n "sendEmail(" --and -l
# Lists all files where sendEmail() is called
# Now you know what to update when you rename/refactor it
```

---

# 4. Branching Basics

## 🌿 4.1 `git branch` — Managing Branches

### How it works (backend)

A branch is simply a **lightweight pointer** (a file in `.git/refs/heads/`) that points to a commit SHA. Creating a branch is nearly instant — it's just creating a new file with 41 bytes.

```
.git/refs/heads/
├── main           → contains: "a1b2c3d4..."  (points to latest main commit)
├── feature/login  → contains: "f9e8d7c2..."  (points to latest feature commit)
└── hotfix/bug-42  → contains: "3c4d5e6f..."

HEAD → .git/HEAD file → contains "ref: refs/heads/main"
       (HEAD tells Git which branch YOU are on)
```

### Branch model visualized

```
main:    A ── B ── C ── D
                    \
feature:             E ── F ── G   ← 3 commits ahead of main
                                ↑
                              HEAD (you are here)
```

### Commands

```bash
git branch                      # list local branches
git branch -a                   # list all (including remote)
git branch -r                   # list only remote branches
git branch feature/login        # create new branch
git branch -d feature/login     # delete (safe — only if merged)
git branch -D feature/login     # force delete (even if not merged)
git branch -m old-name new-name # rename branch
git branch -v                   # show last commit on each branch
git branch --merged             # branches already merged into current
git branch --no-merged          # branches NOT yet merged
```

### 📍 Scenario: Industry branching strategy

```
BRANCHES IN A REAL PROJECT:
  main          → production-ready code, protected, no direct push
  develop       → integration branch, all features merge here first
  feature/JIRA-101-user-auth    → your work
  feature/JIRA-102-payment      → teammate's work
  hotfix/JIRA-99-prod-crash     → urgent production fix
  release/v2.1.0                → staging for next release
```

---

## 🔀 4.2 `git checkout` — Switch + More

### How it works (backend)

`checkout` updates **HEAD** (which branch/commit you're on) AND updates your **working directory** to match. It also updates the **index/staging area**.

```
BEFORE checkout feature:
  HEAD → main → commit D
  Working dir shows D's files

AFTER git checkout feature:
  HEAD → feature → commit G
  Working dir now shows G's files
```

### Commands

```bash
git checkout main                   # switch to main branch
git checkout feature/login          # switch to feature branch
git checkout -b feature/payment     # create AND switch (most common)
git checkout -b hotfix origin/hotfix # create local from remote branch
git checkout -- src/app.js          # DISCARD changes in file (restore from HEAD)
git checkout a1b2c3d                # go to specific commit (detached HEAD)
git checkout HEAD~2                 # go back 2 commits (detached HEAD)
```

### 📍 Scenario: Start a new feature

```
FLOW:
  1. git checkout main          → start from latest main
  2. git pull                   → ensure you have latest code
  3. git checkout -b feature/JIRA-234-user-avatar
  4. ... write code ...
  5. git add . && git commit -m "feat(profile): add avatar upload"
```

---

## 🔀 4.3 `git switch` — Modern Branch Switching

`git switch` was introduced to **split checkout's responsibilities** into cleaner commands:

- `git switch` = only for changing branches
- `git restore` = only for restoring files (replaces `git checkout -- file`)

### Commands

```bash
git switch main                    # switch branch
git switch -c feature/new-thing    # create + switch (-c = --create)
git switch -                       # switch to previous branch
git switch --detach a1b2c3d        # go to commit in detached HEAD
```

> 💡 Use `git switch` for branches, `git restore` for files. It's clearer and safer — you can't accidentally delete file changes with switch.

---

## 🔗 4.4 `git merge` — Combining Branches

### How it works (backend)

Git finds the **common ancestor** of both branches, then combines changes. Two strategies:

**Fast-Forward Merge** (no new commit created):

```
BEFORE:
  main:    A ── B
                \
  feature:       C ── D   ← no new commits on main since branch point

AFTER git merge feature (from main):
  main:    A ── B ── C ── D    ← just moved the pointer forward
```

**3-Way Merge** (new merge commit created):

```
BEFORE:
  main:    A ── B ── E ── F    ← main has moved ahead
                \
  feature:       C ── D

AFTER git merge feature (from main):
  main:    A ── B ── E ── F ── M   ← M is a new merge commit with two parents
                \              /
  feature:       C ─────── D
```

### Commands

```bash
git merge feature/login              # merge into current branch
git merge --no-ff feature/login      # force merge commit even if fast-forward possible
git merge --squash feature/login     # squash all feature commits into one staged change
git merge --abort                    # abort mid-conflict merge
```

### 📍 Scenario: Feature done, merging to main

```
FLOW:
  1. git checkout main
  2. git pull                          → get latest main
  3. git merge feature/JIRA-234 --no-ff
     → --no-ff preserves branch history — you can see "this was a feature"

RESULT IN LOG:
  * M  Merge branch 'feature/JIRA-234' into main    ← merge commit
  |\
  | * D  feat: resize avatar on upload
  | * C  feat: add avatar upload component
  |/
  * F  fix: navbar spacing
  * E  feat: dark mode toggle
```

---

## ⚡ 4.5 Conflicts — What Really Happens

### How it works (backend)

A conflict occurs when two branches changed the **same lines** in the same file. Git cannot auto-decide which is correct, so it marks the file:

```
<<<<<<< HEAD                     ← your current branch's version
const port = 3000;
=======                          ← divider
const port = process.env.PORT;
>>>>>>> feature/config-env       ← incoming branch's version
```

### Conflict resolution flow

```
git merge feature/config-env
    ↓
CONFLICT in src/server.js
    ↓
Open file, find <<<< markers
    ↓
Decide what the correct code should be (may need to combine both)
    ↓
Delete all <<< === >>> markers
    ↓
Save file
    ↓
git add src/server.js            ← mark as resolved
    ↓
git commit                       ← complete the merge
```

### 📍 Scenario: Two developers edited same config file

```
BEFORE (on main):
  const config = { env: 'development', port: 3000 };

Developer A (on feature/ports) changed it to:
  const config = { env: 'development', port: 8080 };

Developer B (on feature/env) changed it to:
  const config = { env: process.env.NODE_ENV, port: 3000 };

WHEN A merges, then B merges:

  <<<<<<< HEAD
  const config = { env: 'development', port: 8080 };
  =======
  const config = { env: process.env.NODE_ENV, port: 3000 };
  >>>>>>> feature/env

RESOLUTION (combining both intentions):
  const config = { env: process.env.NODE_ENV, port: 8080 };

Then:
  git add src/config.js
  git commit -m "merge: resolve config conflict — env dynamic + port 8080"
```

### Tools for easier conflict resolution

```bash
git mergetool            # opens configured visual merge tool
git config --global merge.tool vscode

# Or use VS Code's built-in:
# It shows "Accept Current | Accept Incoming | Accept Both | Compare"
```

---
