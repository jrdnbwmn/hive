---
name: archive-docs
version: 1.1 # bump on meaningful changes
description: >
  Archive the design and plan docs for one merged ticket/branch by moving
  them to docs/designs/done/ and docs/plans/done/. Use when the user runs
  /archive-docs, when invoked inline from branch-finish after a
  successful merge, or when the user says a PR/ticket has merged and
  asks to clean up its docs. Match only on the Ticket/Branch header
  embedded in the docs — never guess by filename. Do NOT auto-invoke for
  any other situation.
model: sonnet
---

# Archive Docs

Moves one ticket's design + plan doc pair to `done/`. Matches only on
the `Ticket:`/`Branch:` header brainstorm and write-plan embed in each
doc — never by filename or session memory — so this works cold, in any
thread.

## 1. Resolve the Identifier

| Case | Resolution |
|---|---|
| Called inline (e.g. from branch-finish) | Use the branch name already known in that session |
| User passed a ticket ID (e.g. `TIC-2`) | Match on `Ticket:` |
| User passed a branch name | Match on `Branch:` |
| User passed a PR URL/number | `gh pr view <ref> --json headRefName` → match on that branch |
| Nothing passed, not inline | Ask which ticket/branch/PR to archive. Don't proceed without one. |

Ticket ID takes priority over branch if both are known — branch names get reused, ticket IDs don't.

## 2. Find Matching Docs

Top-level files only — never recurse into `done/`:

```
grep -l "^> Ticket: <id>$"     docs/designs/*.md docs/plans/*.md
grep -l "^> Branch: <branch>$" docs/designs/*.md docs/plans/*.md
```

## 3. Handle Match Results

| Result | Action |
|---|---|
| One design + one plan match | Proceed to confirm |
| Only one of the two found | Proceed with what exists; note the other's missing |
| Zero matches | STOP — report no docs tagged with [identifier]. May predate tagging or already be archived. Don't fall back to filename guessing. |
| 2+ matches for either type | STOP — list them with headers shown, ask which is correct. Never pick automatically. |

## 4. Confirm and Move

Always confirm first, even for an unambiguous match:

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

Commit: `chore: archive docs for [identifier]`. Skip the commit if
called inline — the caller commits.
