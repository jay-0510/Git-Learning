# Git Versioning & Releases and History Cleanup & Professional Commit Management

# 7. Versioning & Releases

## 🏷️ 7.1 `git tag` — Mark a Release

### How it works (backend)

Tags are refs in `.git/refs/tags/`. There are two types:

```
LIGHTWEIGHT TAG:
  .git/refs/tags/v1.0.0 → points directly to a commit SHA
  (just a pointer, no extra data)

ANNOTATED TAG:
  .git/refs/tags/v1.0.0 → points to a TAG OBJECT
  TAG OBJECT contains:
    - tagger name/email/date
    - message
    - GPG signature (optional)
    - → then points to commit
```

### Commands

```bash
# Lightweight tag
git tag v1.0.0                           # tag current commit

# Annotated tag (use this for releases!)
git tag -a v1.0.0 -m "Release version 1.0.0"
git tag -a v1.2.3 a1b2c3d -m "Tag old commit"    # tag a specific past commit

# View tags
git tag                                  # list all
git tag -l "v1.*"                        # filter by pattern
git show v1.0.0                          # show tag details
```

# Push tags (tags are NOT pushed by default!)

git push origin v1.0.0 # push one tag
git push origin --tags # push all tags

# Delete

git tag -d v1.0.0 # delete local tag
git push origin --delete v1.0.0 # delete remote tag

```

### 📍 Scenario: Releasing version 2.1.0

```

FLOW:

1. Development complete on release/v2.1.0 branch
2. git checkout main
3. git merge release/v2.1.0 --no-ff
4. git tag -a v2.1.0 -m "Release 2.1.0 — dark mode, payment refactor"
5. git push origin main
6. git push origin v2.1.0
7. On GitHub → create Release from tag v2.1.0 (adds release notes, assets)

VERSIONING STANDARD (Semantic Versioning):
MAJOR.MINOR.PATCH

v2.1.0 → MAJOR: breaking changes, MINOR: new features, PATCH: bug fixes

v1.0.0 → first stable release
v1.0.1 → bug fix
v1.1.0 → new feature added
v2.0.0 → breaking API change
v2.1.0-beta.1 → pre-release

```

---
```

# 8. History Cleanup & Professional Commit Management

## 🗜️ 8.1 Squash

### How it works (backend)

Squash combines multiple commits into one. `--squash` in merge doesn't create the merge commit — it stages all the changes for you to commit as one.

```
BEFORE (your messy feature branch):
  A ── B ── C ── D ── E   (main)
              \
               F ── G ── H ── I ── J   (feature — 5 "WIP" commits)

git merge --squash feature (from main):
  All changes from F,G,H,I,J are staged as one commit

AFTER:
  A ── B ── C ── D ── E ── S   (main, S = squash commit)
  (Feature branch history collapsed into one clean commit)
```

---

## ♻️ 8.2 `git rebase` — Replay Commits

### How it works (backend)

Rebase **replays** your commits on top of another base, creating brand-new commits (new SHAs) in the process.

```
BEFORE:
  main:    A ── B ── E ── F     (main moved ahead)
                \
  feature:        C ── D        (your branch, based on B)

git checkout feature
git rebase main:

  Step 1: Find common ancestor (B)
  Step 2: Remove C and D temporarily
  Step 3: Fast-forward feature to F
  Step 4: Replay C as C' on top of F
  Step 5: Replay D as D' on top of C'

AFTER:
  main:    A ── B ── E ── F
                           \
  feature:                  C' ── D'   (new commits, new SHAs!)
```

> ⚠️ Rebase rewrites commit SHAs. Never rebase shared/public branches.

### Commands

```bash
git rebase main                  # rebase current branch onto main
git rebase --abort               # cancel mid-rebase
git rebase --continue            # continue after resolving conflict
git rebase --skip                # skip current problematic commit
```

---

## 🎛️ 8.3 Interactive Rebase — Rewrite History Like a Pro

### How it works (backend)

`git rebase -i` opens an editor showing commits as a list of "actions". You can reorder, squash, edit, drop, or rename any commit.

```
git rebase -i HEAD~5    (edit last 5 commits)

Opens editor:
  pick a1b2c3d feat: add login form
  pick b2c3d4e WIP — half done
  pick c3d4e5f fix: oops forgot import
  pick d4e5f6a WIP still debugging
  pick e5f6a7b feat: login works finally!
```

ACTIONS available:
pick = use commit as-is
reword = use commit, but edit message
edit = use commit, pause to amend it
squash = meld into previous commit (keeps both messages)
fixup = meld into previous commit (discards this message)
drop = remove commit entirely
reorder lines to reorder commits

```

### 📍 Scenario: Clean up before PR
```

```
Your branch history (messy):
  a1b2c3d  feat: add login form
  b2c3d4e  WIP
  c3d4e5f  oops, forgot import
  d4e5f6a  wip still debugging
  e5f6a7b  finally works
  f6a7b8c  fix lint errors
  g7b8c9d  add tests

$ git rebase -i HEAD~7

EDIT to:
  pick  a1b2c3d  feat: add login form
  fixup b2c3d4e  WIP
  fixup c3d4e5f  oops, forgot import
  fixup d4e5f6a  wip still debugging
  reword e5f6a7b  finally works          ← will prompt you to rename
  fixup  f6a7b8c  fix lint errors
  pick   g7b8c9d  add tests

RESULT (clean, reviewable history):
  a1b2c3d  feat: add login form
  h2i3j4k  feat: complete login with validation   ← renamed + squashed
  g7b8c9d  test: add login component tests
```

---

## ⚖️ 8.4 Rebase vs Merge — When to Use What

```
                    MERGE                      REBASE
──────────────────────────────────────────────────────────────────
History           Preserves exact history    Rewrites to linear
Merge commit      Yes (usually)              No
Safe on shared    ✓ Always                  ✗ Never (rewrites SHAs)
Use for           Feature → main            Update branch with main
Team size         Large teams               Small teams / solo

RULE OF THUMB:
  "Rebase before PR to update your branch (local only)"
  "Merge to integrate (shared history preserved)"

COMMON WORKFLOW:
  feature branch: git rebase main   ← keep updated (your local only)
  PR ready:       merge into main   ← preserve history
```

```
DO:    git rebase main         (while on your feature branch, before PR)
DON'T: git rebase feature      (while on main — don't rewrite main!)
DO:    git merge --no-ff feature (to integrate into main)
```

---
