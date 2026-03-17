---
name: write-plan
version: 1.0 # bump on meaningful changes
description: >
  Create a detailed implementation plan from an approved design. Use 
  ONLY when user runs /write-plan or asks for a plan for a multi-file, multi-model change. Do NOT use 
  for small changes, tweaks, bug fixes, UI adjustments,
  single-file edits, or when the user's request is specific enough to
  implement directly.
model: opus
---

# Write Plan

Produce a plan detailed enough that a clone — with zero conversation
history and no judgment — can pick up a single task and execute it
correctly.

Announce: "I'm using the write-plan skill to create an implementation plan from your design."

## Output Template

The plan you produce MUST follow this structure exactly:

```markdown
# Plan: <Feature Name>

## Status

| Task | Description | Assign | Done |
| ---- | ----------- | ------ | ---- |
| 1    | ...         | Master |      |
| 2    | ...         | Clone  |      |

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
4. **Review:** After completion, ALWAYS run review-changes before proceeding. This is not optional.

## Task Dependencies

- [Task 2 depends on Task 1 (needs the model)]
- [Tasks 3–4 can run in parallel]
```

## Pre-Flight

Before writing anything, gather context in this order:

1. **Find the design.** First reference the user-specified path. If no design file specified, look for an active doc in `docs/designs/`
   (skip `done/`). If no file is specified or found, check if there is a clear, specific description from the user. If none of those exist:
   stop. "I need an approved design before planning. Want to brainstorm first?"
2. **Read architecture diagrams** in `docs/architecture/` (if they exist):
   `app-structure.mermaid` (where things live, directory conventions), `data-model.mermaid` (existing models and associations), `routes-map.mermaid` (current controllers and routing).
3. **Scan the component catalog.** Read the Quick Reference table in
   `docs/COMPONENT_CATALOG.md`. Read detailed sections ONLY for components
   this feature will use.
4. **Identify the prototype** (if one exists) for reference in UI tasks.

Goal: every task in the plan should reference real paths and existing
components — not generic placeholders. Only explore the filesystem for
details the diagrams don't cover.

## Writing Tasks

### Sizing

Each task ≈ 2–5 minutes of clone work. If a task description exceeds
~10 lines, split it into two tasks. Each task should touch ≤4 files.

### Scoping

Every task MUST have "In scope" and "NOT in scope" lines. If you can't
define what's NOT in scope, the task is too vague — split or refine it.

### Component Rule

Before writing any UI task, check the catalog. If a needed component
doesn't exist, do NOT include it inline. Add a prerequisite task:
"Add [Component] to the component library" — or flag it as a blocker.

### Phasing

If the plan exceeds ~7 tasks, break into phases (Phase 1: MVP, Phase 2:
Polish). Each phase must be independently deployable.

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
2. Save to `docs/plans/<feature-name>.md` and commit: `docs: add plan for <feature>`.
3. Add to the top of the design doc: `> Plan created: docs/plans/<feature-name>.md`
4. Tell the user: **"Plan approved and saved. Run /execute-plan to start."**

Do NOT begin implementation. Plan and build are separate phases.
