---
description: Finish current feature branch — final checks, then merge, PR, or discard
allowed-tools: Bash(git *), Bash(bin/rails *), Bash(bundle *), Bash(gh *)
model: sonnet
argument-hint: [merge|pr|discard]
---

Finish the current feature branch. This assumes wrap-up has already run.

## 1. Preflight

Quick verification that wrap-up completed cleanly:

- No uncommitted changes (if any exist, ask: "There are uncommitted
  changes, run /wrap-up first?")
- Tests pass: `bin/rails test`
- No pending migrations: `bin/rails db:migrate:status`

If any of these fail, STOP and report.

## 2. Production Readiness (Branch-Finish-Only Checks)

These checks are unique to the merge boundary and not covered
by review-changes or wrap-up:

### Dependencies

- Run `bundle audit` — flag any known gem vulnerabilities
- Verify `Gemfile.lock` is committed

### Platform: Render

- If `bin/render-build.sh` exists, verify it includes any new build
  steps needed for this feature (new gems, asset compilation changes)
- If new environment variables were added: warn "Add these to your
  Render service's Environment settings before this deploy reaches
  production" and list them with expected values
- If migrations were added: verify render-build.sh includes
  `bin/rails db:migrate`

## 3. Report

Present results as:

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

## 4. Finish Branch

If $ARGUMENTS specifies an option, use it. Otherwise present options:
a) **merge** — merge to main locally, push, delete branch
b) **pr** — push branch, create PR via `gh pr create`
c) **discard** — confirm with me, then delete branch

## 5. Clean Up

**If merge:**

- `git checkout main`
- `git merge <feature-branch>` (fast-forward if possible)
- If merge conflicts occur, STOP and report the conflicting files.
  Do NOT auto-resolve — let the user decide.
- `git push origin main`
- Delete the local feature branch: `git branch -d <feature-branch>`
- Delete the remote feature branch if it exists:
  `git push origin --delete <feature-branch>`
- "Code merged and pushed to main. Render will auto-deploy. Check the Render
  dashboard to confirm the deploy succeeds."

**If pr:**

- `git push origin <feature-branch>`
- `gh pr create --fill` (or prompt me for title/description)
- Stay on the feature branch until PR is merged
- Say "Pushed and PR created. Still on branch."

**If discard:**

- Confirm with me first
- `git checkout main`
- `git branch -D <feature-branch>`
- Delete the remote branch if it exists:
  `git push origin --delete <feature-branch>`
- Say "Branch discarded."
