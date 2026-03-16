---
description: Create a handoff document at .claude/whats-next.md that captures everything needed to continue this work in a fresh context
model: sonnet
---

Create or update the `.claude/whats-next.md` file with current session state by doing the following:

1. Read `.claude/whats-next.md` if it exists (for any prior context, dead ends, or
   open questions worth preserving).
2. Gather observations from the current context.
3. If a plan exists in `docs/plans/`, check which tasks are done vs. remaining. If no plan exists, skip this step.
4. Check what changed:
   If on a feature branch: git log --oneline main..HEAD and git diff --stat main
   If on main: git log --oneline -10 and git diff --stat HEAD~5
5. Write (or overwrite) `.claude/whats-next.md` to this:

```
# What's Next

## Work completed and current state
[Brief description of what's committed vs. draft, everything changed and accomplished so far, and the branch name.]

## Work Remaining
[Specific next steps with file paths, dependencies, ordering, and a link to the open plans in `docs/plans/`.]

## Dead Ends
[Approaches tried that failed and why — so next session doesn't repeat them.]

## Open Questions
[Decisions that need user input before proceeding.]
```

This file should capture enough information to continue this work in a fresh context.

If there is no remaining work (feature complete, all tasks done), write only:

```
# What's Next

No pending work. Last completed: [one-sentence description].
```

6. Append to `.claude/session-learnings.md` in the project root (create if it
   doesn't exist). Do NOT overwrite — each catchup adds to it. Review
   the conversation since the last clear (or session start) and note
   anything worth capturing:

```
## Catchup [timestamp]

### Friction
[Things the user had to ask for that should've been automatic.
Or "None".]

### Mistakes
[Mistakes Claude made, approaches that failed. Or "None".]

### Observations
[Skill gaps, clone effectiveness issues, knowledge gaps, patterns
worth remembering. Or "None".]
```

If the session since last clear was short or uneventful, write:

```
## Catchup [timestamp]
Nothing notable.
```

7. Stage and commit both files: "docs: update session handoff". When done, tell the user: "Catchup complete. Ready to clear." When called from wrap-up, skip the commit step — wrap-up handles the commit.

Do NOT implement anything. Only read, summarize, and write the file.
