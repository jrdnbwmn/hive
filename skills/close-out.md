---
name: close-out
version: 1.7 # bump on meaningful changes
description: >
  Single entry point to close out ticket/feature work after
  review-changes and wrap-up have run. Detects whether a PR exists yet
  and what state it's in, then runs the right flow: opens a PR (or
  merges/discards for non-ticket work) if none exists, reports status
  if one's open and unmerged, or syncs main + deletes the branch +
  archives docs if it's been merged.
disable-model-invocation: true
model: sonnet
---

# Close Out

One command for "the work is done, close everything off" — regardless
of whether a PR exists yet, is still open, or already merged. Assumes
`/review-changes` and `/wrap-up` have already run.

Never merges a PR itself — see Section 4.

**Archiving:** wherever this skill or its section files say *archive*,
read and run `~/.claude/skills/archive-docs.md` inline, passing
`<branch>` as the identifier so it skips its own resolution step. Let it
make its own commit — every caller depends on that commit existing as a
standalone. If it finds nothing to archive, that's fine; continue.

## 1. Resolve Branch and Environment

`<branch>` is always the current branch — close-out is run from the
branch being closed. If you find yourself on `main`, stop and ask which
branch to close out rather than guessing; the delete steps below would
otherwise target `main` itself.

Then determine this once, here, and carry the answer through every
section below — do not re-check it later:

**Linked worktree?** `[ "$(git rev-parse --git-dir)" != "$(git
rev-parse --git-common-dir)" ]`. True means this is a linked worktree
(e.g. a Conductor workspace): `main` is checked out in the project root,
so `git checkout main` will fail here and the current branch can't be
deleted from inside it. Several paths below branch on this.

## 2. Check PR State

`gh pr view <branch> --json state,mergedAt,number,url` for the resolved branch.

| `gh pr view` result | Go to |
|---|---|
| Errors / no PR exists for this branch | Read `~/.claude/skills/close-out/open-pr.md` and follow it |
| `state: OPEN` | Section 4 below |
| `state: MERGED` | Read `~/.claude/skills/close-out/post-merge.md` and follow it |
| `state: CLOSED`, `mergedAt: null` | Section 6 below |

Read only the file this table sends you to — the two section files are
mutually exclusive.

## 4. Open, Unmerged — Report and Wait

Report PR status: number, URL, checks (`gh pr checks <number>`), review
state. Then stop:

"PR #<n> is still open — merge it on GitHub, then tell me or run
`/close-out` again and I'll finish up."

**Never merge it yourself.** If, in this same conversation, the user
says they've merged it, don't take their word for it — re-run `gh pr
view` to confirm the state actually changed to `MERGED`, then continue
into `close-out/post-merge.md`. If it still shows open, say so.

## 6. Closed, Not Merged — Ask

The PR was closed without merging — likely abandoned or superseded.
Report this and ask: "This PR was closed without merging. Delete the
branch, or leave it as-is?" Don't guess; don't touch anything until
they answer.
