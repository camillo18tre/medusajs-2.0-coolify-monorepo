# Backend Documentation (MedusaJS 2.0)

## Overview

The backend is a **MedusaJS 2.0** application providing:
- RESTful API for e-commerce operations
- Admin Dashboard UI
- Event-driven architecture
- Modular plugin system

**Package**: `medusa-backend`
**Port**: 9000
**Admin**: http://localhost:9000/app

## Directory Structure

```
apps/backend/
├── src/
│   ├── admin/                  # Admin UI customizations
│   │   ├── routes/            # Custom admin pages
│   │   └── widgets/           # Dashboard widgets
│   │
│   ├── api/                    # HTTP API routes
│   │   ├── admin/             # Admin-only endpoints
│   │   ├── store/             # Storefront endpoints
│   │   └── key-exchange/      # Auth endpoints
│   │
│   ├── jobs/                   # Scheduled jobs
│   │   └── (background tasks)
│   │
│   ├── lib/                    # Shared utilities
│   │   └── constants.ts       # Environment constants
│   │
│   ├── modules/                # Custom modules
│   │   ├── email-notifications/  # Resend email module
│   │   └── minio-file/           # MinIO storage module
│   │
│   ├── scripts/                # Setup and maintenance scripts
│   │   ├── seed.ts            # Database seeding
│   │   └── postBuild.js       # Post-build tasks
│   │
│   ├── subscribers/            # Event subscribers
│   │   ├── invite-created.ts
│   │   └── order-placed.ts
│   │
│   ├── utils/                  # Utility functions
│   │   └── assert-value.ts
│   │
│   └── workflows/              # Business workflows
│       └── (custom workflows)
│
├── .medusa/                    # Generated build output
│   └── server/                 # Compiled server code
│
├── medusa-config.js            # Main Medusa configuration
├── .env.template               # Environment variables template
├── package.json
└── README.md
```

## Configuration

### Environment Variables

Key environment variables (see `.env.template`):

```bash
# Core
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/medusa
REDIS_URL=redis://localhost:6379

# Server
BACKEND_URL=http://localhost:9000
PORT=9000
JWT_SECRET=your-jwt-secret
COOKIE_SECRET=your-cookie-secret

# CORS
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:9000,http://localhost:7001

# Storage (MinIO)
MINIO_ENDPOINT=localhost:9001
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=medusa-media

# Search (MeiliSearch)
MEILISEARCH_HOST=http://localhost:7700
MEILISEARCH_ADMIN_KEY=your-meilisearch-key

# Payment (Stripe)
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Email (Resend)
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=noreply@yourdomain.com

# Optional: SendGrid (alternative to Resend)
SENDGRID_API_KEY=SG...
SENDGRID_FROM_EMAIL=noreply@yourdomain.com

# Admin
DISABLE_ADMIN=false

# Worker Mode
WORKER_MODE=shared
```

### Medusa Configuration (`medusa-config.js`)

The configuration is modular and environment-aware:

#### 1. Project Config
```javascript
projectConfig: {
  databaseUrl: DATABASE_URL,
  redisUrl: REDIS_URL,
  workerMode: WORKER_MODE,  // 'shared' or 'worker' or 'server'
  http: {
    adminCors: ADMIN_CORS,
    authCors: AUTH_CORS,
    storeCors: STORE_CORS,
    jwtSecret: JWT_SECRET,
    cookieSecret: COOKIE_SECRET,
  }
}
```

#### 2. Admin Dashboard
```javascript
admin: {
  backendUrl: BACKEND_URL,
  disable: SHOULD_DISABLE_ADMIN,
  vite: () => ({
    server: {
      allowedHosts: [BACKEND_URL.replace("https://", "")],
      hmr: !IS_DEV ? "false" : "true",
    }
  })
}
```

#### 3. Modules

**File Storage Module**:
- **MinIO** (if configured): S3-compatible object storage
- **Local** (fallback): File system storage

**Event Bus Module** (requires Redis):
- Handles event publishing/subscribing
- Enables async workflows

**Workflow Engine Module** (requires Redis):
- Orchestrates complex business operations
- Transaction management

**Notification Module**:
- **Resend** provider (recommended)
- **SendGrid** provider (alternative)

**Payment Module**:
- **Stripe** integration
- Webhook handling

#### 4. Plugins

**MeiliSearch Plugin**:
- Product search indexing
- Configurable searchable fields
- Real-time sync

## Custom Modules

### 1. MinIO File Module (`src/modules/minio-file`)

