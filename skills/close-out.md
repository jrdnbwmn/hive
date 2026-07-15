---
name: close-out
version: 1.3 # bump on meaningful changes
description: >
  Single entry point to close out ticket/feature work after
  review-changes and wrap-up have run. Detects whether a PR exists yet
  and what state it's in, then runs the right flow: opens a PR (or
  merges/discards for non-ticket work) if none exists, reports status
  if one's open and unmerged, or syncs main + deletes the branch +
  archives docs if it's been merged. Use when the user runs /close-out,
  or says they're done with a branch/ticket and want to wrap it up — in
  any thread, cold or mid-session. Do NOT auto-invoke; this can push,
  open PRs, delete branches, and move files.
model: sonnet
---

# Close Out

One command for "the work is done, close everything off" — regardless
of whether a PR exists yet, is still open, or already merged. Assumes
`/review-changes` and `/wrap-up` have already run.

Never merges a PR itself — see Section 4. Reuses archive-docs for the
doc move rather than duplicating it.

## 1. Resolve What We're Closing

Priority order — first match wins:

1. **$ARGUMENTS given** → resolve it to a branch:
   - Ticket ID (e.g. `TIC-2`) → `gh pr list --search "<id>" --state all --json number,headRefName`
   - PR URL/number → `gh pr view <ref> --json headRefName`
   - Branch name → use directly
   - Zero results for a ticket ID → treat as "no PR yet"; if a local
     branch matching the naming convention exists (`git branch --list
     "*<id>*"`), use it — otherwise ask which branch this ticket is on.
   - 2+ results (ticket ID search only) → STOP. List them, ask which
     one is correct. Never guess.
2. **No argument, on a non-main branch** → use the current branch.
3. **No argument, on main** → ask: "Which ticket/branch/PR do you want
   to close out?" Do not proceed without one.

## 2. Check PR State

`gh pr view <branch> --json state,mergedAt,number,url` for the resolved branch.

| `gh pr view` result | Go to |
|---|---|
| Errors / no PR exists for this branch | Section 3 — No PR Yet |
| `state: OPEN` | Section 4 — Open, Unmerged |
| `state: MERGED` | Section 5 — Merged |
| `state: CLOSED`, `mergedAt: null` | Section 6 — Closed, Not Merged |

## 3. No PR Yet — Open One

### 3A. Choose How to Finish

Check whether the branch is ticket-derived: does it match
`(feature|fix)/<identifier>-...` where `<identifier>` looks like
`[a-z]+-\d+` (the Git Branch Naming Rules ticket format, e.g.
`feature/tic-123-add-account-settings`)?

If $ARGUMENTS specifies `merge`, `pr`, or `discard`, use it — an
explicit choice always wins, even on a ticket branch.

Otherwise:
- **Ticket-derived branch** → default to **pr** automatically, no
  prompt. Ticket work always goes through a PR.
- **Non-ticket branch** → present options:
  a) **merge** — merge to main locally, push, delete branch
  b) **pr** — push branch, create PR via `gh pr create`
  c) **discard** — confirm with me, then delete branch

**If the choice is discard, skip 3B and 3C** — go straight to the
discard steps in 3D. Tests and deploy-readiness don't matter for a
branch you're about to throw away.

### 3B. Preflight

Quick verification that wrap-up completed cleanly:

