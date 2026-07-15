---
description: Archive the design + plan docs for a merged ticket/branch/PR
argument-hint: <ticket ID, branch name, or PR URL/number — omit to be asked>
allowed-tools: Bash(git *), Bash(gh *)
model: sonnet
---

Run the archive-docs skill.

If $ARGUMENTS is provided, pass it as the identifier to match on.
Otherwise let the skill ask which ticket/branch/PR to archive docs for.
