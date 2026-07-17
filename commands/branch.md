---
description: Create a feature branch from main with clean test baseline
allowed-tools: Bash(git *), Bash(bin/rails test *)
model: haiku
argument-hint: <short description, or "<TICKET-ID> <description>" for ticket work>
---

This command is a one-off for the **traditional single-checkout
workflow** — creating a branch from main. You won't normally run it: in a
Conductor workspace the branch already exists (Conductor creates it), and
`/brainstorm`'s Phase 0 renames it to embed the ticket identifier. Use
this command only when you need to create a branch yourself outside the
Conductor flow.

Do the following in order:

1. Check for uncommitted changes: `git status --porcelain`
   If there are changes, STOP: "You have uncommitted changes. Run /commit
   or stash them first."
2. Check out main and pull latest. Create a new branch from main
   following the Git Branch Naming Rules in CLAUDE.md: if $ARGUMENTS
   starts with a ticket identifier (matching `[A-Z]+-\d+`), use the
   ticket-derived format (embed it, lowercased); otherwise the plain
   `feature/<short-description>` form. Derive the description from
   $ARGUMENTS.
3. Run tests to verify clean baseline. If tests fail, STOP and report —
   don't start work on a broken base.
4. After branch creation, report:

- Branch name
- Test status
- "Ready to work"
