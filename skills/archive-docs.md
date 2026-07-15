---
name: archive-docs
version: 1.0 # bump on meaningful changes
description: >
  Archive the design and plan docs for one merged ticket/branch by moving
  them to docs/designs/done/ and docs/plans/done/. Use when the user runs
  /archive-docs, when invoked inline from branch-finish after a
  successful merge, or when the user says a PR/ticket has merged and
  asks to clean up its docs. Do NOT guess by filename — match only on
  the Ticket/Branch header embedded in the docs. Do NOT auto-invoke for
  any other situation.
model: sonnet
---

# Archive Docs

Moves the design + plan doc pair for one merged ticket/branch into their
`done/` directories. Built to work cold, in a thread with no memory of
which docs belong to this ticket — matching is done by reading the
`Ticket:`/`Branch:` header brainstorm and write-plan embed in the docs
themselves, never by filename or session context.

## 1. Resolve the Identifier

Determine what to match on, in this order:

1. **Called inline** (e.g. from branch-finish right after a merge) — use
   the branch name already known in that session. Skip to step 2.
2. **User passed an argument** — classify it:
   - Looks like a ticket ID (e.g. `TIC-2`) → match on `Ticket:`
   - Looks like a branch name → match on `Branch:`
   - Looks like a PR URL or number → `gh pr view <ref> --json headRefName`
     and match on the returned branch name
3. **No argument, not called inline** — ask: "Which ticket ID, branch
   name, or PR do you want to archive docs for?" Do not proceed without one.

## 2. Find Matching Docs

Search only top-level files — never recurse into `done/`:

```
grep -l "^> Ticket: <id>$"     docs/designs/*.md docs/plans/*.md
grep -l "^> Branch: <branch>$" docs/designs/*.md docs/plans/*.md
```

Use whichever header type step 1 resolved (ticket ID takes priority over
branch if both are known — a branch name can be reused across tickets
over time, a ticket ID can't).

## 3. Handle Match Results

| Result | Action |
|---|---|
| Exactly one design + one plan match | Proceed to confirm |
| Only one of the two found | Proceed with what exists; note the other wasn't found |
| Zero matches | STOP. Report: "No docs tagged with [identifier]." They may predate tagging (no header) or already be archived — do not fall back to filename guessing. |
| 2+ matches for either type | STOP. List all matches with their headers shown. Ask the user which is correct. Never pick automatically. |

If a doc has no header at all (created before this tagging convention
existed), it will never match — say so explicitly rather than silently
skipping it, so the user can archive it manually if needed.

## 4. Confirm and Move

Always confirm before moving, even when the match is unambiguous —
cross-thread matches deserve a second look:

```
Found docs for [identifier]:
- docs/designs/<name>.md
- docs/plans/<name>.md

Archive both to done/?
```

On yes:

```
git mv docs/designs/<name>.md docs/designs/done/<name>.md
git mv docs/plans/<name>.md docs/plans/done/<name>.md
```

Commit: `chore: archive docs for [identifier]`.

**If called inline from another skill** (e.g. branch-finish): skip the
commit — the caller commits as part of its own flow.
