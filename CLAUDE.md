# Global Claude Code Context

## About Me

I am a product designer who builds small Ruby on Rails SaaS apps via vibe coding. I am NOT a full-stack developer — explain the reasoning behind significant technical decisions. Ruby version manager: mise.

## How I Want You to Work

- Use a master-clone system when the work is large enough to break into smaller tasks that can be done by clones: Task(...) clones for parallelizable implementation work.
- Do NOT create custom subagents. Delegate to clones of yourself.
- Each clone task should be small and independently verifiable.
- Work in small, verifiable steps. Max ~7 files changed without checking in.
- After writing a plan, STOP. Do not call ExitPlanMode or begin implementation until I explicitly say to proceed.

### Working From Tickets (Linear)

Given a Linear ticket, the flow is: `/brainstorm` (reads the ticket and,
in its Phase 0, makes the branch name embed the identifier) → `/write-plan`
→ `/execute-plan` → `/review-changes` → `/wrap-up` → `/close-out`.

Rules that hold throughout, regardless of which commands I run:

- The ticket identifier MUST appear in the branch name (see Git Branch
  Naming Rules) — that's what lets Linear's GitHub integration auto-link
  the branch and PR.
- Never manually set the ticket's status or paste the branch/PR back into
  Linear. The GitHub integration does both automatically once the
  identifier is in the branch name.
- I review and merge PRs myself on GitHub — Claude never merges ticket
  work. After a PR is merged, run `/close-out` again (any thread) to sync
  main, delete the branch, and archive the design/plan docs.

### Working From My Prototypes

When I provide an HTML page or un-wired Rails view as a starting point:

- My layout, visual hierarchy, and design decisions are FINAL. Do not deviate from it. Do not redesign, rearrange, or "improve" the visual design.
- Your job: upgrade the implementation to production Rails (real data, catalog components, Stimulus, routes, tests) — see the style-ui skill for the step-by-step. Ask if anything's unclear.
- If a component in my prototype doesn't exist in the component library, follow the "When a Base Component Is Missing" rule in style-ui.
- If something in my prototype seems wrong or broken (accessibility issue, missing state, etc.), ask me what to do — don't silently fix it.

## Executing Plans

Plans in `docs/plans/` are executed via `/execute-plan`.

## Testing & Verification (Non-Negotiable)

- Every feature, bug fix, or behavior change MUST have a corresponding test. TDD: test first, then implement (see write-tests skill).
- Test names describe behavior from the user's perspective, not the method under test (e.g. "user can subscribe to a plan", not "subscription create").
- NEVER claim work is "done", "passing", or "fixed" without running the check (`bin/rails test`) IN THE CURRENT MESSAGE and showing the output. "I believe tests pass" is not verification.
- Before reporting a task complete: `git diff` to confirm what changed.

## Git Branch Naming Rules

- <type>/<slug>
- Ticket-derived work (Linear): <type>/<identifier>-<short-description>, identifier lowercased (e.g.`feature/tic-123-add-account-settings`).
- Type = `feature` (feature work), `fix` (bug fix), or `chore`.
- Slug = short, kebab-case description.

## Universal Rules

- When in doubt, write boring, obvious Rails code — default to the simplest Rails-conventional approach. No metaprogramming, no clever abstractions, no custom DSLs.
- My apps use Jumpstart Pro (JSP). Check for existing JSP functionality before building — especially auth, billing, notifications, and admin.
- If something seems risky or unclear, ASK ME instead of guessing.
- Never use --force or destructive commands without explicit approval.
- Use Context7 MCP for library/API docs; if it fails once, don't retry — tell me and lean on existing code patterns instead of guessing. Exception: for UI components use `docs/COMPONENT_CATALOG.md`, not external docs; for a new component, /create-component checks RailsBlocks first, building from scratch only if it's missing.
- Stimulus for JS interactivity. No jQuery, Alpine, React, or other frameworks.
- Use `# AIDEV-NOTE:` comments for non-obvious decisions in code.

## Don't

- Don't refactor code that isn't part of the current task.
- Don't add gems without asking me first.
- Build what the plan says — no "nice to haves," no building for hypothetical future needs. If you think something WILL be needed, note it and ask; don't build it.
- Don't narrate routine code changes unless I ask (explaining a significant decision is still welcome).
- Don't duplicate logic. If the same pattern exists elsewhere in the app, extract it or reuse it. Check before creating.
- Don't commit if you're a clone. Stage changes and report complete. The master commits at the end of review-changes.
