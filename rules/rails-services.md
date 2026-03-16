---
paths:
  - "app/services/**/*.rb"
---

# Service Objects

- One public method: `call`. Use keyword arguments for inputs.
- Return a result object or use `Result`/`Outcome` pattern — don't raise
  for expected failures.
- Name describes the action: `CreateSubscription`, `CancelOrder`,
  `SendWelcomeEmail`.
- Keep services focused on one operation. If it does 3 things, it's 3 services.
- Return the created/modified object on success. Use ActiveModel::Errors
  or raise for expected failures — don't return magic values.
