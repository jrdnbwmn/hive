---
name: fix-bug
version: 1.0 # bump on meaningful changes
description: >
  Structured debugging workflow. Use when fixing any bug — error messages,
  failing tests, unexpected behavior, crashes, or when user says "broken",
  "not working", "something's wrong", "there's a bug", "should do X but
  does Y", or "fix this".
model: sonnet
---

# Fix Bug

Structured debugging that prevents thrashing. Follow these steps in
order — do not skip ahead. A clone can't have a back-and-forth with
the user mid-task, so this structure replaces human judgment.

## Emergency Brake

**If 3 distinct approaches have failed: STOP immediately.**

An "approach" is a theory about the root cause that you tested with a
code change. Minor tweaks within the same theory count as one attempt.

Report back with:
- What you tried (each approach and what you observed)
- Where you think the real issue is
- What information would help narrow it down

Do NOT keep guessing. The architecture may be wrong, or you're
misunderstanding the problem.

---

## Step 1: Understand the Symptom

Before touching any code:

1. Read the error message, stack trace, or logs
2. Run the failing scenario yourself to observe it firsthand
3. State plainly: **what is happening** vs **what should be happening**

Where to look:

| Failure type | Check first |
|---|---|
| Server error / 500 | `log/development.log` — read the full stack trace |
| Failing test | Test output — read the assertion message and line number |
| UI / system test | Browser error console + test screenshot if available |
| Silent wrong behavior | Add `Rails.logger.debug` at boundaries to trace data flow |

If an active plan exists in `docs/plans/`, skim it for context on what
was recently built — bugs often hide in recent work.

**Do NOT guess. Do NOT start changing code. Understand first.**

## Step 2: Find the Root Cause

The symptom is rarely the cause. Explain *why* this is happening.

1. Read the code path from the error location outward
2. Check recent changes: `git log --oneline -10` and `git diff`
3. If unrelated to recent work, trace the relevant code path directly

### Trace Backward

If the cause isn't obvious, work backward from the symptom:

1. What function produced the wrong output?
2. What fed it the wrong input?
3. Keep tracing until you find the original wrong assumption

Add temporary debug logging at layer boundaries to narrow the source.
Remove all debug logging once you find the cause.

```ruby
# Temporary — remove after diagnosis
Rails.logger.debug "=== DEBUG: subscription.status=#{subscription.status} user=#{user.id}"
```

### Not a Bug?

If investigation reveals the behavior is actually correct:

- STOP the fix-bug workflow
- Report: "This isn't a bug — [explanation]. This looks like a
  missing feature or a misunderstanding."
- Do NOT modify working code to add new behavior

## Step 3: Write a Failing Test

Follow write-tests skill. The bug reproduction IS the RED step —
write a test that fails the same way the bug manifests. Run it and
confirm it fails for the right reason.

```bash
bin/rails test test/models/whatever_test.rb
```

If you can't reproduce it in a test, describe why and proceed
carefully.

## Step 4: Minimal Fix

Fix the root cause — not the symptom.

- Change as few lines and files as possible
- Don't refactor, reorganize, or "improve" other code
- Don't add features, even if they seem related
- Don't add nil checks or rescues that hide the real problem

Run the failing test — it MUST now pass.

## Step 5: Verify No Regressions

```bash
bin/rails test
```

All tests must pass, not just yours. If other tests broke, your fix
has side effects — investigate the side effects, don't just patch
those tests to pass.

## Handoff

Report back with:
- **Root cause:** what the bug was and why it happened
- **Fix:** what you changed (files and lines)
- **Tests:** confirmation that full suite passes

Stage your changes. Do NOT commit — review-changes will handle the commit after reviewing your work.
