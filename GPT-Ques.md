# Git Interview Prep – Scenario-Based Revision

---

## 1. Feature Branch Behind Main

### ❓ Scenario

Feature branch is behind main. Need latest changes, clean history, avoid merge commits.

### ✅ Answer

bash git checkout feature/payment </br> git fetch origin</br> git rebase origin/main

### 💡 Explanation

- Rebase replays your commits on top of latest main
- Produces linear history (clean)
- Avoids merge commits

### ⚠️ Risk

- Rewrites history → requires git push --force-with-lease

---

## 2. Rebase Conflict Resolution

### ❓ Scenario

Conflict occurs during rebase.

### ✅ Answer

bash git status </br># fix conflicts manually git add <file></br> git rebase --continue </br># optional git rebase --abort

### 💡 Explanation

- Rebase pauses on conflict
- You resolve → stage → continue
- Do NOT use git commit

### 🔍 Conflict Markers

<<<<<<< HEAD your changes (current commit being replayed) ======= incoming changes (target branch) >>>>>>> branch-name

---

## 3. Push After Rebase

### ❓ Scenario

Push rejected after rebase.

### ✅ Answer

bash git push --force-with-lease </br>

### 💡 Explanation

- History rewritten → normal push fails
- --force-with-lease ensures remote wasn’t changed by others

### ⚠️ Avoid

bash git push --force </br>

- Overwrites blindly (dangerous)

---

## 4. Recover Lost Commits (Reset Hard)

### ❓ Scenario

Accidentally ran git reset --hard HEAD~3

### ✅ Answer

bash git reflog </br> git reset --hard <commit_hash>

### 💡 Explanation

- Reflog stores HEAD history
- Reset moves pointer back to lost commit

---

## 5. Remove Sensitive Data from History

### ❓ Scenario

API key committed and pushed

### ✅ Answer

bash git filter-repo --path <file> --invert-paths </br> git push --force --all

### 💡 Explanation

- Rewrites entire history
- Removes file completely (not just latest commit)

### ❌ Why Not git revert

- Only undoes change
- Sensitive data still exists in history

---

## 6. Feature Branch Workflow

### ❓ Scenario

Working with main, develop, and feature branches

### ✅ Answer

bash git checkout develop </br> git pull origin develop </br>git checkout -b feature/login </br> git fetch origin </br>git rebase origin/develop </br> git push origin feature/login </br> git checkout develop </br> git merge feature/login </br>git push origin develop

### 💡 Explanation

- Start from develop (integration branch)
- Rebase for clean history
- Merge back into develop, not main

---

## 7. Amend Last Commit

### ❓ Scenario

Wrong file added + wrong commit message

### ✅ Answer

bash git rm --cached <file> </br> git commit --amend

### 💡 Explanation

- Remove file from staging
- Amend rewrites last commit (no new commit)

---

## 8. Stash for Context Switching

### ❓ Scenario

Need to switch branch without committing

### ✅ Answer

bash git stash push -m "WIP" </br> git checkout other-branch </br># later git checkout feature-branch </br> git stash pop

### 💡 Explanation

- Saves working directory + staging area
- Acts like temporary commit (outside history)

---

## 9. Recover Deleted Branch

### ❓ Scenario

Deleted branch accidentally

### ✅ Answer

bash git reflog </br> git checkout -b feature/cart <commit_hash>

### 💡 Explanation

- Branch deletion removes pointer only
- Commits still exist → recover via reflog

---

## 10. Merge vs Rebase

### ❓ Question

### 🔁 Merge

bash git merge branch

#### 💡 Concept

- Combines histories
- Creates merge commit

#### ⚙️ Internal

- Two parent commits → merge node

#### ✅ Use When

- Shared/public branches
- Preserve full history

---

### 🔄 Rebase

bash git rebase branch

#### 💡 Concept

- Rewrites history
- Reapplies commits on new base

#### ⚙️ Internal

- Removes commits → reapplies them

#### ✅ Use When

- Feature branches
- Clean linear history

---

### 🎯 Golden Rule

> Use rebase for clean history, merge for shared stability.

# Git Interview Prep – Scenarios 11 to 17

---

## 11. Debugging Changes (git diff)

### ❓ Scenario

- Check unstaged changes
- Check staged changes
- Compare branch with main

### ✅ Answer

bash git diff # unstaged (working dir vs index) </br> git diff --staged # staged (index vs HEAD) </br> git diff main..feature # branch comparison

### 💡 Explanation

- Git compares snapshots, not files directly
- Layers:
  - Working Directory
  - Staging Area (Index)
  - HEAD (last commit)

---

## 12. Search in Repo (git grep)

### ❓ Scenario

Find processPayment in .py files with line numbers

### ✅ Answer

bash git grep -n "processPayment" -- "\*.py" </br>

### 💡 Explanation

- Searches only tracked files
- Faster than normal grep
- Can search specific commits/branches

---

## 13. Apply Specific Commit (git cherry-pick)

### ❓ Scenario

Apply commit abc123 from another branch

### ✅ Answer

bash git checkout feature/cart </br> git cherry-pick abc123

### 💡 Explanation

- Copies changes from commit
- Creates new commit with new hash

### ⚠️ Conflict Handling

bash git add <file> </br> git cherry-pick --continue # </br> or git cherry-pick --abort

---

## 14. Squash Commits

### ❓ Scenario

Convert 5 commits into 1 clean commit

### ✅ Answer

bash git rebase -i HEAD~5 </br>

Then:
pick commit1 squash commit2 squash commit3 squash commit4 squash commit5

### 💡 Explanation

- Combines commits
- Rewrites history → new commit hash

---

## 15. Squash + Push Issue

### ❓ Scenario

Squashed commits already pushed → teammate pulled old history

### ❗ Problem

- History divergence
- Duplicate commits / conflicts

### ✅ Solution

bash git push --force-with-lease

### 👥 Teammate Fix

bash git fetch </br> git reset --hard origin/branch

### 💡 Explanation

- Rebase rewrites commit hashes
- Requires coordination

---

## 16. Clean PR History

### ❓ Scenario

PR has messy merge commits

### ✅ Answer

bash git checkout feature-branch </br> git fetch origin </br> git rebase origin/main </br> git rebase -i origin/main </br> git push --force-with-lease

### 💡 Explanation

- Rebase + squash → clean history
- Avoid on shared branches

### ⚠️ Rule

- Rebase → feature branch
- Merge → shared branch

---

## 17. Git Internals (Core Concepts)

### ❓ Questions

#### 1. What is HEAD?

👉 Pointer to current commit (current branch position)

#### 2. What is origin?

👉 Default remote repository (alias)

#### 3. Difference

| Term        | Meaning                |
| ----------- | ---------------------- |
| main        | Local branch           |
| origin/main | Remote-tracking branch |

---

### 🧠 Internal Working (DAG)

- Git stores commits as nodes in Directed Acyclic Graph (DAG)
- Each commit points to parent commit(s)
- Merge commit → multiple parents
- Rebase → rewrites chain

👉 Think:

> “Git is a graph of commits, not a sequence of files”

---

### 💡 Strong Interview Line

> “Git internally maintains a DAG of commits where branches are just pointers, and HEAD is the current reference to the active commit.”

--
