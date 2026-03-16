---
description: Generate or update Mermaid architecture diagrams (app structure, data model, routes)
model: sonnet
---

Create or update architecture diagrams in `docs/architecture/` following the instructions below. Exception: Do NOT touch `docs/architecture/component-map.mermaid` — that file is managed by /update-catalog.

If $ARGUMENTS is "full" or "regenerate":

- Skip the git diff check
- Regenerate all three diagrams from scratch (see Full Generation below)

If `docs/architecture/` doesn't exist yet, create it and generate all
three diagrams from scratch (skip to Full Generation below).

## Detect Which Diagrams Need Updating

Check what changed:
If on a branch: git diff --name-only main
If on main: git diff --name-only HEAD~5

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

- Read model files in app/models/ for associations, validations,
  and key fields
- Update the diagram to reflect new/changed/removed models
- Include associations between models

**routes-map.mermaid** (graph LR — routes → controllers → actions)

- Read config/routes.rb
- Update the diagram to reflect new/changed/removed routes

Each diagram should include enough detail that a new Claude session
can understand the app architecture WITHOUT reading source files.

If diagrams already exist, preserve any manual annotations or comments.
Update what changed — don't regenerate from scratch.

## Full Generation

If this is the first run or diagrams don't exist yet, generate all
three diagrams by reading the relevant parts of the codebase:

- Directory structure for app-structure
- All model files for data-model
- routes.rb and controller directory for routes-map

## Commit

Commit changed files: "docs: update architecture diagrams"

Say "Diagrams updated."

When called from wrap-up, skip the commit — wrap-up handles it.
