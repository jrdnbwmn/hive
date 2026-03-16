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
- Use factories (FactoryBot), not fixtures.
- Don't mock ActiveRecord. Use factories with real database records.
- `assert_difference` / `assert_no_difference` for count changes.
