# Sequence — Artwork Upload & Validation

```mermaid
sequenceDiagram
    autonumber
    actor E as Editor
    participant CMS as CMS (upload slot)
    participant API as FastAPI
    participant Val as Validator
    participant ST as StorageBackend
    participant DB as PostgreSQL

    E->>CMS: choose file in "Poster" slot
    CMS->>CMS: show live preview + required dims (600×900, 2:3, ≤200 KB)
    CMS->>API: POST /admin/artworks (multipart: file + kind)
    API->>Val: validate(file, kind="poster")
    Val->>Val: decode image → width, height
    Val->>Val: check aspect ratio (2:3), target px, size ≤ 200 KB
    alt invalid aspect
        API-->>CMS: 422 {code:"aspect_ratio", message:"Poster must be 2:3, got 800×900"}
        CMS-->>E: "Poster must be 2:3 (600×900). Got 800×900."
    alt too large
        API-->>CMS: 422 {code:"size_exceeded", message:"File is 310 KB; limit is 200 KB"}
        CMS-->>E: "File is too large (310 KB). Max is 200 KB."
    else valid
        Val->>ST: put("artwork/poster/{uuid}.jpg", bytes, "image/jpeg")
        ST-->>Val: ok
        API->>DB: INSERT artworks(show_id, kind, object_key, width, height, size, sha256)
        DB-->>API: artwork row
        API-->>CMS: 201 {artwork}
        CMS->>CMS: mark slot complete; show saved thumbnail
    end
```

## Editor-readable errors

The API returns a machine-readable `code` **and** a human `message` the CMS can show verbatim.
Codes: `aspect_ratio`, `size_exceeded`, `unsupported_type`, `decode_failed`, `missing_file`.
The CMS maps each code to a sentence an editor can act on ("crop to 2:3", "compress to ≤ 200 KB").
