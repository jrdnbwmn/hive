---
paths:
  - "app/views/**/*.erb"
  - "app/components/**/*"
---

# Views & Components

- Check `docs/COMPONENT_CATALOG.md` (Quick Reference table) for existing
  components before creating new ones.
- All reusable UI MUST be ViewComponents in `app/components/`, not plain
  partials in `app/views/shared/`.
- Each component: `app/components/name_component.rb` + `.html.erb`
- Keep components small and composable — `CardComponent` that accepts a
  block, not `CardWithHeaderAndFooterAndActionsComponent`
- NO `<script>` tags. NO inline `style=""`. No jQuery, Alpine.js, React, or other frameworks.
- Use Rails form helpers (`form_with`), not raw HTML forms.

# Accessibility

- Semantic HTML: `<button>` not `<div onclick>`, `<nav>`, `<main>`, `<section>`
- Every form input needs a visible `<label>` (or `aria-label` if hidden)
- Sufficient color contrast — don't rely on color alone
- Follow ARIA patterns for custom interactive elements
