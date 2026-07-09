# Branch Strategy

Git branching conventions for all Kodama repositories.

## Branch model

Kodama uses a simplified **trunk-based** workflow with a protected main branch.

```
main ─────●─────●─────●─────●─────●───── (always deployable)
           \         /       \
            feature/a         fix/b
```

### Branches

| Branch | Purpose | Lifetime |
|---|---|---|
| `main` | Production-ready code | Permanent |
| `feature/{description}` | New features or enhancements | Deleted after merge |
| `fix/{description}` | Bug fixes | Deleted after merge |
| `docs/{description}` | Documentation-only changes | Deleted after merge |
| `chore/{description}` | Maintenance, dependency updates | Deleted after merge |

No `develop`, `staging`, or long-lived release branches.

## Rules

1. **`main` is always deployable.** Never push broken code to main.
2. **Branch from `main`.** Create feature/fix branches from the latest main.
3. **Keep branches short-lived.** Merge within days, not weeks.
4. **One concern per branch.** A branch should address one feature, fix, or doc change.
5. **Delete after merge.** Remote branches are deleted automatically or manually after merging.

## Branch naming

Use lowercase kebab-case with a type prefix:

```
feature/note-capability-urls
fix/drop-rate-limit-bypass
docs/add-monitoring-guide
chore/update-drizzle-orm
```

For documentation in this repo:

```
docs/adr-0007-why-cloudflare
docs/update-api-standards
```

## Pull requests

All changes to `main` go through pull requests:

1. Create a branch from `main`.
2. Make changes following [Coding Standards](Coding%20Standards.md).
3. Open a PR with a clear title following [Commit Convention](Commit%20Convention.md).
4. Self-review before requesting review.
5. Merge via squash merge to keep main history clean.

### PR title format

```
type(scope): description
```

Examples:

```
feat(note): add capability URL sharing
fix(api): correct rate limit bucket reset
docs: add disaster recovery plan
chore: update @kodama/api-client to v1.2.0
```

## Protected branches

The `main` branch is protected:

- Direct pushes are disabled.
- PRs require at least one approval (when team size permits).
- Status checks must pass before merge.
- Force push is disabled.

## Hotfixes

For urgent production fixes:

1. Branch from `main`: `fix/critical-issue`.
2. Fix, test, and open a PR.
3. Merge and deploy immediately.
4. No special hotfix branch or release process — trunk-based means main is always the target.

## Cross-repository changes

Changes that span multiple repositories (e.g., API change + frontend update):

1. Merge the API change first.
2. Deploy the API.
3. Then merge and deploy the frontend change.
4. Document the coordination in the PR descriptions.

Order matters: backend first, frontend second. This prevents frontends from calling endpoints that do not yet exist.

## Related documentation

- [Commit Convention](Commit%20Convention.md)
- [Release Process](../operations/Release%20Process.md)
- [CI-CD](../deployment/CI-CD.md)
