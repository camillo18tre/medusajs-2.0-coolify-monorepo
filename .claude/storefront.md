# Storefront Documentation (Next.js 14)

## Overview

The storefront is a **Next.js 14** application using the **App Router** architecture, providing:
- Server-side rendering (SSR)
- Server Components by default
- Client Components when needed
- Modern React patterns
- Optimized performance

**Package**: `medusa-storefront`
**Port**: 8000 (default)
**Framework**: Next.js 14.0.0 with React 18

## Directory Structure

```
apps/storefront/
├── src/
│   ├── app/                        # Next.js 14 App Router
│   │   ├── [countryCode]/         # Dynamic country routes
│   │   │   ├── (main)/            # Main layout group
│   │   │   │   ├── page.tsx       # Homepage
│   │   │   │   ├── products/      # Product pages
│   │   │   │   ├── collections/   # Collection pages
│   │   │   │   ├── categories/    # Category pages
│   │   │   │   ├── cart/          # Cart page
│   │   │   │   ├── checkout/      # Checkout flow
│   │   │   │   └── account/       # User account
│   │   │   └── layout.tsx         # Country layout
│   │   ├── api/                   # API routes (middleware)
│   │   ├── layout.tsx             # Root layout
│   │   └── globals.css            # Global styles
│   │
│   ├── lib/                        # Utilities and configs
│   │   ├── context/               # React Contexts
│   │   ├── data/                  # Data fetching functions
│   │   ├── hooks/                 # Custom React hooks
│   │   └── util/                  # Helper functions
│   │
│   ├── modules/                    # Feature modules
│   │   ├── account/               # Account management
│   │   ├── cart/                  # Shopping cart
│   │   ├── categories/            # Category display
│   │   ├── checkout/              # Checkout process
│   │   ├── collections/           # Collection display
│   │   ├── common/                # Shared components
│   │   ├── home/                  # Homepage components
│   │   ├── layout/                # Layout components
│   │   ├── order/                 # Order display
│   │   ├── products/              # Product display
│   │   ├── search/                # Search components
│   │   ├── skeletons/             # Loading skeletons
│   │   └── store/                 # Store utilities
│   │
│   ├── styles/                     # Styling
│   │   └── globals.css            # Tailwind + custom CSS
│   │
│   └── types/                      # TypeScript types
│       └── global.d.ts            # Global type definitions
│
├── public/                         # Static assets
│   ├── images/
│   └── ...
│
├── .env.local.template             # Environment template
├── next.config.js                  # Next.js configuration
├── tailwind.config.js              # Tailwind CSS config
├── postcss.config.js               # PostCSS config
└── package.json
```

## Configuration

### Environment Variables

```bash
# Backend API URL
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000

# Storefront URL (used for absolute URLs)
NEXT_PUBLIC_BASE_URL=http://localhost:8000

# Default region when user region cannot be determined
NEXT_PUBLIC_DEFAULT_REGION=us

# MinIO endpoint (optional, for image URLs)
NEXT_PUBLIC_MINIO_ENDPOINT=bucket-production-eaeb.up.railway.app

# MeiliSearch Configuration
NEXT_PUBLIC_SEARCH_ENDPOINT=http://localhost:7700
NEXT_PUBLIC_SEARCH_API_KEY=your_search_key_here
NEXT_PUBLIC_INDEX_NAME=products
```

### Next.js Configuration

**File**: [next.config.js](../apps/storefront/next.config.js)

Key configurations:
- Image optimization domains
- Standalone output for production
- Webpack configuration
- Environment variables

## Architecture Patterns

### App Router (Next.js 14)

This storefront uses the **App Router**, not the legacy Pages Router.

**Key Concepts**:
1. **File-based routing** in `src/app/`
2. **Server Components** by default
3. **Client Components** with `"use client"` directive
4. **Layouts** for shared UI
5. **Loading states** with `loading.tsx`
6. **Error boundaries** with `error.tsx`

### Route Structure

```
app/
├── layout.tsx                    # Root layout (wraps everything)
├── [countryCode]/               # Dynamic country segment
│   ├── layout.tsx              # Country-specific layout
│   ├── page.tsx                # Homepage
│   ├── products/
│   │   └── [handle]/
│   │       └── page.tsx        # Product detail page
│   ├── collections/
│   │   └── [handle]/
│   │       └── page.tsx        # Collection page
│   ├── categories/
│   │   └── [...category]/
│   │       └── page.tsx        # Category page (catch-all)
│   ├── cart/
│   │   └── page.tsx            # Cart page
│   ├── checkout/
│   │   └── page.tsx            # Checkout page
│   └── account/
│       ├── page.tsx            # Account overview
│       ├── profile/
│       ├── addresses/
│       └── orders/
└── api/                         # API routes (server-side only)
    └── ...
```

### Server Components vs Client Components

**Server Components** (default):
- Run on the server only
- Can directly access backend APIs
- No JavaScript sent to client
- Better performance

**Client Components** (`"use client"`):
- Run on both server and client
- Can use React hooks (useState, useEffect, etc.)
- Interactive elements
- Browser APIs

