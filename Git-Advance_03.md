# Git Large Production Scale & Team Workflow Style

# 13. Large Repos + Production Scale Git

## 📦 13.1 Git LFS — Large File Storage

### How it works (backend)

Git is great for text/code but terrible for large binary files (videos, ML models, design assets). Git LFS **replaces** large files in your repo with **tiny pointer files**, and stores the actual content on a separate server.

```
WITHOUT LFS:
  Your repo: 2MB code + 500MB Photoshop files = 502MB clone
  Every developer clones 502MB
  Every version of every large file stored in .git/ = HUGE repo

WITH LFS:
  Your repo: 2MB code + 1KB pointer files = 2MB clone
  Large files downloaded on-demand from LFS server

  In repo (pointer file content):
    version https://git-lfs.github.com/spec/v1
    oid sha256:a1b2c3d4...
    size 50000000

  Actual 50MB file lives on GitHub's LFS server
```

### Commands

```bash
# Install Git LFS first: https://git-lfs.github.com/
git lfs install                          # enable for your user

git lfs track "*.psd"                   # track Photoshop files
git lfs track "*.mp4"                   # track videos
git lfs track "models/*.pkl"            # track ML models

git add .gitattributes                   # commit the tracking rules
git add design/banner.psd               # now this is an LFS file

git lfs ls-files                         # show tracked LFS files
git lfs status                           # LFS status
git lfs push origin main                 # push LFS objects explicitly
```

### `.gitattributes` (auto-created by `lfs track`)

```
*.psd filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.pkl filter=lfs diff=lfs merge=lfs -text
design/**/*.png filter=lfs diff=lfs merge=lfs -text
```

### 📍 Scenario: Design + code in one repo

```
BEFORE LFS:
  Designers commit Figma exports → repo grows 500MB per sprint
  Developers clone and wait 20 minutes
  CI/CD pipeline slow because it downloads everything

AFTER LFS:
  Designers track *.psd, *.fig, *.mp4 with LFS
  Code clone: 5MB (fast)
  Designers' machines: download assets they need
  CI/CD: only pulls code unless it needs assets
```

---

## 📎 13.2 `git submodule` — Repos Within Repos

### How it works (backend)

Submodules let you include another Git repo inside your repo. The parent repo stores a **pointer** (commit SHA) to the submodule's specific commit, not the files themselves.

```
parent-repo/
├── .gitmodules          ← config: where submodule lives
├── src/
└── libs/
    └── shared-utils/    ← this IS another git repo
        ├── .git         ← separate repo!
        └── ...
```

.gitmodules:
[submodule "libs/shared-utils"]
path = libs/shared-utils
url = https://github.com/company/shared-utils.git
branch = main

````

### Commands

```bash
# Add a submodule
git submodule add https://github.com/company/shared-utils libs/shared-utils

# Clone a repo WITH its submodules
git clone --recurse-submodules <url>

# If already cloned without submodules:
git submodule init
git submodule update

# Update submodule to latest commit
git submodule update --remote libs/shared-utils

# Run command in all submodules
git submodule foreach "git pull origin main"
````

### 📍 Scenario: Microservices sharing a common library

```
company-backend/          ← parent repo
  services/
    auth-service/
    order-service/
  libs/
    shared-models/        ← submodule (company/shared-models repo)
    api-utils/            ← submodule (company/api-utils repo)

BENEFIT:
  shared-models is its own repo (other teams use it too)
  parent repo locks to a SPECIFIC commit of shared-models
  Updating shared-models in parent = deliberate PR + review

WORKFLOW:
  1. Team A updates shared-models repo
  2. Team B: cd libs/shared-models && git pull
  3. cd ../.. && git add libs/shared-models
  4. git commit -m "chore: update shared-models to v1.2"
  5. → Parent now tracks the new version intentionally
```

---

# 14. Team Workflow Style

## 🌊 14.1 GitFlow — The Classic Branching Model

### How it works

GitFlow defines **specific branch types** with specific purposes:

```
BRANCH TYPES:
  main        → production code only. Tagged with versions.
  develop     → integration branch. All features merge here.
  feature/*   → new features (branch from develop, merge to develop)
  release/*   → stabilization (branch from develop, merge to main+develop)
  hotfix/*    → urgent prod fixes (branch from main, merge to main+develop)
```

### Full GitFlow lifecycle

```
main:    v1.0 ─────────────────────────── v1.1 ──── v1.1.1
              \                           /  \          /
develop:       \─── D ─── E ─── F ─── G    \────────/
                \   /         \    /          ↑
feature/login:   A─B    feature/pay: C─D     hotfix/null-crash
                 (merge)              (merge)

release/v1.1:                  E ─── F (stabilize, regression tests)
                                       → merge to main (tag v1.1) + develop
```

### GitFlow commands

```bash
# Install git-flow extension (optional but helpful)
git flow init

# Feature workflow:
git flow feature start JIRA-234-user-login
  # (same as: git checkout -b feature/JIRA-234-user-login develop)
git flow feature finish JIRA-234-user-login
  # (same as: git merge to develop + delete branch)

# Release workflow:
git flow release start v1.1.0
  # (branch from develop)
  # fix bugs, update version numbers
git flow release finish v1.1.0
  # (merge to main + tag + merge back to develop)

# Hotfix:
git flow hotfix start prod-crash
git flow hotfix finish prod-crash
  # (merge to main + tag + merge to develop)
```

### 📍 Scenario: Full GitFlow sprint

```

```

SPRINT FLOW:

Week 1:
Dev A: git flow feature start JIRA-101-auth
Dev B: git flow feature start JIRA-102-dashboard

Week 2:
Dev A finishes: git flow feature finish JIRA-101-auth
→ merged to develop

Dev B finishes: git flow feature finish JIRA-102-dashboard
→ merged to develop

QA: tests develop branch

Week 3:
git flow release start v2.3.0
→ QA runs regression on release/v2.3.0
→ Minor fixes committed directly to release branch

git flow release finish v2.3.0
→ merged to main (tagged v2.3.0), merged back to develop
→ DEPLOYED to production

EMERGENCY (any time):
git flow hotfix start JIRA-999-login-crash
→ branch from MAIN (not develop!)
→ fix, test
git flow hotfix finish JIRA-999-login-crash
→ merged to main (tagged v2.3.1) AND develop

```

```

### GitFlow vs GitHub Flow

```
GITFLOW:
  Best for: scheduled releases, mobile apps, enterprise
  Complexity: higher
  Branches: main, develop, feature, release, hotfix

GITHUB FLOW (simpler alternative):
  Best for: continuous deployment, web apps, SaaS
  Complexity: lower
  Branches: main + feature branches (deploy directly from main)

  Flow:
    main
    → branch feature
    → PR to main
    → review + merge
    → auto-deploy
```

---
