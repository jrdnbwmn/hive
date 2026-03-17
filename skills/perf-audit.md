---
name: perf-audit
version: 1.0 # bump on meaningful changes
description: >
  Invoked by the /perf-audit command.
  Do NOT auto-invoke for any other situation.
model: sonnet
---

# Skill: Performance Audit

## Purpose
Perform a thorough, multi-layered performance audit of a Rails + Hotwire + Tailwind v4 + Jumpstart Pro application. Return ALL findings — no filtering by severity. The developer will triage.

## Inputs
- **Target scope** (optional): A path, feature area, or model name (e.g., `app/models/user.rb`, `accounts`, `dashboard`). If blank, audit the entire app.

## Process

### Step 1: Scope Resolution
- If a target scope is given, identify all related files: models, controllers, views, partials, Stimulus controllers, routes, tests, and migrations that touch that area.
- If no scope, work directory-by-directory through the full app.
- Output the resolved file list before proceeding.

### Step 2: Run All Audit Passes
Execute EVERY pass below against the resolved scope. Do not skip any pass. For each finding, record:
- **File + line** (be specific)
- **Category** (from the pass name)
- **Issue** (what's wrong)
- **Suggestion** (concrete fix or approach)
- **Impact estimate** (high / medium / low)

---

## Audit Passes

### Pass 1: Database & ActiveRecord
- [ ] N+1 queries: trace controller actions → view renders, flag any association accessed in a loop without `includes`/`preload`/`eager_load`
- [ ] Missing indexes: check `db/schema.rb` — every `foreign_key`, `_id` column, and column used in `where`, `order`, `group`, `find_by`, or `pluck` should be indexed
- [ ] Compound indexes: look for queries filtering on multiple columns that would benefit from a composite index
- [ ] Unnecessary `SELECT *`: flag scopes/queries that load full records when only a few columns are needed — suggest `.select()` or `.pluck()`
- [ ] Counter cache candidates: any `has_many` where `.count` or `.size` is called in views
- [ ] Expensive scopes: flag `default_scope`, scopes with subqueries, or scopes chaining many conditions
- [ ] Unscoped pagination: any `.all` or large query without `.limit` / pagination (especially in index actions)
- [ ] Raw SQL review: check for SQL injection risk and optimization opportunities in any `find_by_sql`, `execute`, or `Arel` usage
- [ ] Schema bloat: flag unused columns or tables (cross-reference with model/controller usage)
- [ ] Transaction misuse: flag unnecessarily broad transactions or missing transactions where atomicity is needed

### Pass 2: Controller & Request Lifecycle
- [ ] Fat controllers: flag actions over ~15 lines — suggest extraction to service objects
- [ ] Redundant queries: same data queried in `before_action` and again in the action
- [ ] Missing pagination on index actions
- [ ] Unnecessary `before_action` scope: callbacks that run on actions that don't need them
- [ ] Response format bloat: actions responding to formats (JSON, HTML, etc.) they don't need to
- [ ] Missing `stale?`/`fresh_when` for conditional GET (ETags/Last-Modified)
- [ ] Heavy work in request cycle: anything that should be a background job (emails already handled by JSP, but check for other slow operations like API calls, report generation, file processing)

### Pass 3: Views & Rendering
- [ ] Partial render count: flag any partial rendered 10+ times per page — suggest collection rendering or ViewComponent
- [ ] Collection rendering: any `each` loop rendering a partial should use `render collection:` syntax
- [ ] Nested partials: partials rendering other partials more than 2 levels deep
- [ ] ViewComponent opportunities: repeated UI patterns that aren't yet components
- [ ] Excessive DOM size: estimate node count for complex pages (especially tables/lists without pagination)
- [ ] Inline Ruby computation in views: any non-trivial logic that should be in a helper, presenter, or component
- [ ] Missing `local_assigns` checks for optional partial locals

### Pass 4: Hotwire & Turbo
- [ ] Turbo Frame targeting: frames loading more content than needed — can the frame scope be narrower?
- [ ] Missing Turbo Frame lazy loading (`loading: :lazy`) for below-the-fold content
- [ ] Full-page Turbo visits that could be Turbo Frame updates
- [ ] Turbo Stream broadcasts: broadcasting more data than necessary, or broadcasting to too-wide a channel
- [ ] Missing `turbo_frame_tag` on forms that should submit within a frame
- [ ] Stimulus controllers loading/initializing heavy resources on `connect()` that could be deferred

### Pass 5: Caching
- [ ] Missing fragment caching: any view partial or component rendering database-backed content without `cache` blocks
- [ ] Russian doll caching opportunities: nested cached content where outer cache could wrap inner caches
- [ ] Cache key staleness: cache keys that don't include `updated_at` or a proper cache version
- [ ] Missing HTTP caching headers on static-ish pages
- [ ] Cacheable database queries: expensive queries with results that don't change often — suggest low-level `Rails.cache.fetch`
- [ ] Missing `touch: true` on `belongs_to` for cache invalidation chains

### Pass 6: Assets & Frontend
- [ ] Tailwind v4 output size: check for custom CSS that duplicates Tailwind utilities
- [ ] Unused CSS: classes defined in custom stylesheets but not referenced in views/components
- [ ] JavaScript bundle size: large Stimulus controllers or imported libraries that could be lazy-loaded
- [ ] Image optimization: images served without proper sizing, missing `loading="lazy"`, or not using modern formats (webp/avif)
- [ ] Font loading: font files blocking render — should use `font-display: swap` or be preloaded
- [ ] Unnecessary JS: functionality that could be pure Turbo/HTML instead of Stimulus

### Pass 7: Background Jobs & Async
- [ ] Synchronous work that should be async: API calls, email-adjacent work, report generation, bulk operations
- [ ] Job performance: jobs doing N+1 queries or loading too much data
- [ ] Missing job batching: many small jobs that could be one batch
- [ ] Queue priority: all jobs on default queue vs. proper priority queues

### Pass 8: Memory & Ruby Performance
- [ ] Large object allocation: building big arrays/hashes in memory when streaming or batching would work
- [ ] `find_each` / `in_batches` not used for large record iterations
- [ ] String concatenation in loops (suggest `StringIO` or `Array#join`)
- [ ] Memoization opportunities: methods called multiple times that recompute the same result
- [ ] Unnecessary object creation: e.g., `.map { ... }.select { ... }` chains that could be `.filter_map`

### Pass 9: Security-Adjacent Performance
- [ ] Denial-of-service vectors: endpoints accepting unbounded input (file uploads, search without limits, bulk operations without caps)
- [ ] Rate limiting: public endpoints without throttling
- [ ] Expensive unauthenticated endpoints

### Pass 10: Jumpstart Pro Specific
- [ ] JSP feature usage: using JSP's built-in features (multitenancy scoping, notifications, etc.) vs. rolling custom versions that may be less optimized
- [ ] Pay/Accounts scoping: queries not properly scoped to the current account (loading cross-tenant data unnecessarily)
- [ ] JSP defaults: any JSP default configurations that should be tuned for production (e.g., caching backend, job adapter)

---

## Step 3: Output Format

Produce a single report organized as follows:

### Summary
- Total findings count by severity (high / medium / low)
- Top 5 highest-impact findings as a numbered list

### Findings by Category
One section per Audit Pass. Within each section, list findings in a table:

| # | File:Line | Issue | Suggestion | Impact |
|---|-----------|-------|------------|--------|

### Quick Wins
List any findings that are:
- High impact AND low effort (one-line or config-only changes)
- e.g., adding an index, adding `includes()`, enabling fragment caching

### Needs Investigation
List any areas where you suspect a performance issue but can't confirm without runtime data (e.g., rack-mini-profiler, production logs, actual query plans). State exactly what data would be needed.

---

## Rules
- Be exhaustive. Surface EVERYTHING, even minor issues. The developer will triage.
- Be specific. File names, line numbers, exact code references.
- Don't implement fixes — only report findings and suggest approaches.
- If scope is large, work through it systematically file-by-file. Don't skip files.
- When uncertain whether something is an issue, include it in "Needs Investigation" rather than omitting it.
