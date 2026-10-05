# Lens: Component Baseline

Audit the catalog components themselves (`app/components/`), not a
screen. Best run on the code engine, optionally with Lookbook previews
in the live app.

For each in-scope component, check:

- **Interactive states:** buttons, links, inputs, and other controls
  define hover, focus-visible, active, disabled, and (where relevant)
  loading/pending. A removed focus outline must be replaced with
  something equally visible.
- **Double-submit:** Turbo disables the submitter during form
  submission, so check the gaps — components used in forms with
  `data-turbo="false"` or buttons that run a Stimulus `fetch` must
  disable themselves. Pending text via `data-turbo-submits-with` is a
  nice-to-have.
- **Tap targets:** interactive elements are at least 44×44px, or have
  padding that gets them there.
- **Labels:** form components require or render a visible `<label>`;
  placeholder is never the only label. Icon-only buttons have an
  accessible name.
- **Semantics:** real `<button>`, `<a>`, `<label>`, `<table>` — no
  clickable `<div>`s.
- **Color-only status:** badges/alerts pair color with text or an icon.
- **Contrast:** token color pairs used for text (text-on-background,
  text-on-button, muted/secondary text) meet WCAG 2.2 AA — 4.5:1 for
  body text, 3:1 for large text and UI boundaries.
  - Don't calculate ratios in your head — Tailwind v4 tokens are often
    OKLCH, and mental conversion is unreliable. Write a throwaway
    script in your scratchpad (never in the repo) that converts the
    token values to sRGB and computes the WCAG ratio, and report its
    output.
  - If you can't run a script, report contrast as an estimate (Medium
    confidence), never as computed.

Group findings by component. Fixing here fixes every screen that uses
the component, so weight severity by how widely the component is used.
