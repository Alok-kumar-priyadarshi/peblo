# Testing Standards

## What to test (the "risky parts")

Per the challenge, focus tests where bugs are most costly:

| Area | Must-have tests |
|---|---|
| Publish | atomic write (no half file on failure), content_group collapse, deterministic order, run recorded on success **and** failure, idempotency |
| Artwork | wrong aspect rejected, oversize rejected, unsupported type rejected, valid accepted |
| Roles | editor → 403 on publish, admin → 200, expired/invalid token → 401 |
| CRUD invariants | publish without artwork/duration → 422, duplicate content_group+language → 409, publish show without section → 422 |
| Seed | duplicate content_group+language quarantined; Season 0 imported; null section preserved |
| Search | filters compose; `q` matches show title, episode title, and category |

## Structure

- **Backend:** `pytest` + `httpx` (FastAPI TestClient) + `pytest-asyncio`. Integration tests run
  against a real Postgres (service container in CI); unit tests use fakes for `StorageBackend`.
- **Frontend:** Vitest + React Testing Library. Test behaviour, not implementation — assert on
  rendered output and user interactions.

## Conventions

- Name tests `test_<behaviour>` describing the *outcome*, not the function.
- Use the "arrange / act / assert" shape.
- Keep tests deterministic (no wall-clock flakiness; inject clocks where needed).
- CI must run the full suite; a failing test blocks merge.

## Coverage guidance

Aim for high coverage on the "risky parts" table above, not blanket 100%. Low-value coverage (getters,
rendering-only components) is fine to skip — prioritise the graded invariants.
