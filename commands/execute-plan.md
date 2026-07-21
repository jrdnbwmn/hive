---
description: Execute the current plan
model: sonnet
argument-hint: <path to plan file, or blank to auto-detect>
---

Find and execute the current implementation plan.

## Step 1: Find the Plan

If $ARGUMENTS specifies a path, use that plan file.

Otherwise, look for an active plan:

1. Check docs/plans/ for .md files (exclude the done/ subfolder)
2. If multiple plans exist, list them and ask which one to execute
3. If no plan exists: "No plan found. Run brainstorm then write-plan
   first, or point me to a plan file."

Read the plan file.

## Step 2: Pre-Flight Check

Before executing any tasks, verify the project is in a clean state:

1. `git status --porcelain` — if uncommitted changes exist, STOP:
   "Uncommitted changes detected. Run /commit or stash first."
2. `bin/rails db:migrate:status` — if migrations are pending, STOP:
   "Pending migrations. Run db:migrate first."
3. `bin/rails test --fail-fast` — if tests fail, STOP:
   "Test suite failing before execution. Fix baseline first."

## Step 3: Determine Starting Point

Read the Status table at the top of the plan file. The next task
where the "Done" column is empty is the starting point.

- If no work has started, begin with Task 1
- If all tasks are done, skip to Step 4

Report: "Executing [plan name]. Starting at Task [N] of [total].
[X] tasks complete, [Y] remaining."

## Step 4: Execute Tasks

Execute tasks in order, respecting dependencies.

Follow the delegation assignments in each task header ([Master] or [Clone]). If a task is marked [Clone], delegate it. If marked [Master], execute it in the current session. Do not override the plan's assignments.

For each task:

1. Read the ALL the info for the task before you begin.
2. Follow the task's build order: Test → Implement → Verify
3. If review-changes-mini finds blocking issues, fix them before moving on.
4. After review-changes-mini confirms the task is complete, update the
   Status table: add ✅ to the "Done" column for that task number.

Between tasks, report progress:
"Task [N] complete. [remaining] tasks left. Continuing to Task [N+1]."

### When a Task Fails

If a clone reports failure or review-changes finds blocking issues
that can't be auto-fixed:

- **Small/clear issue** (wrong approach, missing dep, ambiguous spec):
  revert with `git checkout -- .`, clarify the spec if needed, retry.
  If the same task fails twice, STOP and ask the user.
- **Bigger issue** (design was wrong, tasks are in wrong order,
  prerequisite was missed): STOP, report what went wrong, suggest
  whether to revise the plan or skip the task, and wait for user input.

## Step 5: Plan Complete

When all tasks are done, report: "All [N] tasks complete."
