# Production Infrastructure

Overview of the Kodama production environment.

## Architecture summary

```
Users
  │
  ▼
Cloudflare (DNS + CDN + Pages)
  │
  ├── *.kodama.page ──→ Cloudflare Pages (static SPAs)
  ├── cdn.kodama.page ──→ Cloudflare CDN (static assets)
  │
  ▼
api.kodama.page
  │
  ▼
Supabase Edge Functions (Deno + Hono)
  │
  ├── PostgreSQL 15 (database)
  └── Storage (object storage)
```

## Production services

| Service | Provider | Plan | Purpose |
|---|---|---|---|
| DNS + CDN | Cloudflare | Free/Pro | Domain routing, SSL, caching, DDoS protection |
| Static hosting | Cloudflare Pages | Free | Product SPAs, marketing site |
| API runtime | Supabase Edge Functions | Pro | Platform API |
| Database | Supabase PostgreSQL | Pro | Shared data store |
| Object storage | Supabase Storage | Pro | File attachments |
| Source control | GitHub | Free/Team | Code repositories |

## Domain configuration

All domains are registered and managed through Cloudflare:

| Domain | Type | Target |
|---|---|---|
| `kodama.page` | CNAME | Cloudflare Pages |
| `*.kodama.page` | CNAME | Cloudflare Pages (per-project) |
| `api.kodama.page` | CNAME | Supabase Edge Function |
| `cdn.kodama.page` | CNAME | Cloudflare CDN |

SSL/TLS mode: **Full (Strict)** on all domains.

## Compute resources

### Cloudflare Pages

Each frontend project is a separate Cloudflare Pages project:

| Project | Domain | Build command | Output |
|---|---|---|---|
| kodama-web | `kodama.page` | `npm run build` | `dist/` |
| note | `note.kodama.page` | `npm run build` | `dist/` |
| drop | `drop.kodama.page` | `npm run build` | `dist/` |
| brand | `brand.kodama.page` | `npm run build` | `.next/` or `out/` |

Build settings:

- Node.js version: 20.x
- Build command: `npm run build`
- Root directory: `/` (repository root)

### Supabase Edge Functions

| Function | Route | Runtime |
|---|---|---|
| `kodama-api` | `api.kodama.page/*` | Deno |

Deploy command:

```bash
supabase functions deploy kodama-api --project-ref {project-ref}
```

## Database

Production PostgreSQL instance on Supabase:

- Version: PostgreSQL 15
- Region: (configured per Supabase project)
- Connection pooling: enabled (Supavisor)
- Backups: daily automated (see [Backup Policy](../operations/Backup%20Policy.md))

Direct database access is restricted to the Platform API via service role key. No public-facing database endpoints.

## Storage

| Bucket | Purpose | Access |
|---|---|---|
| `page-attachments` | Note file attachments | Signed URLs via API |

Storage limits are governed by the Supabase plan. Monitor usage in the Supabase Dashboard.

## Network security

| Control | Configuration |
|---|---|
| HTTPS | Enforced on all domains |
| CORS | `*.kodama.page` + localhost (dev only) |
| WAF | Cloudflare managed rules (when available) |
| Rate limiting | Application-level (see [API Standards](../standards/API%20Standards.md)) |
| DDoS protection | Cloudflare automatic |

## Cost management

The current infrastructure is designed for minimal cost:

- Cloudflare Pages: free tier (500 builds/month)
- Supabase: Pro plan (~$25/month)
- Cloudflare DNS/CDN: free tier
- GitHub: free tier

Monitor usage monthly. Scale plans only when metrics justify it.

## Scaling triggers

| Metric | Current capacity | Scale action |
|---|---|---|
| API requests | Edge Function auto-scales | Monitor cold start latency |
| Database connections | Supabase pool (default) | Upgrade Supabase plan |
| Storage | Supabase plan limit | Add external storage (R2/S3) |
| Build minutes | 500/month (Cloudflare free) | Upgrade Cloudflare plan |

## Related documentation

- [Infrastructure Overview](../architecture/Infrastructure%20Overview.md)
- [Environment Variables](Environment%20Variables.md)
- [Supabase](Supabase.md)
- [Cloudflare](Cloudflare.md)
- [CI-CD](CI-CD.md)
