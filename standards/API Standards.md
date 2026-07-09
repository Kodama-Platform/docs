# API Standards

Conventions for the Kodama Platform API at `api.kodama.page`.

## Base URL

```
Production:  https://api.kodama.page
Local dev:   http://localhost:8000
```

## Versioning

All routes are prefixed with `/v1/`:

```
POST /v1/note/create
GET  /v1/drop/{slug}/messages
POST /v1/drop/{slug}/send
```

Breaking changes require a new version prefix (`/v2/`). See [Versioning Policy](Versioning%20Policy.md).

## Request format

- Content-Type: `application/json`
- Accept: `application/json`
- Authentication via headers (session tokens, not cookies)

### Headers

| Header | Purpose | Required |
|---|---|---|
| `Content-Type` | Must be `application/json` for POST/PUT/PATCH | Yes (mutations) |
| `Authorization` | Bearer session token | When authenticated |
| `X-Request-Id` | Client-generated UUID for tracing | Recommended |

## Response format

### Success response

```json
{
  "data": { ... },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

For list endpoints, include pagination:

```json
{
  "data": [ ... ],
  "meta": {
    "requestId": "...",
    "page": 1,
    "pageSize": 20,
    "total": 142
  }
}
```

### Error response

```json
{
  "error": {
    "reason": "slug_taken",
    "message": "This slug is already in use for this product.",
    "details": {}
  },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

See [Error Handling](Error%20Handling.md) for the full error vocabulary.

## HTTP status codes

| Code | Usage |
|---|---|
| `200` | Successful GET, PUT, PATCH |
| `201` | Successful POST (resource created) |
| `204` | Successful DELETE (no body) |
| `400` | Invalid request (bad slug, missing fields) |
| `401` | Missing or invalid authentication |
| `403` | Authenticated but not authorized |
| `404` | Resource not found |
| `409` | Conflict (slug taken, stale version) |
| `429` | Rate limited |
| `500` | Internal server error |

## Route naming

Routes follow REST conventions with product-type namespacing:

```
/v1/{product_type}/{resource}/{action}
```

| Pattern | Example | Method |
|---|---|---|
| Create | `POST /v1/note/create` | POST |
| Read | `GET /v1/note/{slug}` | GET |
| Update | `PUT /v1/note/{slug}` | PUT |
| Delete | `DELETE /v1/note/{slug}` | DELETE |
| Sub-resource | `GET /v1/drop/{slug}/messages` | GET |
| Action | `POST /v1/drop/{slug}/send` | POST |

Use nouns for resources, verbs only for non-CRUD actions (`send`, `unlock`, `rotate`).

## Slug validation

All endpoints accepting a slug must validate:

- Pattern: `^[a-z0-9]+(-[a-z0-9]+)*$`
- Length: 1–80 characters
- Not in reserved list (`admin`, `api`, `create`, `login`, `owner`, `settings`, etc.)
- Unique per product type

Return `400` with reason `invalid_slug` or `409` with reason `slug_taken`.

## Rate limiting

Rate limits are enforced per IP address using hashed IP buckets:

| Endpoint type | Limit |
|---|---|
| Create | 10 requests / hour |
| Read | 100 requests / minute |
| Send (Drop) | 5 requests / minute |
| Auth attempts | 5 requests / 15 minutes |

When rate limited, return `429` with reason `rate_limited` and a `Retry-After` header.

## CORS

Allowed origins:

- `https://*.kodama.page` (all product subdomains)
- `https://kodama.page`
- `http://localhost:*` (development only)

No other origins are permitted. Credentials are not included in CORS requests.

## Client library

All frontends must use `@kodama/api-client` — never raw `fetch` to the Platform API:

```typescript
import { createClient } from "@kodama/api-client";

const client = createClient({ baseUrl: import.meta.env.VITE_API_URL });
const place = await client.note.get("my-slug");
```

The client library handles:

- Base URL configuration
- Request/response typing
- Error parsing into typed exceptions
- Request ID generation

## Idempotency

- Create endpoints are **not** idempotent — duplicate slugs return `409`.
- Delete endpoints **are** idempotent — deleting a non-existent resource returns `204`.
- Update endpoints use optimistic concurrency via version numbers — stale updates return `409` with reason `stale_version`.

## Related documentation

- [Error Handling](Error%20Handling.md)
- [Versioning Policy](Versioning%20Policy.md)
- [Database Standards](Database%20Standards.md)
