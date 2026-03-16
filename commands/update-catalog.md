---
description: Generate or update the component catalog and component map diagram
model: sonnet
---

Update the component library documentation based on what has changed.

## Step 0: Detect What Changed

If docs/COMPONENT_CATALOG.md does not exist (first run):

- Read ALL ViewComponents in app/components/ (.rb and .html.erb files)
- Generate the full catalog using the template format in the file
  (Quick Reference table + Component Details sections)
- Then generate docs/architecture/component-map.mermaid from the
  catalog (see Step 2)
- Skip to Step 3

If the catalog already exists, detect changes:
If on a branch: git diff --name-only main -- app/components/
If on main: git diff --name-only HEAD~5 -- app/components/

Build three lists from the results:

- **Added:** new component files in app/components/
- **Modified:** changed component files in app/components/
- **Deleted:** removed component files from app/components/

If no component files changed: say "Component catalog is current —
nothing to update" and stop.

## Step 1: Update the Catalog

Read ONLY the changed component files (from Step 0). For each:

**Added components:** Read the .rb and .html.erb files. Add a new entry to Component Details section of docs/COMPONENT_CATALOG.md using the template format. Include all sections that apply. Omit any section that isn't relevant. Also add a row to the Quick Reference table.

**Modified components:** Read the updated files. When updating an existing entry, search for the ### heading matching
the component name. Read only that section, do not read the full catalog. Update ONLY the changed
fields in the existing catalog entry (e.g., if arguments changed, update
the arguments table). Preserve any manually-added notes.

**Deleted components:** Remove the entry from the Component Details
section and the Quick Reference table.

## Step 2: Update the Component Map

If components were added or deleted, update
docs/architecture/component-map.mermaid:

- Derive the component list and categories from the catalog
  (do not re-read app/components/)
- Add new components to the appropriate category group
- Remove deleted components
- Update composition relationships if templates showed render
  calls to other components (from what you read in Step 1)
- Structure only — no arguments, no usage details
- Do NOT regenerate the entire diagram
- If the file doesn't exist, create it

If only modifications occurred with no category or composition
changes, skip this step.

## Step 3: Commit

Commit all changed files: "docs: update component catalog and map"

Say "Catalog and mermaid diagram updated."

When called from wrap-up, skip the commit — wrap-up handles it.`
