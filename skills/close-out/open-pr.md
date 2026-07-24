# Close Out — Section 3: No PR Yet

Loaded from `~/.claude/skills/close-out.md` when the resolved branch has
no PR. That file already resolved `<branch>` and whether this is a
linked worktree — use those answers, don't re-derive them.

## 3A. Choose How to Finish

If $ARGUMENTS specifies `merge`, `pr`, or `discard`, use it — an
explicit choice always wins, even on a ticket branch.

Otherwise, check whether the branch is ticket-derived: does it carry a
ticket identifier per the Git Branch Naming Rules in CLAUDE.md?

- **Ticket-derived branch** → default to **pr** automatically, no
  prompt. Ticket work always goes through a PR.
- **Non-ticket branch** → present options:
  a) **merge** — merge to main locally, push, delete branch
  b) **pr** — push branch, create PR via `gh pr create`
  c) **discard** — confirm with me, then delete branch

**If the choice is discard, skip 3B and 3C** — go straight to the
discard steps in 3D. Tests and deploy-readiness don't matter for a
branch you're about to throw away.

## 3B. Preflight

Confirm wrap-up completed cleanly. The primary gate is the working tree:

- No uncommitted changes (if any exist, ask: "There are uncommitted
  changes, run /wrap-up first?" and STOP). A clean tree means wrap-up
  committed and its gates — full `bin/rails test` and `db:migrate:status`
  — already passed on this exact code, so don't re-run them here.
- **Exception — running cold** (fresh thread, or you can't confirm
  wrap-up ran this session): run `bin/rails test` and
  `bin/rails db:migrate:status` yourself, since nothing upstream did.
  STOP and report on any failure.

## 3C. Production Readiness

Checks unique to the merge boundary, not covered by review-changes or wrap-up:

- Run `bundle audit` — flag any known gem vulnerabilities
- **Render:** if `bin/render-build.sh` exists, verify it includes any new
  build steps this feature needs (new gems, asset compilation changes),
  and `bin/rails db:migrate` if migrations were added
- If new environment variables were added: warn "Add these to your
  Render service's Environment settings before this deploy reaches
  production" and list them with expected values

Report `## Deploy Readiness: [PASS / FAIL]`, then blocking issues and
non-blocking warnings. Omit the per-check lines when everything passes —
only surface what needs attention.

If blocking issues exist: STOP and report. Do NOT proceed.

## 3D. Execute

**If pr:**

- **Archive first**, before anything else, using the current branch as
  the identifier. This bundles the archive commit into the PR itself, so
  it lands on main via the normal merge instead of needing special
  handling afterward (a worktree can't check out main to commit there
  directly — see post-merge.md).
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

- **If this is a linked worktree** (per Section 1): STOP. `main` is
  checked out in the project root, so `git checkout main` will fail
  here. Say: "Can't merge locally from a worktree — main is checked out
  elsewhere. Use the pr path instead, or merge on GitHub." Do not
  attempt the steps below.
- `git checkout main`
- `git merge <branch>` (fast-forward if possible)
- If conflicts occur, STOP and report the conflicting files. Do NOT auto-resolve.
- `git push origin main`
- Delete the local branch: `git branch -d <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- **Archive**, using `<branch>` as the identifier.
- "Code merged and pushed to main. Render will auto-deploy. Check the
  Render dashboard to confirm the deploy succeeds."

**If discard:**

- Confirm with me first.
- **If this is a linked worktree** (per Section 1): you can't check out
  `main` or delete the branch you're currently sitting on. Delete the
  remote branch if it exists (`git push origin --delete <branch>`), then
  say "Remote branch deleted. Discard this workspace in Conductor to
  remove the worktree and local branch." Stop — don't attempt the steps
  below.
- `git checkout main`
- `git branch -D <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- Say "Branch discarded."
