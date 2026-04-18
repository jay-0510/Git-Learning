# Git Selective Changes & Multi-Branch Control & Stashing

# 9. Selective Changes & Multi-Branch Control

## 🍒 9.1 `git cherry-pick` — Pick Specific Commits

### How it works (backend)

Cherry-pick takes a commit from any branch and applies its changes as a **new commit** on your current branch. The original commit is unchanged.

```
BEFORE:
  main:    A ── B ── C ── D
  hotfix:  A ── B ── X ── Y   (X = critical fix, Y = unrelated test)

You want ONLY X on main:
  git checkout main
  git cherry-pick X

AFTER:
  main:    A ── B ── C ── D ── X'  (X' is a copy of X, new SHA)
  hotfix:  A ── B ── X ── Y        (unchanged)
```

### Commands

```bash
git cherry-pick a1b2c3d                    # pick one commit
git cherry-pick a1b2c3d..f9e8d7c          # pick a range (exclusive start)
git cherry-pick a1b2c3d^..f9e8d7c         # pick a range (inclusive start)
git cherry-pick --no-commit a1b2c3d       # apply changes but don't commit yet
git cherry-pick --abort                    # abort mid-pick
git cherry-pick --continue                 # continue after conflict
```

### 📍 Scenario: Critical bug fixed on wrong branch

```
SITUATION:
  Developer fixed a production bug on 'develop' by mistake.
  The fix needs to go to 'hotfix' and 'main' NOW.

STEPS:
  git log develop --oneline
  a1b2c3d  fix: null check prevents crash on empty cart  ← THIS ONE

  git checkout main
  git cherry-pick a1b2c3d
  git push origin main    → deploy hotfix

  git checkout develop    → already has it, no action needed

RESULT:
  main:    ... ── a1b2c3d'  (fix applied)
  develop: ... ── a1b2c3d   (original)
  (Same fix, two separate commits with different SHAs)
```

### 📍 Scenario: Backport a feature to older version

```

```

Company maintains v1 and v2.
New security patch committed on v2-branch:
b2c3d4e security: fix XSS in comment field

Backport to v1:
git checkout v1-branch
git cherry-pick b2c3d4e
→ Now both versions have the security fix

```

---

## 🔀 9.2 Diverged Branches Reconciliation

```

SCENARIO: main and feature both have new commits (diverged)

main: A ── B ── E ── F
\
 feature: C ── D

OPTIONS:

1. MERGE (preserves history):
   git checkout main
   git merge feature
   Result: A ── B ── E ── F ── M
   \ /
   C ──────── D
2. REBASE (linearizes):
   git checkout feature
   git rebase main
   Result: A ── B ── E ── F ── C' ── D'
   Then: git checkout main && git merge feature (fast-forward)

CHOOSE MERGE when:

- Public/shared branch
- Want to preserve true history
- Large team

CHOOSE REBASE when:

- Your local feature branch only
- Want clean linear log
- Before submitting PR

```

```

---

# 10. Stashing & Multi-work Setup

## 📦 10.1 `git stash` — Temporary Shelf

### How it works (backend)

Stash saves your working directory changes and staged changes into a stack stored in `.git/refs/stash`. Your working directory is restored to HEAD state.

```
STASH STACK:
  stash@{0}  ← most recent (WIP: on feature — half done login)
  stash@{1}  ← older (WIP: on main — debugging)
  stash@{2}  ← oldest

Internally stored as commit objects in .git/objects (not in regular history)
```

### Commands

```bash
git stash                           # stash working dir + staged changes
git stash push -m "half done login form"  # stash with a name
git stash -u                        # also stash untracked files
git stash -a                        # also stash ignored files

git stash list                      # see all stashes
git stash show                      # show what's in most recent stash
git stash show stash@{1}            # show specific stash

git stash pop                       # apply most recent + drop it from stack
git stash apply                     # apply most recent + KEEP it in stack
git stash apply stash@{2}           # apply a specific stash

git stash drop stash@{0}            # delete one stash
git stash clear                     # delete ALL stashes
git stash branch feature/saved stash@{0}  # create branch from stash
```

### 📍 Scenario: Urgent bug, need to switch branches

```
SITUATION:
  You're 30 minutes into writing a new feature.
  Boss: "PROD IS DOWN! Fix this NOW!"

YOUR WORK IN PROGRESS:
  src/checkout.js (modified)
  src/cart.js (modified)
  (not ready to commit — half done)

FLOW:
  1. git stash push -m "WIP: checkout refactor"
     → Working dir is now clean (back to last commit)

  2. git checkout main
  3. git pull
  4. git checkout -b hotfix/prod-crash
  5. ... fix the bug ...
  6. git commit -m "fix: null check prevents prod crash"
  7. git push → PR → merge → deploy

  8. git checkout feature/checkout
  9. git stash pop
     → Your half-done work is back!

  10. Continue where you left off.
```

---

## 🌲 10.2 `git worktree` — Two Branches at Once

### How it works (backend)

Worktree lets you check out multiple branches into **different directories simultaneously** from the same repo. Each directory is a separate working tree but shares `.git/`.

```
/projects/
  myapp/               ← main working tree (.git/ lives here)
    .git/
    src/               ← on branch: main

  myapp-feature/       ← linked working tree (no .git/, uses main's)
    src/               ← on branch: feature/JIRA-234

  myapp-hotfix/        ← another linked working tree
    src/               ← on branch: hotfix/prod-bug
```

### Commands

```bash
git worktree add ../myapp-feature feature/JIRA-234   # create new worktree
git worktree add -b hotfix/bug-99 ../myapp-hotfix    # create + new branch
git worktree list                                      # see all worktrees
git worktree remove ../myapp-feature                  # remove when done
git worktree prune                                     # clean stale worktrees
```

### 📍 Scenario: Review PR while coding

```
WITHOUT worktree:
  You're coding on feature/payment
  Need to review teammate's PR on feature/auth
  → Must stash your work
  → Checkout feature/auth
  → Review
  → Stash pop, go back
  (Annoying and loses context)

WITH worktree:
  cd ~/project
  git worktree add ../project-review feature/auth

  Now:
  Terminal 1: cd ~/project         → coding feature/payment
  Terminal 2: cd ~/project-review  → reviewing feature/auth

  Both open simultaneously! No stashing needed.
  IDE can open both. Tests can run in both simultaneously.
```

---
