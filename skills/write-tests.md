---
name: write-tests
version: 1.0 # bump on meaningful changes
description: >
  TDD: RED-GREEN-REFACTOR. Write the failing test first, then the minimum
  code to pass, then refactor. Use for any code that changes behavior —
  features, bug fixes, model/controller changes. Also use when user says
  "add tests", "test this", or "write a test".
model: sonnet
---

# Write Tests

TDD discipline for any session writing or modifying code.

## Setup

Before writing anything:

1. Look at an existing test file **of the same type** you're about to
   write (model test → `test/models/`, system test → `test/system/`).
   Match the project's patterns, naming, and helpers.
2. If test infrastructure is broken (missing gems, DB not migrated),
   fix that first.

## The Cycle

RED-GREEN-REFACTOR is mandatory. Not suggested. Mandatory.

### RED — Write a failing test

Write a test that describes the desired **behavior**, not implementation. For bug fixes, this means reproducing the bug as a failing test.

```
bin/rails test test/models/whatever_test.rb
```

It MUST fail. If it passes, either the feature already exists or the
test is wrong — investigate before continuing.

### GREEN — Write minimum code to pass

Write the smallest amount of code that makes the test pass. No extras,
no "while I'm here" additions.

```
bin/rails test test/models/whatever_test.rb
```

It MUST pass.

### REFACTOR — Clean up, keep green

Simplify the code you just wrote. Run the specific test after each
change. If a test fails during refactor, fix or revert.

When refactor is complete, run the full suite:

```
bin/rails test
```

The full suite catches regressions your focused tests won't. Never
report a task complete without a green full suite.

## Where Tests Go

| What changed | Test type | Location |
|---|---|---|
| Model, validation, scope, association | Unit test | `test/models/` |
| Controller action, API endpoint | Integration test | `test/integration/` |
| User-facing feature or flow | System test | `test/system/` |
| Background job | Job test | `test/jobs/` |
| Mailer | Mailer test | `test/mailers/` |

Prefer system/integration tests for features, model tests for business
logic. When in doubt, test at the highest level that's still fast.

Hotwire behavior (Turbo Frames, Turbo Streams) must be covered by
system tests — they run in a real browser and verify Turbo Frame/Stream
behavior end-to-end. Integration tests can't exercise this correctly.

## Conventions

| Convention | Rule |
|---|---|
| Test framework | Minitest (not RSpec) |
| Test data | FactoryBot factories. No fixtures unless the project already uses them. |
| Mocking | Only for external APIs (Stripe, email, etc.). Mock the HTTP boundary, not internal classes. Never mock ActiveRecord or ActionController. Test behavior/outcomes, not internal implementation calls. |
| Stuck threshold | If tests fail and you can't figure out why after 2 attempts, stop and report. Don't guess. |

## Special Cases

### Existing code without tests

Write a **baseline test first** covering current behavior — it should
pass immediately. This is documentation, not TDD. Once baseline is
green, start the normal RED-GREEN-REFACTOR cycle for your change.

### Tasks from write-plan

If the plan includes a **Test** section, follow it exactly. The master
wrote it with full context. Don't freelance.
