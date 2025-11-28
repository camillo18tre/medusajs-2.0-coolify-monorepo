# XTCR Medusa 2.0 Monorepo - Claude Code Documentation

> This documentation provides complete project context to assist Claude Code in understanding and developing this monorepo.

## Project Overview

**Name**: XTCR Medusa 2.0 Monorepo
**Version**: 2.10.2
**Type**: E-commerce platform based on MedusaJS 2.0
**Architecture**: Monorepo with Turborepo + pnpm workspaces

### Tech Stack

- **Backend**: MedusaJS 2.0 (Node.js 22.x, TypeScript)
- **Frontend**: Next.js 14 (React 18, TypeScript)
- **Database**: PostgreSQL
- **Cache/Queue**: Redis
- **Search**: MeiliSearch
- **Storage**: MinIO (S3-compatible)
- **Email**: Resend
- **Payments**: Stripe
- **Monorepo Tools**: Turborepo, pnpm

## Monorepo Structure

```
xtcr-medusa-2/
├── apps/
│   ├── backend/         # MedusaJS 2.0 backend + admin dashboard
│   └── storefront/      # Next.js 14 storefront
├── packages/            # Shared packages (future)
├── .claude/             # Claude documentation
├── turbo.json          # Turborepo configuration
├── pnpm-workspace.yaml # Workspace configuration
└── package.json        # Root package with global scripts
```

## Applications

### 1. Backend (`apps/backend`)
- **Package**: `medusa-backend`
- **Framework**: MedusaJS 2.0
- **Port**: 9000
- **Admin Dashboard**: http://localhost:9000/app
- See [.claude/backend.md](.claude/backend.md) for details

### 2. Storefront (`apps/storefront`)
- **Package**: `medusa-storefront`
- **Framework**: Next.js 14
- **Port**: 8000 (default)
- See [.claude/storefront.md](.claude/storefront.md) for details

## Main Scripts

### Root Level (run from monorepo root)

```bash
# Development
pnpm dev                    # Start everything in dev mode
pnpm dev:backend           # Backend only
pnpm dev:storefront        # Storefront only

# Build
pnpm build                 # Build everything
pnpm build:backend         # Build backend only
pnpm build:storefront      # Build storefront only

# Production
pnpm start                 # Start everything in prod
pnpm start:backend         # Start backend only
pnpm start:storefront      # Start storefront only

# Linting
pnpm lint                  # Lint entire monorepo
```

## Infrastructure Dependencies

### Required for Backend
1. **PostgreSQL** - Main database
2. **Redis** - Cache and queues (with simulated fallback)
3. **MinIO** - Object storage (with local fallback)
4. **MeiliSearch** - Search engine

### Deployment Ready
Project is pre-configured for Railway.app with one-click deploy template.

## Detailed Documentation

For specific information, see:

- [Architecture](.claude/architecture.md) - Technical structure and architectural decisions
- [Backend](.claude/backend.md) - MedusaJS backend details
- [Storefront](.claude/storefront.md) - Next.js storefront details
- [Development](.claude/development.md) - Local setup and development workflow
- [Deployment](.claude/deployment.md) - Production configuration and deployment

## Quick Start for Claude

### Typical Development Workflow

1. **Backend Changes**:
   - Source files in `apps/backend/src/`
   - Custom modules in `apps/backend/src/modules/`
   - API routes in `apps/backend/src/api/`

2. **Storefront Changes**:
   - Next.js 14 App Router in `apps/storefront/src/app/`
   - Components in `apps/storefront/src/modules/`

3. **Testing**:
   - Backend: `cd apps/backend && pnpm dev`
   - Storefront: `cd apps/storefront && pnpm dev`

### Important Configuration Files

- `turbo.json` - Build pipeline configuration
- `apps/backend/medusa-config.js` - Backend Medusa config
- `apps/backend/.env.template` - Backend environment template
- `apps/storefront/.env.local.template` - Frontend environment template

## Project Conventions

### Package Manager
- **ALWAYS use pnpm**, never npm or yarn
- Enforced package manager: `pnpm@9.15.5`

### TypeScript
- All development files are in TypeScript
- Strict mode enabled
- Node 22.x target

### Git Workflow
- Main branch: `develop`
- Semantic commits preferred

## Useful Links

- [MedusaJS 2.0 Docs](https://docs.medusajs.com)
- [Next.js 14 Docs](https://nextjs.org/docs)
- [Turborepo Docs](https://turbo.build/repo/docs)
- [Original Template README](./README.md)

## Notes for Claude

- This is a production-ready e-commerce project
- Includes pre-configured third-party integrations
- All custom modules are in `apps/backend/src/modules/`
- Storefront uses Next.js App Router (not Pages Router)
- Backend includes integrated admin dashboard

---

Last updated: 2025-11-27
