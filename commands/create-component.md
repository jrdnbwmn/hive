---
description: Create a new base UI component in the component library
model: sonnet
argument-hint: <component name and description>
---

Invoke the style-ui skill to create a new ViewComponent.

$ARGUMENTS

1. Check the Quick Reference table in `docs/COMPONENT_CATALOG.md` — if a similar component already exists, report it and ask how to proceed.
2. Source it from RailsBlocks first. Use the rails-blocks-cli skill to check whether RailsBlocks has this component; if it does, install/adapt it as the basis (converting to a ViewComponent to match this project's conventions). Only build from scratch if RailsBlocks lacks it.
3. Create the ViewComponent files (app/components/)
4. Add Stimulus controller if needed
5. Add the new component to Quick Reference and Component Details (using the template—include all sections that apply) parts of COMPONENT_CATALOG.md. Only add the content for this new component, don't make any other changes.
6. Add the new component to the appropriate category group in component-map.mermaid following the patterns in the rest of the document. Only add the content for this new component, don't make any other changes.
7. Create Lookbook preview for the new component with at least these scenarios: default (basic usage with required arguments), each major variant, and error/edge states (if the component accepts error arguments). Only add the content for this new component, don't make any other changes.
8. Add the new component to the kitchen sink in `app/views/dev/kitchen_sink/show.html.erb` with representative example data, matching the existing page style and category grouping.
9. Run tests
10. Stage the changes. Do NOT commit — leave that to /commit,
   review-changes, or wrap-up, same as any other implementation work.
11. Say "Component created and staged."
