---
name: resume-work
version: 1.0 # bump on meaningful changes
description: >
  Pick up where you left off after /catchup and /clear: read
  .claude/whats-next.md, summarize the current state, and suggest the
  next step. Use when the user runs /resume-work or asks to pick up
  where they left off (e.g. "where were we?").
model: sonnet
---

Read `.claude/whats-next.md`. Summarize the current
state and suggest what to do next.

If there is an active plan in docs/plans/ with incomplete tasks,
suggest: "Continue building? Run /execute-plan to pick up where
you left off."

If there is no plan but a design exists in docs/designs/ without
a corresponding plan, suggest: "Design is ready. Run write-plan to
create the implementation plan."

Otherwise, let me pick what to work on or tell you what I want.
$ARGUMENTS
