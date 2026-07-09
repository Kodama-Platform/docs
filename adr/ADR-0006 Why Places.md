# ADR-0006: Why Places

## Status

Accepted

## Date

2026-07-09

## Context

Kodama could be built as a traditional SaaS platform with dashboards, workspaces, and multi-feature applications. Instead, the ecosystem is organized around **Places** — single-purpose utilities at dedicated subdomains.

The question: why decompose the platform into separate single-purpose products rather than building one unified application?

## Decision

Kodama is a **forest of Places** — independent, single-purpose web utilities, each at `{product}.kodama.page/{slug}`.

Six product types are defined:

| Place | Domain | Purpose |
|---|---|---|
| Note | `note.kodama.page` | Private encrypted writing |
| Drop | `drop.kodama.page` | Anonymous message inbox |
| Secret | `secret.kodama.page` | One-time secret links |
| Poll | `poll.kodama.page` | No-signup polls |
| Room | `room.kodama.page` | Temporary shared spaces |
| Link | `link.kodama.page` | Focused share links |

## Rationale

**Why Places, not a platform:**

1. **One purpose, done well.** Each Place does exactly one thing. Note is for writing. Drop is for receiving messages. Combining them into one app dilutes focus and increases complexity.

2. **Independent lifecycles.** Note can reach v2 while Drop is still in beta. A Place can be deprecated without affecting others. Each has its own frontend, deploy pipeline, and release cadence.

3. **URL clarity.** `note.kodama.page/my-ideas` tells you exactly what you're getting. A generic `kodama.page/my-ideas` requires context to understand the resource type.

4. **Minimal surface area.** Each Place has a small attack surface. A vulnerability in Drop does not expose Note's encryption logic. Security boundaries are natural, not artificial.

5. **Discoverability.** The marketing site presents each Place as a distinct product. Users find what they need without navigating a complex platform.

6. **Shared infrastructure, independent products.** Places share the Platform API and database but are otherwise independent. This gives economies of scale without product coupling.

**The forest metaphor:**

Kodama is a forest, not a tree. Each Place is a distinct tree — independent, with its own shape, but sharing the same soil (infrastructure). Users visit the tree they need, not a warehouse of tools.

## Architecture implications

| Aspect | Implication |
|---|---|
| Identity | `(product_type, slug)` — slug unique per product, not globally |
| API | Namespaced routes: `/v1/{product_type}/*` |
| Frontend | Separate repo and deployment per Place |
| Database | Shared tables with `product_type` column |
| Domain | One subdomain per Place |
| Marketing | Product catalog on `kodama.page` |

## What Places are not

- **Not microservices.** Places share a single API and database. They are product boundaries, not service boundaries.
- **Not plugins.** Places are not modules within a larger app. Each is a standalone SPA.
- **Not workspaces.** A Place is a single resource, not a container for other resources.

## Consequences

**Positive:**

- Clear product identity and focused UX
- Independent deployment and versioning
- Natural security boundaries
- Easy to explain: "Kodama Note is a place for writing"
- New Places can be added without modifying existing ones

**Negative:**

- More repositories to maintain
- Shared infrastructure must support all product types
- Cross-Place features (e.g., "share this Note via Drop") require explicit integration
- Users manage multiple URLs instead of one dashboard

## Adding a new Place

When a new product type is ready:

1. Define the product spec in a `kodama-{product}` repository.
2. Add API routes under `/v1/{product_type}/` in `api.kodama.page`.
3. Create the frontend repository and deploy to `{product}.kodama.page`.
4. Add to the marketing site product catalog.
5. Document any new architectural decisions as ADRs.

## Related

- [ADR-0005 Why No Accounts](ADR-0005%20Why%20No%20Accounts.md)
- [ADR-0001 Repository Strategy](ADR-0001%20Repository%20Strategy.md)
- [Overall Architecture](../architecture/Overall%20Architecture.md)
- [Platform Roadmap](../roadmap/Platform%20Roadmap.md)
