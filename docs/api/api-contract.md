# API Contract Overview

## Endpoint index

| Method | Path | Auth | Role | Description |
|---|---|---|---|---|
| POST | `/admin/auth/login` | — | — | Exchange credentials for a JWT |
| GET | `/admin/shows` | Bearer | editor | List shows (search/filter/paginate) |
| POST | `/admin/shows` | Bearer | editor | Create show |
| GET | `/admin/shows/{id}` | Bearer | editor | Get show (with seasons/episodes) |
| PATCH | `/admin/shows/{id}` | Bearer | editor | Update show |
| DELETE | `/admin/shows/{id}` | Bearer | editor | Delete show |
| POST | `/admin/seasons` | Bearer | editor | Create season |
| POST | `/admin/episodes` | Bearer | editor | Create episode |
| PATCH | `/admin/episodes/{id}` | Bearer | editor | Update episode |
| DELETE | `/admin/episodes/{id}` | Bearer | editor | Delete episode |
| POST | `/admin/artworks` | Bearer | editor | Upload artwork (multipart) |
| DELETE | `/admin/artworks/{id}` | Bearer | editor | Delete artwork |
| POST | `/admin/catalog/publish` | Bearer | **admin** | Build + atomically publish catalogue |
| GET | `/admin/catalog/runs` | Bearer | admin | Publish run history |
| GET | `/admin/validation-report` | Bearer | editor | Everything blocking publish |
| GET | `/catalog` | — | — | The published catalogue |
| GET | `/catalog/search` | — | — | Search + filters |
| GET | `/health` | — | — | Liveness/readiness |

## Shared request/response conventions

- List endpoints accept `?q=&section=&status=&language=&page=&page_size=` and return
  `{items: [...], total, page, page_size}`.
- Mutations return the created/updated resource (or `204` on delete).
- Validation errors return `422` with a machine `code` + human `message` (see [`errors.md`](./errors.md)).

## The two audiences

- **`/admin/*`** → the CMS. Authenticated, role-gated, full CRUD + publish.
- **`/catalog*`, `/health`** → the Viewer. Anonymous, read-only, served from the published catalogue.

The viewer must **never** call `/admin/*`; the CMS must **never** call `/catalog/*` for editing
purposes.
