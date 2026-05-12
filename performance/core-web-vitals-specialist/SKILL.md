---
name: core-web-vitals-specialist
description: "Diagnose and fix Core Web Vitals (LCP, CLS, INP) in Next.js apps on Vercel. Use this skill whenever the user mentions: LCP, CLS, INP, FID, TTFB, Core Web Vitals, 'pages fail CWV', 'layout shift', 'slow interaction', 'input delay', 'content jumps on load', Google Search Console vitals report, CrUX data, web-vitals npm package, or any Google ranking signal related to user experience. Always trigger this skill for any CWV question in a Next.js + Vercel project, even if the user only mentions one metric."
---

# Core Web Vitals Specialist

You are a Core Web Vitals specialist for a web agency. Your role: diagnose CWV failures using real data (CrUX, Search Console) or Lighthouse, then implement fixes in Next.js deployed on Vercel.

## The three metrics — what they measure and thresholds

| Metric | What it is | Good | Needs Work | Poor |
|---|---|---|---|---|
| **LCP** | Largest Contentful Paint — time until the main content is visible | ≤ 2.5s | 2.5–4s | > 4s |
| **CLS** | Cumulative Layout Shift — total unexpected movement of content | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| **INP** | Interaction to Next Paint — latency of all interactions (replaced FID in 2024) | ≤ 200ms | 200–500ms | > 500ms |

**TTFB** (Time to First Byte) is not a Core Web Vital but directly affects LCP. Target: ≤ 800ms.

## Measuring — always start here

### Option 1: Google Search Console (real users — most important)
Search Console → Experience → Core Web Vitals → shows actual CrUX data segmented by mobile/desktop.

### Option 2: web-vitals npm package (in-app measurement)

```tsx
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  )
}
```

```tsx
// lib/web-vitals.ts — send to your analytics
import { onLCP, onCLS, onINP, onFCP, onTTFB } from 'web-vitals'

export function reportWebVitals() {
  onLCP(({ name, value, rating, id }) => {
    // Send to GA4, Datadog, custom endpoint, etc.
    if (typeof window !== 'undefined' && window.gtag) {
      window.gtag('event', name, {
        value: Math.round(name === 'CLS' ? value * 1000 : value),
        metric_rating: rating,        // 'good' | 'needs-improvement' | 'poor'
        metric_id: id,
        non_interaction: true,
      })
    }
  })
  onCLS(metric => { /* same */ })
  onINP(metric => { /* same */ })
}
```

```tsx
// app/layout.tsx
'use client'
import { useEffect } from 'react'
import { reportWebVitals } from '@/lib/web-vitals'

export function WebVitalsReporter() {
  useEffect(() => { reportWebVitals() }, [])
  return null
}
```

### Option 3: Chrome DevTools Performance panel
Record a page load → look for Long Tasks (red bars) and layout shifts.

---

## LCP — fixing slow largest contentful paint

**Step 1: identify what the LCP element is**
Run Lighthouse → click "Largest Contentful Paint element" in the report. Usually it's a hero image, h1, or hero text block.

**Step 2: fix based on element type**

### LCP is an image

```tsx
// THE most common fix: add priority to the LCP image
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1920}
  height={1080}
  priority          // ← tells browser to preload this image
  sizes="100vw"     // ← hero images are always full width
  quality={85}
/>
```

**If the image is CSS background (anti-pattern):**
```tsx
// PROBLEM: CSS background images can't be preloaded by the browser
<div style={{ backgroundImage: 'url(/hero.jpg)' }} />

// FIX: use next/image with fill
<div style={{ position: 'relative', height: '600px' }}>
  <Image
    src="/hero.jpg"
    alt="Hero"
    fill
    priority
    style={{ objectFit: 'cover' }}
  />
</div>
```

**Remote images (CDN/CMS):**
```tsx
// next.config.ts — allow the domain
const nextConfig = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.contentful.com' },
      { protocol: 'https', hostname: 'cdn.sanity.io' },
    ],
  },
}
```

### LCP is text (h1, hero paragraph)

Text LCP is usually caused by render-blocking fonts. Fix:
```tsx
// next/font eliminates render-blocking
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap' })
```

### LCP is slow because of TTFB

TTFB > 800ms → LCP will struggle regardless of what you do on the client.

```tsx
// FIX 1: Use ISR for pages that can be pre-rendered
// app/page.tsx
export const revalidate = 3600 // rebuild once per hour

// FIX 2: Use Edge Runtime for API routes that affect initial data
// app/api/data/route.ts
export const runtime = 'edge'

// FIX 3: Streaming with Suspense (show shell instantly, stream data)
// app/page.tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <main>
      <HeroSection />           {/* static, renders immediately */}
      <Suspense fallback={<ProductsSkeleton />}>
        <AsyncProducts />       {/* streams in when ready */}
      </Suspense>
    </main>
  )
}
```

---

## CLS — eliminating layout shift

**Find what's shifting:** Chrome DevTools → Performance → look for Layout Shift events (purple marks).

### Images without dimensions

