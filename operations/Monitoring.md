# Monitoring

How the Kodama platform is monitored for availability, performance, and security.

## Monitoring stack

| Layer | Tool | What it monitors |
|---|---|---|
| DNS / CDN | Cloudflare Analytics | Traffic, cache rates, security events, SSL |
| API | Supabase Edge Function logs | Request logs, errors, latency |
| Database | Supabase Dashboard | Connections, query performance, storage |
| Uptime | External uptime checker (planned) | Endpoint availability |
| Frontend | Browser console (dev) | Client-side errors |

## Key metrics

### Availability

| Metric | Target | Alert threshold |
|---|---|---|
| API uptime | 99.9% | 3 consecutive failed health checks |
| Frontend uptime | 99.9% | Cloudflare alert on 5xx rate > 1% |
| Database connectivity | 99.99% | Supabase connection errors |

### Performance

| Metric | Target | Alert threshold |
|---|---|---|
| API response time (p95) | < 500ms | > 2000ms sustained for 5 minutes |
| Frontend load time (LCP) | < 2.5s | > 4s on production |
| Database query time (p95) | < 100ms | > 500ms sustained |

### Security

| Metric | Alert threshold |
|---|---|
| Rate limit triggers | > 100/hour from single IP |
| Failed auth attempts | > 50/hour on single resource |
| 4xx error rate | > 10% sustained for 10 minutes |
| Unusual traffic patterns | Cloudflare anomaly detection |

## Health check endpoint

The Platform API exposes a health check:

```
GET /v1/health

Response: 200 OK
{
  "status": "ok",
  "version": "1.0.0",
  "timestamp": "2026-07-09T05:00:00Z"
}
```

Monitor this endpoint from an external service at 1-minute intervals.

## Logging

### API logs

Supabase Edge Functions produce structured logs:

```json
{
  "level": "info",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "method": "POST",
  "path": "/v1/note/create",
  "status": 201,
  "durationMs": 45
}
```

Log levels:

| Level | Usage |
|---|---|
| `error` | Unhandled exceptions, 5xx responses |
| `warn` | Rate limit hits, auth failures, deprecated endpoint usage |
| `info` | Request/response summary (method, path, status, duration) |
| `debug` | Detailed request/response bodies (development only) |

### What to log

- Request method, path, status code, duration
- Error reason codes
- Rate limit events
- Authentication failures (without credentials)

### What never to log

- Passwords, tokens, or session credentials
- Plaintext content or encryption keys
- Full request/response bodies for encrypted endpoints
- User IP addresses in plaintext (use hashed IP for rate limiting)

## Alerting

### Current alerts

| Source | Alert | Channel |
|---|---|---|
| Cloudflare | DDoS attack detected | Email |
| Cloudflare | SSL certificate expiring | Email |
| Supabase | Database connection limit | Dashboard notification |

### Planned alerts

| Alert | Channel |
|---|---|
| API health check failure | Email + SMS |
| Error rate spike | Email |
| Database storage > 80% | Email |
| Edge Function cold start latency | Dashboard |

## Dashboards

### Cloudflare Dashboard

- Traffic overview per subdomain
- Cache hit ratio
- Security events (WAF, bot management)
- SSL/TLS status

### Supabase Dashboard

- Database size and connection count
- Edge Function invocations and errors
- Storage usage
- API request volume

## On-call

Kodama is currently a small-team operation without formal on-call rotation. The platform maintainer monitors alerts and responds per [Incident Response](Incident%20Response.md) severity levels.

As the platform grows, establish:

- On-call rotation schedule
- Escalation paths
- Runbooks for common alerts

## Related documentation

- [Incident Response](Incident%20Response.md)
- [Production Infrastructure](../deployment/Production%20Infrastructure.md)
- [Supabase](../deployment/Supabase.md)
- [Cloudflare](../deployment/Cloudflare.md)
