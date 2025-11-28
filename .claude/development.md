# Development Guide

## Local Development Setup

### Prerequisites

- **Node.js**: v22.x (required)
- **pnpm**: v9.15.5 (enforced)
- **PostgreSQL**: 17+ (local or remote)
- **Redis**: 7+ (optional, with fallback)
- **MinIO**: Latest (optional, with fallback)
- **MeiliSearch**: Latest (optional)

### Initial Setup

#### 1. Clone and Install

```bash
# Clone the repository
git clone <repo-url>
cd my-medusa-monorepo

# Install dependencies (from root)
pnpm install
```

This will install dependencies for:

- Root workspace
- Backend app
- Storefront app

#### 2. Setup PostgreSQL

**Option A: Local PostgreSQL**

```bash
# macOS (Homebrew)
brew install postgresql@17
brew services start postgresql@17

# Create database
createdb medusa_store
```

**Option B: Docker PostgreSQL**

```bash
docker run -d \
  --name medusa-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=medusa_store \
  -p 5432:5432 \
  postgres:17
```

**Option C: Use Railway/Cloud Database**

- Copy `DATABASE_URL` from Railway
- Add to backend `.env`

#### 3. Setup Redis (Optional)

**Option A: Local Redis**

```bash
# macOS (Homebrew)
brew install redis
brew services start redis

# Or Docker
docker run -d --name medusa-redis -p 6379:6379 redis:7
```

**Option B: Skip Redis**

- Backend will use in-memory fallback (dev only)

#### 4. Setup MinIO (Optional)

**Docker MinIO**:

```bash
docker run -d \
  --name medusa-minio \
  -p 9001:9001 \
  -p 9002:9002 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  minio/minio server /data --console-address ":9002"
```

Access MinIO Console: http://localhost:9002

**Skip MinIO**:

- Backend will use local file storage

#### 5. Setup MeiliSearch (Optional)

**Docker MeiliSearch**:

```bash
docker run -d \
  --name medusa-meilisearch \
  -p 7700:7700 \
  -e MEILI_MASTER_KEY=masterKey \
  getmeili/meilisearch:v1.5
```

#### 6. Configure Backend

```bash
cd apps/backend

# Copy template
cp .env.template .env

# Edit .env with your values
nano .env
```

**Minimal `.env` for local development**:

```bash
# Database (required)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/medusa_store

# Redis (optional)
REDIS_URL=redis://localhost:6379

# Server
BACKEND_URL=http://localhost:9000
PORT=9000
JWT_SECRET=your-random-secret-here
COOKIE_SECRET=your-random-cookie-secret-here

# CORS
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:9000,http://localhost:7001

# MinIO (optional)
MINIO_ENDPOINT=localhost:9001
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=medusa-media

# MeiliSearch (optional)
MEILISEARCH_HOST=http://localhost:7700
MEILISEARCH_ADMIN_KEY=masterKey

# Stripe (for testing)
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Resend (for emails)
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=noreply@yourdomain.com
```

#### 7. Initialize Backend

```bash
# From apps/backend
pnpm ib
```

This will:

- Run database migrations
- Seed initial data
- Create admin user
- Add sample products (optional)

#### 8. Configure Storefront

```bash
cd apps/storefront

# Copy template
cp .env.local.template .env.local

# Edit .env.local
nano .env.local
```

**`.env.local` for local development**:

```bash
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_BASE_URL=http://localhost:8000
NEXT_PUBLIC_DEFAULT_REGION=us

# MeiliSearch (optional)
NEXT_PUBLIC_SEARCH_ENDPOINT=http://localhost:7700
NEXT_PUBLIC_SEARCH_API_KEY=masterKey
NEXT_PUBLIC_INDEX_NAME=products
```

## Starting Development Servers

### Option 1: Start Everything (Recommended)

```bash
# From root directory
pnpm dev
```

This starts:

- Backend on http://localhost:9000
- Storefront on http://localhost:8000

### Option 2: Start Individually

**Backend Only**:

```bash
# From root
pnpm dev:backend

# Or from apps/backend
cd apps/backend
pnpm dev
```

**Storefront Only**:

```bash
# From root
pnpm dev:storefront

# Or from apps/storefront
cd apps/storefront
pnpm dev
```

## Development Workflow

### Daily Development

1. **Start all services**:

```bash
pnpm dev
```

2. **Access applications**:

