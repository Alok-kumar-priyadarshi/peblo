# Contribution Workflow

## Branching

- Work off `main` on a short-lived feature branch named `feat/…`, `fix/…`, `docs/…`, `chore/…`.
- One logical change per PR.

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(api): add atomic catalogue publish job
fix(cms): surface artwork aspect-ratio errors
docs(database): document content_group collapse
```

## Pull request

1. Run lint + tests locally (see [`review-checklist.md`](./review-checklist.md)).
2. Open the PR; CI must pass (lint, test, build).
3. Keep the contract docs in sync — if you change an endpoint, update [`api/`](../api/); if you change
   a table, update [`database/`](../database/).
4. Request review; address feedback; squash-merge.

## Never commit

- Secrets / `.env`
- Built artifacts (`dist/`, `node_modules/`, `__pycache__/`)
- Large binaries or datasets (artwork belongs in storage, not Git)
