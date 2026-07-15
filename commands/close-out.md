---
description: Close out ticket/feature work — opens a PR if none exists, reports status if one's open, or syncs main + archives docs if merged
argument-hint: <ticket ID, branch name, PR URL/number, or merge|pr|discard — omit to use current branch>
allowed-tools: Bash(git *), Bash(bin/rails *), Bash(bundle *), Bash(gh *)
model: sonnet
---

Run the close-out skill.

If $ARGUMENTS is provided, pass it through — it may be an identifier to
resolve (ticket ID, branch name, PR URL/number) or an explicit
merge/pr/discard choice for the no-PR-yet case. Otherwise resolve from
the current branch.
