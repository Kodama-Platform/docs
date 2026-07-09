# Overall Architecture

## Philosophy

Kodama is a platform of single-purpose internet utilities called **Places**. Each Place is a lightweight web application at a dedicated subdomain, backed by a shared API and database layer.

Three principles govern the architecture:

1. **One URL** — Resource identity is `(product_type, slug)`. No accounts, no profiles.
2. **One Purpose** — Each product type has distinct behavior and constraints.
3. **No Account** — Access is granted via URL, optional password, or cryptographic capability.

## System diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Kodama Forest                           │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Note   │  │   Drop   │  │  Secret  │  │   Poll   │   │
│  │  .kodama │  │  .kodama │  │  .kodama │  │  .kodama │   │
│  │  .page   │  │  .page   │  │  .page   │  │  .page   │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │             │             │             │          │
│       └─────────────┴──────┬──────┴─────────────┘          │
│                            │                                │
│                   @kodama/api-client                        │
│                            │                                │
│                   ┌────────▼────────┐                       │
│                   │  Platform API   │                       │
│                   │ api.kodama.page │                       │
│                   │  (Hono + Deno)  │                       │
│                   └────────┬────────┘                       │
│                            │ service role                   │
│                   ┌────────▼────────┐                       │
│                   │   PostgreSQL    │                       │
│                   │   (Supabase)    │                       │
│                   └─────────────────┘                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  Static / Marketing Layer                                   │
│  kodama.page · brand.kodama.page · cdn.kodama.page         │
└─────────────────────────────────────────────────────────────┘
```

## Layers

### Product frontends

Each Place is a static single-page application deployed to its own subdomain:

- **React 19** + **Vite 8** + **TypeScript**
- **TanStack Router** for routing, **TanStack Query** for server state
- **Tailwind CSS v4** + **shadcn/ui** for UI
- Communicates exclusively via HTTP to the Platform API through `@kodama/api-client`

Frontends never connect to the database directly. All data access goes through the Platform API.

### Platform API

A single shared API at `https://api.kodama.page` serves all product types:

- **Hono 4** on **Deno** (Supabase Edge Functions in production)
- Versioned routes under `/v1/{product_type}/`
- Service-role database access only — no Supabase Auth
- CORS restricted to `*.kodama.page` product hosts

The API is product-aware: each product type (`note`, `drop`, `secret`, etc.) has its own route namespace and business logic, but shares common infrastructure (slug validation, rate limiting, session management).

### Database

**PostgreSQL 15** via Supabase:

- Shared schema with product-specific tables
- Drizzle ORM for type-safe queries
- SQL migrations in `supabase/migrations/`
- The database layer is swappable — replacing the Supabase client with a direct Postgres pool requires no frontend changes

### Object storage

Supabase Storage for file attachments. Large ciphertext uploads (per the Security Protocol) will use direct-to-storage signed URLs.

### CDN and static assets

`cdn.kodama.page` serves shared static assets (icons, images). Product frontends and the marketing site reference this CDN.

## Identity model

There are no user accounts. Identity works as follows:

| Concept | Description |
|---|---|
| **Resource** | A Place instance identified by `(product_type, slug)` |
| **Slug** | URL-safe identifier chosen at creation time |
| **Access token** | Product-specific credential (password, edit token, capability URL fragment) |
| **Session** | Short-lived server-side token after successful authentication |

A slug is unique per product type, not globally. `note.kodama.page/wallet` and `drop.kodama.page/wallet` are independent resources.

## Security architecture

Sensitive products (Note, Secret) use client-side encryption:

- Plaintext never leaves the browser
- The server stores only ciphertext, IVs, salts, and wrapped keys
- Access is controlled via capability URLs in the `#fragment` (never sent to the server)
- See `kodama-security-protocol` for the full specification

Non-encrypted products (Drop) use server-side password hashing (bcrypt) and session tokens.

## Request flow

```
Browser → Product SPA → @kodama/api-client → Platform API → PostgreSQL
                                              ↓
                                         Rate limiter
                                         Slug validator
                                         Session manager
```

1. The SPA constructs a typed request via `@kodama/api-client`.
2. The client sends an HTTP request to `api.kodama.page`.
3. The API validates the request (CORS, rate limits, slug rules).
4. Product-specific handlers execute business logic.
5. The API returns a typed JSON response or standardized error.

## Current vs. target state

The ecosystem is migrating toward the target architecture described above. Notable gaps:

| Area | Current | Target |
|---|---|---|
| Note backend access | Direct Supabase RPC calls | Platform API via `@kodama/api-client` |
| Note write auth | Edit token in URL hash | Ed25519 signed edits + capability URLs |
| Drop API | Mixed legacy and `/v1/` routes | Fully on `/v1/drop/*` |
| Security protocol | Spec in `kodama-web` | Dedicated `kodama-security-protocol` repo |

See individual ADRs in [`adr/`](../adr/) for the reasoning behind these decisions.

## Related documentation

- [Repository Structure](Repository%20Structure.md)
- [Infrastructure Overview](Infrastructure%20Overview.md)
- [ADR-0002 Monorepo vs Multi-repo](../adr/ADR-0002%20Monorepo%20vs%20Multi-repo.md)
- [ADR-0003 Capability-Based Security](../adr/ADR-0003%20Capability-Based%20Security.md)
