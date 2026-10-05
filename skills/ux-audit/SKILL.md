---
name: ux-audit
version: 1.1 # bump on meaningful changes
description: >
  Ranked, evidence-backed UX audit — either general (across the app or
  an area) or for one ticket/branch. Manually invoked via /ux-audit
  only. Do NOT auto-invoke.
disable-model-invocation: true
model: opus
argument-hint: <optional: "general" or a ticket ID/branch, a lens, screenshots>
---

# UX Audit

Produce a short, ranked list of UX problems the user can act on the same
day. Every finding points at evidence, names the principle it breaks,
and gives a minimal fix with a Rails implementation hint.

**User input:** $ARGUMENTS

Until the user picks "fix now" in Step 6, this skill only reports — no
edits, no fixes, no commits.

## Step 1: Pick the Audit Type and Scope

Unless the input already makes it clear, ask: **general audit, or a
specific ticket/branch?**

**Ticket/branch audit:**
- Find its design and plan docs by the `> Ticket:` / `> Branch:` header
  lines (never by filename) in `docs/designs/`, `docs/designs/done/`,
  `docs/plans/`, and `docs/plans/done/`.
- One match → confirm it with the user. Several or none → list what you
  found and ask which docs to use (or to continue without).
- Scope = the screens and flows in the design doc. Its Primary tasks
  feed the Task walkthrough.

**General audit:**
- Ask: the whole app, or a specific area?
- Build a candidate screen list from `config/routes.rb` and the main
  nav, then ask the user to confirm or trim it. Max 8 screens; list the
  rest under "Not audited."
- No design doc — ask the user for Primary tasks per screen if the Task
  walkthrough runs, or skip that lens.

**Component baseline** (see Step 3) scopes by components instead —
a named list, or all base components in `app/components/`.

Never audit beyond the confirmed scope.

## Step 2: Pick the Engine

Use the best engine the input allows. Declare it in the report header —
it sets how much the findings can be trusted.

1. **Screenshots** (user pasted images). Contrast and tap-target size
   are estimates only — "appears low," Medium confidence at most.
2. **Code** — the views, components, and Stimulus controllers for the
   scope. Catches token drift, raw HTML where a catalog component
   exists, missing labels, heading order. **Check empty, no-results,
   and error states here, not in the live app** — seeded dev data
   rarely shows them, so a live-only audit passes them by accident.
3. **Live app** in the built-in browser pane (`mcp__Claude_Browser__*`),
   against the local dev server only:
   - Use the URL the user gave, else `http://localhost:3000`. If
     nothing responds, ask the user to start the server — don't start
     it yourself.
   - If a page needs login, ask the user to sign in in the pane. Never
     enter credentials.
   - Read cheaply: `read_page` with `filter: "interactive"` for
     walkthroughs; the full tree only for layout/heading checks; a
     screenshot only for a visual-hierarchy question the user's own
     screenshots don't answer.
   - Phone width: `resize_window` with `preset: "mobile"`, then reset
     with `preset: "desktop"` before finishing.

Code and live combine well: live for what the user experiences, code
for the `file:line` to fix.

**Live-app guardrails (mandatory):**
- Read-only. Follow only plain GET links. Never click a button inside a
  form, a link with a non-GET `data-turbo-method`, or anything that
  writes data.
- Stay inside the scope; don't follow links more than one hop out.
- Max 8 pages, then stop and report.

## Step 3: Pick the Lens

If the input names a lens, use it. Otherwise default to **Task
walkthrough**. Lenses combine — merge findings with the same root cause.
Read only the file(s) for the chosen lens(es):

| Lens | Use for | File |
|---|---|---|
| **Task walkthrough** (default) | Can the user get their tasks done? | `lenses/task-walkthrough.md` |
| **Heuristic sweep** | General "what's off here?" pass | `lenses/heuristic-sweep.md` |
| **Design-system drift** | Does this match the catalog and tokens? | `lenses/design-system-drift.md` |
| **Component baseline** | Do the catalog components cover the basics? | `lenses/component-baseline.md` |

Also read, if they exist:
- `docs/product/ux-notes.md` — voice and copy standards
- The Quick Reference table in `docs/COMPONENT_CATALOG.md` — only for
  the drift or baseline lenses, or when writing Rails hints from code

## Step 4: Run the Audit

Whatever the lens, always check these (they mirror style-ui's
"Before Finishing" checklist, so the audit catches what slipped through):

- Empty, no-results, error, and (where async) loading states exist
- No dead ends — every page, modal, and error has a way forward or back
- Current location, values, and active filters are visible
- No status signaled by color alone
- No placeholder used as the only label
- Copy: one label per action; the user's words, not model/DB names;
  errors name the problem and the next step

While auditing:
- **No evidence, no finding.** Every finding points at a screen region,
  a DOM element, or a `file:line`. If you can't point at it, drop it.
- **Dedupe.** A problem on 6 screens is one finding ("on 6 screens").
- **The user's design is theirs.** If a fix would change their layout or
  visual design, phrase it as a question, not a recommendation.

## Step 5: Report

**Severity:**
- **P0 — Blocker:** stops task completion, or a compliance failure
  (keyboard trap, unreadable primary text, dead-end error)
- **P1 — Major:** significant friction or frequent confusion
- **P2 — Minor:** real but rare or easy to work around
- **P3 — Cosmetic:** polish

Report P0–P1 only, unless the user asks for P2/P3. Max 15 findings — if
there are more, say how many and offer to go deeper.

**Effort:** S (one-line / config), M (one file or component), L (several
files or a new component).

**Format:**

```
# UX Audit: <scope>
Engine: <screenshots | code | live | combination> — <what was NOT verified>
Lens: <lens(es)>

## Summary
<2–3 sentences>

## Top 3 Fixes
1. ...

## Findings
| # | Sev | Effort | Finding | Fix |
|---|-----|--------|---------|-----|

**1.** Evidence: <region / element / file:line> · Principle: <source>
Rails hint: <component, token, Stimulus controller, or file:line>

## What Works
<max 3 bullets>

## Not Audited
<screens or areas out of scope, or "None">
```

**Principle** names the source (e.g. "Nielsen #9, error recovery";
"Task walkthrough step 2, control not visible"; "Token drift").

## Step 6: Next Step

Ask both in one message, then stop and wait:

1. **What to do with the findings** — draft Linear ticket text for
   them, or fix them now? (Either, per finding, is fine.)
2. **Save this report** to `docs/audits/<scope>-<YYYY-MM-DD>.md` for
   comparing after fixes? Never save without a yes.

If they choose **fix now**: on `main`/`master`, ask them to run
`/branch` first — don't create a branch yourself. Then fix following
the write-tests and style-ui skills, in the usual small, verified
steps.

## Hard Rules

- Report only until the user picks "fix now."
- Never audit beyond the confirmed scope.
- Never propose a redesign, a new framework, or a new gem — minimal
  changes that sharpen what exists.
- Never report an estimate (screenshot contrast, tap size) as verified.
- Never write data or enter credentials in the live app.
