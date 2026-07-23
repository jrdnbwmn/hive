---
name: review-changes
version: 2.0
description: >
  Full branch review: security audit, Rails anti-patterns,
  cross-commit integration issues, and a holistic spec check against
  the overall plan. Assumes review-changes-mini already ran cleanly
  at each checkpoint during execution. Manually invoked via
  /review-changes only. Do NOT auto-invoke for any other situation.
disable-model-invocation: true
model: sonnet
---

# Review Changes

Full-branch pass. Assumes each checkpoint already passed `review-changes-mini` (spec compliance, tests, basic lint/debug cleanup).

## Phase 1: Gather Context

Diff `git diff main` (all branch changes).

For branches with 20+ files: focus on cross-file concerns (inconsistent
patterns, duplicate code, missing integration). Per-file quality was
already reviewed by mini at checkpoint time.

Find the full plan in `docs/plans/`. Scan commit history
(`git log --oneline main..HEAD`) for cross-checkpoint issues: conflicting
changes, duplicate definitions, orphaned code, merge artifacts — the kind
of issue mini couldn't see because it only ever looked at one
checkpoint's diff at a time.

## Phase 2: Holistic Spec Check

Mini already verified spec compliance per checkpoint. This phase only
checks for integration-level drift that emerges from combining
checkpoints:

- Does the finished branch, taken as a whole, still satisfy the
  original plan's "In scope" / "NOT in scope" boundaries?
- Did any checkpoint's change quietly break or undo another
  checkpoint's requirement?

**If gaps exist: STOP. Report as BLOCKING. Do NOT proceed to Phase 3.**

## Phase 3: Code Quality

Only after Phase 2 passes. Perform in order.

### A) Full Test Suite (Blocking)

- Run `bin/rails test` for the whole branch — review FAILS if any test
  fails. This catches integration failures between checkpoints that
  isolated per-checkpoint runs couldn't catch.

### B) Security (Blocking) — not covered by mini

- Hardcoded secrets, credentials, or API keys
- Missing strong parameters on new attributes
- Unauthenticated or unauthorized new routes
- SQL injection (raw `.where("...")` with interpolation)
- CSRF protection disabled without API justification
- XSS (unescaped output in views)
- Mass assignment vulnerabilities

### C) Rails Patterns (Blocking or Recommend) — not covered by mini

- N+1 queries (`.each` triggering queries → add `includes`) → blocking
- DRY violations across checkpoints (duplicate logic introduced by
  different checkpoints) → blocking
- Code smells → recommend, don't block:
  - Method > ~20 lines, class > ~200 lines
  - Model with 3+ callbacks
  - `if/elsif/elsif` chains
  - Raw SQL where a scope would work

### D) Auto-Fix (No Approval Needed)

- Fix missing indexes on foreign keys (not in mini's auto-fix list)
- Run `bundle exec rubocop -A` as a final safety net

## Handling Findings

Fix minor issues inline. For blocking issues that need substantial work,
delegate to a Task clone; fix everything else inline yourself.

**Stop and ask the user only for:** security vulnerabilities,
cross-checkpoint spec violations, data loss risk, architectural
concerns.

Present findings to the user:

\```
Review complete.

## Blocking Issues
1. [issue] — [why it's blocking] — [suggested fix]

## Fixed Automatically
1. [what was found] — [what was done]

## Recommendations (Non-Blocking)
1. [suggestion] — [why] — [up to you]
\```

After all issues have been fixed and recommendations addressed, commit
following git-conventions rules. Use the plan description as the commit
message basis. Then say "Review complete. Everything committed."

If no blocking issues or recommendations, commit following
git-conventions rules. Use the plan description as the commit message
basis. Then say "Review passed. Everything committed."
