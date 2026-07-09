# ADR-0004: Why Supabase

## Status

Accepted

## Date

2026-07-09

## Context

Kodama needs a backend platform providing:

- PostgreSQL database (shared across all product types)
- Serverless API runtime (Edge Functions)
- Object storage (file attachments)
- Local development environment

Options evaluated:

1. **Supabase** — managed PostgreSQL + Edge Functions + Storage
2. **AWS (RDS + Lambda + S3)** — individual services composed together
3. **PlanetScale + Vercel + R2** — best-of-breed composition
4. **Self-hosted PostgreSQL + custom server** — full control, full responsibility
5. **Firebase** — Google's BaaS platform

## Decision

Use **Supabase** for database, API runtime, and object storage.

## Rationale

| Factor | Supabase | AWS | Self-hosted |
|---|---|---|---|
| PostgreSQL | Managed, included | RDS (separate setup) | Manual setup |
| API runtime | Edge Functions (Deno) | Lambda (Node/Python) | Custom server |
| Object storage | Storage (included) | S3 (separate) | Manual setup |
| Local dev | `supabase start` (Docker) | LocalStack / manual | Docker compose |
| Auth | Available (not used) | Cognito (not used) | N/A |
| Cost (early stage) | ~$25/month (Pro) | ~$50–100/month | Server cost + time |
| Operational overhead | Low | High | Very high |
| Migration path | Swappable (see below) | N/A | N/A |

**Why Supabase specifically:**

1. **All three needs in one platform.** Database, compute, and storage without stitching services together.
2. **PostgreSQL, not a proprietary database.** Standard SQL, standard migrations, standard ORM. No vendor lock-in at the data layer.
3. **Edge Functions run Deno.** Modern runtime with Web Crypto API built in — important for a security-focused platform.
4. **Excellent local dev experience.** `supabase start` gives a full local environment in one command.
5. **Right-sized for the team.** A small team cannot operate RDS + Lambda + S3 + CloudFront productively.

**Why not Firebase:**

- Firestore is not PostgreSQL — no relational queries, no Drizzle ORM
- Firebase Auth is account-based — contradicts Kodama's no-accounts model
- Vendor lock-in at the database layer

**Why not self-hosted:**

- Operational burden exceeds team capacity
- Security patching, backup management, and uptime are distractions from product development

## Swappability

The Supabase dependency is intentionally shallow:

- **Database:** Standard PostgreSQL. Replace `lib/db.ts` with a connection pool — no frontend changes.
- **Edge Functions:** Hono app runs on any Deno/Node runtime. Not Supabase-specific.
- **Storage:** S3-compatible API exists. Migrate to R2 or S3 with minimal changes.
- **Frontends:** Communicate via HTTP only. Backend swap is invisible to users.

## Consequences

**Positive:**

- Fast time to production
- Single dashboard for database, functions, and storage
- Local development mirrors production
- Standard PostgreSQL — no proprietary query language

**Negative:**

- Supabase Edge Functions have cold start latency
- Supabase Storage lacks cross-region replication on lower plans
- Dependency on Supabase platform availability
- Edge Function runtime limits (memory, execution time)

## When to revisit

Consider migrating individual services away from Supabase if:

- Cold start latency impacts user experience
- Database exceeds Supabase plan limits
- Storage needs cross-region replication
- Team grows large enough to operate dedicated infrastructure

## Related

- [Supabase](../deployment/Supabase.md)
- [Database Standards](../standards/Database%20Standards.md)
- [Infrastructure Overview](../architecture/Infrastructure%20Overview.md)
