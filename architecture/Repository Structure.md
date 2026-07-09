# Repository Structure

## Overview

The Kodama ecosystem uses a **multi-repo** strategy. Each repository has a single, clear purpose. Platform infrastructure and product frontends are separated.

## Repository map

```
Kodama-Platform/                    itcvmaster/ (deployment)
├── kodama-docs          ← you are here
├── kodama-web           ← marketing hub
├── api.kodama.page      ← Platform API + schema
├── kodama-note          ← Note product specs
└── kodama-security-protocol  ← Security protocol spec

                                    ├── note.kodama.page
                                    ├── drop.kodama.page
                                    ├── brand.kodama.page
                                    └── kodama.page
```

## Platform repositories

Maintained under the **Kodama-Platform** GitHub organization. These are the canonical source of truth.

| Repository | Purpose | Deploys to |
|---|---|---|
| `kodama-docs` | Ecosystem documentation (this repo) | — |
| `kodama-web` | Marketing site, manifesto, security pages | `kodama.page` |
| `api.kodama.page` | Platform API, database schema, `@kodama/api-client` | `api.kodama.page` |
| `kodama-note` | Note product specifications and requirements | — |
| `kodama-security-protocol` | Kodama Security Protocol v1 specification | — |

## Product frontend repositories

Each product Place has its own frontend repository containing the SPA that users interact with.

| Repository | Product | Domain | Status |
|---|---|---|---|
| `note.kodama.page` | Kodama Note | `note.kodama.page` | Live |
| `drop.kodama.page` | Kodama Drop | `drop.kodama.page` | Beta |
| `brand.kodama.page` | Brand guide | `brand.kodama.page` | Live |

Future product frontends (`secret.kodama.page`, `poll.kodama.page`, etc.) will follow the same pattern.

## What lives where

### In platform repos

- API route handlers and middleware
- Database schema and migrations
- Shared client library (`@kodama/api-client`)
- Ecosystem-wide documentation
- Security protocol specification
- Marketing and brand content

### In product repos

- Product-specific UI components and pages
- Product-specific business logic (client-side)
- Product-specific tests
- Product README and local development setup
- Product deployment configuration

### Explicitly excluded from this repo

| Content | Belongs in |
|---|---|
| Brand guide, colors, typography | `brand.kodama.page` |
| Note API endpoints, database schema | `kodama-note` |
| Threat model, key hierarchy, protocols | `kodama-security-protocol` |
| Landing page copy, SEO, analytics | `kodama-web` / `kodama.page` |

## Standard repository layout

Every Kodama repository should follow this structure:

```
repository/
├── README.md                 # Purpose, setup, and quick start
├── package.json              # Dependencies and scripts
├── tsconfig.json             # TypeScript configuration
├── .gitignore
├── src/                      # Application source
│   ├── components/           # UI components
│   ├── lib/                  # Shared utilities
│   └── routes/               # Route definitions (if applicable)
├── public/                   # Static assets
└── tests/                    # Test files
```

Platform API repositories additionally include:

```
api.kodama.page/
├── supabase/
│   ├── migrations/           # SQL migrations
│   └── functions/            # Edge function entry points
├── drizzle/                  # Drizzle schema definitions
├── lib/
│   ├── db.ts                 # Database client
│   └── routes/               # Hono route handlers
└── packages/
    └── api-client/           # @kodama/api-client
```

## Naming conventions

| Element | Convention | Example |
|---|---|---|
| Repository name | `{product}.kodama.page` or descriptive kebab-case | `note.kodama.page`, `kodama-docs` |
| npm package scope | `@kodama/{name}` | `@kodama/api-client` |
| Domain | `{product}.kodama.page` | `note.kodama.page` |
| API routes | `/v1/{product_type}/{action}` | `/v1/note/create` |
| Database tables | `kodama_{entity}` | `kodama_resources` |
| Environment variables | `VITE_{NAME}` (frontend), `{NAME}` (backend) | `VITE_API_URL` |

## Dependencies between repositories

```
kodama-web ──────────────────→ (links to all products)

note.kodama.page ──→ @kodama/api-client ──→ api.kodama.page
drop.kodama.page ──→ @kodama/api-client ──→ api.kodama.page

kodama-note ──→ (references) kodama-security-protocol
kodama-security-protocol ──→ (referenced by) api.kodama.page
```

Product frontends depend on `@kodama/api-client`, which is published from `api.kodama.page`. No product frontend should depend on another product frontend.

## Creating a new repository

When adding a new Place to the forest:

1. Create a product spec in a `kodama-{product}` repo (or section in existing specs).
2. Add API routes to `api.kodama.page` under `/v1/{product_type}/`.
3. Create the frontend repo following the standard layout.
4. Register the subdomain (`{product}.kodama.page`).
5. Add the product to the marketing site product catalog.
6. Document any architectural decisions as ADRs in this repo.

## Related documentation

- [Overall Architecture](Overall%20Architecture.md)
- [ADR-0001 Repository Strategy](../adr/ADR-0001%20Repository%20Strategy.md)
- [ADR-0002 Monorepo vs Multi-repo](../adr/ADR-0002%20Monorepo%20vs%20Multi-repo.md)
- [Coding Standards](../standards/Coding%20Standards.md)
