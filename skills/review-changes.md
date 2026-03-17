---
name: review-changes
version: 1.0 # bump on meaningful changes
description: >
  Review recent code changes for bugs, security issues, Rails
  anti-patterns, and spec compliance. Invoked by commands
  (/review-changes), by other skills (execute-plan, wrap-up), by
  task plans, or when the user explicitly asks to review changes.
  Do NOT auto-invoke for any other situation.
model: sonnet
---

# Review Changes

Two-stage review: determine scope, check the work matches the spec, then check
the code is good. Spec compliance comes before code check — don't proceed to
code quality until the spec passes.

Announce: "I'm using the review-changes skill to run a review."

## Phase 1: Determine Review Scope and Gather Context

| Invoked as | What to diff |
|---|---|
| `/review-changes branch` | `git diff main` (all branch changes) |
| `/review-changes` (no args) | `git diff` (uncommitted only). If clean: "Nothing to review — use `/review-changes branch` for the full branch." |
| From another skill or plan task | `git diff` (uncommitted). If clean: `git diff HEAD~1` |

For branch reviews with 20+ files: focus on cross-file concerns
(inconsistent patterns, duplicate code, missing integration). Per-file
quality was already reviewed during task execution.

**Gather context:** Find the original task or plan in `docs/plans/`.
If multiple commits exist on this branch (`git log --oneline main..HEAD`),
scan for cross-commit issues: conflicting changes, duplicate
definitions, orphaned code, merge artifacts.

## Phase 2: Spec Compliance

Read the task or plan, then check:

- Does every requirement have a corresponding implementation?
- Was anything built that WASN'T in the spec? (flag as scope creep)
- Do tests cover the specified behavior?
- Does implementation match "In scope" / "NOT in scope" boundaries?

**If spec gaps exist: STOP. Report as BLOCKING. Do NOT proceed to
Phase 3.**

## Phase 3: Code Quality

Only after Phase 2 passes. Perform in order.

### A) Tests (Blocking)

- For every new or modified model: does a corresponding test file exist?
- For every new route/controller action: does a system or integration test exist?
- Run `bin/rails test` — review FAILS if any test fails
- If test files are missing for new code, list them as BLOCKING issues.

### B) Security (Blocking)

Check for:

- Hardcoded secrets, credentials, or API keys
- Missing strong parameters on new attributes
- Unauthenticated or unauthorized new routes
- SQL injection (raw `.where("...")` with interpolation)
- CSRF protection disabled without API justification
- XSS (unescaped output in views)
- Mass assignment vulnerabilities

### C) Rails Patterns (Blocking or Recommend)

- N+1 queries (`.each` triggering queries — add `includes`) → blocking
- DRY violations (new code duplicating existing logic) → blocking
- Code smells → recommend, don't block:
  - Method > ~20 lines, class > ~200 lines
  - Model with 3+ callbacks
  - `if/elsif/elsif` chains
  - Raw SQL where a scope would work

### D) Auto-Fix (No Approval Needed)

Do these automatically:

- Run `bundle exec rubocop -A`
- Remove debug artifacts: `console.log`, `debugger`, `binding.pry`,
  `byebug`, `binding.irb`, `puts`/`pp`/`p` used for debugging
- Remove task-specific `TODO`/`FIXME` comments (preserve
  `TODO: replace with [ComponentName]` placeholders)
- Fix missing indexes on foreign keys
- Fix trailing whitespace, missing newlines

## Handling Findings

**If running as master (standalone or during execute-plan):**
delegate blocking issues that need substantial work
to a Task clone. Fix everything else inline.

**If running as a clone (during task execution):**
fix minor issues inline. For blocking issues, STOP
and report back — do not spawn sub-clones.

**STOP and ask the user only for:** security vulnerabilities, missing
tests, spec violations, data loss risk, architectural concerns.

Present findings to the user:

```
Review complete.

## Blocking Issues
1. [issue] — [why it's blocking] — [suggested fix]

## Fixed Automatically
1. [what was found] — [what was done]

## Recommendations (Non-Blocking)
1. [suggestion] — [why] — [up to you]
```

After all issues have been fixed and recommendations addressed, commit following git-conventions rules. Use the task or plan description as the commit message basis. Then say "Review complete. Everything committed."

If no blocking issues or recommendations, commit following git-conventions rules. Use the task or plan description as the commit message basis. Then say "Review passed. Everything committed."
