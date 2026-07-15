# Global Claude Code Context

## About Me

I am a product designer who builds small Ruby on Rails SaaS apps via vibe coding. I am NOT a full-stack developer — explain technical decisions when they matter. Ruby version manager: mise.

## How I Want You to Work

- Use a master-clone system when the work is large enough to break into smaller tasks that can be done by clones: Task(...) clones for parallelizable implementation work.
- Do NOT create custom subagents. Delegate to clones of yourself.
- Each clone task should be small and independently verifiable.
- Work in small, verifiable steps. Max ~7 files changed without checking in.
- After writing a plan, STOP. Do not call ExitPlanMode or begin implementation until I explicitly say to proceed.

### Working From Tickets (Linear)

When we start a new task and I give you a Linear ticket reference,
follow this workflow:

1. Fetch the issue (Linear MCP). Read the full description and note its
   identifier (e.g. `TIC-123`) and title.
2. Run `/branch` with the identifier and a short description (e.g.
   `/branch TIC-123 add account settings`). This embeds the identifier
   in the branch name — required for Linear's GitHub integration to
   auto-link the branch/PR to the issue. See Git Branch Naming Rules.
3. Do NOT manually update the issue's status or write the branch name
   back into Linear — the GitHub integration does this automatically
   once the identifier appears in the branch name (step 2 covers this).
4. When I say, plan/implement the task.
5. When I say the work is done, prompt: "Ready to wrap up — want to run
   /review-changes and then /wrap-up?"
6. Once wrap-up finishes, run `/close-out` to open the PR. PR title
   format: `[{identifier}] {issue title}` e.g. `[TIC-123] Add account
   settings form`. Include the identifier in the PR title (Linear
   matches on title/description too, not just branch name) and a
   direct link to the Linear issue in the PR description.
7. Do NOT manually write the PR URL back into Linear — the GitHub
   integration attaches it automatically once linked.
8. I review and merge the PR myself on GitHub — Claude does not merge
   ticket work. Once merged, tell me or run `/close-out` again (any
   thread) to sync main, delete the branch, and archive the design/plan
   docs.

### Working From My Prototypes

When I provide an HTML page or un-wired Rails view as a starting point:

- My layout, visual hierarchy, and design decisions are FINAL. Do not deviate from it. Do not redesign, rearrange, or "improve" the visual design.
- Your job: upgrade the implementation — replace raw HTML with ViewComponents from the catalog, wire up real data, add Stimulus controllers, connect routes and controllers, write tests. Ask questions if something isn't clear.
- If a component in my prototype doesn't exist in the component library, follow the "When a Base Component Is Missing" rule in style-ui.
- If something in my prototype seems wrong or broken (accessibility issue, missing state, etc.), ask me what to do — don't silently fix it.

## Executing Plans

Plans in `docs/plans/` are executed via `/execute-plan`. Do not auto-execute plans without the command.

## Testing (Non-Negotiable)

- Every feature, bug fix, or behavior change MUST have a corresponding test.
- TDD: write the test first, then implement. See write-tests skill.
- Never report a task as "done" without running `bin/rails test`.

## Verification (Non-Negotiable)

- NEVER claim work is "done", "passing", or "fixed" without running the verification command IN THE CURRENT MESSAGE and showing the output.
- "I believe tests pass" is not verification. Run the command and show the result.
- Before reporting a task complete: `git diff` to confirm what changed.

## Git Branch Naming Rules

- Feature work: feature/<short-description>
- Bug fix: fix/<short-description>
- Chore: chore/<short-description>
- Ticket-derived work (Linear): feature/<identifier>-<short-description>
  or fix/<identifier>-<short-description>, identifier lowercased (e.g.
  `feature/tic-123-add-account-settings`). Required for Linear's GitHub
  integration to auto-link — see Working From Tickets.

## Universal Rules

- When in doubt, write boring, obvious code. No metaprogramming, no clever abstractions, no custom DSLs.
- If something seems risky or unclear, ASK ME instead of guessing.
- Never use --force or destructive commands without explicit approval.
- Use Context7 MCP when you need library/API documentation or examples. If Context7 fails on first attempt, don't retry — tell the user and reference existing code patterns where applicable for the rest of this session instead of guessing at API usage. Exception: for information on existing UI components, use `docs/COMPONENT_CATALOG.md` — do NOT look up RailsBlocks, Flowbite, or other external component libraries.
- Stimulus for JS interactivity. No jQuery, Alpine, React, or other frameworks.
- Use `# AIDEV-NOTE:` comments for non-obvious decisions in code.

## Don't

- Don't refactor code that isn't part of the current task.
- Don't add gems without asking me first.
- Don't create migration rollback strategies unless I ask.
- Don't build "nice to have" features that aren't in the plan.
- Don't explain code changes to me unless I ask why.
- Don't build for hypothetical future requirements. Build what the plan says. If you think something WILL be needed, note and ask about it — don't build it.
- Don't duplicate logic. If the same pattern exists elsewhere in the app, extract it or reuse it. Check before creating.
- Don't commit if you're a clone. Stage changes and report complete. The master commits at the end of review-changes.
