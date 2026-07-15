---
name: close-out
version: 1.1 # bump on meaningful changes
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
- `git push origin <branch>`
- Open the PR. For ticket work, title format `[{identifier}] {issue
  title}` (see Working From Tickets in CLAUDE.md) and include a direct
  link to the Linear issue in the description. Otherwise `gh pr create --fill`.
- Stay on the branch.
- Say "Pushed and PR created. Once it's merged on GitHub, tell me or
  run `/close-out` again (any thread) and I'll sync up and archive the docs."

**If merge:**
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

1. `git checkout main && git pull`
2. Delete the local branch if it still exists: `git branch -d <branch>`
   (ignore the error if already gone)
3. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
4. Read and run the archive-docs skill inline, using `<branch>` as the
   identifier — skip its own identifier-resolution step, it's already
   known. Let it run its own commit.
5. Report: "Closed out [identifier]: main synced, branch removed, docs archived."

## 6. Closed, Not Merged — Ask

The PR was closed without merging — likely abandoned or superseded.
Report this and ask: "This PR was closed without merging. Delete the
branch, or leave it as-is?" Don't guess; don't touch anything until
they answer.
