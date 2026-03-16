---
paths:
  - "app/models/**/*.rb"
---

# Models

- Scopes: simple, chainable, in the model. Complex queries → query objects.
- Callbacks: only simple side effects (e.g., setting defaults). Complex
  logic → service object.
- `dependent:` option on every `has_many` and `has_one`. Pick the right
  strategy (destroy, nullify, restrict_with_error).
- `has_many :through` over `has_and_belongs_to_many`.
- Foreign keys: always `null: false` with `index: true` in the migration.
