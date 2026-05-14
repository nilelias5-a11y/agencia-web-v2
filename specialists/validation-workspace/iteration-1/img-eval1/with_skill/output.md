# Image Curation Report — La Nonna Restaurant

**Specialist:** image-curator  
**Client:** La Nonna — Italian restaurant, Gràcia, Barcelona  
**Stack:** Next.js 15, next/image, Unsplash integration  
**Skill version:** v1.0  
**Date:** 2026-05-13

---

## Step 1 — Brand Visual Audit

Before sourcing any image, map visual requirements against the brand brief:

| Visual axis | Target | Avoid |
|---|---|---|
| Lighting | Warm, golden, candlelit interior | Cold flash photography, blue tones |
| Texture | Rustic wood, ceramic, linen, aged tile | Plastic, chrome, clinical surfaces |
| People | Hands, gestures, family moments (not posed smiles) | Corporate food photography poses |
| Food | Steaming, imperfect, abundant | Minimalist plate art (wrong brand register) |
| Setting | Gràcia neighborhood, terrace, intimate dining | Generic "Italian flag" clichés |

**Color harmony check:** All sourced images must work with La Nonna's palette (terracotta `#C4603A`, stone `#FAF6F0`, olive `#3D5A3E`). Images with dominant blue, purple, or cold tones will create chromatic dissonance.

---

## Step 2 — Image Inventory and Sourcing Plan

### 2.1 — Tier Assessment

**Tier 1 (Client-provided — always preferred):**
La Nonna has operated since 2003. Request from director:
- Any professional food photographer sessions (ask for RAW or high-res JPG)
- Smartphone photos of the space (assess quality — modern iPhone/Android = often 12MP+, viable)
- Instagram archive (check @lanonnagrc or similar — can be downloaded at original quality)

**Tier 2 (Unsplash — free, commercial license):**
If client photos don't cover all needs or are below quality threshold.

**Tier 3 (Paid stock — last resort):**
Getty, Stocksy if very specific shots needed (e.g., specific pasta being made by hands).

### 2.2 — Required Images

| ID | Location | Purpose | Dimensions (display) | Source tier |
|---|---|---|---|---|
| IMG-01 | Hero section | Primary mood-setter | Full viewport (2:1 min) | T1 preferred, T2 fallback |
| IMG-02 | About / Our Story | Chef or family moment | 800×600 | T1 preferred |
| IMG-03–06 | Menu section | 4 signature dishes | 600×450 each | T1 preferred |
| IMG-07 | Events section | Table setup / dining room | 1200×800 | T1 preferred |
| IMG-08 | OG Image | Social share preview | 1200×630 | Generated from IMG-01 |
| IMG-09 | favicon.ico | Browser tab | 32×32 | Logo-derived |
| IMG-10 | apple-touch-icon | iOS home screen | 180×180 | Logo-derived |

---

## Step 3 — Unsplash Curation (Tier 2 Fallbacks)

Curated search queries and selection criteria for each need:

### IMG-01 — Hero Fallback
**Query:** `italian restaurant warm evening light`  
**Selection criteria:** Must have warm (golden/amber) tones, identifiable Mediterranean or European space, human presence optional but preferred  
**Reject if:** Flash photography, pure white walls, generic stock feel  
**Next.js implementation:**
```tsx
// src/components/sections/Hero.tsx
import Image from 'next/image'

<Image
  src="/images/hero.jpg"
  alt="Interior del restaurante La Nonna en Gràcia, Barcelona — ambiente cálido y acogedor"
  fill
  priority
  sizes="100vw"
  className="object-cover"
  quality={85}
/>
```

### IMG-03–06 — Menu Dish Fallbacks
**Queries (one per dish):**  
- Pasta: `fresh tagliatelle pasta rustic wooden table`
- Risotto: `risotto parmesan cream close up`
- Pizza: `neapolitan pizza wood fired rustic`
- Antipasti: `italian antipasti spread charcuterie`