**Purpose**: S3-compatible object storage for product images and media

**Files**:
- [index.ts](../apps/backend/src/modules/minio-file/index.ts) - Module registration
- [service.ts](../apps/backend/src/modules/minio-file/service.ts) - MinIO service implementation

**Features**:
- Auto-create bucket on startup
- Image upload/download
- Public URL generation
- Stream support

**Configuration**:
```javascript
{
  resolve: "./src/modules/minio-file",
  id: "minio",
  options: {
    endPoint: MINIO_ENDPOINT,     // e.g., "localhost:9001"
    accessKey: MINIO_ACCESS_KEY,
    secretKey: MINIO_SECRET_KEY,
    bucket: MINIO_BUCKET,         // default: "medusa-media"
  }
}
```

**Usage Example**:
```typescript
// In a workflow or service
const file = await fileModuleService.upload({
  file: uploadedFile,
  provider: 'minio'
})
```

### 2. Email Notifications Module (`src/modules/email-notifications`)

**Purpose**: Transactional emails using Resend and React Email

**Files**:
- [index.ts](../apps/backend/src/modules/email-notifications/index.ts) - Module registration
- [services/resend.ts](../apps/backend/src/modules/email-notifications/services/resend.ts) - Resend service
- `templates/` - React Email templates

**Features**:
- React-based email templates
- Type-safe template rendering
- Event-driven email sending

**Configuration**:
```javascript
{
  resolve: "./src/modules/email-notifications",
  id: "resend",
  options: {
    channels: ["email"],
    api_key: RESEND_API_KEY,
    from: RESEND_FROM_EMAIL,
  }
}
```

**Email Events**:
- Order placed
- Order shipped
- User invited
- Password reset
- (extensible)

## API Routes

### Admin Routes (`src/api/admin`)

Custom admin-only endpoints:
- Authentication required
- CORS restricted to admin origins

Example:
```typescript
// src/api/admin/custom/route.ts
export const GET = async (req, res) => {
  // Admin-only logic
}
```

### Store Routes (`src/api/store`)

Public storefront endpoints:
- Accessible by customers
- CORS restricted to storefront origins

Example:
```typescript
// src/api/store/custom/route.ts
export const GET = async (req, res) => {
  // Public storefront logic
}
```

### Route Structure

```
src/api/
├── admin/
│   └── custom/
│       └── route.ts          # GET/POST/PUT/DELETE /admin/custom
├── store/
│   └── custom/
│       └── route.ts          # GET/POST/PUT/DELETE /store/custom
└── key-exchange/
    └── route.ts              # Custom auth endpoints
```

## Workflows

MedusaJS 2.0 workflows orchestrate complex business logic:

**Location**: `src/workflows/`

**Characteristics**:
- Transactional
- Composable steps
- Automatic compensation on failure
- Async execution via Redis

**Example Use Cases**:
- Multi-step checkout
- Order fulfillment
- Inventory management
- Price calculations

## Subscribers

Event-driven handlers that react to system events:

### Order Placed Subscriber (`src/subscribers/order-placed.ts`)

Listens to: `order.placed` event

Actions:
- Send confirmation email
- Update inventory
- Trigger fulfillment

### Invite Created Subscriber (`src/subscribers/invite-created.ts`)

Listens to: `invite.created` event

Actions:
- Send invitation email

### Creating Custom Subscribers

```typescript
import { SubscriberArgs, type SubscriberConfig } from "@medusajs/medusa"

export default async function handleCustomEvent({
  event: { data },
  container
}: SubscriberArgs) {
  // Access services from container
  const notificationService = container.resolve("notificationService")

  // React to event
  await notificationService.send({
    to: data.email,
    template: "custom-template",
    data: data
  })
}

export const config: SubscriberConfig = {
  event: "custom.event.name"
}
```

## Jobs

Background scheduled tasks:

**Location**: `src/jobs/`

**Use Cases**:
- Cleanup old data
- Generate reports
- Sync with external systems
- Send scheduled emails

## Scripts

### Seed Script (`src/scripts/seed.ts`)

Seeds the database with:
- Initial admin user
- Sample products
- Default regions
- Sample data

**Run**: `pnpm seed` or `npm run seed`

### Initialize Backend Script

**Command**: `pnpm ib` or `npm run ib`

Custom command (via `medusajs-launch-utils`) that:
1. Runs database migrations
2. Seeds initial data
3. Prepares the backend for first use

## Development Commands

