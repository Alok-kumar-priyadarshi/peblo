# ADR-001 — Serve a pre-published catalogue file

- **Status:** accepted

## Context

The viewer needs to render rows, hero, and detail. Two options:
1. Query the DB per viewer request.
2. Publish a materialised `catalogue.json` and serve that.

The challenge explicitly asks why we serve a pre-published file rather than querying per request.

## Decision

**Serve a pre-published catalogue file** built by the publish job.

## Consequences

**Positive**
- Read path is fast, cacheable (ETag / CDN), and independent of CMS load.
- Viewer and DB are decoupled: the DB schema can evolve without touching the read contract.
- A single document is trivial to cache and easy to reason about for a child-facing UI.

**Negative (where it bites)**
- **Eventual consistency** — a publish is explicit, so viewers see content only after a run; fine
  here, but wrong for user-generated/mutable content.
- **Two sources of truth** — the DB (CMS) and the catalogue (viewer) must not drift; mitigated by a
  single publish code path and versioned runs.
- **Search** can't be done client-side over the whole catalogue at scale without cost — hence
  `GET /catalog/search` backed by SQL (see ADR-006).
