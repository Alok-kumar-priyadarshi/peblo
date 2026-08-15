# Error Model

All errors use one envelope so the CMS and Viewer can render consistently:

```json
{
  "error": {
    "code": "published_requires_section",
    "message": "A published show must have a section.",
    "details": { "show_id": "..." }   // optional, machine-readable
  }
}
```

- `code` is a stable, snake_case string (safe to branch on in code).
- `message` is human-readable and editor-actionable.
- `details` is optional structured context.

## HTTP status mapping

| Status | Meaning |
|---|---|
| 400 | Malformed request (e.g. bad filter value) |
| 401 | Missing/invalid/expired token |
| 403 | Authenticated but insufficient role |
| 404 | Resource not found |
| 409 | Conflict (e.g. duplicate content_group+language) |
| 413 | Upload exceeds size limit |
| 422 | Validation failed (invariants, artwork specs) |
| 500 | Unexpected server error (no internals leaked) |

## Error codes

| Code | Status | When |
|---|---|---|
| `invalid_credentials` | 401 | Login failed |
| `token_expired` / `token_invalid` | 401 | Bad JWT |
| `insufficient_permissions` | 403 | editor tried an admin action |
| `not_found` | 404 | Missing resource |
| `content_group_language_conflict` | 409 | Duplicate `(content_group, language)` |
| `published_requires_section` | 422 | Publishing a show with null section |
| `published_requires_artwork_and_duration` | 422 | Publishing an episode missing artwork/duration |
| `aspect_ratio` | 422 | Artwork wrong aspect ratio |
| `size_exceeded` | 413 | Artwork over 200 KB |
| `unsupported_type` | 422 | Artwork not a supported image |
| `decode_failed` | 422 | Artwork unreadable/corrupt |
| `invalid_section` / `invalid_category` / `invalid_language` | 400 | Filter/enum value out of `reference.json` |
| `publish_failed` | 500 | Catalogue build/write failed (message explains) |

## Editor-readability rule

Every `422`/`413` on the upload path **must** carry a `message` an editor can act on unaided:
- "Poster must be 2:3 (600×900). Your image is 800×900 — crop it."
- "File is 310 KB; the limit is 200 KB — compress it."
- "Rhyme Rangers is missing a section — set one before publishing."
