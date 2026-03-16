---
description: Update Lookbook previews and kitchen sink page to match the component catalog
model: sonnet
---

Sync Lookbook previews and the kitchen sink page with the current
component catalog (docs/COMPONENT_CATALOG.md).

## Step 0: Detect What's Out of Sync

Read the Quick Reference table in docs/COMPONENT_CATALOG.md to get the
list of all current components.

Compare against what exists:

- List preview files in test/components/previews/
- If app/views/dev/kitchen_sink/show.html.erb exists, scan it for
  which components are rendered

Build three lists:

- **Missing previews:** components in the catalog with no preview file
- **Missing from kitchen sink:** components in the catalog not on the page
- **Orphaned:** preview files or kitchen sink sections for components
  no longer in the catalog

If everything is in sync: say "Component previews are current —
nothing to update" and stop.

## Step 1: Update Lookbook Previews

For each component that needs a new or updated preview, read ONLY
that component's detailed section in the catalog (search for the

### heading — do not read the full catalog).

Create or update `test/components/previews/[name]_component_preview.rb`
with at least these scenarios:

- Default (basic usage with required arguments)
- Each major variant listed in the catalog
- Error/edge state (if the component accepts error arguments)

For orphaned preview files: delete them.

## Step 2: Update the Kitchen Sink

If `app/views/dev/kitchen_sink/show.html.erb` exists:

- Add a section for each missing component with representative
  example data, matching the existing page style and category grouping
- Remove sections for orphaned components
- Do NOT rewrite existing sections

If the kitchen sink page does not exist, skip this step.

## Step 3: Commit

Commit changed files: "chore: update component previews"

Say "Lookbook and Kitchen Sink updated."
