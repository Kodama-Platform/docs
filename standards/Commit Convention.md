# Commit Convention

Commit message format for all Kodama repositories.

## Format

```
type(scope): short description

Optional longer explanation of why this change was made,
not what it does (the diff shows what).
```

### Types

| Type | Usage |
|---|---|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace (no logic change) |
| `refactor` | Code restructuring (no feature or fix) |
| `test` | Adding or updating tests |
| `chore` | Maintenance, dependencies, tooling |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration |

### Scope

The scope identifies the affected area. Use product type, module name, or repository context:

```
feat(note): add burn-after-read mode
fix(api): handle stale version conflicts
docs(adr): add ADR-0007 for Cloudflare
chore(deps): update drizzle-orm to 0.38
```

Common scopes:

| Scope | Applies to |
|---|---|
| `note` | Kodama Note |
| `drop` | Kodama Drop |
| `api` | Platform API |
| `client` | `@kodama/api-client` |
| `db` | Database schema/migrations |
| `web` | Marketing site |
| `adr` | Architecture Decision Records |

Scope is optional for small or obvious changes:

```
docs: update README with repo map
fix: correct typo in error message
```

## Rules

1. **Subject line:** imperative mood, lowercase, no period, max 72 characters.
2. **Body:** wrap at 72 characters. Explain *why*, not *what*.
3. **One commit per logical change.** Do not bundle unrelated changes.
4. **Reference issues** when applicable: `fix(note): prevent double submission (#42)`.

## Examples

Good:

```
feat(drop): add verified sender via linked Drop

Allow Drop owners to require senders to prove ownership
of another Drop resource before submitting messages.
```

```
fix(api): return 409 instead of 500 for slug conflicts
```

```
docs(adr): add ADR-0004 explaining Supabase choice
```

Bad:

```
updated stuff
```

```
Fix bug
```

```
feat: added a bunch of changes to the note app including new editor features, updated dependencies, and fixed some bugs
```

## Squash merge

PRs are merged via squash merge. The squash commit message should follow this convention:

```
feat(note): add capability URL sharing (#15)
```

The PR title becomes the commit message on main. Write PR titles accordingly.

## Related documentation

- [Branch Strategy](Branch%20Strategy.md)
- [Coding Standards](Coding%20Standards.md)
