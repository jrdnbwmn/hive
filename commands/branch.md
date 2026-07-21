---
description: Create or rename a branch with validation against naming rules
allowed-tools: Bash(git *)
model: haiku
argument-hint: <TICKET-ID> <description>
---

Create or validate a branch name per Git Branch Naming Rules in CLAUDE.md.

1. If $ARGUMENTS is empty or incomplete, ask for a ticket ID and/or short description.

2. Check for uncommitted changes: `git status --porcelain`  
   If any, STOP: "You have uncommitted changes. Run /commit or stash them first."

3. Get current branch: `git branch --show-current`.

4. Determine target name from $ARGUMENTS (following the Git Branch Naming Rules in CLAUDE.md)
   - If it has a ticket identifier (matching `[A-Z]+-\d+`), use the
   ticket-derived format: `<type>/<identifier>-<short-description>`
   - Otherwise use plain `<type>/<slug>` form

5. Validate the name against rules:
   - If invalid:
     - Briefly explain why (wrong prefix, missing id, bad format, etc.).
     - Suggest a valid alternative.
     - Ask user to confirm or provide another.
   - Use confirmed name.

6. If on `main`/`master`: create branch from it:  
   `git checkout -b <name>`  
   Else: rename current branch if necessary:  
   `git branch -m <name>`

7. Report: branch name and "Ready to work".
