# Backlog

Ordered by dependency. Point values mirror the scoring rubric. Each task lists its acceptance criteria
in the area files below.

| ID | Area | Task | Pts | Depends on |
|---|---|---|---|---|
| T-BE-01 | Backend | Schema + Alembic migrations | 10 | — |
| T-BE-02 | Backend | Storage abstraction (local + R2 stubs) | — | T-BE-01 |
| T-BE-03 | Backend | Artwork upload + validation | 15 | T-BE-02 |
| T-BE-04 | Backend | CRUD shows/seasons/episodes + invariants | 15 | T-BE-01 |
| T-BE-05 | Backend | Auth + roles (editor/admin) | 15 | T-BE-01 |
| T-BE-06 | Backend | Publish job (atomic, recorded, grouping) | 20 | T-BE-02..05 |
| T-BE-07 | Backend | `GET /catalog` + `/catalog/search` | 15 | T-BE-06 |
| T-BE-08 | Backend | Validation report endpoint | — | T-BE-04 |
| T-BE-09 | Backend | Tests (risky parts) | — | T-BE-03..08 |
| T-CMS-01 | CMS | List: search/filter/pagination | 15 | T-BE-04 |
| T-CMS-02 | CMS | Create/edit form + 3 artwork slots | 15 | T-BE-03/04 |
| T-CMS-03 | CMS | Publish page (report, button, history) | 15 | T-BE-06/08 |
| T-CMS-04 | CMS | Loading/empty/error/403 states | 15 | T-CMS-01..03 |
| T-VW-01 | Viewer | Home: hero + rows | 10 | T-BE-07 |
| T-VW-02 | Viewer | Search + filters + empty state | 10 | T-BE-07 |
| T-VW-03 | Viewer | Show detail + languages + hide trailers | 10 | T-BE-07 |
| T-VW-04 | Viewer | Slow-image handling | 10 | T-VW-01 |
| T-OPS-01 | Ops | docker-compose (api, db, both UIs, seed) | 10 | all build |
| T-OPS-02 | Ops | GitHub Actions (lint/test/build/deploy) | 10 | — |
| T-OPS-03 | Ops | `.env.example` + secrets write-up | 10 | — |
| T-OPS-04 | Ops | Health endpoint + alerting rationale | 10 | T-BE-07 |
| T-DOC-01 | Docs | README written reasoning (Part E) | 5 | all |

## Suggested order of execution

1. `T-BE-01` → `T-BE-02` (foundations)
2. `T-BE-05` (auth) → `T-BE-04` (CRUD) → `T-BE-03` (upload)
3. `T-BE-06` (publish) → `T-BE-07` (catalog/search) → `T-BE-08` (report)
4. `T-CMS-*` in parallel with `T-VW-*` once the matching API is up
5. `T-OPS-*` throughout; `T-DOC-01` last
