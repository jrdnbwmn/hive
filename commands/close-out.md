---
description: Close out ticket/feature work — opens a PR if none exists, reports status if one's open, or syncs main + archives docs if merged
argument-hint: <optional: merge|pr|discard — omit to use the default for this branch>
allowed-tools: Bash(git *), Bash(bin/rails *), Bash(bundle *), Bash(gh *)
model: sonnet
---

Read and follow the skill file at ~/.claude/skills/close-out.md.

Always operates on the current branch.

If $ARGUMENTS is provided, pass it through — it's an explicit
merge/pr/discard choice for the no-PR-yet case.