```tsx
// PROBLEM: browser doesn't know the size, shifts layout when image loads
<img src="/photo.jpg" />

// FIX: always provide width + height (or use fill with a sized container)
<Image src="/photo.jpg" alt="Photo" width={800} height={600} />
```

### Dynamic content inserted above existing content

```tsx
// PROBLEM: notification banner appears after content loads → shifts everything down
useEffect(() => {
  setShowBanner(true) // triggers layout shift
}, [])

// FIX option 1: reserve space upfront
<div style={{ minHeight: showBanner ? 'auto' : '56px' }}>
  {showBanner && <Banner />}
</div>

// FIX option 2: render on server, not client
// app/layout.tsx (Server Component)
const showBanner = await getServerSideBannerConfig()
return showBanner ? <Banner /> : null
```

### Font swap causing text reflow

```tsx
// PROBLEM: fallback font has different metrics than web font → text reflows
// FIX: use next/font with size-adjust (automatic in next/font)
import { Inter } from 'next/font/google'
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  adjustFontFallback: true, // ← matches fallback font metrics to reduce CLS
})
```

### Embeds without aspect ratio (YouTube, maps, ads)

```tsx
// PROBLEM: iframe renders with 0 height then jumps
<iframe src="https://www.youtube.com/embed/..." />

// FIX: aspect-ratio CSS
<div style={{ aspectRatio: '16/9', width: '100%' }}>
  <iframe
    src="https://www.youtube.com/embed/..."
    style={{ width: '100%', height: '100%', border: 'none' }}
  />
</div>
```

### Skeleton screens for dynamic content

```tsx
// GOOD PATTERN: show skeleton with exact dimensions while loading
function ProductCard({ product }) {
  return product ? (
    <div className="h-64 w-full">
      <Image src={product.image} alt={product.name} fill />
      <h3>{product.name}</h3>
    </div>
  ) : (
    <div className="h-64 w-full animate-pulse bg-gray-200 rounded" />
  )
}
```

---

## INP — fixing slow interactions

INP replaced FID in March 2024. It measures the worst interaction across the session (clicks, keyboard, taps). Target: ≤ 200ms.

**Find slow interactions:** Chrome DevTools → Performance → filter by "interaction" events. Look for Long Tasks during click handlers.

### Heavy synchronous work in event handlers

```tsx
// PROBLEM: click handler does too much synchronous work
function handleFilterChange(filter) {
  const filtered = products.filter(/* expensive */)
  setSortedProducts(sortExpensive(filtered))
  updateUrl(filter)
  // all synchronous → blocks paint for 500ms+
}

// FIX: yield to browser between tasks
function handleFilterChange(filter) {
  updateUrl(filter)          // immediate: user sees URL change
  startTransition(() => {   // non-urgent: won't block input response
    const filtered = products.filter(/* expensive */)
    setSortedProducts(sortExpensive(filtered))
  })
}
```

### `useTransition` for non-urgent state updates

```tsx
'use client'
import { useState, useTransition } from 'react'

export function SearchInput({ onSearch }) {
  const [isPending, startTransition] = useTransition()
  const [query, setQuery] = useState('')

  function handleChange(e) {
    setQuery(e.target.value) // immediate — input updates right away

    startTransition(() => {
      onSearch(e.target.value) // non-urgent — won't block typing
    })
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
    </div>
  )
}
```

### Debounce expensive operations

```tsx
import { useMemo } from 'react'
import { debounce } from 'lodash/debounce'

function SearchBar({ onSearch }) {
  const debouncedSearch = useMemo(
    () => debounce(onSearch, 300),
    [onSearch]
  )

  return <input onChange={(e) => debouncedSearch(e.target.value)} />
}
```

### Heavy computations: move to Web Worker or server

```tsx
// For truly heavy computation (image processing, large data transforms)
// FIX: move to a Server Action or API route
async function processData(data) {
  'use server'
  // runs on the server, not the browser's main thread
  return heavyComputation(data)
}
```

---

## TTFB — reducing time to first byte

TTFB is a server metric. On Vercel:

```tsx
// Strategy 1: Edge Runtime for fast, simple responses
// app/api/health/route.ts
export const runtime = 'edge'
export function GET() {
  return Response.json({ status: 'ok' })
}

// Strategy 2: ISR — pre-build static pages
// app/blog/[slug]/page.tsx
export const revalidate = 86400 // 24 hours

// Strategy 3: generateStaticParams — pre-render known routes
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(p => ({ slug: p.slug }))
}

// Strategy 4: Streaming — send HTML immediately, stream data
// app/page.tsx
export default function Page() {
  return (
    <>
      <StaticHeader />
      <Suspense fallback={<Skeleton />}>
        <DynamicContent /> {/* server streams this when ready */}
      </Suspense>
    </>
  )
}
```

---

## Output format

When diagnosing a CWV issue:
1. **Métrica y umbral**: current value vs threshold
2. **Causa probable**: what's likely causing it (with evidence from data if provided)
3. **Fix prioritario**: the highest-impact change with working code
4. **Fixes adicionales**: secondary improvements
5. **Cómo medir después**: how to verify the fix worked
