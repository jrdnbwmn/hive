---
description: Create a new base UI component in the component library
model: sonnet
argument-hint: <component name and description>
---

Invoke the style-ui skill to create a new ViewComponent.

$ARGUMENTS

1. Check the Quick Reference table in `docs/COMPONENT_CATALOG.md` — if
   a similar component already exists, report it and ask how to proceed.
2. Source it from RailsBlocks first. Use the rails-blocks-cli skill to
   check whether RailsBlocks has this component; if it does, install/
   adapt it as the basis (converting to a ViewComponent to match this
   project's conventions). Only build from scratch if RailsBlocks lacks it.
3. Create the ViewComponent files (`app/components/`). Use design tokens
   from `app/assets/tailwind/theme/_tokens.css` — no arbitrary Tailwind
   values.
4. Add a Stimulus controller if needed, following the naming and
   conventions in the style-ui skill.
5. Add the new component to Quick Reference and Component Details
   (using the template) in `COMPONENT_CATALOG.md`. Only add content for
   this new component.
6. Add the new component to the appropriate category group in
   `component-map.mermaid`, following existing patterns. Only add
   content for this new component.
7. Create a Lookbook preview with at least: default (basic usage),
   each major variant, and error/edge states (if applicable). Only add
   content for this new component.
8. Add the new component to the kitchen sink in
   `app/views/dev/kitchen_sink/show.html.erb` with representative
   example data, matching existing page style and category grouping.
9. Run tests.
10. Stage the changes. Do NOT commit — leave that to /commit,
    review-changes, or wrap-up.
11. Say "Component created and staged."