```bash
# Install dependencies
pnpm install

# Initialize backend (first time setup)
pnpm ib

# Development mode (with watch)
pnpm dev

# Build for production
pnpm build

# Start production build
pnpm start

# Type checking
pnpm check-types

# Database migrations
pnpm medusa db:migrate

# Seed database
pnpm seed

# Email template development
pnpm email:dev
```

## Database

### ORM: MikroORM

MedusaJS 2.0 uses MikroORM for database operations.

**Configuration**:
- Auto-loaded from `medusa-config.js`
- Migrations in `.medusa/server/migrations/`

**Common Operations**:
```bash
# Create migration
pnpm medusa db:create

# Run migrations
pnpm medusa db:migrate

# Rollback
pnpm medusa db:rollback
```

### Models

Models are defined in Medusa core and custom modules.

## Authentication & Authorization

### Admin Authentication
- JWT-based
- Configured via `JWT_SECRET`
- Session cookies

### Storefront Authentication
- JWT for registered users
- Guest checkout supported

### API Keys
- Admin API keys can be generated
- Used for server-to-server communication

## Integrations

### Stripe Payment Integration

**Setup**:
1. Set `STRIPE_API_KEY` (test or live key)
2. Set `STRIPE_WEBHOOK_SECRET`
3. Configure webhook endpoint: `{BACKEND_URL}/hooks/payment/stripe`

**Webhook Events**:
- `payment_intent.succeeded`
- `payment_intent.payment_failed`
- `charge.refunded`

### MeiliSearch Integration

**Setup**:
1. Set `MEILISEARCH_HOST`
2. Set `MEILISEARCH_ADMIN_KEY`

**Indexed Entities**:
- Products (with variants)

**Searchable Fields**:
- Title
- Description
- SKU

**Features**:
- Auto-indexing on product changes
- Typo tolerance
- Instant search

### Redis Integration

**Usage**:
- Event bus
- Workflow engine
- Caching
- Job queues

**Fallback**: If Redis not configured, falls back to in-memory (development only)

## Logging

MedusaJS has built-in logging:

```typescript
// In any service or route
const logger = container.resolve("logger")

logger.info("Info message")
logger.warn("Warning message")
logger.error("Error message")
```

## Testing

```bash
# Run tests (when configured)
pnpm test

# Type checking
pnpm check-types
```

## Common Development Patterns

### Accessing Services in Routes

```typescript
export const GET = async (req, res) => {
  const productService = req.scope.resolve("productService")
  const products = await productService.list()
  res.json({ products })
}
```

### Emitting Events

```typescript
const eventBusService = container.resolve("eventBusService")

await eventBusService.emit("custom.event", {
  data: { foo: "bar" }
})
```

### Using Workflows

```typescript
import { myWorkflow } from "../../workflows/my-workflow"

const result = await myWorkflow(container).run({
  input: { orderId: "order_123" }
})
```

## Performance Considerations

### Database Connection Pooling
- Configured in `medusa-config.js`
- Default pool size based on environment

### Redis Caching
- Cache product listings
- Cache region configurations
- Cache tax calculations

### Build Optimization
- Code splitting enabled
- Tree shaking for unused modules
- Rollup for optimized bundles

## Security Best Practices

1. **Environment Variables**: Never commit `.env` files
2. **CORS**: Restrict to known origins
3. **API Keys**: Rotate regularly
4. **Webhooks**: Verify signatures (Stripe)
5. **JWT Secrets**: Use strong, random values
6. **Database**: Use connection pooling and prepared statements

## Troubleshooting

### Backend Won't Start

**Check**:
1. Database connection (`DATABASE_URL`)
2. Redis connection (if configured)
3. Port 9000 availability
4. Environment variables loaded

### Admin Dashboard 404

**Check**:
1. `DISABLE_ADMIN` is not `true`
2. Backend is running on correct port
3. `BACKEND_URL` is correct

### Migrations Failing

**Solution**:
```bash
# Reset database (development only!)
pnpm medusa db:reset

# Then re-run
pnpm ib
```

### Module Not Loading

**Check**:
1. Environment variables for module are set
2. Module is in `medusa-config.js`
3. Dependencies are installed
4. Build was successful

## Useful Resources

- [MedusaJS 2.0 Docs](https://docs.medusajs.com)
- [API Reference](https://docs.medusajs.com/api)
- [Module Development](https://docs.medusajs.com/modules)
- [Workflow Guide](https://docs.medusajs.com/workflows)

---

Last updated: 2025-11-27
