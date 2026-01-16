# Claude.md

## Project Overview

Monorepo full-stack application with React frontend and NestJS backend, following DDD and hexagonal architecture principles.

## Tech Stack

### Frontend (`/apps/web`)

- **Framework**: React 18+ with TypeScript (strict mode)
- **Build Tool**: Next.js 16+ (App Router)
- **Styling**: Tailwind CSS + shadcn/ui (optional)
- **Deployment**: Vercel
- **State**: TanStack Query for server state, Zustand for client state
- **Testing**: Vitest + Testing Library

### Backend (`/apps/api`)

- **Framework**: NestJS with TypeScript (strict mode)
- **ORM**: Prisma
- **Database**: PostgreSQL (local: Docker, production: Neon)
- **Job Queue**: BullMQ with Redis
- **Deployment**: Railway
- **Architecture**: DDD + Hexagonal (Ports & Adapters) + Event-Driven
- **Testing**: Vitest

### Shared (`/packages/shared`)

- Types, DTOs, validation schemas (Zod), constants

---

## Monorepo Structure

```
/
├── apps/
│   ├── web/                    # React frontend (Next.js)
│   └── api/                    # NestJS backend
├── packages/
│   ├── shared/                 # Shared types, DTOs, constants
│   └── config/                 # Shared configs (ESLint, TS)
├── .claude/
│   └── skills/                 # Claude Code skills
├── docker-compose.yml          # Local PostgreSQL + Redis
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

---

## Environment Variables

### Backend (`apps/api/.env`)

Copy from `.env.example` and adjust:

```env
DATABASE_URL="postgresql://appuser:apppassword@localhost:5433/appdb?schema=public"
JWT_SECRET="change-me-in-production"
JWT_EXPIRES_IN="15m"
JWT_REFRESH_EXPIRES_IN="7d"
PORT=3001
NODE_ENV=development
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
FRONTEND_URL=http://localhost:3000
```

### Frontend (`apps/web/.env.local`)

Copy from `.env.local.example`:

```env
NEXT_PUBLIC_API_URL="http://localhost:3001"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

---

## Commands Reference

```bash
# Docker (local services)
docker compose up -d            # Start PostgreSQL and Redis
docker compose down             # Stop services
docker compose down -v          # Stop + delete volumes (reset DB and Redis)

# Development
pnpm dev                    # Start all apps
pnpm dev:api                # Not configured (use: cd apps/api && pnpm dev)
pnpm dev:web                # Not configured (use: cd apps/web && pnpm dev)

# Build
pnpm build                  # Build all

# Quality
pnpm lint                   # Lint all
pnpm type-check             # TypeScript check
pnpm format                 # Prettier format
pnpm format:check           # Check formatting

# Test
pnpm test                   # Unit tests
pnpm test:coverage          # With coverage

# Database
pnpm db:migrate             # Run migrations (local)
pnpm db:generate            # Generate Prisma client
pnpm db:studio              # Open Prisma Studio
```

---

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
12. **Self-review code before committing** (see code-quality skill)

### NEVER

13. Use `any` type — use `unknown` + type guards
14. Put business logic in controllers or components
15. Skip validation (Zod on frontend, class-validator on backend)
16. Hardcode values — use environment variables or constants
17. Commit without running lint and type-check
18. Throw generic Error — use typed DomainError subclasses
19. Log sensitive data (passwords, tokens, PII)
20. Push code without self-review

### PREFER

21. Composition over inheritance
22. Small, focused functions (< 20 lines)
23. Named exports over default exports
24. Interface for objects, type for unions
25. Early returns over nested conditions
26. Explicit over implicit
27. Result.err() over throwing exceptions in use cases

---

## Skills (Detailed Documentation)

Extended documentation is available in `.claude/skills/`:

### Project Skills

- **[backend-architecture.md](.claude/skills/backend-architecture.md)** - NestJS + Hexagonal architecture patterns
- **[frontend-architecture.md](.claude/skills/frontend-architecture.md)** - React component patterns, TanStack Query
- **[database.md](.claude/skills/database.md)** - Prisma, Docker, PostgreSQL setup
- **[error-handling.md](.claude/skills/error-handling.md)** - DomainError, Result pattern, exception filters
- **[testing.md](.claude/skills/testing.md)** - Vitest, test patterns
- **[observability.md](.claude/skills/observability.md)** - OpenTelemetry, logging
- **[code-quality.md](.claude/skills/code-quality.md)** - Pre-commit checklist, ESLint, Prettier
- **[seo.md](.claude/skills/seo.md)** - Metadata, sitemap, JSON-LD

### Vercel Skills (External)

- **[react-best-practices](.claude/skills/react-best-practices/)** - 45 React/Next.js performance rules
- **[web-design-guidelines](.claude/skills/web-design-guidelines/)** - UI compliance checker

---

## Customization Guide

When starting a new project, customize:

1. **Project Name & Metadata**
   - Update `name` in root `package.json`
   - Update `@app/*` namespace in all packages to your project name
   - Update metadata in `apps/web/src/app/layout.tsx`

2. **Database**
   - Customize `schema.prisma` with your models
   - Run `pnpm db:generate && pnpm db:migrate`

3. **Styling**
   - Customize colors in `tailwind.config.ts`
   - Update global styles in `apps/web/src/app/globals.css`

4. **Environment**
   - Copy `.env.example` files and update values
   - Update CORS origins in `apps/api/src/main.ts`

---

## Plan Mode

- Make the plan extremely concise. Sacrifice grammar for the sake of concision.
- At the end of each plan, give me a list of unresolved questions to answer, if any
