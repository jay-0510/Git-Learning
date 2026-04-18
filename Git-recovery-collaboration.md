# Git Recovery and Collobaration & Remote Workflow

# 5. Undo & Recovery

## 🔄 5.1 `git reset` — Moving HEAD

### How it works (backend)

`git reset` moves the HEAD pointer (and the branch pointer) to a different commit. The difference between modes is what happens to your working files and staging area.

```
RESET MODES:
```

           Working Dir    Staging Area    Commits

--soft kept kept moved
--mixed kept cleared moved ← DEFAULT
--hard cleared cleared moved

VISUAL:
BEFORE:
A ── B ── C ── D (HEAD at D)

git reset --soft HEAD~2 (go back 2):
A ── B (HEAD) C and D's changes are STAGED
(D's and C's code is still there, ready to recommit differently)

git reset --mixed HEAD~2:
A ── B (HEAD) C and D's changes are UNSTAGED in working dir
(code is there, but you need to git add again)

git reset --hard HEAD~2:
A ── B (HEAD) C and D's changes are GONE
(dangerous — use with caution)

````

### Commands

```bash
git reset HEAD~1              # undo last commit, keep changes unstaged (--mixed)
git reset --soft HEAD~1       # undo last commit, keep changes staged
git reset --hard HEAD~1       # undo last commit, DISCARD changes
git reset HEAD src/app.js     # unstage a specific file (keep changes in working dir)
git reset --hard origin/main  # reset local branch to match remote exactly
````

### 📍 Scenario: Committed accidentally to main (should be on feature branch)

```
BEFORE:
  main: A ── B ── C (HEAD) ← committed here by mistake!

STEPS:
  1. git branch feature/my-work        → save the commit to a new branch
  2. git reset --hard HEAD~1           → remove from main
  3. git checkout feature/my-work      → switch to saved branch

```

AFTER:
main: A ── B ← clean
feature/my-work: A ── B ── C ← your work is here

```

### 📍 Scenario: Undo a staged file before commit

```

$ git status -s
A src/secret.env ← oops, staged wrong file

$ git reset HEAD src/secret.env (or: git restore --staged src/secret.env)

$ git status -s
?? src/secret.env ← back to untracked, not staged

```

---
```

## ↩️ 5.2 `git revert` — Safe Undo for Shared History

### How it works (backend)

`git revert` creates a **new commit** that applies the inverse of a previous commit. The old commit stays in history — nothing is rewritten.

```
BEFORE:
  A ── B ── C ── D   (main, HEAD)

git revert C   (undo commit C):

  A ── B ── C ── D ── C'  (HEAD)
                     ↑
              new commit that undoes C's changes

History is preserved — safe for shared/remote branches
```

**revert vs reset**:

```
reset  → rewrites history (dangerous on shared branches)
revert → adds a new "undo" commit (safe on shared branches)

Rule: If it's been pushed → use revert
      If it's local only  → you can use reset
```

### Commands

```bash
git revert HEAD                 # revert last commit
git revert a1b2c3d              # revert specific commit
git revert HEAD~3..HEAD         # revert last 3 commits
git revert --no-commit HEAD     # revert but don't auto-commit (let you review first)
```

### 📍 Scenario: Bad feature deployed to production

```
PRODUCTION TIMELINE:
  A ── B ── C (feat: dark mode) ── D (fix: button color) ── E (current)
                ↑
         This introduced a memory leak!

$ git revert C
  → Creates commit C': "Revert 'feat: dark mode'"
  → History becomes: A ── B ── C ── D ── E ── C'
  → Dark mode is removed, button fix and E are untouched
  → SAFE to push to main (no history rewrite)
```

---

## 🕵️ 5.3 `git reflog` — Your Safety Net

### How it works (backend)

Git keeps a **local-only log** of every time HEAD moved, for about 90 days. Even if you delete a branch or hard-reset, the commits are still in `.git/objects/` and reflog knows where they were.

```
.git/logs/HEAD   ← this file records every HEAD movement

Format:
  <old-sha> <new-sha> <author> <timestamp> <action>
```

### Commands

```bash
git reflog                      # see recent HEAD movements
git reflog show main            # show reflog for specific branch
git reflog --date=iso           # with timestamps
git checkout a1b2c3d            # go to a specific reflog entry
git branch recovered a1b2c3d    # save it as a branch
```

### 📍 Scenario: You `git reset --hard` and lost commits!

```
DISASTER:
  git reset --hard HEAD~5   ← you went back 5 commits, lost 5 commits of work!

RECOVERY:
  $ git reflog

  f9e8d7c HEAD@{0}: reset: moving to HEAD~5
  a1b2c3d HEAD@{1}: commit: feat: add payment form     ← this is the one!
  b2c3d4e HEAD@{2}: commit: feat: add payment validation
  c3d4e5f HEAD@{3}: commit: feat: cart total calculation
  ...

  $ git checkout a1b2c3d                   # go to that commit
  $ git branch recovery/payment-work       # save it as a branch!
  $ git checkout main
  $ git merge recovery/payment-work        # bring it back

