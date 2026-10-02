---
name: create-component
version: 1.0 # bump on meaningful changes
description: >
  Create a new base UI component in the component library, sourcing it
  from RailsBlocks first. Use when the user runs /create-component or
  asks for a new component, when style-ui finds a missing component and
  the user has approved creating it, or for a [Master] plan task that
  names it. Do NOT run to fill a gap mid-task without the user's
  approval.
model: sonnet
argument-hint: "<component name and description>"
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
   (using the template) in `COMPONENT_CATALOG.md`. Fill the Composes
   column with the catalog components its template renders, or `—` if
   none. Only add content for this new component.
6. Create a Lookbook preview with at least: default (basic usage),
   each major variant, and error/edge states (if applicable). Only add
   content for this new component.
7. Add the new component to the kitchen sink in
   `app/views/dev/kitchen_sink/show.html.erb` with representative
   example data, matching existing page style and category grouping.
8. Run tests.
9. Stage the changes. Do NOT commit — leave that to /commit,
   review-changes, or close-out.
10. Say "Component created and staged."
