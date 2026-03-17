---
name: wrap-up
version: 1.0 # bump on meaningful changes
description: >
  End-of-session ritual: ship, remember, improve. Commits code, captures
  learnings, logs system improvements for manual review, and documents
  changes. Use ONLY when the user runs /wrap-up. Do NOT auto-invoke.
model: sonnet
---

## Determine Session Type

Run once at the start to decide how much of Phase 2 to run.

```
# Count non-doc files changed
If on a branch: git diff --name-only main -- app/ config/ db/ lib/ test/
If on main:     git diff --name-only HEAD~5 -- app/ config/ db/ lib/ test/
```

| Condition | Session type |
|-----------|-------------|
| No active plan in `docs/plans/` (skip `done/`) AND < 5 app files changed | **Light** — Phase 2 runs Remember + Changelog only |
| Everything else | **Full** — Phase 2 runs all sections |

---

## Phase 1: Ship It

Work through these gates in order. Do not skip ahead.

### 1A. Update Generated Artifacts

Check changed files (`git diff --name-only main` or `HEAD~5`):

| If changes touch… | Then… |
|---|---|
| `app/models/`, `db/migrate/`, `config/routes.rb`, or directory structure | Read and run `/update-diagrams` inline (skip its commit step) |
| `app/components/` | Read and run `/update-catalog` inline (skip its commit step) |
| Neither | Skip |

### 1B. Update Handoff File

Read and run `/catchup` inline (skip its commit step).

### 1C. Archive Completed Plans

If the plan in `docs/plans/` has all tasks complete:
- Move the plan to `docs/plans/done/`
- If an associated design exists in `docs/designs/`, move it to
  `docs/designs/done/`

### 1D. Preflight Checks

Run in order. Stop and report on any failure.

1. `bin/rails db:migrate:status` — flag any pending or "down" migrations
2. `bin/rails test` — full suite must pass. If tests fail, STOP.
3. `git status` — if nothing to commit, skip to 1E

### 1E. Commit & Push

1. Stage everything and commit following git-conventions rules
2. Push to remote
3. Do NOT auto-deploy

---

## Phase 2: Remember, Review, Improve

Before starting, read `.claude/session-learnings.md` if it exists. This
file has learnings from earlier in the session (captured before context
clears). Combine with observations from the current context window.
Process ALL learnings — from the file and from memory — through the
sections below.

### Remember

Route each learning through these questions in order — stop at first match:

| # | Question | Route to |
|---|----------|----------|
| 1 | Ephemeral? (local URLs, WIP, credentials) | `CLAUDE.local.md` |
| 2 | Permanent project decision? | Project `CLAUDE.md` |
| 3 | Debugging insight or quirk for this project? | `CLAUDE.local.md` |
| 4 | Duplicates existing content? | `@import` reference only |
| 5 | Everything else (skills, commands, rules, global config) | Append to `~/.claude/system-learnings.md` |

**Route 5 format** (create the file if it doesn't exist):

```
### [Short descriptive title]
**Date:** YYYY-MM-DD
**Source project:** [project name]
**Target:** [which global file should change]

[What to change and why. Specific enough to apply without session context.]
```

Never directly modify global files — only append to `system-learnings.md`.

### Review & Improve

> **Skip if session type is Light.**

Analyze the session for improvement opportunities. If the session was
routine with nothing notable, say "Nothing to improve" and move on.

Work through each category:

| Category | Ask yourself… |
|----------|--------------|
| Friction | Did the user have to ask for something that should've been automatic? |
| Skill gap | Did Claude need multiple attempts at something? |
| Mistakes | Did Claude make errors the user had to correct? |
| Knowledge | What project facts was Claude missing? |
| Dead ends | What approaches failed and should be avoided? |
| Automation | Were there repetitive patterns that could become skills/scripts? |
| Clone effectiveness | Were tasks scoped well? Did any clones go off-track? |

### Documentation

> **Skip if session type is Light.**

If a user-facing feature was completed, check if README needs updating.
Architecture diagrams and component catalog are the primary docs — only
update README for user-facing changes. If nothing warrants it, say
"Nothing to document" and skip.

### Changelog

For any changes to project `CLAUDE.md` or `CLAUDE.local.md`, append a
one-line entry to `~/.claude/CHANGELOG.md`:

```
YYYY-MM-DD | [file changed + version if applicable] | [what and why]
```

If entries exceed 20, delete the oldest until 20 remain.

### Approval & Commit Gate

1. Present all Phase 2 findings to the user:
   ```
   Findings:
   1. [title] — [brief description]
   ...

   Did I miss anything? Were there moments where you had to correct me,
   repeat yourself, or work around something I did wrong?
   ```
2. Wait for user feedback. Process any additions.
3. If any files changed in Phase 2, commit: `chore: session learnings`
4. Delete `.claude/session-learnings.md` ONLY after commit and push
   succeed. If either fails, keep the file for next wrap-up.
5. If system learnings were logged:
   "System learnings appended to ~/.claude/system-learnings.md —
   review and apply when ready."
6. "Wrap-up complete and changes committed."
