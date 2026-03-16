---
description: Execute the current plan — work through tasks with clones, review after each
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
3. `bin/rails test` — if tests fail, STOP:
   "Test suite failing before execution. Fix baseline first."
4. Confirm we're on a feature branch (not main)

If all pass: "Pre-flight passed. Starting execution."

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
2. Follow the task's build order: Test → Implement → Verify → Review changes
3. If review-changes finds blocking issues, make sure they are fixed before moving on.
4. After review-changes confirms the task is complete, update the
   Status table: add ✅ to the "Done" column for that task number.

Between tasks, report progress:
"Task [N] complete. [remaining] tasks left. Continuing to Task [N+1]."

### Context Awareness

After completing each task, check context usage. If context exceeds
~50%, STOP and tell the user:
"Context is at ~[X]%. Recommend running /catchup → /clear → /execute-plan
to continue with fresh context. [N] tasks remaining."

### When a Task Fails

If a clone reports failure or review-changes finds blocking issues
that can't be auto-fixed:

1. **Assess:** Is this a task-level issue or a plan-level issue?
   - Task-level: wrong approach, missing dependency, unclear spec
   - Plan-level: the design was wrong, tasks are in wrong order,
     a prerequisite was missed

2. **Task-level recovery:**
   - Revert the clone's changes: `git checkout -- .`
   - Re-read the task spec — is anything ambiguous?
   - If yes: clarify the spec in the plan file, then retry
   - If the task has failed 2x: STOP and ask the user
   - If the failure is a git conflict (staged changes conflict with
     another clone's work): resolve the conflict manually, then retry
     the task. This usually means two tasks touched the same file —
     consider making them sequential in the plan for next time.

3. **Plan-level recovery:**
   - STOP execution
   - Report: "Task [N] revealed a plan issue: [description]"
   - Suggest: revise the plan, or skip this task and continue
   - Wait for user decision

## Step 5: Plan Complete

When all tasks are done, report: "All [N] tasks complete."
