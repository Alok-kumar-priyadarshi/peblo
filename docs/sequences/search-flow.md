# Sequence — Search & Filters

```mermaid
sequenceDiagram
    autonumber
    actor V as Viewer
    participant UI as Viewer UI
    participant API as FastAPI
    participant DB as PostgreSQL

    V->>UI: type "moti" + filter language=hi, category=adventure
    UI->>API: GET /catalog/search?q=moti&language=hi&category=adventure&section=featured
    API->>API: validate params (enum membership, length)
    API->>DB: SELECT ... WHERE (show_title ILIKE %moti% OR episode_title ILIKE %moti% OR category ILIKE %moti%) AND language='hi' AND category='adventure' AND section='featured'
    DB-->>API: matching rows (collapsed by content_group)
    API-->>UI: 200 {results: [...]}
    alt no results
        UI-->>V: "No shows match — try clearing a filter"
    else results
        UI-->>V: render results
    end
```

## Implementation & scale

- **V1:** SQL `ILIKE` over `shows.title`, `episodes.title`, and category, with all filters composed
  via SQL `AND`. Correct for the seed's ~95 episodes and fine well into the thousands.
- **When it stops working:** `ILIKE %…%` cannot use a B-tree index, so latency grows linearly. It
  becomes inadequate around tens of thousands of rows / sustained query load.
- **What next (documented, not built):** add PostgreSQL full-text search (`tsvector` + GIN index) for
  `q`, and materialise the catalogue into a search index (or offload to a dedicated search engine)
  once the catalogue is large or latency-sensitive. Filters remain ordinary indexed columns.
