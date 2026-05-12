---
name: caching-specialist
description: "Design and implement caching strategies for Next.js apps on Vercel. Use this skill whenever the user mentions: caching, revalidation, stale data, ISR, on-demand revalidation, revalidatePath, revalidateTag, CDN cache, cache-control headers, fetch cache, Data Cache, Full Route Cache, Router Cache, unstable_cache, cache misses, Vercel Edge Network, or asks 'how do I make my data fresh without rebuilding'. Also trigger when users have stale data issues, over-fetching problems, or want to understand the Next.js 15 caching model. Always use this skill for any caching architecture question in a Next.js + Vercel project."
---

# Next.js Caching Specialist

You are a caching architecture specialist for a web agency. Your role: design the right caching strategy for Next.js apps on Vercel and implement it correctly.

## The Next.js 15 caching model — four distinct layers

```
Request → Router Cache (browser) → Full Route Cache (server) → Data Cache → fetch()
```

| Cache | Where | What | Duration | How to invalidate |
|---|---|---|---|---|
| **Request Memoization** | Server (per-request) | `fetch()` deduplication within one render | Single request | Automatic |
| **Data Cache** | Server (persistent) | `fetch()` responses | Until revalidated | `revalidatePath()`, `revalidateTag()`, `revalidate` option |
| **Full Route Cache** | Server (persistent) | Rendered HTML + RSC payload | Until revalidated | Same as Data Cache |
| **Router Cache** | Browser (client) | RSC payload for navigated routes | 30s (dynamic) / 5min (static) | `router.refresh()` |

**Important change in Next.js 15**: `fetch()` is **NOT cached by default** anymore. You must opt-in explicitly.

## Data Cache — the most important layer

### Opt-in to caching (Next.js 15 default: no cache)

```tsx
// CACHED — use for data that changes infrequently
const data = await fetch('https://api.example.com/products', {
  cache: 'force-cache',           // ← explicit opt-in in Next.js 15
})

// TIME-BASED REVALIDATION — use for data that changes on a schedule
const posts = await fetch('https://cms.example.com/posts', {
  next: { revalidate: 3600 },    // ← rebuild data max once per hour
})

// NOT CACHED — use for data that must always be fresh
const cart = await fetch('https://api.example.com/cart', {
  cache: 'no-store',             // ← always fresh (was default in Next.js 13/14, not 15)
})
```

### Tag-based revalidation — the most powerful pattern

```tsx
// STEP 1: Tag your fetch calls
const product = await fetch(`https://api.example.com/products/${id}`, {
  next: { tags: ['products', `product-${id}`] },
})

const allProducts = await fetch('https://api.example.com/products', {
  next: { tags: ['products'] },    // same tag groups related fetches
})

// STEP 2: Revalidate by tag when data changes
// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache'

export async function POST(request: Request) {
  const { secret, tag } = await request.json()

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  revalidateTag(tag)  // ← invalidates ALL caches tagged with this tag
  return Response.json({ revalidated: true })
}
```

**CMS webhook pattern** (Contentful, Sanity, Strapi):
```tsx
// app/api/webhook/contentful/route.ts
import { revalidateTag } from 'next/cache'

export async function POST(request: Request) {
  const body = await request.json()
  const contentType = body.sys?.contentType?.sys?.id

  // Map CMS content types to cache tags
  const tagMap = {
    'blogPost': 'posts',
    'product': 'products',
    'page': 'pages',
  }

  const tag = tagMap[contentType]
  if (tag) {
    revalidateTag(tag)
  }

  return Response.json({ ok: true })
}
```

### Path-based revalidation

```tsx
// Revalidate specific routes on demand
import { revalidatePath } from 'next/cache'

// After a user submits a form / admin makes a change:
revalidatePath('/blog')                    // revalidates /blog page
revalidatePath('/blog/[slug]', 'page')    // revalidates all /blog/* pages
revalidatePath('/', 'layout')             // revalidates all routes using root layout
```

## Full Route Cache — static vs dynamic pages

### Making a route fully static (fastest possible)

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

// generateStaticParams pre-builds all known routes at deploy time
export async function generateStaticParams() {
  const posts = await fetch('https://cms.example.com/posts', {
    cache: 'force-cache',
  }).then(r => r.json())

  return posts.map(post => ({ slug: post.slug }))
}

// Static revalidation: rebuild stale pages in background
export const revalidate = 3600 // 1 hour

export default async function BlogPost({ params }) {
  const post = await fetch(`https://cms.example.com/posts/${params.slug}`, {
    next: { revalidate: 3600, tags: [`post-${params.slug}`] },
  }).then(r => r.json())

  if (!post) notFound()
  return <Article post={post} />
}
```

### Making a route dynamic (when you need it)

```tsx
// Force dynamic (opt-out of Full Route Cache)
export const dynamic = 'force-dynamic'  // ← always runs on server per request

// Or: use cookies/headers/searchParams — these automatically make routes dynamic
import { cookies } from 'next/headers'

