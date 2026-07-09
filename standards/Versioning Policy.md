# Versioning Policy

How Kodama versions its API, products, and shared libraries.

## API versioning

The Platform API uses URL path versioning:

```
/v1/note/create    ← current
/v2/note/create    ← future breaking changes
```

### Rules

- The current version is `/v1/`.
- Non-breaking changes (new endpoints, new optional fields, new error codes) do not require a version bump.
- Breaking changes require a new version prefix.
- Old versions are supported for at least **6 months** after a new version is released.
- Deprecation warnings are returned in response headers: `Deprecation: true`, `Sunset: {date}`.

### What counts as breaking

| Breaking | Non-breaking |
|---|---|
| Removing an endpoint | Adding a new endpoint |
| Renaming a field | Adding an optional field |
| Changing field types | Adding new enum values |
| Changing error reason codes | Adding new error reason codes |
| Changing authentication method | Adding optional auth methods |
| Removing response fields | Adding response fields |

## Product versioning

Individual Place products do not have independent version numbers. They deploy continuously from their main branch.

Product status is tracked separately:

| Status | Meaning |
|---|---|
| `live` | Production-ready, publicly available |
| `beta` | Functional but may change; limited announcement |
| `coming-soon` | Registered domain, not yet functional |

Status is declared in the marketing site product catalog, not in version tags.

## Library versioning

Shared packages use semantic versioning:

| Package | Scope | Versioning |
|---|---|---|
| `@kodama/api-client` | Platform API client | Semver (MAJOR.MINOR.PATCH) |

### Semver rules for `@kodama/api-client`

- **MAJOR** — Breaking changes to the client API (removed methods, changed signatures).
- **MINOR** — New methods or endpoints added.
- **PATCH** — Bug fixes, internal improvements.

Product frontends should pin to compatible major versions:

```json
{
  "@kodama/api-client": "^1.0.0"
}
```

## Security protocol versioning

The Kodama Security Protocol uses its own version scheme:

- Current: **v1**
- Breaking crypto or protocol changes require a new major version (v2).
- Protocol versions are documented in `kodama-security-protocol`.
- Products declare which protocol version they implement.

## Database schema versioning

Schema changes are versioned through sequential SQL migrations:

```
supabase/migrations/
├── 0001_initial_schema.sql
├── 0002_add_sessions.sql
├── 0003_add_rate_limits.sql
└── ...
```

Migration files are numbered sequentially and never modified after being applied to production.

## Documentation versioning

This repository (`kodama-docs`) is not versioned with tags. It reflects the current state of the ecosystem. Historical decisions are preserved in ADRs, which are never deleted or rewritten.

## Release coordination

When a breaking API change is planned:

1. Document the change in [Breaking Changes](../roadmap/Breaking%20Changes.md).
2. Add an ADR explaining the rationale.
3. Implement the new version alongside the old one.
4. Update `@kodama/api-client` with a major version bump.
5. Migrate product frontends to the new client version.
6. Set a sunset date for the old API version.
7. Update [Release Milestones](../roadmap/Release%20Milestones.md).

## Related documentation

- [API Standards](API%20Standards.md)
- [Release Process](../operations/Release%20Process.md)
- [Breaking Changes](../roadmap/Breaking%20Changes.md)
