# Part A — Backend Tasks

## T-BE-01 — Schema + migrations
- **Acceptance:** `alembic upgrade head` creates `users`, `shows`, `seasons`, `episodes`, `artworks`,
  `publish_runs` matching [`database/schema.md`](../database/schema.md). Fresh DB + upgrade from
  empty works. Constraints (unique content_group+language, artwork owner CHECK) present.
- **Reference:** [`database/er-diagram.md`](../database/er-diagram.md), [`database/migrations.md`](../database/migrations.md).

## T-BE-02 — Storage abstraction
- **Acceptance:** `StorageBackend` ABC with `LocalDiskStorage` implemented and `CloudflareR2Storage`
  stubbed; `put_atomic` writes temp then renames; a `STORAGE_BACKEND` env var selects the impl.
- **Reference:** [`architecture/storage-abstraction.md`](../architecture/storage-abstraction.md).

## T-BE-03 — Artwork upload + validation
- **Acceptance:** `POST /admin/artworks` validates aspect (2:3 / 16:9), target px, and ≤200 KB for the
  three kinds; returns `code` + human `message` on failure; stores object + DB row. Tests prove wrong
  aspect, oversize, and unsupported type are rejected.
- **Reference:** [`sequences/upload-flow.md`](../sequences/upload-flow.md), [`api/errors.md`](../api/errors.md).

## T-BE-04 — CRUD + invariants
- **Acceptance:** shows/seasons/episodes CRUD; publishing an episode without artwork+duration → 422;
  duplicate `(content_group, language)` → 409; publishing a show without `section` → 422.
- **Reference:** [`sequences/crud-flow.md`](../sequences/crud-flow.md).

## T-BE-05 — Auth + roles
- **Acceptance:** login returns JWT; `require_admin` blocks editor from publish/run-history with 403.
  Tests cover editor-vs-admin and expiry.
- **Reference:** [`api/auth.md`](../api/auth.md), [`sequences/auth-flow.md`](../sequences/auth-flow.md).

## T-BE-06 — Publish job
- **Acceptance:** `POST /admin/catalog/publish` (admin) builds catalogue with only published content,
  collapses content_groups into `languages`, groups by section in deterministic order, writes
  **atomically**, and records the run (who/when/counts/outcome). Idempotent re-publish yields identical
  output.
- **Reference:** [`sequences/publish-flow.md`](../sequences/publish-flow.md),
  [`diagrams/publish-activity.md`](../diagrams/publish-activity.md).

## T-BE-07 — Catalog + search
- **Acceptance:** `GET /catalog` serves the published file (ETag + cache headers); `GET /catalog/search`
  composes `q` (show/episode/category) with `category`, `language`, `section`.
- **Reference:** [`sequences/browse-flow.md`](../sequences/browse-flow.md),
  [`sequences/search-flow.md`](../sequences/search-flow.md).

## T-BE-08 — Validation report
- **Acceptance:** `GET /admin/validation-report` returns `{publishable, blocking[], warnings[]}` with
  `Rhyme Rangers` (missing section) and `ep_0036` (missing artwork) surfaced from seed data.
- **Reference:** [`sequences/validation-report-flow.md`](../sequences/validation-report-flow.md).

## T-BE-09 — Tests
- **Acceptance:** pytest covers publish atomicity, content_group collapse, artwork validation, roles,
  and seed-data edge cases. Integration tests run against a real Postgres (service container in CI).
- **Reference:** [`standards/testing-standards.md`](../standards/testing-standards.md).
