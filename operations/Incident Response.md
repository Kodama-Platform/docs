# Incident Response

How to detect, respond to, and recover from incidents affecting the Kodama platform.

## Severity levels

| Level | Description | Response time | Examples |
|---|---|---|---|
| **SEV-1** | Platform down or data breach | Immediate | API unreachable, database compromised, active exploit |
| **SEV-2** | Major feature broken | < 1 hour | Note creation fails, Drop messages not delivering |
| **SEV-3** | Degraded performance or minor bug | < 4 hours | Slow API responses, UI rendering issue |
| **SEV-4** | Cosmetic or non-urgent | Next business day | Typo, minor styling issue |

## Detection

Incidents may be detected through:

- **Cloudflare alerts** — traffic anomalies, DDoS, SSL issues
- **Supabase monitoring** — database errors, Edge Function failures
- **User reports** — via support channels or social media
- **Security reports** — via `security@kodama.page` (see responsible disclosure below)
- **Automated monitoring** — uptime checks, error rate thresholds (see [Monitoring](Monitoring.md))

## Response process

### 1. Acknowledge

- Confirm the incident is real (not a false alarm).
- Assign a severity level.
- Begin a timeline log (timestamp, actions, observations).

### 2. Communicate

| Audience | Channel | Timing |
|---|---|---|
| Internal team | Direct message / chat | Immediately |
| Users (SEV-1/2) | Status page or social media | Within 30 minutes |
| Security researchers | Reply to reporter | Within 24 hours |

For SEV-1 incidents, post a brief public acknowledgment even before the root cause is known.

### 3. Mitigate

Take the fastest action to stop the bleeding:

| Incident type | Immediate action |
|---|---|
| API down | Check Supabase Edge Function logs; redeploy if needed |
| Database issue | Check Supabase Dashboard; restore from backup if corrupted |
| DDoS / abuse | Enable Cloudflare "Under Attack" mode; tighten rate limits |
| Data breach | Rotate all credentials; assess scope; notify affected users |
| Bad deploy | Rollback to previous Cloudflare Pages deployment |

### 4. Resolve

- Identify and fix the root cause.
- Deploy the fix following [Release Process](Release%20Process.md).
- Verify the fix in production.
- Monitor for recurrence for 24 hours.

### 5. Post-incident review

Within 48 hours of resolution, document:

- Timeline of events
- Root cause
- Impact (users affected, duration, data involved)
- What went well
- What could be improved
- Action items with owners and deadlines

Store post-incident reviews in this repository or a linked issue tracker.

## Security incidents

### Responsible disclosure

Kodama follows a coordinated disclosure policy:

- Contact: `security@kodama.page`
- PGP key: available at `/.well-known/security.txt` on `kodama.page`
- Scope: `kodama.page` and all official subdomains
- Window: 90 days for coordinated disclosure

### Security incident checklist

1. **Confirm** the vulnerability is real and in scope.
2. **Assess** impact — what data or systems are affected?
3. **Contain** — disable affected endpoints, rotate compromised credentials.
4. **Fix** — develop and test a patch.
5. **Deploy** — release the fix to production.
6. **Notify** — inform the reporter; publish advisory if warranted.
7. **Review** — post-incident review with focus on prevention.

### What we can and cannot claim

After a security incident, be accurate about impact:

**Can claim:**
- Database compromise does not reveal encrypted content (zero-knowledge).
- Server-side passwords are bcrypt-hashed.

**Cannot claim:**
- Protection against compromised user devices.
- Recovery of lost passwords or capability URLs.
- Protection against state-level adversaries.

## Escalation

| Condition | Action |
|---|---|
| SEV-1 unresolved after 1 hour | Escalate to all platform maintainers |
| Data breach confirmed | Begin breach notification process |
| Security report received | Acknowledge within 24 hours; begin assessment |
| Third-party service outage (Supabase, Cloudflare) | Monitor status page; communicate to users |

## Related documentation

- [Monitoring](Monitoring.md)
- [Disaster Recovery](Disaster%20Recovery.md)
- [Release Process](Release%20Process.md)
- [Backup Policy](Backup%20Policy.md)
