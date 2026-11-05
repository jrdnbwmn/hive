---
paths:
  - "test/**/*.rb"
  - "spec/**/*.rb"
---

# Tests

- Test framework: Minitest. Do NOT use RSpec syntax.
- Each test must be independent — no ordering dependencies.
- Test behavior, not implementation ("user can subscribe" not "calls Stripe API").
- Descriptive names: `test_user_cannot_access_other_users_data`.
- Test data: use the project's convention (check AGENTS.md/CLAUDE.md). If none is stated, match what the test suite already uses (fixtures or FactoryBot).
- Don't mock ActiveRecord. Use real database records (the project's fixtures or factories).
- `assert_difference` / `assert_no_difference` for count changes.
