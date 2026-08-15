# Sequence — Authentication & Role Enforcement

```mermaid
sequenceDiagram
    autonumber
    actor E as Editor/Admin
    participant CMS as CMS
    participant API as FastAPI
    participant DB as PostgreSQL

    E->>CMS: submit email + password
    CMS->>API: POST /admin/auth/login {email, password}
    API->>DB: SELECT user by email
    DB-->>API: user row (hash, role)
    API->>API: verify_password(password, hash)
    alt invalid credentials
        API-->>CMS: 401 invalid_credentials
        CMS-->>E: "Incorrect email or password"
    else valid
        API->>API: sign JWT {sub, role, exp}
        API-->>CMS: 200 {access_token, role}
        CMS->>CMS: store token; set role in context
    end

    Note over CMS,API: Subsequent admin requests carry the Bearer token

    CMS->>API: POST /admin/catalog/publish (Bearer token)
    API->>API: decode + verify JWT
    API->>API: require_admin dependency
    alt role != admin
        API-->>CMS: 403 insufficient_permissions
        CMS-->>E: "Publish requires an admin account"
    else role == admin
        API->>API: proceed to publish
    end
```

## Key points

- JWT carries a **role claim**; the `require_admin` dependency rejects non-admins **before** the
  handler runs — enforcement, not a runtime `if`.
- Token expiry is short (60 min default); the CMS refreshes or redirects to login on 401.
- Passwords are stored hashed (argon2/bcrypt) — never plaintext, never logged.
