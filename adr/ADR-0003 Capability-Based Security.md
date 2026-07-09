# ADR-0003: Capability-Based Security

## Status

Accepted

## Date

2026-07-09

## Context

Kodama products like Note and Secret handle sensitive user content. We need an access control model that:

- Works without user accounts
- Keeps encryption keys out of server reach
- Supports multiple access levels (read, edit, own)
- Enables secure sharing via URLs

Options considered:

1. **Password-only** — single password grants full access
2. **Token-based sessions** — server-issued tokens after authentication
3. **Capability-based security** — URLs contain secrets in the fragment; server stores only ciphertext

## Decision

Adopt **capability-based security** as defined in the Kodama Security Protocol v1.

Access is controlled by secrets embedded in URL fragments (`#reader_secret=...`), which are never sent to the server. The server stores only encrypted data and public keys.

## Access model

| Role | Capability | Permissions |
|---|---|---|
| **Reader** | `reader_secret` in URL `#fragment` | Decrypt and read content |
| **Editor** | Reader capability + Ed25519 private key | Read and edit (signed updates) |
| **Owner** | Password (never transmitted to server) | Full control: read, edit, rotate keys, delete |

## Key hierarchy

```
Password
  └── Argon2id ──→ Master Secret
                      ├── HKDF ──→ Reader Secret (shared via URL fragment)
                      ├── HKDF ──→ Owner Auth Hash (stored server-side)
                      └── HKDF ──→ Content Encryption Key (AES-256-GCM)
                                      └── Wrapped CEK (stored server-side)
```

## Rationale

**Why not password-only:**

- A single password cannot distinguish read vs. edit vs. own permissions
- Sharing read access would require sharing the password, granting full control
- Password transmission to the server breaks zero-knowledge guarantees

**Why not server-issued tokens:**

- Requires the server to authenticate users — contradicts the no-accounts model
- Tokens stored server-side become a target for theft
- Does not support zero-knowledge encryption

**Why capability-based:**

- URL fragments are never sent to the server (HTTP spec)
- Different capabilities grant different permissions
- Sharing read access does not grant edit or ownership
- Compatible with zero-knowledge encryption
- No server-side session state required for read access

## Consequences

**Positive:**

- True zero-knowledge: server compromise does not reveal content
- Granular access control without accounts
- Share links are revocable by rotating keys (owner action)
- Cryptographically verifiable edit authorship (Ed25519 signatures)

**Negative:**

- Lost capability URL = lost access (no recovery)
- URL fragments are not sent on navigation — requires client-side routing
- More complex client-side crypto implementation
- Key rotation requires owner password (cannot be automated)

## Implementation status

The Security Protocol v1 is fully specified in `kodama-security-protocol`. Note currently uses a simplified edit-token model and is migrating to full capability-based security.

## Related

- [ADR-0005 Why No Accounts](ADR-0005%20Why%20No%20Accounts.md)
- [ADR-0004 Why Supabase](ADR-0004%20Why%20Supabase.md)
- Security protocol specification: `kodama-security-protocol` repository
