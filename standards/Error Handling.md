# Error Handling

Standardized error handling across the Kodama ecosystem.

## Principles

1. **Predictable** — Every error has a machine-readable reason code and a human-readable message.
2. **Safe** — Never expose stack traces, internal paths, or implementation details.
3. **Actionable** — Error messages tell the user what went wrong and what they can do.
4. **Consistent** — The same error format is used in API responses, client library exceptions, and UI displays.

## API error format

All API errors return this structure:

```json
{
  "error": {
    "reason": "slug_taken",
    "message": "This slug is already in use for this product.",
    "details": {
      "slug": "my-slug",
      "productType": "note"
    }
  },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | Yes | Machine-readable error code |
| `message` | string | Yes | Human-readable description |
| `details` | object | No | Additional context for debugging |

## Error reason codes

### Resource errors

| Reason | HTTP | Description |
|---|---|---|
| `not_found` | 404 | Resource does not exist |
| `slug_taken` | 409 | Slug already in use for this product type |
| `slug_reserved` | 400 | Slug is in the reserved list |
| `invalid_slug` | 400 | Slug fails validation (format, length) |
| `stale_version` | 409 | Optimistic concurrency conflict |

### Authentication errors

| Reason | HTTP | Description |
|---|---|---|
| `invalid_password` | 401 | Wrong password for resource |
| `invalid_token` | 401 | Invalid or expired session token |
| `missing_auth` | 401 | Authentication required but not provided |
| `forbidden` | 403 | Authenticated but lacks permission |

### Rate limiting

| Reason | HTTP | Description |
|---|---|---|
| `rate_limited` | 429 | Too many requests; includes `Retry-After` header |

### Validation errors

| Reason | HTTP | Description |
|---|---|---|
| `invalid_request` | 400 | Malformed request body or missing fields |
| `payload_too_large` | 413 | Request body exceeds size limit |
| `unsupported_media` | 415 | Wrong Content-Type |

### Server errors

| Reason | HTTP | Description |
|---|---|---|
| `internal_error` | 500 | Unexpected server failure |
| `service_unavailable` | 503 | Temporary outage or maintenance |

## Client-side error handling

The `@kodama/api-client` library throws typed exceptions:

```typescript
import { ApiError } from "@kodama/api-client";

try {
  await client.note.create({ slug: "taken-slug" });
} catch (error) {
  if (error instanceof ApiError) {
    switch (error.reason) {
      case "slug_taken":
        showError("This name is already taken. Try another.");
        break;
      case "rate_limited":
        showError("Too many requests. Please wait a moment.");
        break;
      default:
        showError("Something went wrong. Please try again.");
    }
  }
}
```

## UI error display

Product frontends should:

- Show user-friendly messages, not raw error codes.
- Provide actionable guidance ("Try a different name" not "slug_taken").
- Never display stack traces or technical details to users.
- Log full error details to the console in development mode only.

## Zero-knowledge guard

Encrypted products (Note, Secret) must validate payloads before sending to the server:

```typescript
function assertNoSecretsInPayload(payload: Record<string, unknown>): void {
  const forbidden = /password|plaintext|^text$|^content$|secret|private/i;
  for (const key of Object.keys(payload)) {
    if (forbidden.test(key)) {
      throw new Error(`Secret field "${key}" must not be sent to the server`);
    }
  }
}
```

If this guard triggers, it is a **client-side bug** — not an API error. Log it as an error in development and prevent the request from being sent.

## Logging

| Environment | What to log | Where |
|---|---|---|
| Production API | Error reason, request ID, endpoint, status code | Supabase Edge Function logs |
| Production frontend | Error reason only (no PII) | Browser console (dev only) |
| Development | Full error object including stack trace | Console |

Never log:

- Passwords or tokens
- Plaintext content
- Encryption keys
- Full request/response bodies for encrypted endpoints

## Adding new error codes

1. Add the reason code to this document.
2. Add it to the `ApiErrorReason` type in `@kodama/api-client`.
3. Handle it in the relevant product frontend.
4. If the error represents an architectural decision, consider an ADR.

## Related documentation

- [API Standards](API%20Standards.md)
- [Coding Standards](Coding%20Standards.md)
