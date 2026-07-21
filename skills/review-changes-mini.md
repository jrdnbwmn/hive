---
name: review-changes-mini
version: 1.0
description: >
  Lightweight review of recent code changes for spec compliance and
  test coverage, run after each checkpoint (≤3 tasks) during plan
  execution. Invoked by execute-plan at checkpoint boundaries, by task
  plans, or when the user explicitly asks for a mini review. For full
  security and Rails-pattern review, use /review-changes instead.
  Do NOT auto-invoke for any other situation.
disable-model-invocation: true
model: sonnet
---

# Review Changes (Mini)

Single-stage review: check the work matches the spec, run tests, auto-fix
mechanical issues. No security audit, no Rails-pattern audit — those are
deferred to the full `/review-changes` at phase/branch level.

## Determine Scope

Diff `git diff HEAD` (staged + unstaged against HEAD) plus
`git status --porcelain` for untracked new files, read directly since
diff commands never show untracked content. If both empty: "Nothing to
review."

**Gather context:** Find the checkpoint's tasks in the plan doc in
`docs/plans/`.

## Spec Compliance

Read the checkpoint's tasks, then check:

- Does every task's requirement have a corresponding implementation?
- Was anything built that WASN'T in scope? (flag as scope creep)
- Do tests cover the specified behavior?
- Does implementation match "In scope" / "NOT in scope" boundaries?

**If spec gaps exist: STOP. Report as BLOCKING. Do NOT proceed further.**

## Tests (Blocking)

- For every new or modified model: does a corresponding test file exist?
- For every new route/controller action: does a system or integration test exist?
- Run `bin/rails test` — review FAILS if any test fails
- If test files are missing for new code, list them as BLOCKING issues.

## Auto-Fix (No Approval Needed)

Do these automatically:

- Run `bundle exec rubocop -A`
- Remove debug artifacts: `console.log`, `debugger`, `binding.pry`,
  `byebug`, `binding.irb`, `puts`/`pp`/`p` used for debugging
- Fix trailing whitespace, missing newlines

## Handling Findings

**If running as master (standalone or during execute-plan):**
delegate blocking issues that need substantial work to a Task clone.
Fix everything else inline.

**If running as a clone (during task execution):**
fix minor issues inline. For blocking issues, STOP and report back —
do not spawn sub-clones.

**STOP and ask the user only for:** missing tests, spec violations.

Present findings to the user:

```
Mini review complete.

Blocking Issues
[issue] — [why it's blocking] — [suggested fix]

Fixed Automatically
[what was found] — [what was done]
```

After all issues have been fixed, commit following git-conventions
rules. Use the checkpoint's task descriptions as the commit message
basis. Then say "Mini review complete. Everything committed."

If no blocking issues, commit following git-conventions rules. Use the
checkpoint's task descriptions as the commit message basis. Then say
"Mini review passed. Everything committed."
