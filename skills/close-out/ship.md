# Close Out — Ship It

Loaded from `close-out/open-pr.md` 3B with the **CHANGED** file list
(`git diff --name-status main`) already computed. Verifies the branch is
safe to ship, regenerates derived docs, and commits. **Does not push** —
3E pushes once, after learnings, so everything lands in a single push.

## 1. Preflight

Runs first, before any doc regeneration — a pending migration or a red
suite should cost you nothing, not a round of wasted clone work.

Stop and report on any failure.

1. `bin/rails db:migrate:status` — flag any pending or "down" migrations.
2. `bin/rails test` — the full suite must be green. Run it every time,
   even if `/review-changes` just ran. You cannot actually check whether
   it ran, or whether code changed since, so any rule that says "skip it
   when…" is really asking you to guess — and guessing wrong here means
   pushing untested code. One suite run per ticket is cheap insurance.

## 2. Update Generated Artifacts

Dispatch the matching rows below as clones, based on CHANGED. If both
match, launch them in a SINGLE message so they run in parallel. Use
`model: sonnet` — this work is mechanical.

| If CHANGED touches… | Then… |
|---|---|
| `app/models/`, `db/migrate/`, `config/routes.rb`, or directory structure | Clone: run `/update-diagrams` |
| `app/components/` | Clone: run `/update-catalog` |
| Neither | Skip — launch nothing |

**Pass CHANGED in each clone's prompt** and tell it to use that list
instead of running its own `git diff`. You already paid for that diff;
three clones re-deriving it is three redundant calls.

Then, only if a component file is marked **A or D** in CHANGED (added or
deleted, not merely modified), launch a clone for
`/update-component-previews` once the `/update-catalog` clone returns. It
runs after, not in parallel — it reads the Quick Reference table
`/update-catalog` may have just changed. A modified-only change can't
orphan a preview or leave one missing, so skip it in that case.

These clones only touch docs and dev-only preview files, which is why
it's safe to run them after the test gate.

Clones stage their changes and do not commit.

## 3. Commit

Stage everything and commit following git-conventions rules. If there's
nothing to commit, say so and move on. Do NOT push, do NOT deploy.
