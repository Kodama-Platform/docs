# Supabase

How Supabase is used in the Kodama ecosystem.

## Role

Supabase provides three services for Kodama:

1. **PostgreSQL database** — shared data store for all product types
2. **Edge Functions** — runtime for the Platform API
3. **Storage** — object storage for file attachments

Supabase Auth is **not used**. Kodama has no user accounts.

## Project structure

```
api.kodama.page/
├── supabase/
│   ├── config.toml           # Supabase project configuration
│   ├── migrations/           # SQL migration files
│   │   ├── 0001_initial.sql
│   │   └── ...
│   └── functions/
│       └── kodama-api/       # Edge Function entry point
│           └── index.ts
├── drizzle/
│   └── schema.ts             # Drizzle ORM schema definitions
└── lib/
    └── db.ts                 # Database client
```

## Local development

### Prerequisites

- Docker Desktop (running)
- Supabase CLI: `npm install -g supabase`

### Setup

```bash
# Start local Supabase (PostgreSQL + Studio + Edge Functions)
npm run db:start

# Apply migrations
npm run db:push

# Get local credentials
npx supabase status
```

Local services:

| Service | URL |
|---|---|
| PostgreSQL | `postgresql://postgres:postgres@127.0.0.1:54322/postgres` |
| Studio (Dashboard) | `http://127.0.0.1:54323` |
| Edge Functions | `http://127.0.0.1:54321/functions/v1/kodama-api` |
| API (via router) | `http://127.0.0.1:54321` |

### Local API development

Two options for running the API locally:

**Option A: Supabase Edge Functions (production-like)**

```bash
npm run db:start
supabase functions serve kodama-api
```

**Option B: Standalone Deno server (faster iteration)**

```bash
npm run db:start
npm run dev:api
# Runs on http://localhost:8000
```

Option B is preferred for daily development. Option A for pre-deploy verification.

## Database

### Connection

The API connects via the Supabase service role key:

```typescript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```

Or via Drizzle ORM with a direct Postgres connection:

```typescript
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";

const client = postgres(process.env.DATABASE_URL!);
const db = drizzle(client);
```

### Migrations

Migrations are numbered SQL files applied sequentially:

```bash
# Generate migration from Drizzle schema changes
npm run db:generate

# Apply migrations locally
npm run db:push

# Apply to production
supabase db push --project-ref {project-ref}
```

Never modify a migration after it has been applied to production. Create a new migration instead.

## Edge Functions

The Platform API runs as a single Edge Function:

| Function | Entry | Route |
|---|---|---|
| `kodama-api` | `supabase/functions/kodama-api/index.ts` | All `/v1/*` routes |

Deploy to production:

```bash
supabase functions deploy kodama-api --project-ref {project-ref}
```

View logs:

```bash
supabase functions logs kodama-api --project-ref {project-ref}
```

## Storage

### Buckets

| Bucket | Public | Purpose |
|---|---|---|
| `page-attachments` | No | Note file attachments |

### Access pattern

1. Frontend requests a signed upload URL from the Platform API.
2. API generates a Supabase Storage signed URL with expiry.
3. Frontend uploads directly to Storage.
4. API stores the file reference in the database.

Files are never served publicly. All access requires signed URLs.

## Why Supabase

See [ADR-0004 Why Supabase](../adr/ADR-0004%20Why%20Supabase.md) for the decision rationale.

Summary: Supabase provides PostgreSQL, Edge Functions, and Storage in a single platform with minimal operational overhead — ideal for a small team building a multi-product platform.

## Swapping Supabase

The database layer is designed to be swappable. To migrate away from Supabase:

1. Replace `lib/db.ts` with a direct PostgreSQL connection pool.
2. Move Edge Functions to a standalone Deno/Node server.
3. Move Storage to S3/R2/compatible provider.
4. Frontends are unaffected (they communicate via HTTP only).

See [ADR-0004](../adr/ADR-0004%20Why%20Supabase.md) for the full analysis.

## Related documentation

- [Database Standards](../standards/Database%20Standards.md)
- [Environment Variables](Environment%20Variables.md)
- [Production Infrastructure](Production%20Infrastructure.md)
- [Backup Policy](../operations/Backup%20Policy.md)
