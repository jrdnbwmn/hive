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

Execute tasks in plan order, respecting the Task Dependencies section.

Follow the delegation assignments in each task header ([Master] or [Clone]). If a task is marked [Clone], delegate it. If marked [Master], execute it in the current session. Do not override the plan's assignments.

### Batching Parallel Clone Work

The plan's Task Dependencies section marks which tasks can run in
parallel. Honor it — don't delegate one clone at a time when the plan
says otherwise.

Before starting a task, look ahead at the run of consecutive [Clone]
tasks beginning there. Batch them into a SINGLE message so they run in
parallel when ALL of these hold:

- The Task Dependencies section doesn't make any of them depend on
  another task in the batch
- None depends on a task that isn't done yet
- **No two tasks in the batch write to the same file.** Check the file
  paths in each task's Build order. If any overlap, run those serially —
  the plan's assignments are a starting point, not a guarantee.
- They're in the same review checkpoint group

Cap a batch at 3 tasks. [Master] tasks always run alone in the current
session — never batch them with anything.

Otherwise, execute serially.

### Per Task

1. Read ALL the info for the task before you begin.
2. Follow the task's build order: Test → Implement → Verify
3. Mark the task done in the Status table: add ✅ to the "Done" column
   for that task number.
4. At a checkpoint boundary, run the review below before starting the
   next checkpoint. If it finds blocking issues, fix them first.

**review-changes-mini runs once per checkpoint**, after every task in
that checkpoint is done. Read checkpoint boundaries from the Checkpoint
column of the Status table.

- **Serial execution:** the checkpoint's final task runs it, as the plan
  instructs.
- **Batched execution:** wait for every clone in the batch to return,
  then the master runs it once over the whole checkpoint and updates the
  Status table for all batched tasks together. A clone must not run it
  itself — in a parallel batch there's no meaningful "final task," and
  a clone finishing early would review an incomplete checkpoint.

Between tasks, report progress:
"Task [N] complete. [remaining] tasks left. Continuing to Task [N+1]."

### When a Task Fails

If a clone reports failure or review-changes-mini finds blocking issues
that can't be auto-fixed:

- **Small/clear issue** (wrong approach, missing dep, ambiguous spec):
  revert and retry. Revert ONLY the failed task's files
  (`git checkout -- <paths>`) — never `git checkout -- .` after a batched
  run, since that would also destroy the work of clones that succeeded.
  Clarify the spec if needed, then retry.
  If the same task fails twice, STOP and ask the user.
- **Bigger issue** (design was wrong, tasks are in wrong order,
  prerequisite was missed): STOP, report what went wrong, suggest
  whether to revise the plan or skip the task, and wait for user input.

## Step 5: Plan Complete

When all tasks are done, report: "All [N] tasks complete."
