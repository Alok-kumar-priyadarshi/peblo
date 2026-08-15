# Review Checklist

Use before merge. Items marked ⚠️ map directly to the challenge's "counts against you" list.

## Correctness & invariants

- [ ] ⚠️ Catalogue is written **atomically** (no overwrite-in-place of the live file).
- [ ] ⚠️ Artwork is validated for aspect + dimensions + 200 KB — never accepted at any size.
- [ ] ⚠️ Roles (`editor` vs `admin`) are enforced server-side, not just declared.
- [ ] ⚠️ Validation is server-side; the UI is not the only gate.
- [ ] `(content_group, language)` uniqueness is enforced and tested.
- [ ] `content_group` variants collapse into one entry with a `languages` list.
- [ ] Season 0 (trailers) excluded from normal season rendering.
- [ ] Publish run is always recorded (success **and** failure).

## API

- [ ] ⚠️ Viewer never calls `/admin/*`; admin never leaks to public routes.
- [ ] Errors use the uniform envelope (`code` + `message`); messages are editor-readable.
- [ ] Filters compose correctly in search.

## Pipeline

- [ ] ⚠️ `docker-compose up` works from a clean checkout.
- [ ] CI runs lint + tests + build; no secrets in the workflow.
- [ ] `.env.example` covers every variable; no real secrets committed.

## Quality

- [ ] Tests cover the risky parts (publish atomicity, upload validation, roles).
- [ ] Lint/type checks pass (ruff, eslint, tsc).
- [ ] Docs updated when a contract changed.

## Written reasoning

- [ ] README answers Part E (atomicity, storage swap, search scale, catalogue-vs-DB, omissions + AI
  tool usage) within ~1 page.