- No uncommitted changes (if any exist, ask: "There are uncommitted
  changes, run /wrap-up first?")
- Tests pass: `bin/rails test`
- No pending migrations: `bin/rails db:migrate:status`

If any of these fail, STOP and report.

### 3C. Production Readiness

Checks unique to the merge boundary, not covered by review-changes or wrap-up:

**Dependencies**
- Run `bundle audit` — flag any known gem vulnerabilities
- Verify `Gemfile.lock` is committed

**Platform: Render**
- If `bin/render-build.sh` exists, verify it includes any new build
  steps needed for this feature (new gems, asset compilation changes)
- If new environment variables were added: warn "Add these to your
  Render service's Environment settings before this deploy reaches
  production" and list them with expected values
- If migrations were added: verify render-build.sh includes `bin/rails db:migrate`

Present results:

```
## Deploy Readiness: [PASS / FAIL]

✅ Tests: [count] passed, 0 failures
✅ Migrations: None pending
✅ Dependencies: No vulnerabilities
✅ Render: Build script current
⚠️ Environment: New ENV var `STRIPE_WEBHOOK_SECRET` — add to Render

### Blocking Issues
[list, or "None"]

### Warnings (Non-Blocking)
[list, or "None"]
```

If blocking issues exist: STOP and report. Do NOT proceed.

### 3D. Execute

**If pr:**
- Read and run the archive-docs skill inline FIRST, using the current
  branch as the identifier — no ambiguity, we're on it. This bundles
  the archive commit into the PR itself, so it lands on main via the
  normal merge instead of needing special handling afterward (a
  worktree can't check out main to commit there directly — see Section
  5). If archive-docs finds nothing to archive, that's fine, continue.
- `git push origin <branch>`
- Open the PR. For ticket work, title format `[{identifier}] {issue
  title}` (see Working From Tickets in CLAUDE.md) and include a direct
  link to the Linear issue in the description. Otherwise `gh pr create --fill`.
- Stay on the branch.
- Say "Pushed and PR created. Once it's merged on GitHub, tell me or
  run `/close-out` again (any thread) — I'll check whether anything
  still needs archiving and finish up."

**If merge:**
- Check for a linked worktree first: `[ "$(git rev-parse --git-dir)" !=
  "$(git rev-parse --git-common-dir)" ]`. If true (e.g. a Conductor
  workspace), `main` is checked out in a different worktree (the
  project root) — `git checkout main` will fail here. STOP and say:
  "Can't merge locally from a worktree — main is checked out
  elsewhere. Use the pr path instead, or merge on GitHub." Do not
  attempt the steps below.
- `git checkout main`
- `git merge <branch>` (fast-forward if possible)
- If conflicts occur, STOP and report the conflicting files. Do NOT auto-resolve.
- `git push origin main`
- Delete the local branch: `git branch -d <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- Read and run the archive-docs skill inline, using `<branch>` as the
  identifier. Let it run its own commit.
- "Code merged and pushed to main. Render will auto-deploy. Check the
  Render dashboard to confirm the deploy succeeds."

**If discard:**
- Confirm with me first
- Check for a linked worktree first: `[ "$(git rev-parse --git-dir)" !=
  "$(git rev-parse --git-common-dir)" ]`. If true (e.g. a Conductor
  workspace), you can't check out `main` or delete the branch you're
  currently sitting on from inside this worktree. Delete the remote
  branch if it exists (`git push origin --delete <branch>`), then say
  "Remote branch deleted. Discard this workspace in Conductor to remove
  the worktree and local branch." Stop — don't attempt the steps below.
- `git checkout main`
- `git branch -D <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- Say "Branch discarded."

## 4. Open, Unmerged — Report and Wait

Report PR status: number, URL, checks (`gh pr checks <number>`), review
state. Then stop:

"PR #<n> is still open — merge it on GitHub, then tell me or run
`/close-out` again and I'll finish up."

**Never merge it yourself.** If, in this same conversation, the user
says they've merged it, don't take their word for it — re-run `gh pr
view` to confirm the state actually changed to `MERGED`, then continue
directly into Section 5. If it still shows open, say so.

## 5. Merged — Sync and Archive

Check for a linked worktree first: `[ "$(git rev-parse --git-dir)" !=
"$(git rev-parse --git-common-dir)" ]`.

**Not a worktree (traditional single checkout):**

1. `git checkout main && git pull`
2. Read and run the archive-docs skill inline, using `<branch>` as the
   identifier — skip its own identifier-resolution step, it's already
   known. Let it run its own commit. Since we're on main, this commits
   directly to main. If Section 3D already archived the docs before
   this PR was opened (the normal case now), archive-docs will likely
   find nothing left to do — that's expected, not an error.
3. Delete the local branch if it still exists: `git branch -d <branch>`
   (ignore the error if already gone)
4. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
5. Report: "Closed out [identifier]: main synced, branch removed, docs archived."

**Linked worktree (e.g. a Conductor workspace):**

`main` is checked out in a different worktree, and this worktree's own
branch can't be deleted while it's checked out here — skip both. No
need to sync a local `main` either; a fresh workspace always starts
from current `origin/main`.

1. Read and run the archive-docs skill inline, using `<branch>` as the
   identifier. Let it run its own commit. There's no way to be "on
   main" in this worktree, so this commits on the current branch.
2. **If archive-docs found nothing to archive** (expected — Section 3D
   should have archived it before this PR was opened): skip to step 5.
3. **If archive-docs did commit something** (fallback: docs weren't
   archived pre-merge, e.g. this PR was opened or merged some other
   way) — that commit needs to land on main without a PR, since it's
   pure doc housekeeping riding on an already-merged branch. Verify
   it's safe before doing anything:
   - `git merge-base --is-ancestor origin/main HEAD` must succeed (this
     branch has no divergence from origin/main)
   - `git rev-list --count origin/main..HEAD` should be a small number
     matching only the archive commit(s) just made
   If either check fails, STOP — there's more ahead of origin/main
   than expected. Report it and ask the user to resolve manually. Do
   NOT push.
4. If the checks pass, ask: "This branch is otherwise identical to
   origin/main (already merged) — OK to push the archive commit
   directly to main? (`git push origin HEAD:main`)" Wait for an
   explicit yes. On yes: `git push origin HEAD:main`.
5. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
6. Report: "Closed out [identifier]: docs archived[, pushed to main if
   step 4 ran]. Archive this workspace in Conductor to remove the
   worktree and local branch."

## 6. Closed, Not Merged — Ask

The PR was closed without merging — likely abandoned or superseded.
Report this and ask: "This PR was closed without merging. Delete the
branch, or leave it as-is?" Don't guess; don't touch anything until
they answer.
