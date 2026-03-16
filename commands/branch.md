---
description: Create a feature branch from main with clean test baseline
allowed-tools: Bash(git *), Bash(bin/rails test *)
model: haiku
argument-hint: <short description of the work>
---

Do the following in order:

1. Check for uncommitted changes: `git status --porcelain`
   If there are changes, STOP: "You have uncommitted changes. Run /commit
   or stash them first."
2. Check out main and pull latest. Create a new branch from main.
   Branch naming rules:

- Feature work: feature/<short-description>
- Bug fix: fix/<short-description>
- Derive the short-description from $ARGUMENTS

3. Run tests to verify clean baseline. If tests fail, STOP and report —
   don't start work on a broken base.
4. After branch creation, report:

- Branch name
- Test status
- "Ready to work"
