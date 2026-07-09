# Environment Variables

Configuration variables used across the Kodama ecosystem.

## Naming conventions

| Prefix | Scope | Example |
|---|---|---|
| `VITE_` | Frontend (exposed to browser) | `VITE_API_URL` |
| No prefix | Backend only (never exposed) | `RATE_LIMIT_IP_SALT` |
| `SUPABASE_` | Supabase connection | `SUPABASE_SERVICE_ROLE_KEY` |

Frontend variables prefixed with `VITE_` are embedded in the build output and visible to users. Never prefix secrets with `VITE_`.

## Platform API

Used in `api.kodama.page` (Supabase Edge Functions / Deno server):

| Variable | Required | Description |
|---|---|---|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key (full database access) |
| `RATE_LIMIT_IP_SALT` | Yes | Salt for hashing IP addresses in rate limit buckets |
| `SENDER_VERIFY_SECRET` | Yes | HMAC secret for Drop verified sender tokens |
| `CORS_EXTRA_ORIGINS` | No | Comma-separated additional CORS origins (dev/testing) |
| `PORT` | No | Local dev server port (default: 8000) |

## Product frontends

Used in all product SPA repositories:

| Variable | Required | Description |
|---|---|---|
| `VITE_API_URL` | Yes | Platform API base URL (`https://api.kodama.page`) |

### Note-specific (current MVP)

Note currently uses direct Supabase access during migration. These will be removed once Note fully adopts `@kodama/api-client`:

| Variable | Required | Description |
|---|---|---|
| `VITE_SUPABASE_URL` | Yes* | Supabase project URL (*temporary) |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Yes* | Supabase anon key (*temporary) |

## Local development

When running locally, create a `.env` or `.env.local` file in the repository root:

### API local dev

```env
SUPABASE_URL=http://127.0.0.1:54321
SUPABASE_SERVICE_ROLE_KEY={from supabase status}
RATE_LIMIT_IP_SALT=local-dev-salt
SENDER_VERIFY_SECRET=local-dev-secret
CORS_EXTRA_ORIGINS=http://localhost:5173,http://localhost:3000
PORT=8000
```

Obtain local Supabase keys:

```bash
npx supabase start
npx supabase status
```

### Frontend local dev

```env
VITE_API_URL=http://localhost:8000
```

## Production values

Production environment variables are configured in:

| Service | Where to set |
|---|---|
| Cloudflare Pages | Project Settings → Environment Variables |
| Supabase Edge Functions | Supabase Dashboard → Edge Functions → Secrets |

Never commit production values to the repository.

## Security rules

1. **Never commit secrets.** All `.env` files are in `.gitignore`.
2. **Never prefix secrets with `VITE_`.** They will be bundled into the frontend.
3. **Rotate on compromise.** See [Disaster Recovery](../operations/Disaster%20Recovery.md).
4. **Minimum necessary scope.** Each service gets only the variables it needs.
5. **Document new variables.** Add to this file when introducing new environment variables.

## Adding a new variable

1. Add the variable to this document.
2. Add it to the `.env.example` file in the relevant repository.
3. Configure it in the deployment platform.
4. Never add production values to the repository.

## Related documentation

- [Production Infrastructure](Production%20Infrastructure.md)
- [Supabase](Supabase.md)
- [CI-CD](CI-CD.md)
