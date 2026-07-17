---
name: brainstorm
version: 1.3 # bump on meaningful changes
description: >
  Explore and refine what to build before writing code through Socratic
  questioning.
disable-model-invocation: true
model: opus
---

# Brainstorm

Socratic design refinement — explore _what_ to build before anyone writes code.
Master-only. Interactive. No code changes, no clones.

## Phase 0: Correct the Branch Name (ticket work only)

If the user is working from a Linear ticket, make the current branch embed
the ticket identifier before anything else. This is what lets Linear's
GitHub integration auto-link the branch/PR, and it ensures the design doc
below gets tagged with the right branch.

1. Get the identifier from the ticket reference (e.g. `TIC-123`).
2. `git branch --show-current`. **Skip the rest** if either:
   - the branch already embeds the identifier (e.g. `feature/tic-123-…`), or
   - you're on `main`/`master` (traditional flow — `/branch` creates the
     branch later; there's nothing to rename yet).
3. Otherwise the branch is a generic/auto-generated name (e.g. a fresh
   Conductor workspace branch). Propose a rename following the **Git Branch
   Naming Rules in CLAUDE.md** — `feature/<id>-<short-slug>` (or `fix/…`),
   identifier lowercased, slug derived from the ticket title. Confirm the
   name with the user, then `git branch -m <new-name>`. If the branch was
   already pushed, note that the remote needs updating too.

No ticket, or no rename needed → go straight to Phase 1.

## Phase 1: Gather Context Before You Start

If `docs/architecture/` exists, read the Mermaid diagrams first:

- `data-model.mermaid` → models and associations
- `component-map.mermaid` → existing ViewComponents
- `routes-map.mermaid` → controllers and routing

If `docs/product/` exists, read any of the following that are present:

- `strategy-brief.md` → personas, JTBD, value prop, positioning
- `product-brief.md` → overall product scope and requirements
- `ux-notes.md` → UX notes like key flows and patterns

Use the context from the architecture and product files to ground your questions and proposals in the product strategy and what already exists in the app.
Don't ask what the files already answer.
Don't propose models that already exist or
components that are already built (unless you're extending them) and don't propose features that contradict the positioning or
solve problems outside the persona needs.

If the user points to a Linear ticket, read
it as primary input. The design doc you will produce replaces the ticket
as the source of truth — incorporate everything implementation-
relevant so write-plan doesn't need to read the original.

If the user provides a prototype (HTML, ERB, screenshot), read it.
The visual design is FINAL — don't propose layout alternatives.
Focus questions on what the prototype doesn't show.

## Phase 2: Understand

Ask questions to surface what you don't know. Don't assume. Cover:

- **Purpose:** What problem does this solve? For whom?
- **Success:** How will we know it works?
- **Constraints:** Timeline, existing models, JSP features to leverage
- **Data:** What data backs this? Which models and associations?
- **Flows:** What are the required and related pages/flows/features?
- **Edges:** What happens when things go wrong, data is missing,
  user is unauthorized?
- **Gaps:** What is the user not thinking about? Unknown unknowns?

2-4 questions per round. Use as many rounds as necessary. Go deeper on unclear answers before moving on.

## Phase 3: Explore

### Propose

If a prototype exists: propose changes or improvements based on what
you've learned. Explain reasoning for each.

If no prototype: propose 2-3 approaches with honest tradeoffs:

1. **Simplest version** — minimum that solves the core problem
2. **Recommended approach** — what you'd actually pick and why
3. **Bigger vision** — what this looks like if we go further (scope for later)

For each: what it gives you, what it costs, what it leaves out.
Be specific — "more complex: adds 2 models and 1 controller" not just "more complex."

Favor solutions in this order:

1. JSP built-ins (accounts, billing, notifications)
2. Rails conventions and built-ins
3. Existing ViewComponents (check Quick Reference in
   `docs/COMPONENT_CATALOG.md`)
4. Custom code (justify why nothing above fits)

### Refine

After the user picks an approach, walk through the design in sections.
Present one at a time, get approval before moving on:

1. **User flow** — what the user sees and does, step by step
2. **Data model** — models, associations, key fields, validations
3. **Key screens/UI** — reference existing components where possible
4. **Edge cases** — what happens when things go wrong or data is
   missing

State assumptions explicitly and ask the user to confirm them.

## Phase 4: Document

After user approves the design, determine tagging values so this doc can
be found accurately later, from any thread:

- **Ticket:** the Linear identifier if one was provided this session
  (e.g. `TIC-123`), otherwise `None`.
- **Branch:** current branch (`git branch --show-current`) if not on
  `main`/`master`, otherwise `TBD`.

Save to `docs/designs/<feature-name>.md`, with the tags as the first two
lines of the file, above the heading:

```
> Ticket: <ticket ID or "None">
> Branch: <branch name or "TBD">

# Feature: <name>

## Problem
<1-2 sentences — what problem this solves and for whom>

## Approach
<summary of the chosen approach>

## Acceptance Criteria
<how we will know this feature is a success>

## Prototype
<path to prototype file, or "None">
<what the prototype covers and what decisions it locks in>

## Data Model
<models, associations, key fields>

## Screens / Flows
<what the user sees, step by step>

## Scope
**In:** <what we're building>
**Deferred:** <what we're not building yet>

## Open Questions
<anything unresolved — or "None">

## More Info
<Relevant info from the feature doc not covered above.
Only include what affects implementation. Omit section if
no feature doc exists.>
```

Commit the design file: `docs: add design for <feature>`.

End with: "Design approved and saved in [path]. Ready to plan?
Run `/write-plan`."

## Hard Rules

- Do NOT write code, create app files, generate migrations, or
  spawn clones. The only file this skill creates is the design doc.
- Do NOT skip to implementation — that's write-plan's job.
- Do NOT present more than one design section at a time without checking in.
- **Scope guard:** If the feature touches 3+ models or 5+ screens
  or keeps growing — stop and say so. Propose an MVP scope and a "Deferred"
  list for everything else, then let themd decide what to do.
- If the user says "just build it" — push back once. Five minutes of
  brainstorming prevents hours of rework. If they insist, respect it and
  hand off to `write-plan` with whatever context you have.
