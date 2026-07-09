# Cloudflare

How Cloudflare is used in the Kodama ecosystem.

## Role

Cloudflare provides four services for Kodama:

1. **DNS** — domain management for `kodama.page` and all subdomains
2. **Pages** — static hosting for product frontends and marketing site
3. **CDN** — caching and delivery for static assets at `cdn.kodama.page`
4. **Security** — SSL/TLS, DDoS protection, WAF

## DNS configuration

All Kodama domains are managed in a single Cloudflare zone: `kodama.page`.

| Record | Type | Target | Proxy |
|---|---|---|---|
| `kodama.page` | CNAME | Cloudflare Pages project | Yes |
| `note.kodama.page` | CNAME | Cloudflare Pages project | Yes |
| `drop.kodama.page` | CNAME | Cloudflare Pages project | Yes |
| `brand.kodama.page` | CNAME | Cloudflare Pages project | Yes |
| `api.kodama.page` | CNAME | Supabase Edge Function URL | Yes |
| `cdn.kodama.page` | CNAME | Cloudflare CDN / R2 | Yes |
| `secret.kodama.page` | CNAME | (reserved) | Yes |
| `poll.kodama.page` | CNAME | (reserved) | Yes |
| `room.kodama.page` | CNAME | (reserved) | Yes |
| `link.kodama.page` | CNAME | (reserved) | Yes |

All records are proxied through Cloudflare (orange cloud enabled).

## SSL/TLS

| Setting | Value |
|---|---|
| Mode | Full (Strict) |
| Minimum TLS | 1.2 |
| Automatic HTTPS Rewrites | Enabled |
| Always Use HTTPS | Enabled |
| HSTS | Enabled (max-age: 31536000) |

Cloudflare manages certificate issuance and renewal automatically.

## Cloudflare Pages

Each frontend is a separate Pages project connected to its GitHub repository.

### Project configuration

| Setting | Value |
|---|---|
| Production branch | `main` |
| Build command | `npm run build` |
| Build output | `dist/` |
| Node.js version | 20 |
| Root directory | `/` |

### Environment variables

Set per-project in Cloudflare Pages Dashboard → Settings → Environment Variables:

| Variable | Production | Preview |
|---|---|---|
| `VITE_API_URL` | `https://api.kodama.page` | `https://api.kodama.page` |
| `NODE_VERSION` | `20` | `20` |

### Preview deployments

Every pull request automatically gets a preview URL:

```
https://{commit-hash}.{project-name}.pages.dev
```

Use preview deployments for visual review before merging.

### Rollback

Cloudflare Pages retains all previous deployments. Rollback is instant:

1. Dashboard → Project → Deployments
2. Find the target deployment
3. Click "Rollback to this deployment"

## CDN (cdn.kodama.page)

Static assets shared across the ecosystem:

- Product icons and illustrations
- Shared UI assets
- Security page diagrams

### Cache configuration

| Asset type | Cache-Control |
|---|---|
| Icons, images | `public, max-age=31536000, immutable` |
| Versioned assets | `public, max-age=31536000, immutable` |
| HTML (if served) | `no-cache` |

## Security

### DDoS protection

Cloudflare provides automatic DDoS mitigation on all proxied domains. No configuration required.

### WAF

Cloudflare managed WAF rules are enabled when available on the current plan. Custom rules may be added for:

- Blocking known bad bots
- Rate limiting at the edge (supplementing application-level rate limits)
- Geo-blocking (if needed)

### Under Attack mode

If experiencing an active attack:

1. Enable "Under Attack" mode in Cloudflare Dashboard.
2. This adds a JavaScript challenge for all visitors.
3. Disable once the attack subsides.

### Security headers

Configure via Cloudflare Transform Rules or `_headers` file in Pages projects:

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## Analytics

Cloudflare Analytics provides per-domain metrics:

- Requests and bandwidth
- Status code distribution
- Cache hit ratio
- Security events (blocked requests, bot traffic)
- Geographic distribution

Access via Cloudflare Dashboard → Analytics & Logs.

## Adding a new subdomain

When launching a new Place:

1. Create a Cloudflare Pages project for the frontend repository.
2. Add a CNAME record: `{product}.kodama.page` → Pages project URL.
3. Enable proxy (orange cloud).
4. Verify SSL certificate is active.
5. Add the domain to CORS allowed origins in the Platform API.

## Related documentation

- [Production Infrastructure](Production%20Infrastructure.md)
- [CI-CD](CI-CD.md)
- [Monitoring](../operations/Monitoring.md)
- [Incident Response](../operations/Incident%20Response.md)
