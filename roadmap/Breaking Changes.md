# Breaking Changes

A log of breaking changes across the Kodama ecosystem. Consult this before upgrading dependencies or deploying cross-repo changes.

## Format

Each entry documents:

- **What changed** — the specific breaking change
- **Affected repos** — which repositories need updates
- **Migration steps** — how to adapt to the change
- **Timeline** — when the old behavior was removed or will be removed

---

## Upcoming breaking changes

### Note: Supabase RPC → Platform API

| Field | Value |
|---|---|
| **What** | Note frontend will stop calling Supabase RPCs directly (`kodama_read_page`, etc.) and use `@kodama/api-client` exclusively |
| **Affected** | `note.kodama.page` |
| **Migration** | Replace Supabase client calls with `@kodama/api-client` methods. Remove `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` env vars. |
| **Timeline** | Target: 2026-Q3. Old RPC calls will stop working when Note is fully migrated. |

### Drop: Legacy API routes removal

| Field | Value |
|---|---|
| **What** | Legacy Drop API routes (`/createDrop`, etc.) will be removed. All clients must use `/v1/drop/*` routes. |
| **Affected** | `drop.kodama.page` |
| **Migration** | Update all API calls to use `/v1/drop/*` prefix. Update `@kodama/api-client` to latest version. |
| **Timeline** | Target: 2026-Q3. Legacy routes deprecated but still functional. |

### Note: Edit token → Capability URLs

| Field | Value |
|---|---|
| **What** | Note will migrate from edit-token authentication to capability-based security (Security Protocol v1). Edit tokens in URL hash will be replaced by `#reader_secret` fragments. |
| **Affected** | `note.kodama.page`, existing Note URLs with edit tokens |
| **Migration** | Existing Places will need key rotation to adopt the new model. Old edit-token URLs will stop working after migration. Users must save new capability URLs. |
| **Timeline** | Target: 2026-Q4. Breaking for existing Note Places with edit tokens. |

---

## Completed breaking changes

_No breaking changes have been deployed yet._

---

## Deprecation policy

When introducing a breaking change:

1. **Announce** — add an entry to this document under "Upcoming."
2. **Deprecate** — keep the old behavior working alongside the new behavior.
3. **Warn** — return deprecation headers on old endpoints: `Deprecation: true`, `Sunset: {date}`.
4. **Migrate** — update all internal clients to the new behavior.
5. **Remove** — delete the old behavior after the sunset date (minimum 6 months).
6. **Archive** — move the entry to "Completed" with the removal date.

## Version compatibility

| Component | Current version | Breaking change policy |
|---|---|---|
| Platform API | v1 | New version prefix for breaking changes |
| `@kodama/api-client` | v1.x | Semver — major bump for breaking changes |
| Security Protocol | v1 | New version for crypto/protocol breaking changes |
| Database schema | Sequential migrations | Forward-only; no rollback migrations |

## Related documentation

- [Versioning Policy](../standards/Versioning%20Policy.md)
- [Platform Roadmap](Platform%20Roadmap.md)
- [Release Milestones](Release%20Milestones.md)
- [Release Process](../operations/Release%20Process.md)
