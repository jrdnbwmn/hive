# Close Out — Section 3: No PR Yet

Loaded from `~/.claude/skills/close-out/SKILL.md` when the resolved branch has
no PR. That file already resolved `<branch>` and whether this is a
linked worktree — use those answers, don't re-derive them.

This is the full finish-the-work path. Work through 3A–3E in order.

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

**If the choice is discard, skip 3B, 3C, and 3D** — go straight to 3E.
Don't regenerate docs, capture learnings, or run readiness checks for a
branch you're about to throw away.

## 3B. Ship It

Run once, and keep the output — 3D and the ship clones all reuse it
instead of re-running their own diffs:

```bash
git diff --name-status main
```

Call this list **CHANGED**. Use `--name-status`, not `--name-only`:
downstream steps need to tell an added file from a modified one, and a
bare path list can't.

Then read `~/.claude/skills/close-out/ship.md` and follow it, passing
CHANGED.

## 3C. Remember, Review, Improve

Read `~/.claude/skills/close-out/learnings.md` and follow it. Runs
before the PR opens so anything captured lands in the same push.

## 3D. Production Readiness

Merge-boundary checks not covered by review-changes or 3B. Both are
gated on CHANGED — skip a check outright when its trigger isn't there,
and say you skipped it.

- **`Gemfile.lock` in CHANGED** → run `bundle audit` and flag known gem
  vulnerabilities. Otherwise skip: dependencies didn't move, so the
  audit can only repeat what it said last time.
- **`db/migrate/`, `Gemfile.lock`, or `app/assets/` in CHANGED** → if
  `bin/render-build.sh` exists, verify it covers the new build steps
  (new gems, asset compilation) and includes `bin/rails db:migrate` when
  migrations were added.
- **New environment variables added** → warn "Add these to your Render
  service's Environment settings before this deploy reaches production"
  and list them with expected values.

Report `## Deploy Readiness: [PASS / FAIL]`, then blocking issues and
non-blocking warnings. Omit the per-check lines when everything passes —
only surface what needs attention.

If blocking issues exist: STOP and report. Do NOT proceed.

## 3E. Execute

**If merge or discard:** read
`~/.claude/skills/close-out/merge-or-discard.md` and follow it. Ignore
the rest of this section.

**If pr:**

- **Archive first**, before anything else, using the current branch as
  the identifier. This bundles the archive commit into the PR itself, so
  it lands on main via the normal merge instead of needing special
  handling afterward (a worktree can't check out main to commit there
  directly — see post-merge.md).
- `git push origin <branch>` — this is the single push for the whole
  run, carrying 3B's, 3C's, and the archive commits.
- Open the PR. For ticket work, title format `[{identifier}] {issue
  title}` (e.g. `[TIC-123] Add account settings form`) — Linear matches
  on the PR title too, not just the branch — and include a direct link to
  the Linear issue in the description. Otherwise `gh pr create --fill`.
- Stay on the branch.
- Say "Pushed and PR created. Once it's merged on GitHub, tell me or
  run `/close-out` again (any thread) — I'll check whether anything
  still needs archiving and finish up."
