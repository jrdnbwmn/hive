---
description: Pick up where you left off after /catchup and /clear
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
