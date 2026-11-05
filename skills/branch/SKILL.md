---
name: branch
version: 1.3 # bump on meaningful changes
description: >
  Create or rename a branch, validating it against the Git Branch Naming
  Rules in CLAUDE.md. Use when the user runs /branch or asks for a
  branch to be created or renamed. Do NOT run from another skill or
  workflow step, and never create or rename a branch the user didn't ask
  for.
allowed-tools: Bash(git *)
model: haiku
argument-hint: "<TICKET-ID> <description>"
---

Create or validate a branch name per Git Branch Naming Rules in CLAUDE.md.

1. If $ARGUMENTS is empty or incomplete, ask for a ticket ID and/or short description.

2. Check for uncommitted changes: `git status --porcelain`  
   If any:
   - **The user asked to bring them onto the new branch** (e.g. "branch and
     commit this", "move these changes to a branch"): carry them over. On
     `main`/`master`, set them aside before step 6's pull with
     `git stash push -u -m "<name>"`, and run `git stash pop` right after
     `git checkout -b <name>`. If the pop reports a conflict, STOP and report
     it; the changes stay safe in the stash. When renaming a non-main branch,
     no stash is needed — `git branch -m` keeps them.
   - **Otherwise** STOP: "You have uncommitted changes. Commit or stash them
     first, or tell me to carry them onto the new branch."

3. Get current branch: `git branch --show-current`.

4. Read the applicable Git Branch Naming Rules (CLAUDE.md / AGENTS.md) and
   determine the required type prefix. For a ticket, use the task's type;
   default to `feature` unless the user clearly describes a bug fix (`fix`)
   or maintenance task (`chore`). Lowercase the ticket identifier.
   - Ticket: `<type>/<ticket-id>-<short-description>`
   - No ticket: `<type>/<short-description>`
   Never use the Conductor workspace name as a branch prefix. A workspace
   slug such as `pangyo/...` is not a valid substitute for `feature/...`.

5. Validate the complete target name against the rules, including the type
   prefix and ticket identifier. Do not report success if only the slug
   matches or if the branch still has a workspace-name prefix:
   - If invalid:
     - Briefly explain why (wrong prefix, missing id, bad format, etc.).
     - Suggest a valid alternative.
     - Ask user to confirm or provide another.
   - Use confirmed name.

6. Every path ends with the branch up to date with `origin/main` (or
   `origin/master`). Never skip the sync, and never start ticket work from a
   stale base. Start with `git fetch origin`. If the fetch fails (no network,
   no remote), STOP and report the error.

   - **On `main`/`master`:** sync, then create the branch:  
     `git pull --ff-only`  
     If the pull fails (diverged history, no upstream), STOP and report the
     error. Then: `git checkout -b <name>`

   - **On any other branch** (e.g. a Conductor workspace branch): rename it
     if necessary with `git branch -m <name>`, then check how it compares:  
     `git rev-list --left-right --count origin/main...HEAD`  
     (left = commits behind, right = commits ahead)
     - **0 ahead, some behind:** fast-forward it:
       `git merge --ff-only origin/main`. This only moves the branch forward;
       it never rewrites or merges anything. If carrying uncommitted changes,
       `git stash push -u -m "<name>"` first and `git stash pop` after (same
       conflict rule as step 2).
     - **0 ahead, 0 behind:** already current, nothing to do.
     - **Some ahead (the branch has its own commits):** STOP. Don't rebase or
       merge on your own. Report the ahead/behind counts and ask whether to
       rebase onto or merge `origin/main`.

   Finally, confirm with the same `rev-list` command that the branch is 0
   behind `origin/main`.

7. Report: branch name, that it is up to date with `origin/main` (the commit
   it now sits on, and how many commits the sync brought in), whether
   uncommitted changes were carried over, and "Ready to work". If the sync
   brought in new migrations (`db/migrate/`), say to run
   `bin/rails db:migrate`.
