# Assets

Placeholder for rendered images and static assets referenced by the docs.

The documentation currently uses **Mermaid source** embedded in Markdown, which renders natively on
GitHub/GitLab. If you need static images (e.g. for slides or a wiki that doesn't render Mermaid),
export the diagrams here:

```bash
# example using mermaid-cli
mmdc -i ../architecture/system-architecture.md -o system-architecture.png
```

Suggested exports (one per file):

| Source | Output |
|---|---|
| `architecture/system-architecture.md` | `system-architecture.png` |
| `database/er-diagram.md` | `er-diagram.png` |
| `sequences/publish-flow.md` | `publish-flow.png` |

Keep rendered PNGs out of Git unless a specific workflow requires them (they can be regenerated from
the Mermaid source, which is the source of truth).
