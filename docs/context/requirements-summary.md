# Requirements Summary

Condensed from `requirements.md` (the take-home challenge spec). This is the source of truth for
scope; see [`tasks/`](../tasks/) for the actionable breakdown.

## Part A — Backend (FastAPI + PostgreSQL)

1. **Schema + migrations** — shows → seasons → episodes, artwork records, publish runs.
2. **Artwork upload** — three sizes per spec; validate aspect/dimensions + 200 KB ceiling; editor-readable
   errors; storage abstraction (local/MinIO → Cloudflare R2 = one class swap).
3. **CRUD** for shows/seasons/episodes with validation:
   - episode can't be `published` without artwork + duration
   - `(content_group, language)` unique
   - published show must have a `section`
4. **`POST /admin/catalog/publish`** — build + write catalogue: only published items, content_group
   variants collapse with `languages`, grouped by section, deterministic ordering, run recorded,
   **atomic**.
5. **`GET /catalog`** — serve the published file (or fast equivalent).
6. **`GET /catalog/search?q=&category=&language=&section=`** — `q` matches show title **and** episode
   title **and** category; all filters compose.
7. **`GET /admin/validation-report`** — everything blocking publish, grouped for editors.
8. **Roles** — `editor` (CRUD) vs `admin` (CRUD + publish), actually enforced.
9. **Tests** on the risky parts.

## Part B — Internal CMS (React + TS)

1. Show/episode list: search, filters (section, status, language), pagination.
2. Create/edit form with **three labelled artwork upload slots** — required dims, live preview,
   human-readable errors.
3. Publish page: validation report, publish button disabled *with reasons*, run history.
4. Loading / empty / error / permission-denied states.
5. TanStack Query (or stated alternative + why).

## Part C — Viewer browse UI (React + TS)

Reads **only** the published catalogue:
1. Netflix-style home: featured hero + horizontal rows by section; right artwork per surface
   (banner → hero, poster → rows, thumbnail → episode lists).
2. Search + filters (category, language) with sensible empty state.
3. Show detail: synopsis, seasons/episodes, language options for grouped episodes; trailers
   (Season 0) not shown as a normal season.
4. Pleasant while images are slow.

## Part D — Pipeline & operability

1. `docker-compose up` → API + DB + both UIs, seeded and working.
2. GitHub Actions: lint, tests, build images; deploy step written + explained (no real cloud needed).
3. `.env.example` covering every variable + secrets management paragraph.
4. Health endpoint + one thing to alert on, with reasoning.

## Part E — Written (max 1 page, in README)

1. How publishing is atomic; what happens if the process dies mid-publish.
2. Storage abstraction: what changes to move local disk → Cloudflare R2.
3. Search implementation; at what catalogue size it stops working; what next.
4. Why serve a pre-published catalogue instead of querying DB per request; where that bites.
5. What was left out and why; which AI tools used and where output was accepted/rejected.

## Scoring weights (drives priority)

| Area | Pts |
|---|---|
| Upload & validation | 15 |
| Publish job (atomic, recorded, idempotent, language grouping) | 20 |
| API design & auth | 15 |
| Data modelling | 10 |
| CMS usability | 15 |
| Viewer UI | 10 |
| Pipeline & operability | 10 |
| Written reasoning | 5 |

**Anti-patterns that count against you:** overwriting the live catalogue file; accepting artwork at
any size; roles declared but never enforced; client-side-only validation; browser-side search with no
scale comment; viewer calling admin endpoints; broken `docker-compose up`.
