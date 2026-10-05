# Lens: Design-System Drift

Compare the scope against `docs/COMPONENT_CATALOG.md` and
`app/assets/tailwind/theme/_tokens.css`. Best run on the code engine.

- **Token drift:** arbitrary Tailwind values (`w-[347px]`,
  `text-[13px]`) or hardcoded colors/spacing where a token exists.
- **Component reuse:** raw HTML or one-off partials re-implementing
  something the catalog already has.
- **Pattern divergence:** this screen solves an already-solved problem
  differently than other screens (e.g. a different confirm pattern,
  a different empty-state layout).
- **Consistency:** same word, action, or icon meaning different things
  across the scope; mixed date/number formats.

Rails hint should name the catalog component or token to use instead.
