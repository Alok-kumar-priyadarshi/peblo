# Glossary

A shared vocabulary for the codebase, docs, and API. Prefer these exact terms in identifiers and
commit messages to keep the system self-consistent.

| Term | Definition | Where it lives |
|---|---|---|
| **CMS** | Internal Content Management System (React) used by editors/admins. | `cms/` |
| **Viewer** | Public-facing browse UI (React) that reads only the published catalogue. | `viewer/` |
| **API** | FastAPI service exposing admin + catalog endpoints. | `backend/` |
| **Catalogue** | The single published JSON document the viewer consumes. | storage: `catalogue.json` |
| **Catalogue entry** | One collapsed show/episode record in the catalogue (content_group variants merged). | catalogue JSON |
| **Publish** | The job that builds + atomically writes the catalogue. | `backend/publish/` |
| **Publish run** | A single recorded execution of publish (who/when/counts/outcome). | `publish_runs` table |
| **Storage abstraction** | Interface behind which local disk, MinIO, or Cloudflare R2 live. | `backend/storage/` |
| **object key** | Storage path/identifier for a stored blob (e.g. `artwork/{uuid}-poster.jpg`). | `artworks.object_key` |
| **Artwork kind** | One of `poster`, `banner`, `thumbnail` with fixed aspect/target px. | `reference.json` |
| **Section** | Top-level browse row: `featured`, `series`, `minisodes`, `songs`. | `reference.json` |
| **Category** | Editorial tag; 15 allowed values. | `reference.json` |
| **content_group** | Key shared by language variants of the same episode. | `episodes.content_group` |
| **Language variant** | An episode sharing a content_group but differing in `language`. | `episodes.language` |
| **Validation report** | Grouped list of everything blocking publish, for editors. | `GET /admin/validation-report` |
| **Role** | `editor` (CRUD) or `admin` (CRUD + publish). | `users.role` |
| **Draft** | Content status: not yet publishable. | `content_status` |
| **Published** | Content status: eligible to appear in the catalogue. | `content_status` |
| **Trailer** | A Season-0 item, excluded from normal season views. | `seasons.number = 0` |
| **Atomic write** | Temp file + rename (or storage conditional put) so readers never see partial data. | publish job |
| **Catalogue version** | Monotonic integer per publish run, used for rollback/audit. | `publish_runs.version` |
