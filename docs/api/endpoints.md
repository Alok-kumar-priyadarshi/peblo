# Endpoint Reference

> Full machine-readable detail lives in [`openapi.yaml`](./openapi.yaml). This is the readable guide.

## Auth

### `POST /admin/auth/login`
```jsonc
// request
{ "email": "admin@peblo.tv", "password": "secret" }
// 200
{ "access_token": "eyJ...", "token_type": "bearer", "role": "admin" }
```

## Shows

### `GET /admin/shows`
Query: `q`, `section`, `status`, `language`, `page`, `page_size`.
```jsonc
// 200
{ "items": [ { "id": "...", "title": "Moti's Many Lives", "slug": "motis-many-lives",
  "section": "featured", "status": "published", "synopsis": "..." } ],
  "total": 8, "page": 1, "page_size": 20 }
```

### `POST /admin/shows`
```jsonc
// request
{ "title": "New Show", "slug": "new-show", "synopsis": "...", "section": "series" }
// 201 → the created show resource
```

### `PATCH /admin/shows/{id}`
Accepts partial updates, e.g. `{ "status": "published" }`. Returns `422 published_requires_section`
if `section` is null.

## Seasons / Episodes

### `POST /admin/seasons`
```jsonc
{ "show_id": "...", "number": 1 }
```

### `POST /admin/episodes`
```jsonc
{ "season_id": "...", "number": 1, "title": "The Lost Kite",
  "duration_seconds": 510, "language": "en", "content_group": "motis-many-lives-s01e01",
  "status": "draft" }
// 409 content_group_language_conflict on duplicate (content_group, language)
// 422 published_requires_artwork_and_duration when status=published but incomplete
```

## Artwork

### `POST /admin/artworks` (multipart/form-data)
Fields: `file`, `kind` (`poster|banner|thumbnail`), `show_id` (poster/banner) **or** `episode_id`
(thumbnail).
```jsonc
// 201
{ "id": "...", "kind": "poster", "object_key": "artwork/poster/uuid.jpg",
  "width": 600, "height": 900, "size_bytes": 180000, "url": "https://..." }
```

## Publish

### `POST /admin/catalog/publish` — **admin only**
```jsonc
// 200
{ "run_id": "...", "version": 42, "status": "succeeded",
  "counts": { "shows": 7, "episodes": 90, "entries": 82 } }
```

### `GET /admin/catalog/runs`
```jsonc
{ "items": [ { "version": 42, "triggered_by": "admin@peblo.tv", "status": "succeeded",
  "counts": {...}, "started_at": "...", "finished_at": "..." } ], "total": 10, "page": 1 }
```

### `GET /admin/validation-report`
See [`sequences/validation-report-flow.md`](../sequences/validation-report-flow.md) for the shape.

## Catalogue (public)

### `GET /catalog`
Returns the published catalogue (see [`diagrams/catalogue-shape.md`](../diagrams/catalogue-shape.md)).
Supports `If-None-Match` / `ETag` and `Cache-Control: public, max-age=60`.

### `GET /catalog/search`
Query: `q`, `category`, `language`, `section` — all compose via AND.
```jsonc
// 200
{ "results": [ /* collapsed catalogue entries */ ], "total": 3 }
```

### `GET /health`
```jsonc
// 200
{ "status": "ok", "db": "ok", "storage": "ok" }
// 503 (any component down)
{ "status": "degraded", "db": "ok", "storage": "error" }
```
