# Image Curation Report — Café Vermut E-Commerce

**Specialist:** image-curator  
**Client:** Café Vermut — Specialty Coffee E-Commerce, Barcelona  
**Stack:** Next.js 16, next/image, Supabase Storage, Stripe  
**Skill version:** v1.0  
**Date:** 2026-05-13

---

## Step 1 — E-Commerce Visual Strategy Audit

Café Vermut sells specialty single-origin coffee. The purchase decision is sensory — it happens through smell memory, origin narrative, and aesthetic trust. Images must do three jobs simultaneously:

1. **Trigger sensory anticipation** (the customer should almost smell the coffee)
2. **Signal provenance credibility** (single-origin means origin imagery matters)
3. **Simplify the purchase decision** (clean product photography reduces friction)

**Visual direction constraint:** "Japan meets Barcelona" — this means less is more. No cluttered product pages. One hero product, surrounded by space.

---

## Step 2 — Image Architecture for E-Commerce

### 2.1 — Product Image System

Each product in the catalog requires a standardized image set:

| Image type | Slot | Display size | Source minimum | Purpose |
|---|---|---|---|---|
| `main` | Product card thumbnail | 400×400 | 800×800 | Grid listing |
| `hero` | Product detail page | 640×640 | 1280×1280 | Primary product view |
| `angle_1` | PDP gallery | 640×640 | 1280×1280 | Side/back view |
| `lifestyle` | PDP gallery | 640×480 | 1280×960 | Brewed / ritual context |
| `origin` | PDP — origin story | 800×500 | 1600×1000 | Farm/country of origin |

**Total per product: 5 images** (main + hero + angle + lifestyle + origin)

For an initial catalog of ~20 products: **100 images** needed.

### 2.2 — Shared Assets (not per-product)

| Asset | Display | Source | Purpose |
|---|---|---|---|
| `hero-landing` | Full viewport | 2400×1350 | Homepage hero |
| `hero-tienda` | Full viewport | 2400×1350 | /tienda category page |
| `about-team` | 1200×800 | 2400×1600 | /nosotros page |
| `brewing-tutorial-1–4` | 800×600 | 1600×1200 | Blog / how-to content |
| `og-default` | 1200×630 | — | Default social share |
| `og-product-[slug]` | 1200×630 | — | Per-product social share |

---

## Step 3 — Sourcing Plan

### Tier 1 — Client Photography (Priority)

**Brief to director for client handoff:**

> Café Vermut needs product photography for their online store. We recommend a professional food/product photographer for a 1-day studio session:
> - 20 products × (main + hero + angle_1 views) = 60 studio shots
> - 10–15 lifestyle shots (brewing ritual, hands, ceramic cups, morning light)
> - Setup: white seamless background for product shots; linen/wood surface for lifestyle
> - Lighting: natural or warm-balanced (match `#F8F7F4` base tone)
> - Budget estimate: 800–1500€ for Barcelona-based food photographer (1-day rate)
> - Deliverable: RAW + JPEG, 24MP+, at least 3000px on shortest side

**Origin imagery:** Request directly from coffee suppliers (most specialty roasters have origin farm photos available under usage license).

### Tier 2 — Unsplash (Staging/Placeholder)

For development phase, use Unsplash as placeholder images:

| Need | Query | Selection notes |
|---|---|---|
| Hero landing | `specialty coffee minimal white` | Warm, controlled — not dark moody |
| Product lifestyle | `coffee pour over ceramic morning` | Japanese aesthetic, light-filled |
| Ethiopia origin | `ethiopia coffee landscape farm` | Natural, green, authentic |
| Colombia origin | `colombia coffee mountain harvest` | Natural, mountain, human element |
| Guatemala origin | `guatemala coffee plantation altitude` | Altitude/fog aesthetic |
| Brewing detail | `coffee ground espresso macro` | Texture, sensory detail |

### Tier 3 — Paid Stock (If Needed)

- **Stocksy:** Best lifestyle and editorial-quality coffee photography. ~$30–80/image. Reserve for hero shots and OG images if client photography delayed.
- **Avoid:** Getty clichés (white mug on white table), anything with visible brand names.

---

## Step 4 — Technical Implementation

### 4.1 — next/image Configuration

```tsx
// next.config.ts — allow Supabase storage + Unsplash for placeholders
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '*.supabase.co',
        pathname: '/storage/v1/object/public/**',
      },
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
      },
    ],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384, 400, 640],
  },
}

export default nextConfig
```

### 4.2 — Product Image Components

```tsx
// components/product/ProductImage.tsx
import Image from 'next/image'

interface ProductImageProps {
  src: string
  alt: string
  priority?: boolean
  variant?: 'card' | 'detail'
}

export function ProductImage({ src, alt, priority = false, variant = 'card' }: ProductImageProps) {
  const sizes =
    variant === 'card'
      ? '(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw'
      : '(max-width: 768px) 100vw, 640px'

  return (
    <div className="relative aspect-square overflow-hidden bg-[#F8F7F4]">
      <Image
        src={src}
        alt={alt}
        fill
        priority={priority}
        sizes={sizes}
        className="object-contain transition-transform duration-500 hover:scale-[1.03]"
        quality={85}
      />
    </div>
  )
}
```

