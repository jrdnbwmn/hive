---
name: safe-migration
version: 1.0 # bump on meaningful changes
description: >
  Safe database migration guardrails. Use when creating a migration,
  adding/removing/renaming columns, adding indexes, changing tables,
  or when user says "add a field", "change the schema", "new table",
  or "add a model". Also use when running `rails generate migration`
  or `rails generate model`.
model: sonnet
---

# Safe Migration

Guardrails for database migrations. Migrations are hard to undo, and a
clone can't ask "is this safe?" mid-task. This skill provides automatic
safety checks.

## Before You Start

1. Run `bin/rails db:migrate:status` — don't create a migration if
   others are pending. Run or understand them first.
2. If the project uses the `strong_migrations` gem, follow its error
   messages — they tell you exactly what to do.

## Migration Naming

Use descriptive names that state the change:

```
# ✅ Clear intent
rails generate migration AddStatusToSubscriptions status:string
rails generate migration CreateNotificationPreferences

# ❌ Vague
rails generate migration UpdateSubscriptions
rails generate migration FixStuff
```

## Column Rules

| Rule | Why |
|------|-----|
| Add `null: false` on required columns | State intent explicitly — don't rely on implicit nil |
| Add `default:` where it makes sense | Only omit default when nil is a valid state |
| Add `foreign_key: true` on references | DB-level referential integrity |
| Add DB constraints, not just model validations | The database is the last line of defense |

```ruby
# ✅ Explicit constraints + default
add_column :subscriptions, :status, :string, null: false, default: "active"
add_reference :subscriptions, :user, null: false, foreign_key: true

# Then in the model:
validates :status, inclusion: { in: %w[active canceled past_due] }
```

## Indexes

- Add indexes on all foreign keys (`index: true` on references, or
  `add_index` separately).
- Add indexes on columns used in `where`, `order`, or `find_by`.

**Large tables only (10k+ rows, typically production):** Use concurrent
indexing to avoid table locks:

```ruby
disable_ddl_transaction!
def change
  add_index :large_table, :column_name, algorithm: :concurrently
end
```

For small/dev tables, normal `add_index` is fine and simpler.

## Destructive Operations

These rules protect against data loss. For solo dev apps with no
production data yet, the rename shortcut is acceptable — but build
the habit now.

| Operation | Safe approach |
|-----------|--------------|
| Remove column/table | Two deploys: first remove the code that uses it, then drop the column in a later migration |
| Rename column | Add new column → migrate data → update code → drop old column. (Dev-only: `rename_column` is acceptable.) |
| Raw SQL (`execute`) | Avoid unless absolutely necessary. If you must, add a comment explaining why. |

## Data Backfills

Schema changes and data changes have different failure modes. Keep them
in separate migrations so each is independently reversible.

```ruby
# Migration 1: schema change
class AddTierToPlans < ActiveRecord::Migration[8.0]
  def change
    add_column :plans, :tier, :string, null: false, default: "standard"
  end
end

# Migration 2: data backfill (separate file)
class BackfillPlanTiers < ActiveRecord::Migration[8.0]
  def up
    Plan.where(name: "Pro").update_all(tier: "premium")
  end

  def down
    Plan.where(tier: "premium").update_all(tier: "standard")
  end
end
```

## Verification Checklist

Run immediately after writing the migration — before tests or
implementation code that depends on the new schema.

```bash
bin/rails db:migrate            # 1. Must succeed
bin/rails db:rollback           # 2. Must succeed (if not, add a `down` method)
bin/rails db:migrate            # 3. Confirm clean re-apply
bin/rails test                  # 4. Migrations can break existing tests
```

Then confirm:
- `db/schema.rb` was updated — stage it alongside the migration
- Model validations match DB constraints (add them if in scope,
  note for review if not)

If a migration feels complex or risky, stop and report back rather
than guessing.
