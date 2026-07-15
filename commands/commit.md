---
description: Quick mid-session commit following project git conventions
allowed-tools: Bash(git *)
model: haiku
argument-hint: <commit message or leave blank to auto-generate>
---

1. Check for changes: `git status --porcelain`
   If empty, say "Nothing to commit." and stop.
2. Check current branch: `git branch --show-current`
   If on main, STOP and warn: "You're on main. Create a feature branch
   first with /branch, or confirm you intentionally want to commit to main."
3. Read the full diff (`git diff` + `git status`). Check for debug
   artifacts and secrets — flag and confirm before proceeding if found.
4. Decide how many commits this needs:
   - $ARGUMENTS provided → one commit, using it as the message. Skip
     straight to step 5.
   - Diff reads as one logical change (touching multiple files is
     normal for a single feature/fix — that's still one change) → one
     commit, message generated from the diff.
   - Diff reads as several unrelated changes (different files/areas
     serving unrelated purposes) → describe the clusters you see and
     ask: "These look like unrelated changes: [clusters]. Split into
     separate commits, or keep together as one?" Do not guess — wait
     for the answer, then follow it exactly.
5. Stage and commit accordingly, following git-conventions rules for
   each message. For a split, stage and commit each group separately
   (`git add` scoped to that group's files, then commit) before moving
   to the next group.
   Do NOT push — just commit locally.
6. When done say: "Commit complete." If split, list what went into
   each commit.
