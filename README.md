# Fullstack Monorepo Boilerplate

Production-ready monorepo boilerplate with React (Next.js) frontend and NestJS backend, featuring TypeScript, Prisma, BullMQ, and comprehensive tooling.

## 🚀 Features

- **🏗️ Monorepo Architecture**: Turborepo + pnpm workspaces
- **⚡ Frontend**: Next.js 16 (App Router) + React 19 + Tailwind CSS
- **🔥 Backend**: NestJS + Prisma + PostgreSQL + Redis
- **📦 Shared Packages**: Types, DTOs, validation schemas (Zod)
- **🧪 Testing**: Vitest for both frontend and backend
- **✅ Code Quality**: ESLint + Prettier + Husky + lint-staged
- **🐳 Docker**: PostgreSQL + Redis with docker-compose
- **🚢 Deployment**: Vercel (frontend) + Railway (backend) + Neon (database)
- **🤖 AI-Ready**: Comprehensive Claude Code skills and documentation

## 📋 Prerequisites

- Node.js >= 20
- pnpm >= 9
- Docker Desktop (for local database)

## 🛠️ Quick Start

### 1. Clone and Install

```bash
# Clone (or use this directory as-is)
cd fullstack-boilerplate

# Install dependencies
pnpm install
```

### 2. Environment Setup

```bash
# Backend
cd apps/api
cp .env.example .env
# Edit .env with your values

# Frontend
cd ../web
cp .env.local.example .env.local
# Edit .env.local with your values
```

### 3. Start Local Services

```bash
# From root directory
docker compose up -d

# Verify services are running
docker compose ps
```

### 4. Database Setup

```bash
# Generate Prisma client and run migrations
pnpm db:generate
pnpm db:migrate

# Optional: Open Prisma Studio
pnpm db:studio
```

### 5. Run Development Servers

```bash
# Start both frontend and backend
pnpm dev

# Frontend: http://localhost:3000
# Backend: http://localhost:3001
# API Health: http://localhost:3001/health
```

## 📁 Project Structure

```
fullstack-boilerplate/
├── apps/
│   ├── web/                    # Next.js frontend
│   │   ├── src/
│   │   │   └── app/            # App Router pages
│   │   ├── public/             # Static assets
│   │   └── package.json
│   └── api/                    # NestJS backend
│       ├── src/
│       │   ├── app.module.ts
│       │   └── main.ts
│       ├── prisma/
│       │   └── schema.prisma
│       └── package.json
├── packages/
│   ├── shared/                 # Shared types & constants
│   │   └── src/
│   │       ├── types.ts
│   │       └── constants.ts
│   └── config/                 # Shared configs
│       ├── eslint/
│       │   ├── base.js
│       │   ├── react.js
│       │   └── nestjs.js
│       └── typescript/
│           └── base.json
├── .claude/
│   └── skills/                 # Claude Code skills
├── docker-compose.yml          # Local PostgreSQL + Redis
├── turbo.json                  # Turborepo config
├── pnpm-workspace.yaml
├── CLAUDE.md                   # AI assistant instructions
└── README.md
```

## 🧰 Available Scripts

### Root Level

```bash
pnpm dev              # Start all apps in development mode
pnpm build            # Build all apps
pnpm test             # Run all tests
pnpm test:coverage    # Run tests with coverage
pnpm lint             # Lint all packages
pnpm type-check       # TypeScript type checking
pnpm format           # Format code with Prettier
pnpm format:check     # Check code formatting
```

### Database Commands

```bash
pnpm db:generate      # Generate Prisma client
pnpm db:migrate       # Run migrations (dev)
pnpm db:deploy        # Deploy migrations (production)
pnpm db:push          # Push schema changes (dev)
pnpm db:studio        # Open Prisma Studio
```

### Docker Commands

```bash
docker compose up -d           # Start services in background
docker compose down            # Stop services
docker compose down -v         # Stop and remove volumes (reset data)
docker compose logs -f         # View logs
docker compose ps              # List running services
```

## 🎨 Customization

### 1. Update Project Name

Replace `@app` namespace everywhere:

```bash
# Find and replace in all files
# @app/web → @yourproject/web
# @app/api → @yourproject/api
# @app/shared → @yourproject/shared
# @app/config → @yourproject/config
```

Update root `package.json` name:

```json
{
  "name": "your-project-name"
}
```

### 2. Customize Styling

**Tailwind Colors** (`apps/web/tailwind.config.ts`):

```ts
theme: {
  extend: {
    colors: {
      primary: '#your-color',
      secondary: '#your-color',
    },
  },
},
```

**Global Styles** (`apps/web/src/app/globals.css`):

```css
:root {
  --background: #ffffff;
  --foreground: #171717;
}
```

### 3. Database Schema

Edit `apps/api/prisma/schema.prisma`:

```prisma
model YourModel {
  id        String   @id @default(cuid())
  // Add your fields
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Then run:

```bash
pnpm db:generate
pnpm db:migrate
```

### 4. SEO & Metadata

Update `apps/web/src/app/layout.tsx`:

```ts
export const metadata: Metadata = {
  title: 'Your App Name',
  description: 'Your app description',
  // Add more metadata
};
```

## 🚀 Deployment

### Frontend (Vercel)

1. Push to GitHub
2. Import project in Vercel
3. Set environment variables:
   - `NEXT_PUBLIC_API_URL`: Your backend URL
   - `NEXT_PUBLIC_APP_URL`: Your frontend URL
4. Deploy

### Backend (Railway)

1. Push to GitHub
2. Create new project in Railway
3. Add PostgreSQL and Redis services
4. Set environment variables from `apps/api/.env.example`
5. Deploy

### Database (Neon)

1. Create account at [neon.tech](https://neon.tech)
2. Create database
3. Copy connection string
4. Update `DATABASE_URL` in Railway

## 🧪 Testing

```bash
# Run all tests
pnpm test

# Run tests with coverage
pnpm test:coverage

# Run tests in watch mode
cd apps/web && pnpm test:watch
cd apps/api && pnpm test:watch
```

## 📚 Documentation

- **[CLAUDE.md](./CLAUDE.md)** - Complete guide for AI-assisted development
- **[.claude/skills/](./.claude/skills/)** - Detailed architecture documentation
  - Backend architecture (NestJS, DDD, Hexagonal)
  - Frontend architecture (React, TanStack Query)
  - Database patterns (Prisma)
  - Error handling (Result pattern)
  - Testing strategies
  - Observability (OpenTelemetry)
  - Code quality guidelines

## 🤖 Working with Claude Code

This boilerplate is optimized for Claude Code CLI. Key features:

- Comprehensive project documentation in `CLAUDE.md`
- Detailed skills in `.claude/skills/`
- Architecture guidelines and best practices
- Pre-configured code quality tools

Start a conversation:

```bash
claude
# Or use the VSCode extension
```

Example prompts:
- "Add user authentication"
- "Create a new feature module"
- "Set up email notifications with BullMQ"
- "Add error boundary to the frontend"

## 🔒 Security

- [ ] Change `JWT_SECRET` in production
- [ ] Update database credentials
- [ ] Configure CORS origins
- [ ] Enable rate limiting (already configured with `@nestjs/throttler`)
- [ ] Add helmet.js for security headers
- [ ] Set up proper environment variable validation

## 📝 License

MIT - Feel free to use this boilerplate for any project

## 🙏 Credits

Based on modern web development best practices and the Vercel performance guidelines.

---

**Happy coding! 🚀**
