---
name: wrap-up
version: 1.3
description: >
  End-of-session ritual: ship, remember, improve. Commits code, captures
  learnings, and logs system improvements for manual review.
disable-model-invocation: true
model: sonnet
---

## Phase 1: Ship It

Work through these gates in order. Do not skip ahead.

### 1A. Update Generated Artifacts

Check changed files:

```bash
git diff --name-only main
```

| If changes touch… | Then… |
|---|---|
| `app/models/`, `db/migrate/`, `config/routes.rb`, or directory structure | Read and run `/update-diagrams` inline (skip its commit step) |
| `app/components/` | Read and run `/update-catalog` inline (skip its commit step) |
| Neither | Skip |

### 1B. Preflight Checks

Run in order. Stop and report on any failure.

1. `bin/rails db:migrate:status` — flag any pending or "down" migrations
2. `git status` — if nothing to commit, skip to 1C

### 1C. Commit & Push

1. Stage everything and commit following git-conventions rules
2. Push to remote
3. Do NOT auto-deploy

---

## Phase 2: Remember, Review, Improve

Before starting, read `.claude/session-learnings.md` and `.codex/session-learnings.md` if they exist. These files have learnings from earlier in the session (captured before context clears). Combine with observations from the current context window.

Process all learnings from the file and current session through the sections below.

### Remember

Route each learning through these questions in order — stop at first match:

| # | Question | Route to |
|---|----------|----------|
| 1 | Ephemeral? (local URLs, WIP, credentials) | `CLAUDE.local.md` |
| 2 | Project decision, convention, or debugging insight? | `AGENTS.md` |
| 3 | Duplicates existing content? | `@import` reference only |
| 4 | Everything else (skills, commands, rules, global config) | Append to `~/.claude/system-learnings.md` |

Use `AGENTS.md` as the source of truth for project-specific learnings. Do not add new project-specific content to `CLAUDE.md`.

When writing to `CLAUDE.local.md`, warn if it is a plain file in a Conductor worktree and not a symlink, because that note will be lost when the workspace is archived.

For `~/.claude/system-learnings.md`, append only. It lives in the home directory and is not affected by worktree or symlink concerns.

**Route 4 format** (create the file if it doesn't exist):

```md
### [Short descriptive title]
**Date:** YYYY-MM-DD
**Source project:** [project name]
**Target:** [which global file should change]

[What to change and why. Specific enough to apply without session context.]
```

Never directly modify global files — only append to `system-learnings.md`.

### Review & Improve

Analyze the session only for notable improvement opportunities.

Only include an item if:
- The user had to correct or repeat something.
- The AI needed multiple attempts or made mistakes.
- A project convention or fact was missing and should be recorded.
- A repeated pattern should become a script, command, or skill.

Limit to at most 5 findings.

If nothing notable happened, say:

> Nothing to improve.

### Changelog

For any changes to project `AGENTS.md`, `CLAUDE.md`, or `CLAUDE.local.md`, append a one-line entry to `~/.claude/CHANGELOG.md`:

```md
YYYY-MM-DD | [file changed + version if applicable] | [what and why]
```

If entries exceed 20, delete the oldest until 20 remain.

### Commit

1. Present all Phase 2 findings to the user.
2. Run `git status`. If any files have changed, do a chore commit.
3. Say "Wrap-up complete and changes committed. Review `~/.claude/system-learnings.md` and apply learnings when ready."
