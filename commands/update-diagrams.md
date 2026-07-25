---
description: Generate or update Mermaid architecture diagrams (app structure, data model, routes)
model: sonnet
---

Create or update architecture diagrams in `docs/architecture/` following the instructions below.

If $ARGUMENTS is "full" or "regenerate":

- Skip the git diff check
- Regenerate all three diagrams from scratch, using the sources listed
  under Update Affected Diagrams below

If `docs/architecture/` doesn't exist yet, create it and generate all
three diagrams the same way.

## Detect Which Diagrams Need Updating

Check what changed. **If a caller passed you a changed-file list, use it
and skip this command** — don't re-derive what you were given:

```bash
git diff --name-status main
```

Determine which diagrams are affected:

- **app-structure.mermaid** — update if files/directories were added,
  deleted, or moved (new controllers, models, components, config files)
- **data-model.mermaid** — update if any files changed in
  app/models/ or db/migrate/
- **routes-map.mermaid** — update if config/routes.rb changed or
  files were added/deleted in app/controllers/

If no relevant files changed: say "Architecture diagrams are current —
nothing to update" and stop.

## Update Affected Diagrams Only

For each diagram that needs updating:

**app-structure.mermaid** (graph TD — directory structure & key files)

- Run a directory listing to see current structure
- Update the diagram to reflect additions/removals
- Do NOT read file contents — just names and locations

**data-model.mermaid** (erDiagram — models with associations & key fields)

- Read model files in app/models/ for associations and key fields.
  Skip validations — they don't belong in an ERD, and looking for them
  means reading the whole model body.
- Update the diagram to reflect new/changed/removed models
- Include associations between models

**routes-map.mermaid** (graph LR — routes → controllers → actions)

- Read config/routes.rb
- Update the diagram to reflect new/changed/removed routes

Each diagram should include enough detail that a new Claude session
can understand the app architecture WITHOUT reading source files.

If diagrams already exist, preserve any manual annotations or comments.
Update what changed — don't regenerate from scratch.

## Stage

Stage the changed files. Do NOT commit — /commit, review-changes, or
close-out handles that.

Say "Diagrams updated and staged."
