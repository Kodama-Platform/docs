# Release Process

How changes move from development to production across the Kodama ecosystem.

## Overview

Kodama uses **continuous deployment** from the `main` branch. There are no scheduled release cycles — changes deploy when merged.

```
Developer → PR → Review → Merge to main → CI build → Deploy to production
```

## Release types

| Type | Trigger | Deploy target |
|---|---|---|
| **Frontend** | Merge to product repo `main` | Cloudflare Pages (automatic) |
| **API** | Merge to `api.kodama.page` `main` | Supabase Edge Functions (manual or CI) |
| **Database** | Migration merged to `api.kodama.page` `main` | Supabase (manual apply) |
| **Documentation** | Merge to `kodama-docs` `main` | GitHub (no deploy needed) |

## Frontend release

Product frontends deploy automatically via Cloudflare Pages:

1. Developer merges PR to `main`.
2. Cloudflare Pages detects the push and runs `npm run build`.
3. Build output (`dist/`) is deployed to the product subdomain.
4. Previous deployment remains available for instant rollback.

### Pre-merge checklist

- [ ] Code builds without errors (`npm run build`)
- [ ] Tests pass (`npm test`)
- [ ] No secrets in the build output
- [ ] Environment variables documented if new ones were added
- [ ] API changes deployed first (if this frontend depends on new API endpoints)

### Rollback

In Cloudflare Pages Dashboard:

1. Go to the project's Deployments tab.
2. Find the last known-good deployment.
3. Click "Rollback to this deployment".

Rollback is instant — no rebuild required.

## API release

The Platform API requires a manual deploy step after merging:

1. Merge PR to `api.kodama.page` `main`.
2. Deploy the Edge Function:

```bash
supabase functions deploy kodama-api --project-ref {project-ref}
```

3. Verify the deployment:

```bash
curl https://api.kodama.page/v1/health
```

4. Monitor Edge Function logs for errors.

### Database migrations

When a PR includes schema changes:

1. Review the migration SQL carefully.
2. Create a manual backup (see [Backup Policy](Backup%20Policy.md)).
3. Apply the migration:

```bash
supabase db push --project-ref {project-ref}
```

4. Verify schema in Supabase Dashboard.
5. Deploy the API code that uses the new schema.

**Order: backup → migrate → deploy API → deploy frontends.**

## Coordinated releases

Changes spanning multiple repositories follow this order:

```
1. api.kodama.page  (schema + API endpoints)
2. @kodama/api-client  (updated types and methods)
3. product frontend  (uses new client)
4. kodama-web  (marketing updates, if applicable)
5. kodama-docs  (documentation updates)
```

Each step must be verified before proceeding to the next.

## Version tagging

Kodama does not use git tags for releases. Deployment history is tracked by:

- Cloudflare Pages deployment log (frontends)
- Supabase Edge Function deployment log (API)
- Git commit history on `main`

`@kodama/api-client` is the only component with semver tags, published via npm.

## Release communication

| Change type | Communication |
|---|---|
| New product launch | Update marketing site, announce publicly |
| Breaking API change | Update [Breaking Changes](../roadmap/Breaking%20Changes.md), notify product maintainers |
| Security fix | Follow [Incident Response](Incident%20Response.md) disclosure process |
| Documentation | No external communication needed |

## Environment promotion

Currently, Kodama operates with two environments:

| Environment | Purpose |
|---|---|
| **Local** | Developer machines (`supabase start` + Vite dev server) |
| **Production** | Live platform (`*.kodama.page`) |

A staging environment is planned but not yet implemented. Until then:

- Test thoroughly in local development.
- Use Cloudflare Pages preview deployments for frontend review.
- Apply database migrations to production with backup first.

## Related documentation

- [Branch Strategy](../standards/Branch%20Strategy.md)
- [CI-CD](../deployment/CI-CD.md)
- [Versioning Policy](../standards/Versioning%20Policy.md)
- [Breaking Changes](../roadmap/Breaking%20Changes.md)
