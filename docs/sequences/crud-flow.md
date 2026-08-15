# Sequence — CRUD with Validation

Creating/updating an episode exercises three invariants. Shows and seasons follow the same pattern.

```mermaid
sequenceDiagram
    autonumber
    actor E as Editor
    participant CMS as CMS
    participant API as FastAPI
    participant Svc as EpisodeService
    participant DB as PostgreSQL

    E->>CMS: save episode (title, duration, language, content_group, status)
    CMS->>API: POST /admin/episodes
    API->>Svc: create(dto)
    Svc->>Svc: validate status rules
    alt status == published AND (no artwork OR no duration)
        Svc-->>API: 422 published_requires_artwork_and_duration
        API-->>CMS: error
        CMS-->>E: "Publish requires artwork and a duration."
    end
    Svc->>DB: check UNIQUE (content_group, language)
    alt duplicate
        DB-->>Svc: unique violation
        Svc-->>API: 409 content_group_language_conflict
        API-->>CMS: error
        CMS-->>E: "This episode already exists in Hindi."
    else ok
        Svc->>DB: INSERT ... RETURNING
        DB-->>Svc: row
        Svc-->>API: 201 episode
        API-->>CMS: 201
        CMS-->>E: saved
    end
```

## Show-level rule

When publishing a **show**, `section` must be set:

```mermaid
sequenceDiagram
    participant CMS as CMS
    participant API as FastAPI
    participant DB as PostgreSQL
    CMS->>API: PATCH /admin/shows/{id} {status:"published"}
    API->>API: validate section != null
    alt section is null
        API-->>CMS: 422 published_requires_section
    else ok
        API->>DB: UPDATE shows SET status='published'
        DB-->>API: ok
        API-->>CMS: 200
    end
```
