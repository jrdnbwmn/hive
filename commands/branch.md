---
description: Create a feature branch from main with clean test baseline
allowed-tools: Bash(git *), Bash(bin/rails test *)
model: haiku
argument-hint: <short description, or "<TICKET-ID> <description>" for ticket work>
---

Do the following in order:

1. Check for uncommitted changes: `git status --porcelain`
   If there are changes, STOP: "You have uncommitted changes. Run /commit
   or stash them first."
2. Check out main and pull latest. Create a new branch from main,
   following the Git Branch Naming Rules:
   - If $ARGUMENTS starts with a ticket identifier (e.g. `TIC-123`,
     matching `[A-Z]+-\d+`), lowercase it and embed it:
     `feature/tic-123-<short-description>` (derive the description
     from the rest of $ARGUMENTS). This is required for Linear's
     GitHub integration to auto-link the branch — don't drop it.
   - Otherwise derive the branch name from $ARGUMENTS as before:
     `feature/<short-description>`
3. Run tests to verify clean baseline. If tests fail, STOP and report —
   don't start work on a broken base.
4. After branch creation, report:

- Branch name
- Test status
- "Ready to work"
