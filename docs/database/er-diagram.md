# ER Diagram

```mermaid
erDiagram
    users ||--o{ publish_runs : "triggers"

    shows ||--|{ seasons : "has"
    seasons ||--|{ episodes : "has"
    shows ||--o{ artworks : "poster/banner"
    episodes ||--o{ artworks : "thumbnail"

    users {
        uuid      id            PK
        text      email         UK "unique"
        text      password_hash
        user_role role          "editor|admin"
        timestamptz created_at
        timestamptz updated_at
    }

    shows {
        uuid         id        PK
        text         title
        text         slug      UK "unique"
        text         synopsis
        text         section   "nullable; validated vs reference.json"
        content_status status  "draft|published"
        timestamptz  created_at
        timestamptz  updated_at
    }

    seasons {
        uuid        id         PK
        uuid        show_id    FK "→ shows.id"
        int         number     "0 = trailers"
        timestamptz created_at
    }

    episodes {
        uuid         id               PK
        uuid         season_id        FK "→ seasons.id"
        int          number
        text         title
        text         synopsis
        int          duration_seconds "nullable until ready"
        text         language         "en|hi"
        text         content_group
        content_status status         "draft|published"
        timestamptz  created_at
        timestamptz  updated_at
    }

    artworks {
        uuid         id          PK
        uuid         show_id     FK "nullable; poster/banner"
        uuid         episode_id  FK "nullable; thumbnail"
        artwork_kind kind        "poster|banner|thumbnail"
        text         object_key  "storage path"
        int          width
        int          height
        int          size_bytes
        text         mime_type
        text         sha256
        timestamptz  created_at
    }

    publish_runs {
        uuid          id           PK
        int           version      UK "monotonic"
        uuid          triggered_by FK "→ users.id"
        publish_status status      "running|succeeded|failed"
        jsonb         counts       "shows/episodes/entries"
        text          catalog_key  "storage key"
        text          error
        timestamptz   started_at
        timestamptz   finished_at
    }
```

## Relationship notes

- **shows 1—N seasons**: a season belongs to exactly one show.
- **seasons 1—N episodes**: an episode belongs to exactly one season (and thus one show).
- **artworks**: polymorphic-ish — exactly one of `show_id`/`episode_id` is set (enforced by a CHECK),
  so `poster`/`banner` hang off a show and `thumbnail` off an episode.
- **content_group**: *not* a physical table in v1 — it's a logical grouping key on `episodes` that
  the publish job collapses into a `languages` list. (It appears in the domain diagram for clarity.)
- **publish_runs → users**: every run records who triggered it.
