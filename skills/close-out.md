---
name: close-out
version: 1.5 # bump on meaningful changes
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

Confirm wrap-up completed cleanly. The primary gate is the working tree:

- No uncommitted changes (if any exist, ask: "There are uncommitted
  changes, run /wrap-up first?" and STOP). A clean tree means wrap-up
  committed and its gates — full `bin/rails test` and `db:migrate:status`
  — already passed on this exact code, so don't re-run them here.
- **Exception — running cold** (fresh thread, or you can't confirm
  wrap-up ran this session): run `bin/rails test` and
  `bin/rails db:migrate:status` yourself, since nothing upstream did.
  STOP and report on any failure.

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
  branch as the identifier — no ambiguity, we're on it. Let it run its
  own commit. This bundles the archive commit into the PR itself, so it
  lands on main via the normal merge instead of needing special
  handling afterward (a worktree can't check out main to commit there
  directly — see Section 5). If archive-docs finds nothing to archive,
  that's fine, continue.
- `git push origin <branch>`
- Open the PR. For ticket work, title format `[{identifier}] {issue
  title}` (e.g. `[TIC-123] Add account settings form`) — Linear matches
  on the PR title too, not just the branch — and include a direct link to
  the Linear issue in the description. Otherwise `gh pr create --fill`.
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

1. Check for a linked worktree: `[ "$(git rev-parse --git-dir)" !=
   "$(git rev-parse --git-common-dir)" ]`. True → go to **Linked
   worktree** below. False → continue below. Check this before running
   anything else in this section — do not jump straight to `git
   checkout main`.

**Not a worktree (traditional single checkout):**

2. `git checkout main && git pull`
3. Read and run the archive-docs skill inline, using `<branch>` as the
   identifier — skip its own identifier-resolution step, it's already
   known. Let it run its own commit. Since we're on main, this commits
   directly to main. If Section 3D already archived the docs before
   this PR was opened (the normal case now), archive-docs will likely
   find nothing left to do — that's expected, not an error.
4. Delete the local branch if it still exists: `git branch -d <branch>`
   (ignore the error if already gone)
5. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
6. Report: "Closed out [identifier]: main synced, branch removed, docs archived."

**Linked worktree (e.g. a Conductor workspace):**

`main` is checked out in a different worktree, and this worktree's own
branch can't be deleted while it's checked out here — skip both. No
need to sync a local `main` either; a fresh workspace always starts
from current `origin/main`. Do NOT run `git checkout main` in this
worktree — it will fail, `main` is checked out elsewhere.

2. `git fetch origin`, then `git checkout -b <branch>-archive
   origin/main` — a temp branch based on current main, not on
   `<branch>` itself. This matters if the PR was squash-merged (check:
   `git log -1 --oneline origin/main` — a squash-merge commit title
   looks like `<title> (#<pr-number>)`). After a squash merge,
   `<branch>`'s own history never contains that squash commit, so
   committing directly on `<branch>` and then checking its ancestry
   against `origin/main` would fail even though pushing is perfectly
   safe. Branching fresh from `origin/main` sidesteps that — the temp
   branch is `origin/main` plus one commit, by construction, so there's
   nothing left to verify.
3. Read and run the archive-docs skill inline, using `<branch>` as the
   identifier, on this temp branch. Let it run its own commit.
4. **If archive-docs found nothing to archive** (expected — Section 3D
   should have archived it before this PR was opened): `git checkout
   <branch> && git branch -D <branch>-archive`, then skip to step 7.
5. **If archive-docs did commit something** (fallback: docs weren't
   archived pre-merge, e.g. this PR was opened or merged some other
   way) — that commit needs to land on `main`. This is a real push to a
   shared branch: always ask, every time, regardless of what a past
   archive commit did.

   **STOP. Ask exactly this, and wait for an explicit yes:** "Ready to
   push a doc-archive commit directly to main (`git push origin
   <branch>-archive:main`). OK to push?" Do not proceed on silence, and
   do not treat a prior similar push (e.g. an earlier ticket's archive
   commit) as standing permission for this one.

   On yes only: `git push origin <branch>-archive:main`.
6. `git checkout <branch> && git branch -D <branch>-archive`
7. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
8. Report: "Closed out [identifier]: docs archived[, pushed to main if
   step 5 ran]. Archive this workspace in Conductor to remove the
   worktree and local branch."

## 6. Closed, Not Merged — Ask

The PR was closed without merging — likely abandoned or superseded.
Report this and ask: "This PR was closed without merging. Delete the
branch, or leave it as-is?" Don't guess; don't touch anything until
they answer.
