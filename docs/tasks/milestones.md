# Milestones & Definition of Done

## Milestones

| Milestone | Tasks | Exit criterion |
|---|---|---|
| **M1 — Foundations** | T-BE-01, T-BE-02, T-BE-05 | Schema migrates clean; storage abstracted; auth enforced |
| **M2 — Write path** | T-BE-03, T-BE-04, T-BE-08 | Editor can CRUD + upload with correct validation errors |
| **M3 — Publish + read path** | T-BE-06, T-BE-07 | Atomic publish works; catalogue + search served |
| **M4 — UIs** | T-CMS-*, T-VW-* | Editor can work unaided; child can browse |
| **M5 — Operability + docs** | T-OPS-*, T-DOC-01 | `docker-compose up` works; CI green; README reasoning written |

## Definition of Done (every task)

- [ ] Code implements the acceptance criteria.
- [ ] Validation is server-side (not client-only).
- [ ] Relevant tests added and passing (see [`standards/testing-standards.md`](../standards/testing-standards.md)).
- [ ] Lint/type checks pass (ruff, eslint, tsc).
- [ ] Documentation touched if the contract changed ([`api/`](../api/), [`database/`](../database/)).
- [ ] No secrets or binaries committed.

## "Good 70% over rushed 100%" prioritization

Per the challenge's guidance, if time runs short, cut in this order and say so in the README:
1. Optional stretch (versioned rollback, dry-run diff, audit log).
2. Polish/styling.
3. Secondary filters in the Viewer.
Never cut: atomic publish, artwork validation, role enforcement, or a working `docker-compose up` —
those are explicitly graded against you if wrong.
