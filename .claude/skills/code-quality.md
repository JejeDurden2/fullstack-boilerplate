# Code Quality & Review

## Rules for Claude

### ALWAYS

1. Create reusable, typed components — no one-off implementations
2. Implement mobile-first, then scale up (sm → md → lg → xl)
3. Add proper TypeScript types with explicit return types
4. Consider SEO impact (metadata, structure, performance, Core Web Vitals)
5. Follow hexagonal architecture — domain has NO infrastructure dependencies
6. Use conventional commits with proper scope
7. Handle errors with Result pattern + typed DomainError classes
8. Write tests for business logic
9. Use dependency injection for all services
10. Add structured logging with context (traceId, spanId, relevant metadata)
11. Use docker-compose for local dev, Neon for production
12. **Self-review code before committing** (see checklist below)
13. **Use i18n for ALL user-facing strings** — French first, then English

### NEVER

14. Use `any` type — use `unknown` + type guards
15. Put business logic in controllers or components
16. Skip validation (Zod on frontend, class-validator on backend)
17. Hardcode values — use environment variables or constants
18. Commit without running lint and type-check
19. Throw generic Error — use typed DomainError subclasses
20. Log sensitive data (passwords, tokens, PII)
21. Push code without self-review
22. Hardcode user-facing text — use translation keys

### PREFER

23. Composition over inheritance
24. Small, focused functions (< 20 lines)
25. Named exports over default exports
26. Interface for objects, type for unions
27. Early returns over nested conditions
28. Explicit over implicit
29. Result.err() over throwing exceptions in use cases
30. Translation interpolation over string concatenation

---

## Pre-Commit Checklist

### 1. Static Analysis

```bash
pnpm lint              # Zero errors, zero warnings
pnpm type-check        # Zero TypeScript errors
pnpm format:check      # Formatting consistent
```

### 2. Code Quality Review

#### TypeScript

- [ ] No `any` types — all types explicit or properly inferred
- [ ] Explicit return types on all exported functions
- [ ] No type assertions (`as`) unless absolutely necessary with comment
- [ ] Null/undefined handled properly (optional chaining, nullish coalescing)

#### Architecture

- [ ] Domain layer has zero imports from infrastructure
- [ ] Use cases return `Result<T, E>`, not throwing exceptions
- [ ] Controllers only orchestrate, no business logic
- [ ] Components are reusable, no hardcoded values

#### Error Handling

- [ ] All errors are typed `DomainError` subclasses
- [ ] Error messages are user-friendly and actionable
- [ ] Errors include relevant metadata for debugging
- [ ] No swallowed errors (empty catch blocks)

#### Performance

- [ ] No N+1 queries — use includes/joins
- [ ] Large lists paginated
- [ ] Expensive computations memoized
- [ ] Images optimized, lazy loaded

#### Security

- [ ] No secrets in code — use env variables
- [ ] User input validated (Zod/class-validator)
- [ ] No SQL injection (Prisma parameterized)
- [ ] Sensitive data not logged

#### Internationalization

- [ ] All user-facing strings use translation keys
- [ ] French translation added (source of truth)
- [ ] English translation added
- [ ] No string concatenation — use interpolation `{variable}`
- [ ] Metadata translated (title, description, OG tags)

### 3. Testing

```bash
pnpm test              # All tests pass
pnpm test:coverage     # Coverage >= 80% on new code
```

- [ ] Unit tests for new business logic
- [ ] Edge cases covered (null, empty, boundary values)
- [ ] Error paths tested

### 4. Documentation

- [ ] Complex logic has comments explaining "why"
- [ ] Public APIs have JSDoc
- [ ] README updated if new setup steps

### 5. Final Verification

```bash
pnpm build             # Build succeeds
docker compose up -d && pnpm db:migrate  # DB migrations work
pnpm dev               # App runs without errors
```

---

## Self-Review Workflow

```
1. WRITE      → Implement the feature/fix
2. LINT       → Run pnpm lint && pnpm type-check
3. FIX        → Fix any errors found
4. REVIEW     → Go through checklist above
5. REFACTOR   → Improve code quality issues found
6. TEST       → Run pnpm test, add missing tests
7. VERIFY     → Run pnpm build && pnpm dev
8. COMMIT     → Only if all checks pass
```

---

## Common Issues to Self-Fix

| Issue                        | Fix                                         |
| ---------------------------- | ------------------------------------------- |
| Missing return type          | Add explicit `: ReturnType`                 |
| `any` type                   | Replace with proper type or `unknown`       |
| Long function (>20 lines)    | Extract helper functions                    |
| Nested conditionals          | Use early returns                           |
| Duplicated code              | Extract to shared util/component            |
| Missing error handling       | Add Result pattern or try/catch             |
| Hardcoded string             | Extract to constant or env var              |
| Missing validation           | Add Zod schema or class-validator           |
| No tests                     | Write unit test for business logic          |
| Console.log left in          | Remove or replace with logger               |
| Hardcoded UI text            | Extract to translation file (fr.json first) |
| String concatenation in i18n | Use interpolation `{variable}`              |

---

## Review Questions

1. **"Would I approve this in a PR review?"** — If no, fix it
2. **"Is this the simplest solution?"** — If not, simplify
3. **"Can this break in production?"** — If yes, add safeguards
4. **"Will future me understand this?"** — If not, add comments
5. **"Is this reusable?"** — If not, make it generic
6. **"Is every user-facing string translated?"** — If no, add to fr.json + en.json

---

## ESLint Configuration

```javascript
// packages/config/eslint-preset.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/strict-type-checked',
    'plugin:@typescript-eslint/stylistic-type-checked',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  rules: {
    // TypeScript
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/explicit-function-return-type': [
      'error',
      {
        allowExpressions: true,
      },
    ],
    '@typescript-eslint/no-unused-vars': [
      'error',
      {
        argsIgnorePattern: '^_',
      },
    ],
    '@typescript-eslint/consistent-type-imports': 'error',

    // React
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',

    // Import order
    'import/order': [
      'error',
      {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling'],
        'newlines-between': 'always',
        alphabetize: { order: 'asc' },
      },
    ],
  },
};
```

---

## Prettier Configuration

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

---

## Conventional Commits

```
feat(domain): add user registration use case
fix(api): handle duplicate email error
test(web): add login form tests
refactor(auth): extract token validation
docs(readme): add setup instructions
```