LESSON: reflog is your 90-day time machine for local work.
```

---

## 👤 5.4 Detached HEAD — Don't Panic

### How it works (backend)

Normally HEAD points to a branch, which points to a commit:

```
HEAD → main → commit D
```

In **detached HEAD**, HEAD points directly to a commit (no branch):

```
HEAD → commit C  (no branch!)
```

Any new commits you make here are "orphaned" — they'll be lost when you switch branches.

### When you get into detached HEAD

```bash
git checkout a1b2c3d       # go to specific commit → detached HEAD
git checkout HEAD~3        # go back 3 commits     → detached HEAD
git checkout v1.0.0        # checkout a tag        → detached HEAD
```

### How to safely work in detached HEAD

```
SCENARIO: You want to explore what the code looked like 3 weeks ago.

```

1. git checkout HEAD~10 → detached HEAD (safe to look around)
2. You realize you want to MAKE changes here.

OPTION A: Save work to a new branch:
git switch -c explore/old-state → now you're on a real branch

OPTION B: Return without saving:
git switch main → go back, detached commits will be garbage collected

OPTION C: You made commits in detached HEAD and want to save them:
git branch rescue/my-work → save current HEAD as a branch
git switch rescue/my-work → you're now on a real branch

```

```

### Visual

```
DETACHED:
  A ── B ── C ── D   (main)
              ↑
            HEAD (you're here, not on any branch)

If you commit:
  A ── B ── C ── D   (main)
              \
               X ── Y   ← these will be lost when you leave!
               ↑
             HEAD

FIX: git branch save-me (while still at Y) → now X and Y are reachable
```

---

# 6. Collaboration & Remote Workflow

## 🌐 6.1 Remote (`origin`, tracking branches)

### How it works (backend)

A "remote" is just a name for a URL stored in `.git/config`. When you clone a repo, Git automatically names the source remote `origin`.

```
.git/config (after clone):
  [remote "origin"]
    url = https://github.com/company/project.git
    fetch = +refs/heads/*:refs/remotes/origin/*

  [branch "main"]
    remote = origin
    merge = refs/heads/main       ← this is a "tracking branch"
```

**Remote-tracking branches** (like `origin/main`) are LOCAL read-only copies of what the remote looks like. They update only when you `fetch`/`pull`.

```
YOUR LOCAL:              WHAT GIT REMEMBERS REMOTE LOOKS LIKE:
  main  → D               origin/main → B   (stale until you fetch)

After git fetch:
  main  → D               origin/main → D   (now in sync)
```

### Commands

```bash
git remote -v                              # list remotes and their URLs
git remote add origin <url>               # add a remote
git remote add upstream <url>             # add a second remote (fork workflow)
git remote rename origin backup           # rename
git remote remove backup                  # remove
git remote set-url origin <new-url>       # change URL (e.g., SSH vs HTTPS)
git remote show origin                    # detailed info about remote
```

---

## 📋 6.2 `git clone` — Get a Copy

### How it works (backend)

Clone:

1. Creates a new directory
2. Initializes a `.git/` repo
3. Adds `origin` remote pointing to source
4. Fetches all objects (commits, trees, blobs)
5. Checks out the default branch

```

```

github.com/company/app.git
↓ git clone
my-machine/app/
├── .git/
│ ├── refs/remotes/origin/ ← copies of remote branches
│ └── objects/ ← all history downloaded
└── (working files checked out)

````

### Commands

```bash
git clone https://github.com/company/app.git          # clone to folder 'app'
git clone <url> my-folder                              # clone to custom name
git clone --depth 1 <url>                             # shallow clone (latest only)
git clone --branch develop <url>                       # clone specific branch
git clone --single-branch --branch main <url>         # only download main branch
git clone git@github.com:company/app.git              # SSH clone (preferred for auth)
````

### 📍 Scenario: Joining a team project

```
1. Find repo URL on GitHub
2. git clone git@github.com:company/backend-api.git
3. cd backend-api
4. git log --oneline -10     → see recent history
5. git branch -a             → see all branches
6. git checkout -b feature/JIRA-301-my-task origin/develop
   → create your branch from the team's develop branch
```

---

## 🍴 6.3 Fork — Your Own Copy on GitHub

### How it works

A fork is a **GitHub/GitLab concept** (not a Git command). It creates a full server-side copy of a repo under your account.

```
ORIGINAL:  github.com/facebook/react
                    ↓ Fork (on GitHub)
YOUR COPY: github.com/arjunshah/react     ← you own this

LOCAL:     git clone github.com/arjunshah/react
                    ↓
           git remote add upstream github.com/facebook/react
```

### Fork workflow

```
1. Fork on GitHub (creates your copy)
2. git clone your-fork (work locally)
3. Add upstream remote → track original
4. Create branch → make changes → push to YOUR fork
5. Open Pull Request: your-fork/feature → original/main
```

```
1. Fork on GitHub (creates your copy)
2. git clone your-fork (work locally)
3. Add upstream remote → track original
4. Create branch → make changes → push to YOUR fork
5. Open Pull Request: your-fork/feature → original/main
```

```bash
# After forking on GitHub:
git clone git@github.com:arjunshah/react.git
cd react
git remote add upstream https://github.com/facebook/react.git

# Keep your fork updated with original:
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

## 📥 6.4 `git fetch` — Download Without Merging

### How it works (backend)

`git fetch` downloads new commits and branches from the remote but does NOT touch your working directory or current branch. It only updates the `origin/*` remote-tracking refs.

```
BEFORE fetch:
  Local:         main → D
  Remote-tracking: origin/main → B    ← stale
  Actual remote: main → F (E, F added since last time)

AFTER git fetch origin:
  Local:         main → D             ← untouched
  Remote-tracking: origin/main → F   ← updated
  (E and F are now downloaded into .git/objects)

You can now:
  git log origin/main     → see what came in
  git diff main origin/main → see what's different
  git merge origin/main   → integrate (your choice, your timing)
```

### Commands

```bash
git fetch origin                    # fetch from origin
git fetch --all                     # fetch from all remotes
git fetch origin main               # fetch only main branch
git fetch --prune                   # fetch + remove local refs to deleted remote branches
```

---

## 📤 6.5 `git pull` — Fetch + Merge

### How it works (backend)

`git pull` = `git fetch` + `git merge` (by default)

```
git pull origin main

Step 1: git fetch origin main
  → Downloads new commits, updates origin/main

Step 2: git merge origin/main
  → Merges origin/main into your current branch

If there are new commits on both sides → merge commit created
If remote is ahead only → fast-forward
```

```bash
git pull                            # pull from tracked remote branch
git pull origin main                # explicit: pull main from origin
git pull --rebase                   # fetch + rebase instead of merge
git pull --ff-only                  # fail if not fast-forward (safe for clean history)
git pull --no-commit                # pull but don't auto-commit merge
```

---

## ⚖️ 6.6 Fetch vs Pull — The Real Difference

```
                    git fetch               git pull
────────────────────────────────────────────────────────
Downloads changes?    ✓ Yes                  ✓ Yes
Merges into branch?   ✗ No                   ✓ Yes (auto)
Safe always?          ✓ Very safe             ⚠ Can cause conflicts
Use when?           Want to look first     Trust + want it now

MENTAL MODEL:
  fetch = "Download the mail to the mailbox"
  pull  = "Download the mail AND read it immediately"
```

### 📍 Scenario: Review before integrating

```
FLOW (professional approach):
  1. git fetch origin
  2. git log main..origin/main --oneline
     → See exactly what commits are coming in
  3. git diff main origin/main
     → See exactly what code is changing
  4. git merge origin/main
     → Now merge, knowing exactly what you're getting

FLOW (quick approach):
  git pull         → fetch + merge in one step
```

### 📍 Scenario: `git pull --rebase` for clean history

```
WITHOUT rebase:
  Your:    A ── B ── C ── D (feature)
  Remote:  A ── B ── E ── F (main)

  git pull (merge):
  Result: A ── B ── E ── F ── M   (M = merge commit, messy)
                  \         /
                   C ── D

WITH rebase:
  git pull --rebase:
  Result: A ── B ── E ── F ── C' ── D'  (clean, linear)
  (Your commits moved on top, no merge commit)
```

---

## 📋 6.7 Industry Collaboration Standards

```
✅ DO:
  - Always pull/fetch before starting work each day
  - Create feature branches from latest main/develop
  - Keep branches short-lived (merge within 1-3 days ideally)
  - Write meaningful commit messages (conventional commits)
  - Update your branch before opening PR

```

❌ DON'T:

- Force push to shared branches (main, develop)
- Commit directly to main
- Leave branches open for weeks
- Push broken/non-compiling code
- Include unrelated changes in a PR

````

```bash
# ✅ Proper daily start:
git fetch --all --prune
git checkout main
git pull --ff-only
git checkout feature/my-task
git rebase main     # keep my branch updated with latest main
````

---

## 🔀 6.8 Pull Requests (PR Workflow)

### Industry PR flow

```
1. FORK or BRANCH from main/develop
      ↓
2. Write code on feature branch
      ↓
3. Push branch to remote
      ↓
4. Open PR on GitHub/GitLab:
   - Target: main or develop
   - Description: What, Why, How to test
   - Link JIRA ticket
      ↓
5. Code Review (teammates comment)
      ↓
6. Address feedback → push more commits
      ↓
7. CI/CD passes (tests green, lint clean)
      ↓
8. Approved → MERGE (squash or merge commit or rebase)
      ↓
9. Delete feature branch
      ↓
10. Auto-deploy (if set up)
```

### 📍 Scenario: Real PR description format

```markdown
## What

Added user avatar upload feature (JIRA-234)

## Why

Users requested profile personalization.

## Changes

- POST /api/users/avatar endpoint
- Client-side image resize before upload
- S3 integration for storage

## How to test

1. Go to /profile
2. Click "Upload Photo"
3. Upload a JPG > 1MB — should auto-resize
4. Verify shows in nav bar

## Screenshots

[before] [after]
```

---
