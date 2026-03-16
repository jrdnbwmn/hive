---
paths:
  - "db/migrate/**/*.rb"
---

# Migrations

- Use `rails generate migration` to create migration files — don't
  create them manually.
- After writing: `db:migrate` → `db:rollback` → `db:migrate` to verify.
- Data backfills go in a separate migration or rake task, not inline.
