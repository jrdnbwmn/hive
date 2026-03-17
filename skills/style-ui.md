---
name: style-ui
version: 1.0 # bump on meaningful changes
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
   doesn't answer your questions, check the source in
   `app/components/`.
3. If a component exists that does what you need, USE IT.
   Don't build a new one.
4. If a component exists but needs modification, modify it — don't
   create a parallel version.
5. If no component exists, see "When a Base Component Is Missing."
6. Check `app/assets/stylesheets/theme/_tokens.css` for design tokens.

Do NOT reference RailsBlocks, Flowbite, or any external component
library. `docs/COMPONENT_CATALOG.md` and `app/components/` are the
only sources of truth.

### When a Base Component Is Missing

If you need a generic UI element (form input, feedback element, overlay,
navigation pattern) that doesn't exist in the catalog:

- Do NOT build it from scratch
- STOP and report: "This task needs a [component type] that doesn't
  exist in the component library. Add it to the library before
  proceeding."
- Continue with other parts of the task if possible, using a placeholder
  comment: `<%# TODO: replace with [ComponentName] when available %>`

### Creating New Components During Implementation

You may create **feature-specific components** that compose existing
base components. Examples: PlanComparisonComponent, ActivityFeedComponent,
OnboardingStepComponent.

Rules:

- The new component MUST be built from existing catalog components
- It must be a ViewComponent
- It should render base components, not reimplement their HTML/CSS
- Add an entry to `docs/COMPONENT_CATALOG.md` following the existing
  format (see the catalog for the template)

You may NOT create new **base UI components** (form elements, feedback
elements, overlays, navigation patterns). See "When a Base Component
Is Missing" above.

## Working From Prototypes

When the user gives a raw HTML or un-wired ERB prototype:

1. Read the prototype and identify every UI element
2. Map each element to a component in COMPONENT_CATALOG.md:
   - Raw `<select>` → SelectComponent
   - Raw `<div class="card...">` → CardComponent
   - Raw `<div class="modal...">` → ModalComponent
   - etc.
3. If an element maps to a base component that doesn't exist in the
   catalog, STOP and report it (follow the "When a Base Component Is Missing" instructions above)
4. Replace raw HTML with ViewComponent render calls, preserving:
   - The exact layout and visual hierarchy
   - All Tailwind classes that aren't handled by the component
   - Any custom spacing, sizing, or positioning (but ask if there are new tokens that need to be made in `app/assets/stylesheets/theme/_tokens.css`)
5. Wire up real data where the prototype uses placeholder/hardcoded content
6. Add Stimulus controllers for interactive elements
7. Add loading, empty, and error states if not already prototyped

## Tailwind CSS v4

This project uses **Tailwind CSS v4**. Claude's training data skews v3 —
follow these rules explicitly:

- Do NOT create `tailwind.config.js`. Config is CSS-first via `@theme`.
- Use design tokens from `_tokens.css`, not arbitrary values
  (`w-[347px]`). Ask when you think you should deviate.
- Mobile-first: start with base layout, add `sm:`, `md:`, `lg:` overrides
- Use `dark:` variants when the project supports dark mode

## JavaScript: Stimulus Only

- Check `app/javascript/controllers/` for existing controllers before
  writing new ones.
- If you need a new controller, match the project conventions
  in `app/javascript/controllers/`
- Namespace custom controllers to avoid JSP conflicts: `ui-modal`
  not `modal` (file: `ui_modal_controller.js`,
  attribute: `data-controller="ui-modal"`).

## Hotwire Interaction Patterns

- **Turbo Frames** for partial page updates (inline editing, tabs, modals)
- **Turbo Streams** for real-time updates and multi-element changes
- **Stimulus** for client-side-only interactivity (toggles, validation, clipboard)
- Don't build SPAs. Server-rendered HTML + Hotwire is the architecture.
- Test Hotwire behavior with system tests (they run in a real browser
  and verify Turbo Frame/Stream behavior end-to-end).

## Every Data-Driven Component Needs Three States

Every data-driven component needs these (ask if unsure what to do):

1. **Loading** — what the user sees while data fetches
2. **Empty** — what the user sees when there's no data yet
3. **Error** — what the user sees when something goes wrong

## Hard Rules

- NEVER hardcode styling that is already available as a token in `app/assets/stylesheets/theme/_tokens.css` or add arbitrary Tailwind values — always use token-based Tailwind classes. If you need a new token, ask.
- NEVER add JS dependencies without asking
- NEVER use icon libraries not already in the project
- NEVER create `tailwind.config.js` — this is Tailwind v4
