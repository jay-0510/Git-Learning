# Git Debugging and Automation & Team Rules

# 11. Debugging Like a Pro

## 🔍 11.1 `git bisect` — Binary Search for Bugs

### How it works (backend)

Bisect uses **binary search** on your commit history. Given a "bad" commit and a "good" commit, Git checks out the midpoint, lets you test, and halves the search space each time. For 1000 commits, it finds the culprit in ~10 steps.

```
BISECT FLOW:

  [good] ── ... many commits ... ── [bad]
                   ↑
          Git checks out middle

  You test → "still bad" → left half eliminated

  [good] ── ... ── [mid was bad]
              ↑ new middle

  You test → "good here!" → found the range

  Continue until:
  The exact commit that introduced the bug is identified!
```

### Commands

```bash
git bisect start
git bisect bad                   # current HEAD is broken
git bisect good v1.2.0           # last known good tag/commit

# Git checks out a commit. You test:
# (run your app, run test, whatever)
git bisect bad                   # this commit is still broken
git bisect good                  # this commit is fine

# Repeat until Git says:
# "a1b2c3d is the first bad commit"

git bisect reset                 # return to HEAD when done
```

### Automated bisect

```bash
# If you have a test script that exits 0 (pass) or 1 (fail):
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run npm test -- --testFile cart.test.js

# Git automatically tests each midpoint!
```

### 📍 Scenario: Performance regression appeared somewhere

```
SITUATION:
  App response time was <100ms at v2.0.0 release
  Now at HEAD (after 200 commits): response is 800ms
  No idea which commit caused it

FLOW:
  git bisect start
  git bisect bad HEAD          # current is slow
  git bisect good v2.0.0       # this was fast

  [Git checks out commit #100 of 200]
  → Run performance test
  → Still slow (800ms)
  git bisect bad

  [Git checks out commit #50]
  → Test: fast! (95ms)
  git bisect good

  [Git checks out commit #75]
  → Test: slow
  git bisect bad

  [Git checks out commit #62]
  → Test: fast
  git bisect good

... (continues narrowing) ...

Result: "commit #68 is the first bad commit"
git show <commit-68> → "refactor: change DB query to use ORM"
→ Found! The ORM change caused N+1 query problem.

git bisect reset

---

# 12. Automation & Team Rules

## 🪝 12.1 Git Hooks — Basics
### How it works (backend)

Hooks are **scripts** stored in `.git/hooks/` that Git runs automatically at specific points. They can be any executable (bash, Python, Node.js).

```

.git/hooks/
├── pre-commit ← runs before git commit
├── commit-msg ← runs after you type commit message
├── pre-push ← runs before git push
├── post-commit ← runs after commit completes
├── pre-merge-commit ← runs before merge commit
└── prepare-commit-msg ← modify default commit message

````

> **Important**: `.git/hooks/` is NOT committed to the repo. Each developer must set up their own hooks, OR you use a tool like **husky** (Node) to auto-install them.

### 📍 Scenario: Run tests before every commit

```bash
# .git/hooks/pre-commit

#!/bin/bash
echo "Running tests before commit..."

npm test

if [ $? -ne 0 ]; then
  echo "❌ Tests failed! Commit aborted."
  exit 1
if
echo "✅ Tests passed. Proceeding with commit."
exit 0
```

```bash
chmod +x .git/hooks/pre-commit   # make executable
```

```
FLOW:
  git commit -m "feat: something"
        ↓
  pre-commit hook runs npm test
        ↓
  Tests FAIL → commit is blocked!
        ↓
  Fix tests → git commit works
```

---
## 🪝 12.2 Git Hooks — Advanced

### Using Husky (team-shared hooks in Node.js projects)

```bash
# Install
npm install husky --save-dev
npx husky init

# Creates .husky/ folder (COMMITTED to repo!)
# Everyone who clones + npm installs gets hooks automatically
```

```
.husky/           ← committed to git!
├── pre-commit
├── commit-msg
└── pre-push
```

```bash
# .husky/pre-commit
npm run lint
npm run test:unit

# .husky/pre-push
npm run test:integration
npm run build    # ensure it builds
```### Hook types and when they fire

```
pre-commit         → before commit (lint, test)
prepare-commit-msg → modify default message (add branch name)
commit-msg         → validate message format
post-commit        → after commit (notifications, logging)
pre-push           → before push (full test suite)
pre-rebase         → before rebase starts
post-checkout      → after branch switch (install deps?)
post-merge         → after merge (install deps if package.json changed)
```

---

## 📝 12.3 Commit Message Enforcement (JIRA-style)

### 📍 Scenario: Enforce JIRA ticket prefix on all commits

```
RULE: All commits must start with a JIRA ticket number like JIRA-123
Valid:   JIRA-234: feat: add login
Invalid: "just some changes"
```

```bash
# .git/hooks/commit-msg  (or .husky/commit-msg)

#!/bin/bash

COMMIT_MSG=$(cat "$1")
PATTERN="^(JIRA|BUG|FEAT)-[0-9]+: (feat|fix|docs|style|refactor|test|chore)"

if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
  echo ""
  echo "❌ Invalid commit message: $COMMIT_MSG"
  echo ""
  echo "Format required: JIRA-123: feat: short description"
  echo "Example: JIRA-234: feat(auth): add JWT refresh"
  echo ""
  exit 1
fi

echo "✅ Commit message format is valid."
exit 0
```
```
Valid commits:
  JIRA-234: feat(auth): add JWT refresh   ✓
  BUG-99: fix: null check on cart         ✓

Blocked commits:
  "WIP"                                    ✗
  "fixed stuff"                            ✗
  "JIRA-234 added login"                  ✗ (missing colon and type)
```

### Using commitlint (more powerful)

```bash
npm install @commitlint/cli @commitlint/config-conventional --save-dev

# commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'references-empty': [2, 'never'],   // must have JIRA ref
  }
};

# .husky/commit-msg
npx --no -- commitlint --edit "$1"
```
````
