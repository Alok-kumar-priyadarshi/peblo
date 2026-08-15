# Publish Activity Diagram

```mermaid
flowchart TD
    Start([Admin triggers publish]) --> Auth{require_admin?}
    Auth -- no --> Deny[403 forbidden]
    Auth -- yes --> Record[INSERT publish_runs running]
    Record --> Build[SELECT published shows/seasons/episodes/artwork]
    Build --> Collapse[Collapse content_group → languages list]
    Collapse --> Group[Group by section, deterministic order]
    Group --> Validate{all entries valid?}
    Validate -- no --> FailBuild[mark run failed + error]
    Validate -- yes --> Write[put_atomic catalogue.json]
    Write --> Version[write catalogue-version.json]
    Version --> Succeed[mark run succeeded + counts]
    Succeed --> Respond[200 run_id, version, counts]
    FailBuild --> RespondErr[500 + error payload]
    Deny --> [*]
    Respond --> [*]
    RespondErr --> [*]
```

This mirrors [`sequences/publish-flow.md`](../sequences/publish-flow.md) but emphasizes the
decision/branching structure and the failure path.