**Example**:
```tsx
// Server Component (default)
export default async function ProductPage({ params }) {
  const product = await getProduct(params.handle)  // Direct data fetch
  return <ProductTemplate product={product} />
}

// Client Component
'use client'
export function AddToCart({ productId }) {
  const [loading, setLoading] = useState(false)

  const handleClick = () => {
    // Interactive logic
  }

  return <button onClick={handleClick}>Add to Cart</button>
}
```

## Data Fetching

### Medusa SDK Integration

The storefront uses `@medusajs/js-sdk` for API communication.

**Location**: `src/lib/data/`

**Pattern**:
```typescript
// src/lib/data/products.ts
import { sdk } from "@lib/config"

export async function getProduct(handle: string) {
  const { products } = await sdk.store.product.list({
    handle,
    fields: "+variants.prices,+images"
  })

  return products[0]
}
```

### Data Fetching Strategies

1. **Server Component Fetch** (recommended):
```tsx
export default async function Page() {
  const data = await fetchData()  // Fetched on server
  return <Component data={data} />
}
```

2. **Client-Side Fetch**:
```tsx
'use client'
export function Component() {
  const { data } = useSWR('/api/data', fetcher)
  return <div>{data}</div>
}
```

3. **API Routes**:
```tsx
// app/api/custom/route.ts
export async function GET() {
  const data = await fetchFromBackend()
  return Response.json(data)
}
```

## Feature Modules

### 1. Account Module (`modules/account`)

**Features**:
- User registration
- Login/logout
- Profile management
- Address management
- Order history

**Key Components**:
- `Login` - Login form
- `Register` - Registration form
- `AccountNav` - Account navigation
- `ProfileForm` - Profile editor
- `AddressBook` - Address management

### 2. Cart Module (`modules/cart`)

**Features**:
- Add/remove items
- Update quantities
- Apply discounts
- Cart persistence

**Key Components**:
- `CartDropdown` - Mini cart
- `CartTemplate` - Full cart page
- `CartTotals` - Price breakdown
- `ItemsTemplate` - Cart item list

**State Management**: React Context

### 3. Checkout Module (`modules/checkout`)

**Features**:
- Shipping address
- Shipping method selection
- Payment method selection
- Order review
- Stripe integration

**Key Components**:
- `CheckoutForm` - Main checkout form
- `ShippingForm` - Shipping details
- `PaymentForm` - Payment details
- `StripeWrapper` - Stripe Elements wrapper

**Flow**:
```
Cart → Shipping Address → Shipping Method → Payment → Confirmation
```

### 4. Products Module (`modules/products`)

**Features**:
- Product display
- Variant selection
- Image gallery
- Related products

**Key Components**:
- `ProductTemplate` - Main product page
- `ProductInfo` - Product details
- `ImageGallery` - Product images
- `VariantSelector` - Size/color picker
- `ProductActions` - Add to cart

### 5. Search Module (`modules/search`)

**Features**:
- Instant search (MeiliSearch)
- Faceted filters
- Search suggestions
- Product grid

**Key Components**:
- `SearchModal` - Search overlay
- `SearchResults` - Results display
- `Filters` - Faceted navigation

**Integration**: `react-instantsearch-hooks-web` with MeiliSearch

### 6. Collections & Categories

**Collections** (`modules/collections`):
- Curated product groups
- Marketing collections

**Categories** (`modules/categories`):
- Hierarchical product taxonomy
- Nested category support

### 7. Layout Module (`modules/layout`)

**Shared Layout Components**:
- `Header` - Site header
- `Footer` - Site footer
- `Navigation` - Main nav
- `MobileMenu` - Mobile navigation
- `SearchBar` - Search input

## Styling

### Tailwind CSS

The storefront uses **Tailwind CSS** for styling.

**Configuration**: [tailwind.config.js](../apps/storefront/tailwind.config.js)

**Usage**:
```tsx
<div className="container mx-auto px-4 py-8">
  <h1 className="text-3xl font-bold">Product Title</h1>
  <button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
    Add to Cart
  </button>
</div>
```

### Medusa UI Components

Uses `@medusajs/ui` for consistent UI components:
- Buttons
- Inputs
- Modals
- Dropdowns
- etc.

### Custom Styles

Global styles in `src/styles/globals.css`:
- Tailwind directives
- Custom CSS variables
- Component-specific styles

## State Management

### React Context

**Cart Context**:
```tsx
// lib/context/cart-context.tsx
export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState(null)

  return (
    <CartContext.Provider value={{ cart, setCart }}>
      {children}
    </CartContext.Provider>
  )
}

// Usage
const { cart, setCart } = useCart()
```

**Account Context**:
- User authentication state
- User profile data

### URL State

- Search filters in query params
- Pagination in query params
- Region in URL path (`[countryCode]`)

## Internationalization (i18n)

### Multi-Region Support

**Pattern**: `/{countryCode}/...`

Examples:
- `/us/products/shirt` - US region
- `/eu/products/shirt` - EU region
- `/uk/products/shirt` - UK region

**Benefits**:
- Region-specific pricing
- Region-specific products
- Localized content

**Middleware**: Detects user region and redirects appropriately

