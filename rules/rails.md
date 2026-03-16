---
paths:
  - "**/*.rb"
---

# Rails Conventions

- When in doubt, write boring, obvious Rails code. Default to the simplest
  Rails-conventional approach. No metaprogramming, no clever abstractions,
  no custom DSLs.
- Prefer Rails built-ins over adding gems, unless the gem provides unique,
  important functionality that is otherwise hard to replicate.
- Prefer `rails generate` over manually creating files when it creates
  needed boilerplate (model, migration, test, factory).
- `Time.current` not `Time.now`. `Date.current` not `Date.today`.
- `find_by` not `where(...).first`. `exists?` not `count > 0`.
- Prefer `where.missing(:association)` over LEFT JOIN for finding orphans.
- This app uses Jumpstart Pro. Check for existing JSP functionality
  before building — especially auth, billing, notifications, and admin.
