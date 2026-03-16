---
description: Quick mid-session commit following project git conventions
allowed-tools: Bash(git *)
model: haiku
argument-hint: <commit message or leave blank to auto-generate>
---

1. Check current branch first: `git branch --show-current`
   If on main, STOP and warn: "You're on main. Create a feature branch
   first with /branch, or confirm you intentionally want to commit to main."
2. Stage and commit current changes following git-conventions rules.
   Review staged changes for debug artifacts and secrets before committing.
   If $ARGUMENTS provided, use as commit message. Otherwise, generate
   from the diff.
   Do NOT push — just commit locally.
3. When done say: "Commit complete."