**Selection criteria:** Warm lighting, natural textures, food should look abundant and authentic, NOT flat-lay / overhead food editorial

---

## Step 4 — Technical Optimization Pipeline

### 4.1 — Source → Web Pipeline

```
Client provides: JPG/HEIC/PNG (raw, full resolution)
  ↓
1. Resize: max 2400px on longest side (imagemagick or sharp CLI)
2. Convert: → WebP (quality 85) + JPEG fallback (quality 88)
3. Next.js serves: automatically via next/image built-in optimization
4. CDN cache: Vercel CDN handles caching headers

Target file sizes:
  Hero (full viewport): < 180KB WebP
  Dish photos (600×450): < 60KB WebP  
  About photo (800×600): < 90KB WebP
  Events photo (1200×800): < 130KB WebP
```

### 4.2 — Responsive `sizes` Props

```tsx
// Hero — full viewport always
sizes="100vw"

// Full-width section image
sizes="(max-width: 768px) 100vw, 1200px"

// Menu dish grid (3-col desktop, 2-col tablet, 1-col mobile)
sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"

// About section (half-width desktop, full mobile)
sizes="(max-width: 1024px) 100vw, 50vw"
```

### 4.3 — Alt Text Standards

All alt text must be:
- Descriptive (describes the actual content, not "image of")
- In Spanish (site language)
- Max 125 characters
- Include context (dish name, setting)

```tsx
// Correct
alt="Tagliatelle al ragù de la nonna servidos en plato de cerámica rústica"
alt="Comedor principal de La Nonna con mesas de madera y luz cálida de velas"

// Incorrect
alt="image1"
alt="food photo"
alt="La Nonna restaurant image"
```

---

## Step 5 — Favicon and Social Asset Production

### Favicon Set

| File | Size | Format | Notes |
|---|---|---|---|
| `favicon.ico` | 32×32 + 16×16 multi-size | ICO | Multi-size preferred, legacy support |
| `icon.svg` | Scalable | SVG | Modern browsers — best quality |
| `apple-touch-icon.png` | 180×180 | PNG | iOS home screen, no transparency |
| `favicon-192.png` | 192×192 | PNG | Android PWA |
| `favicon-512.png` | 512×512 | PNG | Splash screen |

**Source:** La Nonna logo on `#C4603A` (terracotta) background. If logo is letterform ("N" or "LN"), render at 512×512 first, downscale with anti-aliasing.

**Next.js placement:** `src/app/favicon.ico`, `src/app/icon.svg`, `src/app/apple-icon.png`

### Open Graph Image

**Spec:** 1200×630px, JPEG, < 200KB  
**Composition:** Centered hero food photo + "La Nonna" wordmark in Playfair Display + tagline in Lato  
**Background fallback:** Solid `#C4603A` with white text if no photo available  

```tsx
// src/app/opengraph-image.tsx (Next.js OG generation)
import { ImageResponse } from 'next/og'

export default function OGImage() {
  return new ImageResponse(
    <div style={{ background: '#C4603A', width: '100%', height: '100%', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
      <span style={{ color: 'white', fontSize: 72, fontFamily: 'serif' }}>La Nonna</span>
    </div>,
    { width: 1200, height: 630 }
  )
}
```

---

## Step 6 — Pre-Launch Image Checklist

- [ ] All images converted to WebP with JPEG fallback
- [ ] Hero < 180KB WebP confirmed in browser DevTools
- [ ] All dish photos < 60KB WebP
- [ ] All `alt` attributes present and in Spanish
- [ ] favicon.ico renders correctly in Chrome, Firefox, Safari
- [ ] apple-touch-icon.png shows correctly when added to iOS home screen
- [ ] OG image renders correctly in https://opengraph.xyz preview
- [ ] No images served from external domains except Vercel CDN
- [ ] `priority` prop on hero image (above-fold critical)
- [ ] `loading="lazy"` on all below-fold images
