# Git Workflow

- **Branches:** short-lived off `main`; names `feat/…`, `fix/…`, `docs/…`, `chore/…`.
- **Commits:** Conventional Commits, e.g. `feat(api): atomic publish`.
- **PRs:** one logical change; CI (lint + test + build) must pass; squash-merge to `main`.
- **Never commit:** secrets, `.env`, build artifacts, node_modules, large binaries.
- **Docs in sync:** endpoint change → update [`api/`](../api/); schema change → update
  [`database/`](../database/).

Full detail in [`instructions/contribution.md`](../instructions/contribution.md).
