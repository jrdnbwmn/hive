# Git Conventions

- Commit messages: imperative mood, ~50-char subject, blank line, then body if needed.
- Prefix: feature: / fix: / docs: / refactor: / style: / chore: / test:
- One logical change per commit. Don't bundle unrelated changes.
- Before committing: review staged changes for debug artifacts,
  secrets, and TODOs. Remove them from the commit.
- Never commit .env, credentials, or API keys.
- Never force-push to main.
- Branch naming: `feature/short-description` or `fix/short-description`.
