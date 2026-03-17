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

**System tests only:** Verify Capybara is configured before writing.
Run an existing system test first if unsure:
`bin/rails test test/system/ -n test_something_basic`

## The Cycle

RED-GREEN-REFACTOR is mandatory. Not suggested. Mandatory.

### RED — Write a failing test

Write a test that describes the desired **behavior**, not implementation.

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
change. If a test fails during refactor, undo immediately.

When refactor is complete, run the full suite:

```
bin/rails test
```

The full suite catches regressions your focused tests won't. Never
report a task complete without a green full suite.

## Test Naming

Test names describe the behavior from the user's perspective, not the
method being called:

```ruby
# ❌ Vague or implementation-focused
test "subscription create" do ...
test "it works" do ...

# ✅ Behavior-focused
test "user can subscribe to a plan" do ...
test "expired subscriptions are excluded from active scope" do ...
```

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

## Behavior Over Implementation

```ruby
# ❌ Tests implementation — brittle, breaks on refactor
test "creating subscription calls Stripe API" do
  mock = Minitest::Mock.new
  mock.expect :create, true, [Hash]
  Stripe::Subscription.stub :create, mock do
    Subscription.create!(user: @user, plan: @plan)
  end
end

# ✅ Tests behavior — stable, describes what the user gets
test "user can subscribe to a plan" do
  assert_difference "Subscription.count", 1 do
    post subscriptions_path, params: { plan_id: @plan.id }
  end
  assert @user.reload.subscribed?
end
```

## Conventions

| Convention | Rule |
|---|---|
| Test framework | Minitest (not RSpec) |
| Test data | FactoryBot factories. No fixtures unless the project already uses them. |
| Mocking | Only for external APIs (Stripe, email, etc.). Mock the HTTP boundary, not internal classes. Never mock ActiveRecord or ActionController. |
| Stuck threshold | If tests fail and you can't figure out why after 2 attempts, stop and report. Don't guess. |

## Special Cases

### Existing code without tests

Write a **baseline test first** covering current behavior — it should
pass immediately. This is documentation, not TDD. Once baseline is
green, start the normal RED-GREEN-REFACTOR cycle for your change.

### Bug fixes

The failing test IS the RED step. Reproduce the bug as a test that
fails, then fix it (GREEN). Natural TDD — don't skip it.

### Tasks from write-plan

If the plan includes a **Test** section, follow it exactly. The master
wrote it with full context. Don't freelance.
