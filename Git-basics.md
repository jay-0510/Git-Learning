# Mastery of Git - Scratch to Industry Level

## Table of Contents

1. [Git Setup + Start a Repo](#1-git-setup--start-a-repo)
2. [Clean Repo & Ignore Rules](#2-clean-repo--ignore-rules)
3. [Basic Daily Workflow](#3-basic-daily-workflow)
4. [Branching Basics](#4-branching-basics)
5. [Undo & Recovery](#5-undo--recovery)
6. [Collaboration & Remote Workflow](#6-collaboration--remote-workflow)
7. [Versioning & Releases](#7-versioning--releases)
8. [History Cleanup & Professional Commit Management](#8-history-cleanup--professional-commit-management)

# 1. Git Setup + Start a Repo

## 🔧 1.1 `git config` — Who Are You?

Git needs to know who is making changes. Every commit is stamped with a name and email.

### How it works (backend)

Git stores config in three layers, each overriding the one above it:

```
[System]   → /etc/gitconfig          (all users on machine)
    ↓ overridden by
[Global]   → ~/.gitconfig            (your user account)
    ↓ overridden by
[Local]    → .git/config             (this specific repo)
```

### Commands

```bash
# Set globally (most common — do this once after installing Git)
git config --global user.name "Arjun Shah"
git config --global user.email "arjun@company.com"

# Set locally (override just for one repo — e.g., work vs personal)
git config --local user.email "arjun@freelance.com"

# Check what's configured
git config --list
git config user.name

# Set default branch name (industry standard is 'main')
git config --global init.defaultBranch main

# Set default editor (for commit messages)
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "vim"
```

### 📍 Scenario: Work laptop with two identities

```
PROBLEM:
  You work at a company (arjun@company.com) but also have
  a personal open-source project (arjun@gmail.com).

SOLUTION:
  Global config → company email (used everywhere by default)
  Local config  → override in the personal project folder

```

~/company-project/ → uses arjun@company.com (from global)
~/oss-project/ → uses arjun@gmail.com (from local .git/config)

````

```bash
# Inside ~/oss-project only:
git config --local user.email "arjun@gmail.com"
````

---

## 🏗️ 1.2 `git init` — Creating a Repo

### How it works (backend)

Running `git init` creates a hidden `.git/` folder. This IS your repository — everything Git knows lives here.

```
my-project/
├── .git/                  ← Git's brain
│   ├── HEAD               ← points to current branch
│   ├── config             ← local repo config
│   ├── objects/           ← all commits, files, trees stored as blobs
│   ├── refs/
│   │   ├── heads/         ← local branches (e.g., refs/heads/main)
│   │   └── tags/          ← tags
│   └── index              ← staging area (what's ready to commit)
├── src/
└── README.md
```

### Commands

```bash
# Start a new repo in current directory
git init

# Start with a specific branch name
git init -b main

# Initialize a bare repo (used on servers — no working directory)
git init --bare my-repo.git
```

## 📊 1.3 `git status` — What's Going On?

### How it works (backend)

Git compares three things:

1. **HEAD** (last commit)
2. **Index/Staging area** (what you've `git add`-ed)
3. **Working directory** (your actual files right now)

```
Working Dir ──[git add]──▶ Staging Area ──[git commit]──▶ Repository (HEAD)
                                                                ↑
                              git status shows differences between all three
```

### Commands

```bash
git status          # full status
git status -s       # short/compact format
git status -sb      # short + branch info
```

### Reading `git status -s` output

```
?? file.txt         → Untracked (Git doesn't know about it)
A  file.txt         → Added to staging (new file)
M  file.txt         → Modified and staged
 M file.txt         → Modified but NOT staged (column 2 = working dir)
MM file.txt         → Modified, partially staged (both columns)
D  file.txt         → Deleted and staged
```

### 📍 Scenario: You edited 3 files, forgot which ones

```bash
$ git status -s

 M  src/app.js        → modified, not staged yet
A   src/utils.js      → new file, added to staging
?? logs/debug.log    → untracked, Git ignoring it (you should .gitignore this)
```

---

## 🔍 1.4 Tracked vs Untracked

### How it works (backend)

Git maintains an **object database** in `.git/objects/`. A file is "tracked" only when Git has stored a snapshot of it (via `git add` → creates a blob object).

```
UNTRACKED:  file exists on disk → Git has ZERO knowledge of it
TRACKED:    file is in Git's index → Git watches it for changes

                 ┌─────────────────────────────────────────┐
                 │           File Lifecycle                 │
                 │                                         │
  New file ──▶  UNTRACKED                                  │
                   │                                       │
               git add                                     │
                   ↓                                       │
               STAGED (tracked, in index)                  │
                   │                                       │
             git commit                                    │
                   ↓                                       │
               COMMITTED (in history)                      │
                   │                                       │
           edit the file again                             │
                   ↓                                       │
               MODIFIED (tracked but has changes)          │
                                                           │
  git rm → back to UNTRACKED (or gone)                    │
                 └─────────────────────────────────────────┘
```

---

# 2. Clean Repo & Ignore Rules

## 📄 2.1 `.gitignore` — Keep Junk Out

### How it works (backend)

`.gitignore` is just a plain text file. Git reads it and uses **glob pattern matching** to decide which untracked files to completely ignore. It does NOT affect already-tracked files.

> ⚠️ **Critical**: If you already committed `node_modules/`, adding it to `.gitignore` won't remove it. You must `git rm -r --cached node_modules/` first.

### Pattern Rules

```
node_modules/       → ignore the folder and everything inside
*.log               → ignore all .log files anywhere
!important.log      → EXCEPT this specific one (negation)
/secret.env         → ignore only at root level (not in subfolders)
dist/               → ignore dist folder
**/*.tmp            → ignore .tmp files in any nested folder
```

### Pattern Rules

```
node_modules/       → ignore the folder and everything inside
*.log               → ignore all .log files anywhere
!important.log      → EXCEPT this specific one (negation)
/secret.env         → ignore only at root level (not in subfolders)
dist/               → ignore dist folder
**/*.tmp            → ignore .tmp files in any nested folder
```

### 📍 Scenario: Node.js project going to GitHub

```
BEFORE (without .gitignore):
  What gets committed:
  ├── src/
  ├── node_modules/   ← 50,000 files, 300MB — DISASTER
  ├── .env            ← your database password — SECURITY RISK
  └── dist/           ← generated output — no need

AFTER (with .gitignore):
  What gets committed:
  ├── src/            ← your actual code ✓
  ├── .gitignore      ← the rules file ✓
  └── package.json    ← so others can run npm install ✓
```

### Standard `.gitignore` for a Node.js project

```gitignore
# Dependencies
node_modules/
.pnp
.pnp.js

# Environment variables — NEVER commit these
.env
.env.local
.env.*.local

# Build outputs
dist/
build/
out/

# Logs
*.log
logs/
npm-debug.log*

# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
*.swp
```

### Scope of `.gitignore` files

```
my-project/
├── .gitignore          ← applies to whole repo
├── src/
│   └── .gitignore      ← applies only inside src/
└── backend/
    └── .gitignore      ← applies only inside backend/
```

### Global gitignore (for your machine)

```bash
# Create a global ignore file (OS files, editor junk — never project-specific)
git config --global core.excludesFile ~/.gitignore_global

# ~/.gitignore_global:
.DS_Store
.idea/
*.swp
```

---

## 🧹 2.2 `git clean` — Remove Untracked Junk Safely

### How it works (backend)

`git clean` removes files from the **working directory** that are NOT tracked by Git and NOT in `.gitignore`. Think of it as "reset to what Git knows about."

> ⚠️ This is **permanent** — cleaned files do NOT go to trash.

### Commands

```bash
git clean -n          # dry run — preview what WOULD be deleted (always do this first!)
git clean -f          # force delete untracked files
git clean -fd         # force delete untracked files AND directories
git clean -fx         # also delete files ignored by .gitignore
git clean -fdx        # nuclear option: delete ALL untracked + ignored
```

### 📍 Scenario: Build process created junk files

```

```

BEFORE git clean:
project/
├── src/app.js ← tracked ✓
├── dist/ ← untracked (build output)
│ ├── app.bundle.js
│ └── app.bundle.css
├── .cache/ ← untracked (temp cache)
└── debug.log ← untracked

$ git clean -n
Would remove dist/
Would remove .cache/
Would remove debug.log

$ git clean -fd

AFTER git clean:
project/
├── src/app.js ← untouched ✓
(everything else gone)

```

```

### 📍 Scenario: Switching branches and build artifacts conflict

```
You're on feature-branch, ran the build.
Now switching to main — old dist/ files conflict.

git clean -fd   → remove all untracked artifacts
git checkout main  → clean switch
```

---
