---
name: style-ui
version: 1.1
description: >
  UI consistency and design system enforcement. Use when building or
  editing any user-facing interface — pages, components, forms, layouts.
  Auto-invoke when editing files in app/views/ or app/components/.
model: sonnet
---

# Style UI

UI consistency enforcement for all clones and the master. This app
uses a ViewComponent-based design system with design tokens.

## Component-First Rule

**Before building ANY UI element, check the project's component library.**

1. Read the Quick Reference table in `docs/COMPONENT_CATALOG.md`.
   Do NOT read the entire catalog — just the table.
2. If a component exists, read ONLY its detailed section. If that
   doesn't answer your questions, check the source in `app/components/`.
3. If a component exists that does what you need, USE IT.
   Don't build a new one.
4. If a component exists but needs modification, modify it — don't
   create a parallel version.
5. If no component exists, see "When a Base Component Is Missing."
6. Check `app/assets/tailwind/theme/_tokens.css` for design tokens.

`docs/COMPONENT_CATALOG.md` and `app/components/` are the sources of
truth — don't reach for external libraries inline. New base components
come from RailsBlocks via `/create-component`, never mid-task.

### When a Base Component Is Missing

- Do NOT build it from scratch.
- Propose the new component: name it, describe what it should do, and
  explain why no existing catalog component covers this need.
- If running interactively: STOP and wait for approval. Do not proceed with implementation or run /create-component until the user explicitly approves. Once approved, run /create-component <name and description> yourself. After the component is created, resume the original task using it.
- If running as a subagent/clone: STOP and return the proposal as your
  final result — do not attempt to wait for approval or run
  /create-component. Clearly flag this in your result, e.g.:
  "BLOCKED — missing component: [name]. Reason: [why]. Awaiting approval to run /create-component."
  The parent session will handle it.

### Creating New Components During Implementation

You may create **feature-specific components** that compose existing
base components (e.g. PlanComparisonComponent, ActivityFeedComponent).

Rules:

- MUST be built from existing catalog components
- MUST be a ViewComponent
- Should render base components, not reimplement their HTML/CSS
- Add an entry to `docs/COMPONENT_CATALOG.md` following the existing
  template

You may NOT create new **base UI components** — see "When a Base
Component Is Missing" above.

## Working From Prototypes

*(Only applies if the user provides a raw HTML or un-wired ERB prototype.
Policy — design is final, ask before "fixing" anything — lives in CLAUDE.md.)*

1. Read the prototype and identify every UI element
2. Map each element to a component in COMPONENT_CATALOG.md
   (e.g. raw `<select>` → SelectComponent, `<div class="modal...">` → ModalComponent)
3. If an element maps to a missing base component, follow
   "When a Base Component Is Missing" above
4. Replace raw HTML with ViewComponent render calls, preserving exact
   layout, visual hierarchy, and any custom spacing/sizing (ask if new
   tokens are needed in `_tokens.css`)
5. Wire up real data where prototypes or scaffolded views use placeholder/hardcoded content.
6. Add loading, empty, and error states per "Every Data-Driven
   Component Needs Three States" below

## Tailwind CSS v4

This project uses **Tailwind CSS v4**. Claude's training data skews v3 —
follow these rules explicitly:

- Do NOT create `tailwind.config.js`. Config is CSS-first via `@theme`.
- Use design tokens from `_tokens.css`, not arbitrary values (`w-[347px]`).
  Ask when you think you should deviate.
- Mobile-first: start with base layout, add `sm:`, `md:`, `lg:` overrides.

## JavaScript: Stimulus Only

- Check `app/javascript/controllers/` for existing controllers before
  writing new ones.
- Match project conventions when writing new controllers.
- Namespace custom controllers to avoid conflicts: `ui-modal`
  not `modal` (file: `ui_modal_controller.js`,
  attribute: `data-controller="ui-modal"`).

## Every Data-Driven Component Needs Three States

Every data-driven component needs these (ask if unsure what to do):

1. **Loading** — what the user sees while data fetches
2. **Empty** — what the user sees when there's no data yet
3. **Error** — what the user sees when something goes wrong

## Hard Rules

- NEVER hardcode styling available as a token, or add arbitrary Tailwind
  values — always use token-based classes. If a new token is needed, ask.
- NEVER add JS dependencies without asking
- NEVER use icon libraries not already in the project
