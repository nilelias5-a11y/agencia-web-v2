# Image Curation Plan — La Nonna Restaurant

**Project:** La Nonna — Italian restaurant, Gràcia, Barcelona  
**Date:** 2026-05-13

---

## Image Needs

For a restaurant website, the critical images are:

1. **Hero image** — wide, appetizing, sets the mood
2. **Food/dish photos** — menu items, best sellers
3. **Interior/atmosphere** — the dining space
4. **Team/chef** — human connection, trust signal
5. **Open Graph image** — social share preview

---

## Sourcing Recommendations

### Option A — Client photographs (preferred)
Ask the client for photos taken by a food photographer. La Nonna has been operating since 2003 and likely has professional photos. Quality check:
- Minimum 2000px wide
- Warm lighting (avoid cold/blue tones, inconsistent with terracotta palette)
- Natural light or warm artificial light preferred

### Option B — Stock photography (fallback)
If client has no professional photos, recommend from:
- **Unsplash** (free): search "italian restaurant", "pasta closeup", "mediterranean food"
- **Pexels** (free): similar search terms
- Avoid: top-down flat-lay with white backgrounds — too editorial, not "la nonna" enough
- Prefer: natural light, rustic textures, steam, hands, communal settings

---

## Recommended Unsplash queries for La Nonna

| Use | Query | Notes |
|-----|-------|-------|
| Hero | "italian restaurant interior warm light" | Choose 2:1 ratio, warm tones |
| Hero alt | "barcelona restaurant evening" | Local/Mediterranean feel |
| Pasta hero | "fresh pasta making hands" | Human element |
| Menu items | "italian food rustic table" | Multiple dishes |
| Interior | "cozy restaurant wooden tables" | Matches Gràcia aesthetic |

---

## Technical Specs

**Hero image:**
- Source: 2400×1600px minimum
- Display: full-width, covers viewport
- Use `next/image` with `fill` and `priority` prop
- Add `sizes="100vw"` 

**Food gallery images:**
- Source: 1200×900px minimum  
- Use `next/image` with explicit `width` and `height`
- Format: jpg (photos), lazy load with `loading="lazy"`

**Open Graph:**
- 1200×630px
- Can be a cropped version of the hero or a composed title card

---

## Optimization

- Convert to WebP using sharp (already in Next.js pipeline)
- Compress to target: hero < 200KB WebP, gallery items < 80KB WebP
- Add `alt` text describing the food/scene (SEO + accessibility)

---

## Favicon

- Provide: 32×32 favicon.ico, 180×180 apple-touch-icon.png
- Source: La Nonna logo on terracotta background (`#C4603A`)
- If no vector logo available, use a stylized "N" or pasta/fork icon
