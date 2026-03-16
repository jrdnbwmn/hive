---
description: Create a new base UI component in the component library
model: sonnet
argument-hint: <component name and description>
---

Invoke the style-ui skill to create a new ViewComponent.

$ARGUMENTS

1. Check the Quick Reference table in `docs/COMPONENT_CATALOG.md` — if a similar component already exists, report it and ask how to proceed.
2. Create the ViewComponent files (app/components/)
3. Add Stimulus controller if needed
4. Add the new component to Quick Reference and Component Details (using the template—include all sections that apply) parts of COMPONENT_CATALOG.md. Only add the content for this new component, don't make any other changes.
5. Add the new component to the appropriate category group in component-map.mermaid following the patterns in the rest of the document. Only add the content for this new component, don't make any other changes.
6. Create Lookbook preview for the new component with at least these scenarios: default (basic usage with required arguments), each major variant, and error/edge states (if the component accepts error arguments). Only add the content for this new component, don't make any other changes.
7. Add the new component to the kitchen sink in `app/views/dev/kitchen_sink/show.html.erb` with representative example data, matching the existing page style and category grouping.
8. Run tests
9. Commit following git-conventions: "feature: add [ComponentName]"
10. Say, "Component created and committed."
