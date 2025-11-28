# Architecture Documentation

## Monorepo Architecture

### Overview

This is a **Turborepo-based monorepo** using **pnpm workspaces** to manage multiple applications and packages efficiently.

### Key Architectural Decisions

1. **Monorepo Pattern**: Single repository containing multiple applications
2. **Build Orchestration**: Turborepo for intelligent caching and parallel execution
3. **Package Management**: pnpm for efficient disk space usage and strict dependency resolution
4. **TypeScript**: Type safety across entire codebase
5. **Headless Architecture**: Decoupled backend and frontend

## Directory Structure

```
my-medusa-monorepo/
├── apps/
│   ├── backend/                    # MedusaJS backend application
│   │   ├── src/
│   │   │   ├── admin/             # Admin UI customizations
│   │   │   ├── api/               # API routes
│   │   │   ├── jobs/              # Background jobs
│   │   │   ├── lib/               # Shared utilities
│   │   │   ├── modules/           # Custom modules
│   │   │   │   ├── email-notifications/
│   │   │   │   └── minio-file/
│   │   │   ├── scripts/           # Setup and seed scripts
│   │   │   ├── subscribers/       # Event subscribers
│   │   │   ├── utils/             # Utilities
│   │   │   └── workflows/         # Business workflows
│   │   ├── medusa-config.js       # Medusa configuration
│   │   ├── .env.template          # Environment template
│   │   └── package.json
│   │
│   └── storefront/                 # Next.js storefront application
│       ├── src/
│       │   ├── app/               # Next.js 14 App Router
│       │   ├── modules/           # Feature modules
│       │   ├── lib/               # Utilities and configs
│       │   └── styles/            # Global styles
│       ├── public/                # Static assets
│       ├── .env.local.template    # Environment template
│       └── package.json
│
├── packages/                       # Shared packages (future)
│   └── (currently empty)
│
├── .claude/                        # Claude documentation
├── .github/                        # GitHub workflows
├── turbo.json                      # Turborepo configuration
├── pnpm-workspace.yaml            # Workspace definition
└── package.json                   # Root package
```

## Application Architecture

### Backend (MedusaJS 2.0)

```
┌─────────────────────────────────────────────────────────┐
│                    Admin Dashboard                       │
│                   (Port 9000/app)                        │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────┐
│                   MedusaJS Core                          │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐ │
│  │   API Routes  │  │   Workflows   │  │    Jobs     │ │
│  └───────────────┘  └───────────────┘  └─────────────┘ │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐ │
│  │  Subscribers  │  │   Modules     │  │   Scripts   │ │
│  └───────────────┘  └───────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌─────▼──────┐ ┌───────▼────────┐
│   PostgreSQL   │ │   Redis    │ │  MeiliSearch   │
│   (Database)   │ │ (Cache/Q)  │ │    (Search)    │
└────────────────┘ └────────────┘ └────────────────┘
                          │
                  ┌───────▼────────┐
                  │     MinIO      │
                  │   (Storage)    │
                  └────────────────┘
```

#### Backend Layers

1. **API Layer** (`src/api/`)

   - HTTP endpoints
   - Request validation
   - Response formatting

2. **Workflow Layer** (`src/workflows/`)

   - Business logic orchestration
   - Multi-step operations
   - Transaction management

3. **Module Layer** (`src/modules/`)

   - Custom business modules
   - Third-party integrations
   - Email notifications (Resend)
   - File storage (MinIO)

4. **Subscriber Layer** (`src/subscribers/`)

   - Event-driven reactions
   - Async processing triggers

5. **Job Layer** (`src/jobs/`)
   - Scheduled tasks
   - Background processing

### Storefront (Next.js 14)

```
┌─────────────────────────────────────────────────────────┐
│                  Next.js App Router                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Pages (RSC + Client)                  │  │
│  │  - Product Listing  - Product Detail              │  │
│  │  - Cart            - Checkout                      │  │
│  │  - Account         - Orders                        │  │
│  └───────────────────────────────────────────────────┘  │
│                          │                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │                   Modules                          │  │
│  │  - Product Display  - Cart Management             │  │
│  │  - Checkout Flow   - Account Management           │  │
│  └───────────────────────────────────────────────────┘  │
│                          │                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │                 Data Layer                         │  │
│  │  - @medusajs/js-sdk                               │  │
│  │  - Server Actions                                  │  │
│  │  - API Routes                                      │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
                   HTTP/REST API
                          │
                          ▼
              ┌──────────────────────┐
              │   Backend API        │
              │   (Port 9000)        │
              └──────────────────────┘
```

#### Frontend Architecture Patterns

1. **App Router (Next.js 14)**

   - Server Components by default
   - Client Components when needed
   - Streaming and Suspense

