# Close Out — Remember, Review, Improve

Loaded from `close-out/open-pr.md` 3C, after shipping and before the PR
opens, so anything captured lands in the same push.

Runs **only on this path** — never from `post-merge.md`. That run
happens in a fresh thread days later with no session context to mine.

First read `.claude/session-learnings.md` and `.codex/session-learnings.md`
if they exist (learnings captured earlier, before context cleared).
Combine with the current context window, then process everything below.

## Remember

Route each learning through these questions in order — stop at first match:

| # | Question | Route to |
|---|----------|----------|
| 1 | Ephemeral? (local URLs, WIP, credentials) | `CLAUDE.local.md` |
| 2 | Product behavior rule or decision (what users experience)? | `docs/product/product-brief.md` (or `ux-notes.md` for voice/UX) |
| 3 | Project decision, convention, or debugging insight? | `AGENTS.md` — the source of truth for project learnings. Never put these in `CLAUDE.md`. |
| 4 | Duplicates existing content? | `@import` reference only |
| 5 | Everything else (skills, commands, rules, global config) | Append to `~/.claude/system-learnings.md` |

Route 1: warn if `CLAUDE.local.md` is a plain file in a Conductor
worktree rather than a symlink — the note will be lost when the
workspace is archived.

Route 5: append-only, never modify global files directly. It lives in
the home directory, so worktree and symlink concerns don't apply. Create
the file if missing. One entry per learning:

```md
### [Title]
**Date:** YYYY-MM-DD | **Source project:** [name] | **Target:** [which global file should change]

[What to change and why. Specific enough to apply without session context.]
```

## Review & Improve

Surface at most 5 notable improvement opportunities. Include an item
only if: the user had to correct or repeat something; the AI needed
multiple attempts or made mistakes; a project convention or fact was
missing and should be recorded; or a repeated pattern should become a
script, command, or skill.

If nothing notable happened, say "Nothing to improve."

## Changelog

For any change to project `AGENTS.md`, `CLAUDE.md`, or `CLAUDE.local.md`,
append one line to `~/.claude/CHANGELOG.md` —
`YYYY-MM-DD | [file + version if applicable] | [what and why]` — then
trim the oldest entries until 20 remain.

## Commit

Present the findings above. Stage and commit any in-repo files that
changed (e.g. `AGENTS.md`) following git-conventions rules; files
outside the repo aren't committed. Do NOT push — 3E pushes everything
at once. Then tell the user: "Learnings captured. Review
`~/.claude/system-learnings.md` and apply them when ready."
