# Coding Standards

Standards that apply to every Kodama repository.

## Languages

| Layer | Language | Minimum version |
|---|---|---|
| Frontends | TypeScript | 5.x |
| Platform API | TypeScript (Deno) | 5.x |
| Database migrations | SQL | PostgreSQL 15 |
| Scripts | TypeScript or shell | — |

JavaScript files are not permitted in new code. All source files must be TypeScript.

## Formatting

- **Indentation:** 2 spaces (no tabs)
- **Semicolons:** omitted (ASI)
- **Quotes:** double quotes for strings
- **Trailing commas:** required in multiline structures
- **Line length:** 100 characters soft limit

Use the project's configured formatter (Prettier or equivalent). Do not hand-format.

## Naming conventions

| Element | Convention | Example |
|---|---|---|
| Files (components) | PascalCase | `PlaceHero.tsx` |
| Files (utilities) | kebab-case | `slug-validator.ts` |
| Files (routes) | kebab-case | `create-place.ts` |
| Variables / functions | camelCase | `validateSlug()` |
| Constants | UPPER_SNAKE_CASE | `MAX_SLUG_LENGTH` |
| Types / interfaces | PascalCase | `ApiResponse` |
| Enums | PascalCase (members too) | `ProductType.Note` |
| CSS classes | kebab-case | `place-header` |
| Database tables | snake_case with prefix | `kodama_resources` |
| Database columns | snake_case | `created_at` |
| Environment variables | UPPER_SNAKE_CASE | `VITE_API_URL` |
| API routes | kebab-case | `/v1/note/create` |

## Folder structure

Every frontend repository follows this layout:

```
src/
├── components/       # Reusable UI components
│   └── ui/           # shadcn/ui primitives
├── lib/              # Utilities, helpers, constants
├── routes/           # Route/page components
├── hooks/            # Custom React hooks
└── types/            # Shared TypeScript types
```

Platform API repositories:

```
lib/
├── db.ts             # Database client
├── middleware/        # Hono middleware (CORS, rate limit, auth)
├── routes/           # Route handlers by product type
│   ├── note/
│   └── drop/
└── utils/            # Shared utilities
```

## TypeScript rules

- Enable `strict` mode in `tsconfig.json`.
- No `any` — use `unknown` and narrow with type guards.
- Prefer `interface` for object shapes, `type` for unions and intersections.
- Export types alongside the functions that use them.
- Use Zod schemas for runtime validation at API boundaries.

```typescript
// Good
interface CreatePlaceRequest {
  slug: string;
  productType: ProductType;
}

// Bad
function createPlace(data: any) { ... }
```

## React conventions

- Functional components only — no class components.
- Use hooks for state and side effects.
- Colocate component-specific hooks in the same file or a `use-{name}.ts` file.
- Prefer composition over prop drilling; use context sparingly.
- Server state via TanStack Query — no manual fetch + useState for API data.

## Import order

1. External packages (`react`, `@tanstack/react-query`)
2. Internal aliases (`@/components`, `@/lib`)
3. Relative imports (`./PlaceHero`)
4. Type-only imports (`import type { ... }`)

Separate each group with a blank line.

## Error handling

- Never swallow errors silently.
- Use typed error responses at API boundaries (see [Error Handling](Error%20Handling.md)).
- Client-side: display user-friendly messages, log details to console in development.
- Never expose stack traces or internal details in API responses.

## Security rules

- **Never** send plaintext secrets to the server. Use `assertNoSecretsInPayload()` before API calls in encrypted products.
- **Never** log passwords, tokens, or encryption keys.
- **Never** store secrets in localStorage without encryption.
- Validate all user input with Zod schemas before processing.
- Sanitize any user-generated content rendered as HTML.

## Testing

- Unit tests for business logic, crypto functions, and validators.
- Test files colocated: `slug-validator.test.ts` next to `slug-validator.ts`.
- Use Vitest as the test runner.
- Name tests descriptively: `it("rejects slugs longer than 80 characters")`.

## Comments

- Code should be self-explanatory. Prefer clear naming over comments.
- Add comments only for non-obvious business logic or security-critical behavior.
- Use JSDoc for public API functions and exported types.

## Related documentation

- [API Standards](API%20Standards.md)
- [Error Handling](Error%20Handling.md)
- [Database Standards](Database%20Standards.md)
- [Commit Convention](Commit%20Convention.md)
