# Sequence — Validation Report

```mermaid
sequenceDiagram
    autonumber
    actor E as Editor/Admin
    participant CMS as CMS (Publish page)
    participant API as FastAPI
    participant DB as PostgreSQL

    E->>CMS: open Publish page
    CMS->>API: GET /admin/validation-report (Bearer)
    API->>DB: scan published (or publish-intended) content for violations
    DB-->>API: rows
    API->>API: group violations by type + show
    API-->>CMS: 200 report

    CMS->>CMS: if blocked → disable Publish button, list reasons
    CMS-->>E: "Blocked: (1) Rhyme Rangers missing section (2) ep_0036 missing artwork"
```

## Report shape

```json
{
  "publishable": false,
  "blocking": [
    {
      "type": "show_missing_section",
      "show": "Rhyme Rangers",
      "count": 1
    },
    {
      "type": "episode_missing_artwork",
      "show": "Discover India with Moti",
      "episode": "ep_0036",
      "count": 1
    }
  ],
  "warnings": [
    {
      "type": "duplicate_content_group_language",
      "content_group": "motis-many-lives-s01e02",
      "language": "hi",
      "episodes": ["ep_0004", "ep_9001"]
    }
  ]
}
```

Grouping lets an editor fix issues **without asking an engineer** — the exact requirement.
