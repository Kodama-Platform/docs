# Dependency Policy

Rules for managing dependencies across the Kodama ecosystem.

## Principles

1. **Minimal dependencies.** Every dependency is a liability — prefer standard library and built-in APIs.
2. **Pinned majors.** Lock major versions; allow minor and patch updates.
3. **Audited regularly.** Review dependencies quarterly for security advisories.
4. **Consistent across repos.** Products sharing a stack should use the same major versions.

## Approved stack

These are the standard dependencies for Kodama projects. Deviating requires justification.

### Frontend (all product SPAs)

| Dependency | Purpose | Version |
|---|---|---|
| React | UI framework | 19.x |
| Vite | Build tool | 8.x |
| TypeScript | Language | 5.x |
| TanStack Router | Routing | latest |
| TanStack Query | Server state | latest |
| Tailwind CSS | Styling | 4.x |
| shadcn/ui | UI components | latest |
| Radix UI | Accessible primitives | latest |
| Zod | Runtime validation | latest |
| Vitest | Testing | latest |

### Platform API

| Dependency | Purpose | Version |
|---|---|---|
| Hono | HTTP framework | 4.x |
| Drizzle ORM | Database queries | latest |
| bcryptjs | Password hashing | latest |
| Zod | Request validation | latest |

### Shared

| Dependency | Purpose | Version |
|---|---|---|
| `@kodama/api-client` | Typed API client | semver |

## Adding a dependency

Before adding a new dependency, answer:

1. **Is it necessary?** Can the standard library or an existing dependency do this?
2. **Is it maintained?** Last commit within 6 months? Active issue responses?
3. **Is it secure?** No known critical vulnerabilities?
4. **Is it sized appropriately?** Check bundle size impact for frontend deps (use `bundlephobia.com`).
5. **Is it licensed compatibly?** MIT, Apache 2.0, or BSD preferred. No GPL in frontend bundles.

If all answers are satisfactory, add it. Document the choice in the PR description.

## Forbidden dependencies

| Category | Reason |
|---|---|
| jQuery, lodash (full) | Use native APIs or targeted utilities |
| moment.js | Use native `Intl` or `date-fns` |
| Supabase client in frontends | Use `@kodama/api-client` instead |
| Crypto libraries (Node) | Use Web Crypto API in browsers, Deno crypto in API |
| ORMs in frontends | All data access via API |

## Package managers

| Repository | Manager | Lockfile |
|---|---|---|
| `api.kodama.page` | npm | `package-lock.json` |
| `drop.kodama.page` | npm | `package-lock.json` |
| `note.kodama.page` | yarn | `yarn.lock` |
| `brand.kodama.page` | yarn | `yarn.lock` |
| `kodama-web` | npm | `package-lock.json` |

Do not mix package managers within a single repository. Do not commit multiple lockfiles.

## Updating dependencies

### Patch and minor updates

- Update freely for security patches.
- Run tests after updating.
- Commit with `chore(deps): update {package} to {version}`.

### Major updates

- Review changelog for breaking changes.
- Update one major dependency at a time.
- Test thoroughly before merging.
- Update all repos using the same dependency to maintain consistency.

### Automated updates

Dependabot or Renovate may be configured per repository. Auto-merge is permitted for patch-level updates to dev dependencies only.

## Security auditing

Run `npm audit` (or equivalent) before each release. Critical and high vulnerabilities must be resolved before deploying.

For frontend dependencies, also check that no secrets or credentials are bundled into the production build.

## Internal packages

`@kodama/api-client` is the only internal package currently published. It lives in `api.kodama.page/packages/api-client/`.

Future internal packages follow the same `@kodama/{name}` scope and semver rules.

## Related documentation

- [Coding Standards](Coding%20Standards.md)
- [Versioning Policy](Versioning%20Policy.md)
