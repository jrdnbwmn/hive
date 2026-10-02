---
name: close-out
version: 2.1 # bump on meaningful changes
description: >
  Single entry point to close out ticket/feature work after
  review-changes has run. Detects whether a PR exists yet and what state
  it's in, then runs the right flow: regenerates derived docs, commits,
  captures learnings and opens a PR (or merges/discards for non-ticket
  work) if none exists, reports status if one's open and unmerged, or
  syncs main + deletes the branch + archives docs if it's been merged. Manually invoked via /close-out only. Do NOT auto-invoke.
disable-model-invocation: true
model: sonnet
argument-hint: "<optional: merge|pr|discard — omit to use the default for this branch>"
---

# Close Out

One command for "the work is done, close everything off" — regardless
of whether a PR exists yet, is still open, or already merged. Assumes
`/review-changes` has already run; this skill verifies rather than
assumes what that gate covered.

On the no-PR-yet path this also regenerates derived docs, commits, and
captures session learnings — see `close-out/ship.md` and
`close-out/learnings.md`, both loaded from `open-pr.md`.

Never merges a PR itself — see Section 4.

**Archiving:** wherever this skill or its section files say *archive*,
read and run `~/.claude/skills/archive-docs/SKILL.md` inline, passing
`<branch>` as the identifier so it skips its own resolution step. Let it
make its own commit — every caller depends on that commit existing as a
standalone. If it finds nothing to archive, that's fine; continue.

## 1. Resolve Branch and Environment

One command, once — every section below reuses these answers rather than
re-checking:

```bash
git branch --show-current && git rev-parse --git-dir --git-common-dir
```

**`<branch>`** is the current branch; close-out is run from the branch
being closed. If it's `main`, stop and ask which branch to close out
rather than guessing — the delete steps below would otherwise target
`main` itself.

**Finish choice passed to /close-out:** $ARGUMENTS

If that's `merge`, `pr`, or `discard`, it's an explicit choice for the
no-PR-yet case — carry it into `open-pr.md` §3A. Empty means none.

**Linked worktree?** True when the two `rev-parse` paths differ (e.g. a
Conductor workspace): `main` is checked out in the project root, so
`git checkout main` will fail here and the current branch can't be
deleted from inside it. Several paths below branch on this.

## 2. Check PR State

`gh pr view <branch> --json state,mergedAt,number,url` for the resolved branch.

| `gh pr view` result | Go to |
|---|---|
| Errors / no PR exists for this branch | Read `~/.claude/skills/close-out/open-pr.md` and follow it |
| `state: OPEN` | Section 4 below |
| `state: MERGED` | Read `~/.claude/skills/close-out/post-merge.md` and follow it |
| `state: CLOSED`, `mergedAt: null` | Section 6 below |

Read only the file this table sends you to. `open-pr.md` and
`post-merge.md` are mutually exclusive; `open-pr.md` pulls in `ship.md`,
`learnings.md`, and `merge-or-discard.md` itself, only when it needs
each one.

## 4. Open, Unmerged — Report and Wait

Report number, URL, checks (`gh pr checks <number>`), review state, then
stop: "PR #<n> is still open — merge it on GitHub, then tell me or run
`/close-out` again and I'll finish up."

**Never merge it yourself.** If the user says they've merged it, don't
take their word for it — re-run `gh pr view` to confirm the state
actually changed to `MERGED`, then continue into `post-merge.md`. If it
still shows open, say so.

## 6. Closed, Not Merged — Ask

Closed without merging — likely abandoned or superseded. Report it and
ask: "This PR was closed without merging. Delete the branch, or leave it
as-is?" Don't guess; don't touch anything until they answer.