### 4.3 — Performance Budget

| Category | Target WebP size | Budget rationale |
|---|---|---|
| Product card (400×400) | < 30KB | Grid has 12+ cards — total < 360KB |
| Product detail (640×640) | < 65KB | Primary viewport image |
| Lifestyle (640×480) | < 55KB | Below fold, lazy loaded |
| Origin (800×500) | < 75KB | Below fold |
| Hero landing (full vp) | < 160KB | Above fold — optimize aggressively |
| Total per product page | < 400KB | Including hero + 3 gallery + origin |

### 4.4 — Alt Text Standards (E-Commerce)

Alt text for e-commerce must be both descriptive and SEO-relevant:

```tsx
// Pattern for product images
alt={`${product.name} — café de especialidad de ${product.origin}, ${product.weight}g`}
// Example: "Etiopía Yirgacheffe — café de especialidad de Etiopía, 250g"

// Pattern for lifestyle
alt="Preparación de café con método pour over en cerámica artesanal"

// Pattern for origin
alt="Plantación de café en las laderas del Valle del Rift, Etiopía — origen del Café Vermut Yirgacheffe"
```

---

## Step 5 — Favicon and Social Asset System

### Complete Favicon Set

| File | Size | Format | Placement |
|---|---|---|---|
| `favicon.ico` | 16×16 + 32×32 | ICO (multi-size) | `src/app/favicon.ico` |
| `icon.svg` | Scalable | SVG | `src/app/icon.svg` |
| `apple-icon.png` | 180×180 | PNG (no alpha) | `src/app/apple-icon.png` |
| `icon-192.png` | 192×192 | PNG | `public/icon-192.png` (PWA) |
| `icon-512.png` | 512×512 | PNG | `public/icon-512.png` (PWA) |

**Design spec:** Café Vermut wordmark or "V" lettermark on `#1A1209` background with `#C9A84C` gold accent. Must read at 16px — simplify to letterform at small sizes.

### Open Graph System

**Static OG (default):**
```tsx
// src/app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const size = { width: 1200, height: 630 }
export const contentType = 'image/jpeg'

export default function OGImage() {
  return new ImageResponse(
    <div style={{ background: '#F7F5F0', width: '100%', height: '100%', display: 'flex', flexDirection: 'column', justifyContent: 'center', padding: '80px' }}>
      <span style={{ fontSize: 28, color: '#6A5748', letterSpacing: '0.15em', textTransform: 'uppercase', marginBottom: 20 }}>Café Vermut</span>
      <span style={{ fontSize: 56, color: '#0D0B08', fontWeight: 300, lineHeight: 1.1 }}>Café de especialidad.</span>
      <span style={{ fontSize: 56, color: '#0D0B08', fontWeight: 300, lineHeight: 1.1 }}>Origen único.</span>
    </div>,
    size
  )
}
```

**Dynamic OG (per product):**
```tsx
// src/app/tienda/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'
import { getProduct } from '@/lib/supabase/products'

export async function generateImageMetadata({ params }: { params: { slug: string } }) {
  const product = await getProduct(params.slug)
  return [{ id: params.slug, alt: product.name }]
}

export default async function OGImage({ params }: { params: { slug: string } }) {
  const product = await getProduct(params.slug)
  return new ImageResponse(
    <div style={{ background: '#F7F5F0', width: '100%', height: '100%', display: 'flex', alignItems: 'center', padding: '80px', gap: '80px' }}>
      {/* eslint-disable-next-line @next/next/no-img-element */}
      <img src={product.imageUrl} style={{ width: 460, height: 460, objectFit: 'contain' }} alt="" />
      <div style={{ display: 'flex', flexDirection: 'column', gap: 16 }}>
        <span style={{ fontSize: 20, color: '#6A5748', letterSpacing: '0.12em', textTransform: 'uppercase' }}>{product.origin}</span>
        <span style={{ fontSize: 48, color: '#0D0B08', fontWeight: 300, lineHeight: 1.1 }}>{product.name}</span>
        <span style={{ fontSize: 24, color: '#C4751A', marginTop: 8 }}>{(product.price / 100).toFixed(2)} €</span>
      </div>
    </div>,
    { width: 1200, height: 630 }
  )
}
```

---

## Step 6 — Pre-Launch Checklist

### Product Images
- [ ] All 20 products have `main` image (400×400 WebP < 30KB)
- [ ] All 20 products have `hero` image (640×640 WebP < 65KB)
- [ ] All alt texts in Spanish with origin and weight
- [ ] No images served from external domains (all uploaded to Supabase Storage)
- [ ] Supabase Storage bucket is public with correct CORS policy

### Global Assets
- [ ] Hero landing image < 160KB WebP — confirmed in DevTools
- [ ] `priority` prop on hero and first product row
- [ ] `loading="lazy"` on all below-fold product cards
- [ ] favicon.ico renders correctly in all browsers
- [ ] apple-icon.png shows on iOS home screen test
- [ ] Default OG image renders in opengraph.xyz preview
- [ ] Per-product OG images render (test 3 products)
- [ ] `formats: ['image/avif', 'image/webp']` confirmed in next.config.ts
