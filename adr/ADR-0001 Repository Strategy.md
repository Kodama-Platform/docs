# ADR-0001: Repository Strategy

## Status

Accepted

## Date

2026-07-09

## Context

Kodama is a multi-product platform with shared infrastructure (API, database, security protocol) and independent product frontends (Note, Drop, Secret, etc.). We need a clear strategy for organizing code across Git repositories.

Options considered:

1. **Single monorepo** — all code in one repository
2. **Multi-repo with shared packages** — separate repos with published shared libraries
3. **Multi-repo, fully independent** — no shared code between repos

## Decision

Use a **multi-repo strategy** with a shared Platform API and client library:

- Platform infrastructure in `Kodama-Platform` GitHub organization
- Product frontends in individual repositories (may be under personal or org accounts)
- Shared code published as `@kodama/api-client` from the API repository
- Ecosystem documentation centralized in `kodama-docs`

## Rationale

**Why not a monorepo:**

- Products deploy independently to separate subdomains
- Different products may have different release cadences
- A monorepo creates coupling — a Drop bug fix should not require rebuilding Note
- The team is small; monorepo tooling (Turborepo, Nx) adds complexity without proportional benefit

**Why not fully independent repos:**

- The Platform API, database schema, and client library are genuinely shared
- Duplicating API types across repos leads to drift
- A single API with typed client ensures consistency

**Why multi-repo with shared packages:**

- Each product can deploy independently
- Shared infrastructure has a single source of truth
- `@kodama/api-client` provides type safety without monorepo overhead
- New products follow a proven pattern: create frontend repo, add API routes, connect via client library

## Consequences

**Positive:**

- Clear ownership boundaries per repository
- Independent deployment per product
- Shared types via published client library
- New contributors can focus on one repo at a time

**Negative:**

- Cross-repo changes require coordinated PRs (API first, then frontend)
- Client library must be published/updated separately
- No atomic commits across repos
- Repository proliferation as products are added

## Related

- [ADR-0002 Monorepo vs Multi-repo](ADR-0002%20Monorepo%20vs%20Multi-repo.md)
- [Repository Structure](../architecture/Repository%20Structure.md)
