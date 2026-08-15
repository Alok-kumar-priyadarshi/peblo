# Part B — CMS Tasks

## T-CMS-01 — List page
- **Acceptance:** show/episode list with search, filters (section, status, language), and pagination;
  empty and error states handled.
- **Reference:** [`architecture/frontend-architecture.md`](../architecture/frontend-architecture.md).

## T-CMS-02 — Create/edit form + artwork slots
- **Acceptance:** three labelled slots (poster 2:3 600×900, banner 16:9 1280×720, thumbnail 16:9
  640×360) each showing required dimensions, a **live preview**, and human-readable errors mapped from
  API error codes. Editor can act on errors unaided.
- **Reference:** [`sequences/upload-flow.md`](../sequences/upload-flow.md).

## T-CMS-03 — Publish page
- **Acceptance:** renders the validation report; publish button is **disabled with reasons** when
  blocked; run history table shown. Uses `POST /admin/catalog/publish` + `GET /admin/catalog/runs`.
- **Reference:** [`sequences/validation-report-flow.md`](../sequences/validation-report-flow.md).

## T-CMS-04 — State handling
- **Acceptance:** loading skeletons, empty states, inline errors, and a permission-denied (403) screen;
  admin-only actions hidden for editors. TanStack Query used for server state (or alternative stated +
  justified).
- **Reference:** [`architecture/frontend-architecture.md`](../architecture/frontend-architecture.md).
