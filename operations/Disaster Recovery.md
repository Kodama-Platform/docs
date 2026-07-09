# Disaster Recovery

Procedures for recovering the Kodama platform from catastrophic failure.

## Recovery objectives

| Metric | Target |
|---|---|
| **RTO** (Recovery Time Objective) | 4 hours |
| **RPO** (Recovery Point Objective) | 24 hours (daily backup cadence) |

These targets reflect the current scale and team size. They will be tightened as the platform matures.

## Failure scenarios

### Scenario 1: Database corruption or loss

**Impact:** All Places inaccessible. API returns 500 errors.

**Recovery:**

1. Confirm the issue via Supabase Dashboard (connection errors, query failures).
2. Check if Supabase infrastructure is experiencing an outage (status.supabase.com).
3. If Supabase outage: wait for resolution; communicate to users.
4. If data corruption:
   a. Stop all API traffic (disable Edge Function or enable maintenance mode).
   b. Restore from the most recent Supabase backup via Dashboard → Database → Backups.
   c. If PITR is available, restore to the point before corruption.
   d. Verify data integrity with spot checks on `kodama_resources`.
   e. Redeploy the API Edge Function.
   f. Verify health check endpoint.
   g. Re-enable traffic.

**Data loss:** Up to 24 hours of changes (since last daily backup).

### Scenario 2: API Edge Function failure

**Impact:** All product frontends unable to communicate with backend.

**Recovery:**

1. Check Supabase Edge Function logs for errors.
2. Attempt redeployment:

```bash
supabase functions deploy kodama-api --project-ref {project-ref}
```

3. If redeployment fails, roll back to the previous function version.
4. Verify with health check: `curl https://api.kodama.page/v1/health`.
5. If Supabase platform issue, monitor status page and communicate.

**Data loss:** None (database unaffected).

### Scenario 3: Frontend deployment failure

**Impact:** Single product subdomain serving broken or blank page.

**Recovery:**

1. Identify the failed deployment in Cloudflare Pages Dashboard.
2. Rollback to the previous deployment (instant, no rebuild).
3. If rollback fails, manually deploy from a known-good commit:

```bash
git checkout {good-commit}
npm run build
# Upload dist/ via Cloudflare Pages direct upload or wrangler
```

4. Verify the product loads and functions correctly.

**Data loss:** None.

### Scenario 4: DNS / domain failure

**Impact:** All or specific subdomains unreachable.

**Recovery:**

1. Check Cloudflare Dashboard for DNS record status.
2. Verify SSL/TLS certificate validity.
3. If records were deleted, restore from Cloudflare audit log.
4. If domain expired, renew immediately via registrar.
5. DNS propagation may take up to 24 hours (typically minutes with Cloudflare).

**Data loss:** None.

### Scenario 5: Credential compromise

**Impact:** Unauthorized access to database, API, or deployment platforms.

**Recovery:**

1. Immediately rotate all compromised credentials:
   - Supabase service role key
   - Cloudflare API token
   - GitHub access tokens
   - `RATE_LIMIT_IP_SALT`
   - `SENDER_VERIFY_SECRET`
2. Redeploy API with new credentials.
3. Audit recent database changes for unauthorized modifications.
4. Review access logs for suspicious activity.
5. Follow [Incident Response](Incident%20Response.md) security incident process.

**Data loss:** Depends on attacker actions. Encrypted content remains protected (zero-knowledge).

### Scenario 6: Cloudflare outage

**Impact:** All frontends and CDN unavailable. API may still be reachable directly.

**Recovery:**

1. Confirm outage at cloudflarestatus.com.
2. API (`api.kodama.page`) may remain functional if DNS resolves directly to Supabase.
3. Communicate to users via alternative channels.
4. Wait for Cloudflare resolution — no local recovery possible.

**Data loss:** None.

## Recovery testing

| Test | Frequency | Procedure |
|---|---|---|
| Database restore | Quarterly | Restore latest backup to a test Supabase project; verify data |
| Frontend rollback | Quarterly | Rollback one product to previous deployment; verify functionality |
| API redeployment | Monthly | Redeploy Edge Function; verify health check |
| Credential rotation | Quarterly | Rotate one credential type; verify all services recover |

Document test results and any issues discovered.

## Communication during recovery

| Timing | Action |
|---|---|
| Incident detected | Internal acknowledgment |
| 15 minutes | Public status update (for SEV-1/2) |
| Every 30 minutes | Progress update until resolved |
| Resolution | All-clear announcement |
| 48 hours post | Post-incident review published |

## What cannot be recovered

These are architectural constraints, not failures:

- **Lost capability URLs** — reader secrets in URL fragments are never stored server-side.
- **Forgotten passwords** — only bcrypt hashes exist; no recovery mechanism.
- **Burned content** — intentionally destroyed per product burn settings.
- **Expired sessions** — short-lived by design.

Users must be informed of these limitations at creation time.

## Related documentation

- [Backup Policy](Backup%20Policy.md)
- [Incident Response](Incident%20Response.md)
- [Production Infrastructure](../deployment/Production%20Infrastructure.md)
