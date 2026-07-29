# Close Out — Section 5: Merged

Loaded from `~/.claude/skills/close-out.md` when the PR has merged. That
file already resolved `<branch>` and whether this is a linked worktree —
use those answers, don't re-derive them.

## Step 1: Is There Anything Left to Archive?

Do this before touching git. Run archive-docs' header greps (Section 2
of that skill) against the resolved branch — top-level files only, never
recursing into `done/`:

```
grep -l "^> Branch: <branch>$" docs/designs/*.md docs/plans/*.md
grep -l "^> Ticket: <id>$"     docs/designs/*.md docs/plans/*.md
```

**Zero matches is the normal case.** Section 3E archives docs before the
PR is opened, so they already landed on main via the merge. When nothing
matches, skip every archive step below — don't create branches or commits
for work that's already done.

## Not a Linked Worktree

1. `git checkout main && git pull`
2. If Step 1 matched docs: **archive** using `<branch>` as the
   identifier. We're on main, so this commits directly to main.
3. Delete the local branch if it still exists: `git branch -d <branch>`
   (ignore the error if already gone)
4. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
5. Report: "Closed out [identifier]: main synced, branch removed[, docs
   archived if step 2 ran]."

## Linked Worktree (e.g. a Conductor workspace)

`main` is checked out in a different worktree, and this worktree's own
branch can't be deleted while it's checked out here — skip both. No need
to sync a local `main` either; a fresh workspace always starts from
current `origin/main`. Never run `git checkout main` in this worktree.

**If Step 1 found nothing (normal case):**

1. Delete the remote branch if it still exists: `git push origin
   --delete <branch>` (ignore the error if GitHub already auto-deleted it)
2. Report: "Closed out [identifier]. Archive this workspace in Conductor
   to remove the worktree and local branch, and sync main."

**If Step 1 matched docs** (fallback: docs weren't archived pre-merge,
e.g. this PR was opened or merged some other way) — that archive commit
needs its own small PR, never a direct push to `main`:

1. `git fetch origin`, then `git checkout -b <branch>-archive
   origin/main` — branch from `origin/main`, not from `<branch>`, so this
   survives a squash-merge (after a squash, `<branch>`'s own history
   never contains the squash commit, and an ancestry check against
   `origin/main` would fail even though pushing is safe).
2. **Archive** on this temp branch, using `<branch>` as the identifier.
3. Push the branch: `git push -u origin <branch>-archive`
4. Open a PR: `gh pr create --base main --title "chore: archive docs
   for [identifier]" --body "..."` (see `~/.claude/CLAUDE.md` for PR body
   conventions). Do not merge it — that's the user's call, same as any
   other PR.
5. `git checkout <branch> && git branch -D <branch>-archive`
6. Delete the original branch's remote copy if it still exists: `git
   push origin --delete <branch>`
7. Report: "Closed out [identifier]: docs archived via PR #<n>
   (<url>). Archive this workspace in Conductor to remove the worktree
   and local branch, and sync main."
