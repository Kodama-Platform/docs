# CI-CD

Continuous integration and deployment for the Kodama ecosystem.

## Current state

Kodama is transitioning toward fully automated CI/CD. Current deployment patterns:

| Component | CI | CD | Method |
|---|---|---|---|
| Product frontends | Planned | Automatic | Cloudflare Pages (on push to `main`) |
| Platform API | Planned | Manual | `supabase functions deploy` |
| Database migrations | Planned | Manual | `supabase db push` |
| Documentation | N/A | N/A | GitHub (no deploy) |

## Target pipeline

Every repository should implement this pipeline via GitHub Actions:

```yaml
# .github/workflows/ci.yml (target)
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run typecheck
      - run: npm run test
      - run: npm run build
```

## Frontend deployment (Cloudflare Pages)

Cloudflare Pages provides built-in CI/CD for frontend repositories:

1. Connect the GitHub repository to Cloudflare Pages.
2. Configure build settings:
   - **Build command:** `npm run build`
   - **Build output directory:** `dist`
   - **Node.js version:** 20
3. Set environment variables in Cloudflare Pages project settings.
4. Every push to `main` triggers a production deployment.
5. Every PR triggers a preview deployment with a unique URL.

### Preview deployments

Preview deployments are valuable for reviewing UI changes before merge:

- URL format: `{hash}.{project}.pages.dev`
- Environment: uses production API (or staging when available)
- Lifetime: until the PR is closed or merged

## API deployment

The Platform API deploys separately from frontends:

### Manual deployment (current)

```bash
# 1. Run checks locally
npm run typecheck
npm test

# 2. Deploy Edge Function
supabase functions deploy kodama-api --project-ref {project-ref}

# 3. Verify
curl https://api.kodama.page/v1/health
```

### Target automated deployment

```yaml
# .github/workflows/deploy-api.yml (target)
name: Deploy API

on:
  push:
    branches: [main]
    paths:
      - 'lib/**'
      - 'supabase/functions/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - run: supabase functions deploy kodama-api --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
```

## Database migrations

Migrations are applied manually to maintain control over schema changes:

```bash
# 1. Create backup (see Backup Policy)
# 2. Apply migration
supabase db push --project-ref {project-ref}
# 3. Verify schema
# 4. Deploy API code
```

Automated migration deployment is intentionally deferred until the team size warrants it. Schema changes are infrequent and high-impact.

## Required checks

Before any code reaches production, these checks must pass:

| Check | Command | Required for |
|---|---|---|
| Type check | `npm run typecheck` | All repos |
| Tests | `npm test` | All repos with tests |
| Build | `npm run build` | Frontends |
| Lint | `npm run lint` | All repos (when configured) |
| Secret scan | GitHub secret scanning | All repos (automatic) |

## Branch protection

GitHub branch protection rules for `main`:

- Require PR before merge
- Require status checks to pass
- No force pushes
- No direct commits

See [Branch Strategy](../standards/Branch%20Strategy.md) for branching conventions.

## Rollback procedures

| Component | Rollback method | Time |
|---|---|---|
| Frontend | Cloudflare Pages → previous deployment | Instant |
| API | Redeploy previous Edge Function version | ~1 minute |
| Database | Restore from backup | ~30 minutes |

See [Release Process](../operations/Release%20Process.md) for detailed rollback steps.

## Secrets management

CI/CD secrets are stored in GitHub repository secrets:

| Secret | Used by |
|---|---|
| `SUPABASE_ACCESS_TOKEN` | API deployment workflow |
| `SUPABASE_PROJECT_REF` | API deployment workflow |
| `CLOUDFLARE_API_TOKEN` | Pages deployment (if using wrangler) |

Never log secrets in CI output. Use `${{ secrets.NAME }}` syntax in workflows.

## Related documentation

- [Release Process](../operations/Release%20Process.md)
- [Branch Strategy](../standards/Branch%20Strategy.md)
- [Production Infrastructure](Production%20Infrastructure.md)
- [Environment Variables](Environment%20Variables.md)
