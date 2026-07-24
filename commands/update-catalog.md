---
description: Generate or update the component catalog
model: sonnet
---

Update the component library documentation based on what has changed.

The Quick Reference table is the only component index. It MUST carry a
**Composes** column listing the other catalog components each component
renders in its template, comma-separated, or `—` if it renders none.

## Step 0: Detect What Changed

If docs/COMPONENT_CATALOG.md does not exist (first run):

- Read ALL ViewComponents in app/components/ (.rb and .html.erb files)
- Generate the full catalog using the template format in the file
  (Quick Reference table + Component Details sections)
- Skip to Step 2

Otherwise detect changes:

```bash
git diff --name-only main -- app/components/
```

Build three lists from the results:

- **Added:** new component files in app/components/
- **Modified:** changed component files in app/components/
- **Deleted:** removed component files from app/components/

If no component files changed: say "Component catalog is current —
nothing to update" and stop.

## Step 1: Update the Catalog

Read ONLY the changed component files (from Step 0). For each:

**Added components:** A check, not a generation step —
/create-component and style-ui already write the entry at creation
time. Search the Quick Reference table; if the component is there, do
nothing. If missing, read its .rb and .html.erb and add a Component
Details entry (template format, omitting sections that don't apply)
plus a Quick Reference row with Composes filled from the template's
render calls.

**Modified components:** Read the updated files. When updating an
existing entry, search for the ### heading matching the component name.
Read only that section, do not read the full catalog. Update ONLY the
changed fields in the existing catalog entry (e.g., if arguments
changed, update the arguments table). If the template's render calls
changed, update the component's Composes cell. Preserve any
manually-added notes.

**Deleted components:** Remove the entry from the Component Details
section and the Quick Reference table. Also remove it from any other
row's Composes cell.

## Step 2: Stage

Stage the changed files. Do NOT commit — /commit, review-changes, or
wrap-up handles that.

Say "Catalog updated and staged."
