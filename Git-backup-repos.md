# Git Backup / Sharing Repos Offline

## 📦 16.1 `git archive` — Export Snapshot

### How it works (backend)

`git archive` creates a snapshot of your repo (or a specific commit/branch) as a `.tar` or `.zip` **without the `.git/` folder**. Good for distributing source code, deployment packages, or backups of a specific release.

```
git archive HEAD

  Takes all files in current HEAD commit
  Packages them into tar/zip
  NO version history, NO .git/
  Just clean source files
```

### Commands

```bash
# Export current HEAD as zip
git archive --format=zip --output=release.zip HEAD

# Export specific tag
git archive --format=zip --output=v2.1.0-src.zip v2.1.0

# Export only a subfolder
git archive --format=tar HEAD src/ | tar -xv -C /tmp/output/

# With a prefix in the archive
git archive --format=zip --prefix=myapp-v2.1.0/ --output=release.zip v2.1.0
```

### 📍 Scenario: Send code to client without Git history

```
Client needs your source code but you don't want to share
your entire commit history (internal notes, author info, etc.)

git archive --format=zip --prefix=project-v1.0/ \
  --output=client-delivery-v1.0.zip v1.0.0

→ client gets: project-v1.0/src/, project-v1.0/README.md, etc.
→ No .git/, no history, no internal branches
```

---

## 🎁 16.2 `git bundle` — Repo in a File (WITH History)

### How it works (backend)

`git bundle` packages your **entire Git repository** (or part of it) into a single binary file. Unlike archive, it contains the full object database and refs — it behaves like a real remote.

```
.git/ (full repo) ──[git bundle create]──▶ repo.bundle
                                               ↓
                                     Can be cloned, fetched from,
                                     just like a real remote!
```

### Commands

```bash
# Bundle entire repo
git bundle create repo.bundle --all

# Bundle only main branch
git bundle create main.bundle main

# Bundle changes since a tag (incremental backup)
git bundle create incremental.bundle v1.0.0..HEAD

# Verify a bundle
git bundle verify repo.bundle

# Clone from a bundle
git clone repo.bundle cloned-repo

# Fetch from a bundle (incremental update)
git fetch repo.bundle main:main
```

### 📍 Scenario: Transfer repo with no internet

```
SCENARIO: Moving a private project between two machines
          on the same local network, or via USB drive.

ON SOURCE MACHINE:
  git bundle create /usb-drive/myproject.bundle --all
  → Everything packed: all branches, all history, all tags
```

ON TARGET MACHINE:
git clone /usb-drive/myproject.bundle ~/projects/myproject
→ Full repo cloned locally with complete history!

# Set real remote after:

git remote set-url origin git@github.com:company/myproject.git
git push origin --all
git push origin --tags

```

### 📍 Scenario: Incremental offline sync

```

Team A works without internet for a week.
At end of week, sync with Team B via bundle.

TEAM A:
git bundle create week3-changes.bundle v2.0.0..HEAD

# Only commits since v2.0.0 (what Team B has)

TEAM B:
git fetch ./week3-changes.bundle main:refs/remotes/teama/main
git merge refs/remotes/teama/main

# Merged Team A's week of work!

```

---
```
