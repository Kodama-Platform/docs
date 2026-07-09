# Database Standards

Conventions for the shared PostgreSQL database used by all Kodama products.

## Provider

**Supabase PostgreSQL 15** — single shared database for the entire ecosystem.

Local development uses `supabase start` (Docker). See [Supabase](../deployment/Supabase.md) for setup.

## Schema management

| Tool | Purpose |
|---|---|
| Drizzle ORM | Type-safe queries and schema definitions |
| SQL migrations | Authoritative schema changes in `supabase/migrations/` |
| Drizzle Kit | Schema introspection and migration generation |

### Migration workflow

1. Modify the Drizzle schema in `drizzle/schema.ts`.
2. Generate a migration: `npm run db:generate`.
3. Review the generated SQL in `supabase/migrations/`.
4. Apply locally: `npm run db:push`.
5. Test thoroughly before deploying to production.

Never modify production schema directly. All changes go through migrations.

## Naming conventions

| Element | Convention | Example |
|---|---|---|
| Tables | `kodama_{entity}` (snake_case) | `kodama_resources` |
| Columns | snake_case | `created_at`, `product_type` |
| Primary keys | `id` (UUID) | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` |
| Foreign keys | `{entity}_id` | `resource_id` |
| Indexes | `idx_{table}_{columns}` | `idx_resources_product_slug` |
| Constraints | `{table}_{description}` | `kodama_resources_slug_product_unique` |
| Enums | snake_case type name | `product_type`, `burn_mode` |

## Standard columns

Every table must include:

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
```

Tables with soft delete add:

```sql
deleted_at TIMESTAMPTZ
```

Use a trigger or application logic to update `updated_at` on modification.

## Core tables

| Table | Purpose |
|---|---|
| `kodama_resources` | All Place instances (note, drop, secret, etc.) |
| `kodama_resource_versions` | Version history for versioned products |
| `kodama_drop_messages` | Drop inbox messages |
| `kodama_sessions` | Server-side session tokens |
| `kodama_drop_message_fingerprints` | Message deduplication hashes |
| `kodama_reserved_slugs` | Globally reserved slug list |
| `api_rate_limits` | IP-based rate limiting buckets |

## Data access rules

1. **Only the Platform API accesses the database.** Frontends never connect directly.
2. **Service role only.** The API uses the Supabase service role key. Row Level Security (RLS) is not used — authorization is handled in application code.
3. **No Supabase Auth.** There are no user accounts. Sessions are managed in `kodama_sessions`.
4. **Parameterized queries only.** Never interpolate user input into SQL strings.

## Indexing guidelines

- Index all foreign keys.
- Index columns used in WHERE clauses for common queries.
- Use composite indexes for multi-column lookups (e.g., `(product_type, slug)`).
- Avoid over-indexing — measure before adding indexes to infrequently queried columns.

## Data types

| Use case | Type |
|---|---|
| Primary keys | `UUID` |
| Timestamps | `TIMESTAMPTZ` |
| Short strings | `TEXT` with CHECK constraints |
| Enums | PostgreSQL `ENUM` types or `TEXT` with CHECK |
| JSON metadata | `JSONB` |
| Binary data | `BYTEA` (ciphertext, hashes) |
| Booleans | `BOOLEAN NOT NULL DEFAULT false` |
| Counters | `INTEGER` or `BIGINT` |

Store ciphertext as `BYTEA`, not base64-encoded text. Encode/decode at the application layer.

## Sensitive data

The database may contain:

| Data | Storage | Notes |
|---|---|---|
| Ciphertext | `BYTEA` | Encrypted client-side; server cannot decrypt |
| Password hashes | `TEXT` | bcrypt for Drop owner passwords |
| Owner auth hashes | `BYTEA` | SHA-256 of master secret |
| Wrapped content keys | `BYTEA` | AES-wrapped CEK |
| Editor public keys | `BYTEA` | Ed25519 public keys |
| Session tokens | `TEXT` | Hashed before storage |

The database must **never** contain:

- Plaintext content
- User passwords (only hashes)
- Private keys
- Reader secrets or capability tokens

## Backup and recovery

See [Backup Policy](../operations/Backup%20Policy.md) and [Disaster Recovery](../operations/Disaster%20Recovery.md).

Supabase provides automated daily backups. Additional backup procedures are documented in operations.

## Swappable database layer

The database client in `lib/db.ts` is designed to be swappable. Replacing the Supabase client with a direct PostgreSQL connection pool requires changes only in the API repository — frontends are unaffected because they communicate via HTTP only.

## Related documentation

- [API Standards](API%20Standards.md)
- [Supabase](../deployment/Supabase.md)
- [Backup Policy](../operations/Backup%20Policy.md)
