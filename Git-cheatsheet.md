# Git Cheat Sheet

# 🎓 Quick Reference: Command Cheat Sheet

## Daily Workflow

```bash
git status -sb                    # what's going on
git diff                          # what changed (unstaged)
git diff --staged                 # what will be committed
git add -p                        # stage selectively
git commit -m "type: message"     # commit
git push                          # push to remote
```

## Branch Operations

```bash
git switch -c feature/name        # create + switch
git switch main                   # switch back
git branch --merged               # branches done
git branch -d feature/name        # delete merged branch
```

## Undo / Recovery

```bash
git restore src/file.js           # discard file changes
git restore --staged src/file.js  # unstage
git commit --amend --no-edit      # add to last commit
git reset --soft HEAD~1           # undo commit, keep staged
git revert HEAD                   # safe undo (adds new commit)
git reflog                        # find lost commits
```

## Remote

```bash
git fetch --all --prune           # fetch all, clean stale refs
git pull --rebase                 # pull cleanly
git push -u origin feature/name  # push + set upstream
git remote -v                     # check remotes
```

## History

```bash
git log --oneline --graph --all   # visual branch graph
git log --author="Arjun"          # filter by author
git log --since="1 week ago"      # filter by time
git log src/app.js                # file history
git show a1b2c3d                  # show specific commit
git blame -L 10,20 src/app.js     # who wrote lines 10-20
```

## Dangerous but useful

```bash
git reset --hard origin/main      # throw away local, match remote
git push --force-with-lease       # safer force push (checks remote first)
git clean -fdx                    # delete everything untracked
git reflog expire --expire=now --all && git gc  # clean up
```

---

# 🗺️ Decision Trees

## When to Merge vs Rebase?

```
Is the branch shared with others (pushed to remote)?
  YES → Use MERGE (never rebase shared history)
  NO  → Either is fine

Are you updating your feature branch with latest main?
  YES → Use REBASE (git rebase main → cleaner, linear)

Are you integrating a finished feature into main?
  YES → Use MERGE --no-ff (preserves feature branch context)
```

## When something breaks — what to use?

```
Committed wrong thing on LOCAL branch?
  → git reset --soft HEAD~1    (undo, keep code)
  → git reset --hard HEAD~1    (undo, delete code)

Committed wrong thing + PUSHED to shared branch?
  → git revert HEAD            (safe — adds undo commit)

Lost commits after reset?
  → git reflog                 (find them)
  → git branch recover <SHA>  (rescue)

Wrong file in commit?
  → git commit --amend         (if not pushed yet)

Need to move a commit from one branch to another?
  → git cherry-pick <SHA>

Need to undo one specific old commit (not the latest)?
  → git revert <SHA>
```

## Which remote command to use?

```
Just see what's on remote without changing anything?
  → git fetch
```

Update your branch with remote changes?
→ git pull (or git pull --rebase for cleaner history)

Check remote changes before merging?
→ git fetch → git log HEAD..origin/main → git merge

```

---

```
