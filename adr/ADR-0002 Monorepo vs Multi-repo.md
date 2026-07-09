# ADR-0002: Monorepo vs Multi-repo

## Status

Accepted

## Date

2026-07-09

## Context

Following [ADR-0001](ADR-0001%20Repository%20Strategy.md), we evaluated whether a monorepo tooling approach (Turborepo, Nx, Lerna) could provide the benefits of shared code without the overhead of multi-repo coordination.

## Decision

**Multi-repo without monorepo tooling.** Each repository is independent with its own build, test, and deploy pipeline. Shared code is limited to `@kodama/api-client`.

## Evaluation

| Factor | Monorepo | Multi-repo |
|---|---|---|
| Independent deploys | Requires complex CI filtering | Native — each repo deploys separately |
| Shared types | Automatic via workspace | Via published `@kodama/api-client` |
| Build tooling | Turborepo/Nx setup and maintenance | Standard per-repo (Vite, npm) |
| Cross-repo changes | Single atomic commit | Coordinated PRs across repos |
| Onboarding | Must understand entire workspace | Focus on one repo |
| CI complexity | Shared pipeline with path filters | Simple per-repo pipeline |
| Team size fit | Better for 5+ developers | Better for 1–3 developers |

## Rationale

1. **Team size.** Kodama is built by a small team. Monorepo tooling pays off at scale — not at 1–3 developers.
2. **Deploy independence.** Each Place deploys to its own Cloudflare Pages project on its own subdomain. Monorepo CI would need path-based deploy filtering, adding complexity for no gain.
3. **Minimal shared code.** The only truly shared artifact is the API client library. This does not justify monorepo infrastructure.
4. **Product isolation.** A security fix in Drop should not trigger builds for Note, Secret, or the marketing site.

## When to revisit

Reconsider monorepo tooling if:

- The team grows beyond 5 active contributors
- Shared packages exceed 3 (e.g., UI kit, crypto library, testing utilities)
- Cross-repo changes become a weekly pain point
- Atomic multi-product releases become common

## Consequences

**Positive:**

- Simple, familiar tooling in every repo
- Zero monorepo configuration overhead
- Products are truly independent

**Negative:**

- `@kodama/api-client` must be published and versioned separately
- No shared ESLint/Prettier config package (copy conventions via docs instead)
- Cross-repo refactoring requires multiple PRs

## Related

- [ADR-0001 Repository Strategy](ADR-0001%20Repository%20Strategy.md)
- [Repository Structure](../architecture/Repository%20Structure.md)
- [Dependency Policy](../standards/Dependency%20Policy.md)