2. **Data Fetching**

   - Server-side with Medusa SDK
   - Client-side with React hooks
   - Optimistic UI updates

3. **State Management**
   - URL state for navigation
   - React Context for cart/user
   - Server state via React Query patterns

## Integration Architecture

### Third-Party Services

```
┌──────────────────────────────────────────────────────────┐
│                    Medusa Backend                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    Stripe    │  │    Resend    │  │    MinIO     │  │
│  │  (Payment)   │  │   (Email)    │  │  (Storage)   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                           │
│  ┌──────────────┐  ┌──────────────┐                     │
│  │ MeiliSearch  │  │    Redis     │                     │
│  │   (Search)   │  │ (Cache/Jobs) │                     │
│  └──────────────┘  └──────────────┘                     │
└──────────────────────────────────────────────────────────┘
```

### Custom Modules

#### 1. MinIO File Module (`modules/minio-file`)

- **Purpose**: S3-compatible object storage
- **Features**:
  - Automatic bucket creation
  - Image upload/download
  - URL generation
- **Fallback**: Local file system

#### 2. Email Notifications (`modules/email-notifications`)

- **Purpose**: Transactional emails
- **Service**: Resend
- **Templates**: React Email components
- **Events**:
  - Order confirmation
  - Shipping updates
  - Password reset

## Data Flow

### Example: Product Purchase Flow

```
1. User browses products
   Storefront → Backend API → MeiliSearch (search)
                            → PostgreSQL (products)

2. User adds to cart
   Storefront → State (React Context)

3. User proceeds to checkout
   Storefront → Backend API → PostgreSQL (create order)
                            → Redis (lock inventory)

4. Payment processing
   Storefront → Backend → Stripe API → Payment confirmed

5. Order confirmation
   Backend → Event Bus → Email Subscriber → Resend → Email sent
          → Workflow → Update inventory
                    → Create fulfillment

6. Admin reviews order
   Admin Dashboard → Backend API → PostgreSQL
```

## Caching Strategy

### Turborepo Cache

- Build outputs cached
- Test results cached
- Remote cache supported

### Redis Cache

- Session data
- Cart contents
- Rate limiting
- Job queues

### Next.js Cache

- Static pages cached
- API responses cached (when configured)
- Image optimization cache

## Security Architecture

### Backend Security

- Environment variables for secrets
- CORS configuration
- Rate limiting via Redis
- JWT for admin authentication
- Webhook signature verification (Stripe)

### Frontend Security

- Server-side rendering for sensitive data
- Client-side validation + server validation
- Secure cookie handling
- XSS prevention via React
- CSRF tokens for mutations

## Scalability Considerations

### Current Architecture

- Single backend instance
- Single database (PostgreSQL)
- Redis for caching/jobs
- MinIO for file storage

### Scaling Path

1. **Horizontal Backend Scaling**

   - Load balancer in front
   - Multiple backend instances
   - Shared Redis and PostgreSQL

2. **Database Scaling**

   - Read replicas
   - Connection pooling (PgBouncer)

3. **File Storage**

   - MinIO distributed mode
   - Or switch to AWS S3

4. **Search**
   - MeiliSearch cluster

## Development Architecture

### Hot Reload

- Backend: Medusa dev server watches files
- Storefront: Next.js Fast Refresh

### Build Pipeline

```
pnpm build
    │
    ├─> turbo run build
    │       │
    │       ├─> build:backend
    │       │       └─> medusa build
    │       │
    │       └─> build:storefront
    │               └─> next build
    │
    └─> Outputs cached by Turborepo
```

## Deployment Architecture

### Railway.app (Default)

- Single template deploys all services
- Automatic environment setup
- Health checks configured
- Automatic SSL

### Architecture on Railway

```
┌─────────────────────────────────────────────┐
│              Railway Project                 │
├─────────────────────────────────────────────┤
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  Backend Service (Node.js)           │  │
│  │  Port: 9000                          │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  Storefront Service (Next.js)        │  │
│  │  Port: 8000                          │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  PostgreSQL (Managed)                │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  Redis (Managed)                     │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  MinIO (Object Storage)              │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  MeiliSearch (Search Engine)         │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

## Technology Decisions

### Why Turborepo?

- Intelligent build caching
- Parallel task execution
- Simple configuration
- Great monorepo DX

### Why pnpm?

- Disk space efficiency
- Strict dependency resolution
- Fast installs
- Native monorepo support

### Why MedusaJS 2.0?

- Modern e-commerce framework
- Headless architecture
- Extensible module system
- Active development

### Why Next.js 14?

- App Router for modern patterns
- Server Components
- Built-in optimization
- Great DX

---

Last updated: 2025-11-27
