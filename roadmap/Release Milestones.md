# Release Milestones

Significant ecosystem milestones and their target dates.

## Completed

| Milestone | Date | Details |
|---|---|---|
| Platform API v1 | 2026-Q2 | Hono API on Supabase Edge Functions |
| Kodama Note launch | 2026-Q2 | Live at `note.kodama.page` with client-side encryption |
| Kodama Drop beta | 2026-Q2 | Beta at `drop.kodama.page` with anonymous inbox |
| Marketing site | 2026-Q2 | `kodama.page` with product catalog and security pages |
| Security Protocol v1 spec | 2026-Q2 | Full specification in `kodama-web` |
| Brand guide site | 2026-Q2 | `brand.kodama.page` |
| Ecosystem documentation | 2026-Q3 | `kodama-docs` repository populated |

## Upcoming

| Milestone | Target | Details |
|---|---|---|
| Note → Platform API migration | 2026-Q3 | Note off direct Supabase RPC; full `@kodama/api-client` adoption |
| Drop → Platform API migration | 2026-Q3 | All Drop routes on `/v1/drop/*`; legacy routes removed |
| CI/CD for all repos | 2026-Q3 | GitHub Actions: typecheck, test, build on every PR |
| Security protocol repo migration | 2026-Q3 | Spec moved to `kodama-security-protocol` |
| Note capability URLs | 2026-Q4 | Reader sharing via `#reader_secret` fragment |
| Note signed edits | 2026-Q4 | Ed25519 edit signatures per Security Protocol v1 |

## Planned

| Milestone | Target | Details |
|---|---|---|
| Kodama Secret launch | 2027-Q1 | One-time secret links at `secret.kodama.page` |
| Kodama Poll launch | 2027-Q1 | No-signup polls at `poll.kodama.page` |
| Staging environment | 2027-Q1 | Supabase branch + Cloudflare preview for pre-production testing |
| Kodama Link launch | 2027-Q2 | Focused share links at `link.kodama.page` |
| Kodama Room launch | 2027-Q2 | Temporary shared spaces at `room.kodama.page` |
| Security audit | 2027-Q2 | Third-party review of crypto implementation |
| External monitoring | 2027-Q2 | Uptime checks, error rate alerting |

## Milestone format

When a milestone is completed:

1. Move it from "Upcoming" or "Planned" to "Completed" with the actual date.
2. Update the [Platform Roadmap](Platform%20Roadmap.md) phase checklist.
3. If the milestone involved breaking changes, archive the entry in [Breaking Changes](Breaking%20Changes.md).

When a milestone is delayed:

1. Update the target date.
2. Add a brief note explaining the delay (in the Details column).

## Related documentation

- [Platform Roadmap](Platform%20Roadmap.md)
- [Breaking Changes](Breaking%20Changes.md)
- [Release Process](../operations/Release%20Process.md)