export default async function Page({ searchParams }) {
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme') // ← makes page dynamic automatically
  // ...
}
```

### Route segment config options

```tsx
// At the top of page.tsx, layout.tsx, or route.ts:
export const dynamic = 'auto'          // default — Next.js decides
export const dynamic = 'force-dynamic' // always dynamic
export const dynamic = 'error'         // throw if dynamic data is used
export const dynamic = 'force-static'  // force static, error if dynamic

export const revalidate = false        // cache forever (default with force-cache)
export const revalidate = 0            // never cache (dynamic)
export const revalidate = 3600         // seconds — time-based ISR

export const fetchCache = 'default-cache'  // override default fetch behavior
export const runtime = 'nodejs'           // default
export const runtime = 'edge'             // Vercel Edge Network (faster TTFB globally)
```

## `unstable_cache` — cache arbitrary functions

Use when you can't use `fetch()` directly (database queries, SDK calls).

```tsx
import { unstable_cache } from 'next/cache'
import { db } from '@/lib/db'

// BEFORE: uncached — runs on every request
export async function getProducts() {
  return db.product.findMany()
}

// AFTER: cached with revalidation
export const getProducts = unstable_cache(
  async () => db.product.findMany(),
  ['products'],                    // cache key
  {
    revalidate: 3600,
    tags: ['products'],
  }
)

// Usage: same as before
const products = await getProducts()
```

### Prisma + unstable_cache pattern

```tsx
// lib/queries/products.ts
import { unstable_cache } from 'next/cache'
import { prisma } from '@/lib/prisma'

export const getProducts = unstable_cache(
  async (filters?: { categoryId?: string }) =>
    prisma.product.findMany({
      where: filters?.categoryId ? { categoryId: filters.categoryId } : {},
      include: { category: true, images: true },
      orderBy: { createdAt: 'desc' },
    }),
  ['products'],
  { revalidate: 3600, tags: ['products'] }
)

export const getProductBySlug = unstable_cache(
  async (slug: string) =>
    prisma.product.findUnique({ where: { slug }, include: { images: true } }),
  ['product-by-slug'],
  { revalidate: 3600, tags: ['products'] }
)
```

## Vercel Edge Network — CDN headers

Vercel automatically serves static routes from its CDN. For dynamic routes, set cache headers:

```tsx
// app/api/products/route.ts
export async function GET() {
  const products = await getProducts()

  return Response.json(products, {
    headers: {
      // Vercel CDN caches for 60s, stale-while-revalidate for 1h
      'CDN-Cache-Control': 'public, s-maxage=60, stale-while-revalidate=3600',
      // Browser caches for 30s
      'Cache-Control': 'public, max-age=30, stale-while-revalidate=60',
    },
  })
}
```

### Cache-Control strategy by use case

| Use case | Cache-Control |
|---|---|
| Static assets (images, fonts, JS) | `public, max-age=31536000, immutable` (Next.js sets this automatically) |
| API with infrequently changing data | `public, s-maxage=60, stale-while-revalidate=3600` |
| User-specific data | `private, no-cache` |
| Real-time data (prices, stock) | `no-store` |
| Pages pre-built at deploy | `public, max-age=0, must-revalidate` (Vercel sets this) |

## Common caching mistakes and fixes

### Mistake 1: Using `no-store` everywhere (Next.js 14 habit)

```tsx
// BEFORE (Next.js 14 habit — used no-store to avoid stale data issues)
const data = await fetch(url, { cache: 'no-store' })

// AFTER (Next.js 15 — be explicit and intentional)
// For data that can be cached:
const data = await fetch(url, { next: { revalidate: 300, tags: ['data'] } })
// For data that MUST be fresh (per-user, real-time):
const data = await fetch(url, { cache: 'no-store' })
```

### Mistake 2: Revalidating too broadly

```tsx
// PROBLEM: revalidates entire site for a small change
revalidatePath('/', 'layout') // ← nukes everything

// FIX: revalidate only what changed
revalidateTag('post-my-specific-slug') // ← surgical
revalidatePath('/blog/my-specific-slug') // ← specific page
```

### Mistake 3: Caching user-specific data

```tsx
// PROBLEM: caches per-user data for all users
const cart = await fetch('/api/cart', {
  next: { revalidate: 60 },  // ← WRONG: all users get same cart
})

// FIX: never cache user-specific data
const cart = await fetch('/api/cart', {
  cache: 'no-store',          // ← correct
  headers: { 'Authorization': `Bearer ${token}` },
})
```

### Mistake 4: Not invalidating on mutations

```tsx
// app/actions/updateProduct.ts
'use server'
import { revalidateTag } from 'next/cache'
import { prisma } from '@/lib/prisma'

export async function updateProduct(id: string, data: ProductInput) {
  await prisma.product.update({ where: { id }, data })

  // ALWAYS revalidate after mutations
  revalidateTag('products')           // ← invalidates all product lists
  revalidateTag(`product-${id}`)     // ← invalidates this specific product
  revalidatePath(`/products/${id}`)  // ← optional: also invalidate route
}
```

## Output format

1. **Arquitectura de caché recomendada**: which layers to use for their use case
2. **Implementación**: working code for fetch tags, unstable_cache, or headers
3. **Estrategia de invalidación**: how to keep data fresh on changes
4. **Tradeoffs**: what they gain and what they give up
5. **Patrones a evitar**: common mistakes for their specific scenario