- Admin Dashboard: http://localhost:9000/app
- Storefront: http://localhost:8000
- Backend API: http://localhost:9000

3. **Make changes**:

- Backend: `apps/backend/src/`
- Storefront: `apps/storefront/src/`

4. **Watch for automatic reload**:

- Backend: Medusa dev server watches files
- Storefront: Next.js Fast Refresh

### Working on Backend

#### Adding API Endpoints

```bash
# Create new route
apps/backend/src/api/
  └── store/
      └── custom-endpoint/
          └── route.ts
```

```typescript
// route.ts
export const GET = async (req, res) => {
  // Your logic
  return res.json({ message: "Hello" });
};

export const POST = async (req, res) => {
  const body = req.body;
  // Your logic
  return res.json({ success: true });
};
```

#### Creating Custom Module

```bash
apps/backend/src/modules/
  └── my-module/
      ├── index.ts          # Module registration
      ├── service.ts        # Service implementation
      └── README.md         # Documentation
```

#### Adding Event Subscriber

```bash
apps/backend/src/subscribers/
  └── my-event.ts
```

```typescript
import { SubscriberArgs, type SubscriberConfig } from "@medusajs/medusa";

export default async function myEventHandler({
  event,
  container,
}: SubscriberArgs) {
  const logger = container.resolve("logger");
  logger.info("Event received", event.data);
}

export const config: SubscriberConfig = {
  event: "my.event.name",
};
```

#### Database Changes

```bash
# Create migration
cd apps/backend
pnpm medusa db:create my_migration_name

# Run migrations
pnpm medusa db:migrate

# Rollback
pnpm medusa db:rollback
```

### Working on Storefront

#### Adding New Page

```bash
# Create page in app router
apps/storefront/src/app/[countryCode]/
  └── my-page/
      └── page.tsx
```

```tsx
export default async function MyPage() {
  return (
    <div className='container mx-auto py-8'>
      <h1 className='text-3xl font-bold'>My Page</h1>
    </div>
  );
}
```

#### Creating Components

```bash
apps/storefront/src/modules/
  └── my-feature/
      ├── components/
      │   └── my-component.tsx
      ├── templates/
      │   └── my-template.tsx
      └── index.ts
```

#### Data Fetching

**Server Component**:

```tsx
import { sdk } from "@lib/config";

export default async function ServerPage() {
  const { products } = await sdk.store.product.list();

  return <ProductGrid products={products} />;
}
```

**Client Component**:

```tsx
"use client";
import { useState, useEffect } from "react";

export function ClientComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api/data")
      .then((r) => r.json())
      .then(setData);
  }, []);

  return <div>{data?.title}</div>;
}
```

## Common Development Tasks

### Reset Database

```bash
cd apps/backend

# Drop and recreate database
pnpm medusa db:reset

# Re-initialize
pnpm ib
```

### Seed Sample Data

```bash
cd apps/backend
pnpm seed
```

### Clear Build Cache

```bash
# From root
rm -rf .turbo
rm -rf apps/backend/.medusa
rm -rf apps/storefront/.next

# Rebuild
pnpm build
```

### Update Dependencies

```bash
# From root
pnpm update --latest

# Or specific package
pnpm update @medusajs/medusa --latest
```

### Type Checking

```bash
# Check all
pnpm check-types

# Backend only
cd apps/backend && pnpm check-types

# Storefront only
cd apps/storefront && pnpm check-types
```

## Debugging

### Backend Debugging

**VSCode Launch Configuration** (`.vscode/launch.json`):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "cwd": "${workspaceFolder}/apps/backend",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["dev"],
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

**Console Logging**:

```typescript
const logger = container.resolve("logger");
logger.info("Debug info", { data });
logger.error("Error occurred", error);
```

### Storefront Debugging

**Console Logging**:

```tsx
"use client";
export function Component() {
  console.log("Debug:", data);
  return <div>...</div>;
}
```

**React Developer Tools**:

- Install browser extension
- Inspect component tree
- View props and state

### Network Debugging

**Backend Logs**:

- All requests logged to console
- Check terminal where `pnpm dev` is running

**Storefront Logs**:

- Next.js logs in terminal
- Browser network tab for client requests

## Testing

### Backend Testing

```bash
cd apps/backend

# Run tests (when configured)
pnpm test

# Run specific test
pnpm test path/to/test.ts
```

