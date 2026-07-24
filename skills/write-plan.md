---
name: write-plan
version: 1.5 # bump on meaningful changes
description: >
  Create a detailed implementation plan from an approved design, for a
  multi-file or multi-model change.
disable-model-invocation: true
model: opus
---

# Write Plan

Produce a plan detailed enough that a clone — with zero conversation
history and no judgment — can pick up a single task and execute it
correctly.

## Output Template

The plan you produce MUST follow this structure exactly:

```markdown
> Ticket: <copied verbatim from the design doc's header>
> Branch: <copied verbatim from the design doc's header>

# Plan: <Feature Name>

## Status

| Task | Phase | Checkpoint | Description | Assign | Done |
| ---- | ----- | ---------- | ----------- | ------ | ---- |
| 1    | 1     | 1          | ...         | Master |      |
| 2    | 1     | 1          | ...         | Clone  |      |

## Prerequisites

- Design: [path to design doc or description]
- Prototype: [path to file or "None"]
- Feature branch exists (run /branch first if needed)

## Tasks

### Task N [Master|Clone]: <Short description>

**Skills:** [applicable skills — e.g., safe-migration, write-tests, style-ui]
**Reference:** Read [`path/to/file`] for patterns to follow
**Prototype:** [`path/to/prototype`] — match layout/hierarchy (UI tasks only)

**In scope:**

- [exactly what to build]

**NOT in scope:**

- [what to leave out]

**Build order:**

1. **Test:** [test file path, what to assert — be specific]
2. **Implement:** [code to write, exact file paths]
3. **Verify:** `bin/rails test [specific test file]`

## Task Dependencies

- [Task 2 depends on Task 1 (needs the model)]
- [Tasks 3–4 can run in parallel]
```

## Pre-Flight

Before writing anything, gather context in this order:

1. **Find and read the design**, in this order:
   - The user-specified path, if given
   - Otherwise an active doc in `docs/designs/` (skip `done/`)
   - Otherwise a clear, specific description from the user in this
     conversation
   - None of the above → STOP: "I need an approved design before
     planning. Want to brainstorm first?"

2. **Classify the work as UI or backend-only.** Does it touch views,
   forms, partials, components, pages, emails, admin screens, or
   JS/Stimulus controllers? If the design is ambiguous or doesn't
   clearly rule UI out, treat it as UI work. Skip steps 4–5 below only
   when the design is clearly and entirely backend/data-only — a
   migration, background job, API endpoint with no new views, or an
   internal refactor.
3. **Read architecture diagrams** in `docs/architecture/` (if they exist):
   `app-structure.mermaid` (where things live, directory conventions), `data-model.mermaid` (existing models and associations), `routes-map.mermaid` (current controllers and routing).
4. **Scan the component catalog.** Read the Quick Reference table in
   `docs/COMPONENT_CATALOG.md`. Read detailed sections ONLY for components
   this feature will use.
5. **Identify the prototype** (if one exists) for reference in UI tasks.

Goal: every task in the plan should reference real paths and existing
components — not generic placeholders. Only explore the filesystem for
details the diagrams don't cover.

## Writing Tasks

### Sizing

Each task touches ≤4 files. If a task description exceeds ~10 lines,
split it into two tasks.

### Scoping

Every task MUST have "In scope" and "NOT in scope" lines. If you can't
define what's NOT in scope, the task is too vague — split or refine it.

### Component Rule

Before writing any UI task, reference the catalog scan from Pre-Flight
step 4 — do not re-read the catalog file. If a needed component doesn't
exist in what you already scanned, do NOT include it inline. Add a prerequisite task:
"[Master] Run /create-component to add [Component] to the component
library" — or flag it as a blocker. Assign it to Master, not Clone —
it touches shared infra other tasks will depend on.

### Phasing

If the plan exceeds ~6 tasks, break into phases (e.g. Phase 1: MVP, Phase 2:
Polish). Each phase must be independently deployable.

### Review Checkpoints

Group tasks into review checkpoints of ≤3 tasks each. Align checkpoints with
phase boundaries first — split further only if a phase exceeds 3 tasks,
preferring splits at dependency boundaries (or other natural breakpoints)
over splitting tightly-coupled work. A single-task phase is still its own
checkpoint.

Record each task's checkpoint in the **Checkpoint** column of the Status
table — that column is how execute-plan finds checkpoint boundaries.

On the final task of each checkpoint group — including single-task
checkpoints — add an explicit instruction to run review-changes-mini
when the task's work is finished, naming the checkpoint group it covers.
Note in that instruction that if the checkpoint's tasks were executed as
a parallel batch, the master runs this review once the whole batch
returns, rather than the task running it itself. Either way it runs
exactly once per checkpoint, after every task in that checkpoint is done.

### Assigning Master vs Clone

Put the assignment tag directly in the task header: `### Task 3 [Clone]: Build subscription form`

| Assign to | When |
|-----------|------|
| **Clone** | No dependency on incomplete tasks AND touches ≤4 files AND scope is unambiguous AND doesn't modify shared infra (routes, base models, auth, migrations other tasks need) |
| **Master** | Touches shared files OR involves migrations later tasks depend on OR scope fuzzy and requires judgment OR is the first task (sets patterns) |

Default to Master when uncertain.

### Task Ordering

1. Sequential dependencies come first.
2. Group parallelizable tasks together.
3. Note parallelism explicitly in the Task Dependencies section.

## Approval & Save

1. Present the full plan. Wait for explicit approval.
2. Put a `Ticket:`/`Branch:` header at the top of the plan doc.
   archive-docs matches on these headers and never falls back to
   filenames, so a plan without them can never be archived.
   - **Design doc exists:** copy both lines verbatim from its header.
   - **No design doc** (planning from a description): derive them —
     `Ticket:` is the Linear identifier if one was given, otherwise
     `None`; `Branch:` is `git branch --show-current`. If that returns
     `main`/`master`, STOP and ask the user to run `/branch` first.
3. Save plan doc to `docs/plans/<feature-name>.md`. Commit it — plus the
   design doc if one exists:
   `git add docs/plans/<feature-name>.md [docs/designs/<feature-name>.md]`
   `git commit -m "docs: add design and plan for <feature>"`
4. If a design doc exists, add to the top of it:
   `> Plan created: docs/plans/<feature-name>.md`
5. Tell the user: **"Plan approved and saved. Run /execute-plan to start."**

Do NOT begin implementation. Plan and build are separate phases.
