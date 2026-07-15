---
name: execute-plan
description: Execute an implementation plan from docs/plans or a provided plan path. Use ONLY when the user says /execute-plan.
---

# Execute Plan

Execute only when the user explicitly asks to execute or continue a plan. Do not auto-execute a plan just because one exists.

## Find The Plan

If the user provides a path, use that plan file.

Otherwise:

1. Check `docs/plans/` for `.md` files, excluding `docs/plans/done/`.
2. If exactly one plan exists, use it.
3. If multiple plans exist, list them and ask which one to execute.
4. If no plan exists, stop and say: `No plan found. Run brainstorm then write-plan first, or point me to a plan file.`

Read the full plan before acting.

## Pre-Flight

Before changing files, verify the project is ready.

1. Run `git status --porcelain`.
   - If output is non-empty, stop and say: `Uncommitted changes detected. Commit or stash first.`
2. Confirm the branch is not `main` or `master`.
   - If on `main` or `master`, stop and ask whether to create a feature branch.
3. Run the project’s migration check when applicable.
   - For Rails apps with `bin/rails`, run `bin/rails db:migrate:status`.
   - If migrations are pending, stop and say: `Pending migrations. Run db:migrate first.`
4. Run the project’s required baseline test command.
   - Prefer instructions in `AGENTS.md`.
   - For Rails apps, default to `bin/rails test`.
   - For npm apps, use the repo’s documented test command.
   - If baseline tests fail, stop and say: `Test suite failing before execution. Fix baseline first.`

If all checks pass, say: `Pre-flight passed. Starting execution.`

## Determine Starting Point

Read the Status table at the top of the plan file.

Start at the first task whose `Done` column is empty.

If all tasks are marked done, skip to completion.

Report:

`Executing [plan name]. Starting at Task [N] of [total]. [X] tasks complete, [Y] remaining.`

## Execute Tasks

Execute tasks in order and respect dependencies.

For each task:

1. Read the full task section before starting.
2. Follow the task’s assignment exactly:
   - `[Master]`: complete the task in the current session.
   - `[Clone]`: delegate to a clone/multi-agent worker when clone tools are available.
3. Follow the task’s build order:
   - Test
   - Implement
   - Verify
   - Review changes
4. Use TDD when the plan changes behavior: write or update the test first.
5. After implementation, run the verification command required by the plan or `AGENTS.md`.
6. Run a review pass for the task before moving on.
7. If blocking issues are found, fix them before continuing.
8. Update the plan Status table by marking the task done.
9. Run `git diff` and inspect what changed.

Between tasks, report:

`Task [N] complete. [remaining] tasks left. Continuing to Task [N+1].`

## Clone Task Rules

Use clones only for small, independently verifiable work.

When delegating:

1. Give the clone only the relevant task section, plan path, repo context, and verification command.
2. Tell the clone not to commit.
3. Tell the clone to stage nothing unless explicitly instructed by the plan.
4. Require the clone to report changed files, tests run, results, and any unresolved risk.
5. Review the clone’s diff before marking the task complete.

If clone tooling is unavailable, stop and tell the user the task requires clone execution but no clone tool is available in this session.

## Failure Handling

If a task fails, classify the failure.

Task-level failures include wrong implementation approach, missing local dependency, unclear task details, or test failures contained to that task.

Plan-level failures include wrong design, tasks in the wrong order, missing prerequisites, or conflicts with repo architecture.

For task-level failures:

1. Do not use destructive git commands without explicit user approval.
2. Re-read the task and relevant code.
3. If the task is ambiguous, clarify the plan file, then retry.
4. If the task fails twice, stop and ask the user.
5. If there is a file conflict between tasks, resolve manually when safe, then continue.

For plan-level failures:

1. Stop execution.
2. Report: `Task [N] revealed a plan issue: [description]`
3. Suggest revising the plan or skipping the task.
4. Wait for the user’s decision.

## Context Check

After each task, check context usage if the environment exposes it.

If context is above roughly 50%, stop and say:

`Context is at ~[X]%. Recommend running /catchup -> /clear -> /execute-plan to continue with fresh context. [N] tasks remaining.`

If exact context usage is unavailable, use judgment and stop when continuing would risk losing important plan/task state.

## Completion

When all tasks are done:

1. Run the final verification required by `AGENTS.md` or the plan.
2. Run `git diff` and summarize changed files.
3. Report: `All [N] tasks complete.`
