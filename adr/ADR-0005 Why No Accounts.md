# ADR-0005: Why No Accounts

## Status

Accepted

## Date

2026-07-09

## Context

Every product platform faces a foundational decision: require user accounts or not. Accounts enable personalization, cross-device sync, and user management — but they also introduce signup friction, password management, privacy concerns, and infrastructure complexity.

Kodama's product philosophy is "a place, not a profile." Each Place is a standalone utility accessed via URL.

## Decision

**Kodama will not have user accounts.** Resource identity is `(product_type, slug)`. Access is granted via URL, optional password, or cryptographic capability.

## Rationale

**Why no accounts:**

1. **Frictionless creation.** Creating a Place takes seconds — pick a name, done. No email verification, no password requirements, no onboarding flow.

2. **Privacy by default.** Without accounts, there is no user database to breach. No email addresses, no profile data, no behavioral tracking tied to identity.

3. **Honest security model.** Accounts create an implicit promise: "sign up and we'll keep your stuff safe." Kodama's model is different: "here's a URL with a key — you keep it safe." This is more honest about what the platform can and cannot protect.

4. **Architectural simplicity.** No auth provider, no session management for users, no password reset flows, no account deletion compliance (GDPR right to erasure), no email infrastructure.

5. **Product identity.** "No account. No performance. No noise." Accounts would contradict the brand's core message.

6. **Zero-knowledge compatibility.** Capability-based security (see [ADR-0003](ADR-0003%20Capability-Based%20Security.md)) works naturally without accounts. The URL *is* the credential.

**What we give up:**

- Cross-device sync (user must transfer the URL themselves)
- "My places" dashboard (no list of owned resources)
- Password recovery (by design — see consequences)
- User analytics tied to identity
- Notification systems (email, push)

**What we keep:**

- Full ownership via cryptographic capabilities
- Shareable access without granting ownership
- No signup friction
- No user data to protect (beyond ciphertext)

## Identity model

Without accounts, identity works differently:

| Traditional (accounts) | Kodama (no accounts) |
|---|---|
| User ID | `(product_type, slug)` |
| Login | Open URL |
| Password | Optional; protects owner actions |
| Share | Invite user to workspace | Share capability URL |
| "My stuff" | Dashboard | Bookmark the URL |
| Account recovery | Email reset | Not possible (by design) |

## Consequences

**Positive:**

- Instant Place creation — no signup flow
- No user database to secure or breach
- No GDPR account deletion workflows
- Simpler API (no auth endpoints, no user management)
- Genuine zero-knowledge architecture

**Negative:**

- Users who lose their URL lose access permanently
- No cross-device sync without manual URL transfer
- No "list all my places" feature
- Cannot notify users of security updates
- Some enterprise/compliance scenarios require identifiable users

## Mitigations

| Limitation | Mitigation |
|---|---|
| Lost URL | Clear warnings at creation; export/backup guidance |
| No cross-device sync | QR codes, copy-to-clipboard, manual transfer |
| No place list | Browser bookmarks; future optional encrypted index (local only) |
| No notifications | Public security advisories on `kodama.page/security` |

## When to revisit

Reconsider if:

- A product type genuinely requires persistent user identity (e.g., collaborative editing with attribution)
- Enterprise customers require SSO or audit trails
- Regulatory requirements mandate identifiable users

Any account system must be optional and product-specific — never a platform-wide requirement.

## Related

- [ADR-0003 Capability-Based Security](ADR-0003%20Capability-Based%20Security.md)
- [ADR-0006 Why Places](ADR-0006%20Why%20Places.md)
- [Overall Architecture](../architecture/Overall%20Architecture.md)