### Storefront Testing

```bash
cd apps/storefront

# E2E tests with Playwright
pnpm test-e2e

# Run specific test
pnpm test-e2e tests/checkout.spec.ts
```

### Manual Testing Checklist

**Backend**:

- [ ] Admin login works
- [ ] Products can be created
- [ ] Orders can be placed
- [ ] Emails are sent
- [ ] File uploads work

**Storefront**:

- [ ] Homepage loads
- [ ] Product pages work
- [ ] Search works
- [ ] Cart functions
- [ ] Checkout completes

## Environment-Specific Considerations

### Development Environment

**Characteristics**:

- Hot reload enabled
- Verbose logging
- Source maps
- No optimization
- Mock payment methods

**Performance**: Slower, but better DX

### Production-like Local Testing

```bash
# Build everything
pnpm build

# Start production builds
pnpm start
```

**Tests**:

- Performance
- Optimization
- Production configs
- Error handling

## Turborepo Development

### Understanding Cache

Turborepo caches build outputs:

```bash
# View cache
.turbo/

# Clear cache
rm -rf .turbo
```

### Running Tasks

```bash
# Run task in all workspaces
turbo run build

# Run task in specific workspace
turbo run build --filter=medusa-backend

# Run with force (ignore cache)
turbo run build --force

# Run in parallel
turbo run lint build
```

### Filtering

```bash
# Run dev for backend only
pnpm dev:backend
# Equivalent to: turbo run dev --filter=medusa-backend

# Run for multiple
turbo run dev --filter=medusa-backend --filter=medusa-storefront
```

## Git Workflow

### Branch Strategy

**Main Branches**:

- `main` - Production
- `develop` - Development (current)

**Feature Branches**:

```bash
git checkout -b feature/my-feature
git checkout -b fix/bug-fix
git checkout -b chore/update-deps
```

### Commit Convention

Semantic commit messages:

```
feat: add new product filter
fix: resolve cart calculation bug
chore: update dependencies
docs: improve setup documentation
refactor: simplify checkout flow
style: format code
test: add product tests
```

### Pre-commit Checklist

Before committing:

- [ ] Code builds: `pnpm build`
- [ ] Types check: `pnpm check-types`
- [ ] Lint passes: `pnpm lint`
- [ ] Tests pass: `pnpm test`

## Common Issues & Solutions

### Issue: Port Already in Use

**Error**: `Port 9000 is already in use`

**Solution**:

```bash
# Find process
lsof -i :9000

# Kill process
kill -9 <PID>

# Or use different port
PORT=9001 pnpm dev
```

### Issue: Database Connection Failed

**Check**:

1. PostgreSQL is running
2. `DATABASE_URL` is correct
3. Database exists
4. User has permissions

### Issue: Redis Connection Failed

**Solution**:

- Start Redis, or
- Remove `REDIS_URL` from `.env` (uses fallback)

### Issue: Module Not Found

**Solution**:

```bash
# Reinstall dependencies
rm -rf node_modules
rm pnpm-lock.yaml
pnpm install
```

### Issue: Build Fails

**Common Causes**:

1. TypeScript errors - run `pnpm check-types`
2. Missing environment variables
3. Dependency version conflicts

**Solution**:

```bash
# Clean and rebuild
rm -rf .turbo apps/backend/.medusa apps/storefront/.next
pnpm install
pnpm build
```

### Issue: Hot Reload Not Working

**Backend**:

- Restart dev server
- Check file is in `src/`

**Storefront**:

- Check for syntax errors
- Restart Next.js dev server
- Clear `.next` folder

## IDE Setup

### VSCode Recommended Extensions

- ESLint
- Prettier
- Tailwind CSS IntelliSense
- TypeScript Importer
- GitLens
- Prisma (if using)

### VSCode Settings

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"]
  ]
}
```

## Performance Tips

### Backend

1. **Use connection pooling** for database
2. **Enable Redis** for caching
3. **Index frequently queried fields**
4. **Use workflows** for complex operations

### Storefront

1. **Prefer Server Components** when possible
2. **Lazy load** heavy components
3. **Optimize images** with Next.js Image
4. **Use Suspense** for loading states

## Documentation Workflow

When adding features:

1. Add JSDoc comments to functions
2. Update relevant `.claude/*.md` files
3. Add examples to documentation
4. Update README if needed

---

Last updated: 2025-11-27
