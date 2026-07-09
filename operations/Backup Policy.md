# Backup Policy

How Kodama platform data is backed up and retained.

## Scope

This policy covers all persistent data in the Kodama ecosystem:

- PostgreSQL database (Supabase)
- Object storage (Supabase Storage)
- DNS and domain configuration (Cloudflare)
- Source code (GitHub)

Static frontend builds are not backed up — they are reproducible from source via CI/CD.

## Database backups

### Automated backups (Supabase)

Supabase provides automated daily backups for the production PostgreSQL instance:

| Setting | Value |
|---|---|
| Frequency | Daily |
| Retention | 7 days (Supabase Pro plan) |
| Type | Full database snapshot |
| Recovery | Point-in-time recovery (PITR) available on Pro plan |

### Manual backups

Before major schema migrations or data operations:

1. Create a manual backup via the Supabase Dashboard → Database → Backups.
2. Verify the backup completed successfully.
3. Document the backup in the migration PR or release notes.

```bash
# Local backup via pg_dump (requires direct connection)
pg_dump -h db.{project-ref}.supabase.co -U postgres -d postgres > backup-$(date +%Y%m%d).sql
```

### Backup verification

- Test restore procedures quarterly (see [Disaster Recovery](Disaster%20Recovery.md)).
- Verify backup integrity after each manual backup.
- Monitor Supabase Dashboard for backup failure alerts.

## Object storage backups

Supabase Storage buckets are included in the database backup scope for metadata. File contents are stored in Supabase's underlying object storage with its own redundancy.

For critical attachments:

- Supabase Storage provides built-in redundancy within the region.
- Cross-region replication is not currently configured.
- Deleted files are not recoverable after bucket-level deletion.

## Configuration backups

| Asset | Backup method |
|---|---|
| DNS records | Cloudflare (managed, versioned via audit log) |
| SSL certificates | Cloudflare (auto-managed) |
| Environment variables | Documented in [Environment Variables](../deployment/Environment%20Variables.md); stored in deployment platform |
| Source code | GitHub (distributed, immutable commit history) |
| Database migrations | Git repository (`supabase/migrations/`) |

## Retention schedule

| Data type | Retention | Notes |
|---|---|---|
| Database daily backups | 7 days | Supabase automated |
| Manual pre-migration backups | 30 days | Stored locally or in secure storage |
| Application logs | 30 days | Supabase Edge Function logs |
| Cloudflare analytics | 90 days | Traffic and security events |
| Deleted Place data | Immediate | No soft-delete retention for user data |
| Session tokens | Until expiry | Auto-cleaned by TTL |

## Data that is not backed up

By design, the following cannot be recovered if lost:

- **Encryption keys in URL fragments** — capability URLs are never stored server-side.
- **Owner passwords** — only hashes are stored; passwords cannot be recovered.
- **Burned/expired content** — intentionally destroyed per product settings.

This is a security feature, not a gap. Users must be informed that losing their URL or password means losing access permanently.

## Responsibilities

| Role | Responsibility |
|---|---|
| Platform maintainer | Verify automated backups, perform manual backups before migrations |
| Product developer | Ensure no unrecoverable data is stored without user acknowledgment |
| All contributors | Never store credentials or secrets in the repository |

## Related documentation

- [Disaster Recovery](Disaster%20Recovery.md)
- [Database Standards](../standards/Database%20Standards.md)
- [Supabase](../deployment/Supabase.md)
