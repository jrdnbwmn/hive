# Close Out — Merge or Discard

Loaded from `open-pr.md` 3E only when 3A chose `merge` or `discard`.
Ticket-derived branches always choose `pr` and never load this file.

`<branch>` and the linked-worktree answer come from close-out.md
Section 1 — don't re-derive them.

## If merge

- **If this is a linked worktree:** STOP. `main` is checked out in the
  project root, so `git checkout main` will fail here. Say: "Can't merge
  locally from a worktree — main is checked out elsewhere. Use the pr
  path instead, or merge on GitHub." Do not attempt the steps below.
- **Archive first**, using `<branch>` as the identifier, so the archive
  commit merges to main with everything else.
- `git checkout main`
- `git merge <branch>` (fast-forward if possible)
- If conflicts occur, STOP and report the conflicting files. Do NOT auto-resolve.
- `git push origin main`
- Delete the local branch: `git branch -d <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- "Code merged and pushed to main. Render will auto-deploy. Check the
  Render dashboard to confirm the deploy succeeds."

## If discard

- Confirm with me first.
- **If this is a linked worktree:** you can't check out `main` or delete
  the branch you're currently sitting on. Delete the remote branch if it
  exists (`git push origin --delete <branch>`), then say "Remote branch
  deleted. Discard this workspace in Conductor to remove the worktree
  and local branch." Stop — don't attempt the steps below.
- `git checkout main`
- `git branch -D <branch>`
- Delete the remote branch if it exists: `git push origin --delete <branch>`
- Say "Branch discarded."