## Search Integration (MeiliSearch)

### Configuration

Uses `@meilisearch/instant-meilisearch` adapter with `react-instantsearch-hooks-web`.

**Setup**:
```tsx
import { instantMeiliSearch } from '@meilisearch/instant-meilisearch'
import { InstantSearch } from 'react-instantsearch-hooks-web'

const searchClient = instantMeiliSearch(
  NEXT_PUBLIC_SEARCH_ENDPOINT,
  NEXT_PUBLIC_SEARCH_API_KEY
)

<InstantSearch
  indexName={NEXT_PUBLIC_INDEX_NAME}
  searchClient={searchClient}
>
  {/* Search components */}
</InstantSearch>
```

### Search Features

- **Instant results** as you type
- **Faceted filters** (category, price, etc.)
- **Typo tolerance**
- **Highlighting** of search terms
- **Pagination**

## Payment Integration (Stripe)

### Stripe Elements

Uses `@stripe/react-stripe-js` for secure payment forms.

**Components**:
- `StripeWrapper` - Provides Stripe context
- `PaymentElement` - Stripe's unified payment UI

**Flow**:
1. Create payment intent on backend
2. Load Stripe Elements
3. User enters payment details
4. Confirm payment
5. Handle success/failure

## Image Optimization

### Next.js Image Component

```tsx
import Image from 'next/image'

<Image
  src={product.thumbnail}
  alt={product.title}
  width={600}
  height={600}
  priority={true}
  className="object-cover"
/>
```

**Benefits**:
- Automatic lazy loading
- Responsive images
- WebP/AVIF support
- Blur placeholder

### Image Domains

Configure allowed image domains in `next.config.js`:
```javascript
images: {
  domains: [
    'localhost',
    'medusa-public-images.s3.eu-west-1.amazonaws.com',
    // Add MinIO domain
  ]
}
```

## Performance Optimization

### Server Components

Default to Server Components for:
- Static content
- Data fetching
- SEO-critical pages

### Client Components

Use sparingly for:
- Interactive elements
- Browser APIs
- React hooks

### Code Splitting

Automatic code splitting per route.

**Manual splitting**:
```tsx
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <Skeleton />,
  ssr: false  // Disable SSR if needed
})
```

### Caching

- Static pages cached by default
- ISR (Incremental Static Regeneration) for semi-static pages
- CDN caching for public assets

## Development Commands

```bash
# Install dependencies
pnpm install

# Development mode
pnpm dev

# Build for production
pnpm build

# Start production build
pnpm start

# Lint
pnpm lint

# Type checking
pnpm check-types

# Build Next.js only (no wait for backend)
pnpm build:next
```

## Common Development Patterns

### Creating a New Page

1. Create file in `app/[countryCode]/`
```tsx
// app/[countryCode]/new-page/page.tsx
export default async function NewPage() {
  return <div>New Page</div>
}
```

2. Add to navigation if needed

### Adding a New Component

1. Create in appropriate `modules/` directory
```tsx
// modules/products/components/new-component.tsx
export function NewComponent() {
  return <div>New Component</div>
}
```

2. Export from module index
```tsx
// modules/products/index.ts
export { NewComponent } from './components/new-component'
```

### Fetching Data

**Server Component**:
```tsx
// app/[countryCode]/products/page.tsx
import { getProducts } from '@lib/data/products'

export default async function ProductsPage() {
  const products = await getProducts()

  return <ProductGrid products={products} />
}
```

**Client Component**:
```tsx
'use client'
import { useState, useEffect } from 'react'

export function ClientComponent() {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetch('/api/data')
      .then(r => r.json())
      .then(setData)
  }, [])

  return <div>{data?.title}</div>
}
```

## Troubleshooting

### Storefront Won't Start

**Check**:
1. Backend is running
2. `NEXT_PUBLIC_MEDUSA_BACKEND_URL` is correct
3. Port 8000 is available
4. Dependencies installed

### Images Not Loading

**Check**:
1. Image domain in `next.config.js`
2. Backend serving images correctly
3. MinIO configured if used

### Search Not Working

**Check**:
1. MeiliSearch is running
2. Environment variables set
3. Products indexed in MeiliSearch
4. API key is correct

### Build Failures

**Common Issues**:
1. TypeScript errors - run `pnpm check-types`
2. Missing environment variables
3. Backend not accessible during build

## Testing

```bash
# E2E tests (Playwright)
pnpm test-e2e

# Type checking
pnpm check-types
```

## Security Considerations

1. **Environment Variables**: Only `NEXT_PUBLIC_*` exposed to client
2. **API Routes**: Server-side only logic in `/api` routes
3. **XSS Prevention**: React automatically escapes content
4. **CORS**: Backend must allow storefront origin
5. **Payment**: Use Stripe Elements, never handle card data directly

## Useful Resources

- [Next.js 14 Docs](https://nextjs.org/docs)
- [App Router Guide](https://nextjs.org/docs/app)
- [MedusaJS Storefront Docs](https://docs.medusajs.com/storefront)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Stripe Elements](https://stripe.com/docs/stripe-js)

---

Last updated: 2025-11-27
