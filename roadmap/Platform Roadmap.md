# Platform Roadmap

Ecosystem-wide planning for the Kodama platform. Product-specific roadmaps live in their respective repositories.

## Vision

Kodama is a forest of single-purpose Places — quiet internet utilities that respect privacy, require no accounts, and do one thing well.

## Current state (July 2026)

| Place | Status | Backend | Security model |
|---|---|---|---|
| Note | Live | Supabase RPC (migrating) | Password + edit token |
| Drop | Beta | Platform API (partial) | bcrypt + session token |
| Secret | Planned | — | Capability-based (planned) |
| Poll | Planned | — | — |
| Room | Planned | — | — |
| Link | Planned | — | — |

## Phase 1: Foundation (current)

**Goal:** Establish shared platform infrastructure and migrate existing products.

- [x] Platform API (`api.kodama.page`) with Hono + Supabase Edge Functions
- [x] Shared database schema with Drizzle ORM
- [x] `@kodama/api-client` typed client library
- [x] Kodama Note live at `note.kodama.page`
- [x] Kodama Drop beta at `drop.kodama.page`
- [x] Marketing site at `kodama.page`
- [x] Security Protocol v1 specification
- [x] Ecosystem documentation (`kodama-docs`)
- [ ] Note migration to Platform API (off direct Supabase RPC)
- [ ] Drop full migration to `/v1/drop/*` routes
- [ ] Security protocol migration to `kodama-security-protocol` repo
- [ ] CI/CD pipelines for all repositories
- [ ] GitHub Actions for typecheck, test, build

## Phase 2: Security & Protocol

**Goal:** Implement full capability-based security across encrypted products.

- [ ] Note: capability URL sharing (`#reader_secret`)
- [ ] Note: Ed25519 signed edits
- [ ] Note: key rotation (owner action)
- [ ] Note: migrate from edit-token to Security Protocol v1
- [ ] Security protocol spec in dedicated repository
- [ ] Direct-to-storage upload for large ciphertext
- [ ] Security audit of crypto implementation

## Phase 3: New Places

**Goal:** Launch the next products in the forest.

- [ ] Kodama Secret — one-time secret links (`secret.kodama.page`)
- [ ] Kodama Poll — no-signup polls (`poll.kodama.page`)
- [ ] Kodama Link — focused share links (`link.kodama.page`)
- [ ] Kodama Room — temporary shared spaces (`room.kodama.page`)

Each new Place follows the established pattern: product spec → API routes → frontend → deploy.

## Phase 4: Platform maturity

**Goal:** Operational excellence and developer experience.

- [ ] Staging environment (Supabase branch + Cloudflare preview)
- [ ] Automated database migration pipeline
- [ ] External uptime monitoring
- [ ] Structured logging and alerting
- [ ] `@kodama/ui` shared component library (if warranted)
- [ ] API v2 planning (if breaking changes accumulate)

## Non-goals

These are explicitly out of scope for the platform:

- User accounts or authentication system
- Mobile native apps
- Real-time collaboration (CRDT, WebSocket sync)
- Marketplace or third-party Place plugins
- Enterprise SSO or team management
- Monetization infrastructure

Individual products may have their own non-goals documented in their repositories.

## How to propose changes

1. Open an issue or discussion in the relevant repository.
2. For architectural changes, write an ADR in `kodama-docs/adr/`.
3. For breaking changes, document in [Breaking Changes](Breaking%20Changes.md).
4. For new Places, start with a product spec and an ADR explaining the product type.

## Related documentation

- [Release Milestones](Release%20Milestones.md)
- [Breaking Changes](Breaking%20Changes.md)
- [ADR-0006 Why Places](../adr/ADR-0006%20Why%20Places.md)
