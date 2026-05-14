# Image Plan — Café Vermut E-Commerce

**Project:** Café Vermut — Specialty Coffee E-Commerce, Barcelona  
**Date:** 2026-05-13

---

## Image Needs for Coffee E-Commerce

The main image categories for an online coffee store:

1. **Hero / Landing** — brand mood image
2. **Product photos** — individual bags/products (main, multiple angles)
3. **Origin stories** — coffee farm or origin country images
4. **Lifestyle / brewing** — at-home brewing ritual shots
5. **OG / social** — share previews

---

## Product Photography Requirements

For Café Vermut's "Japan meets Barcelona" direction, product photos should be:
- **Clean white or near-white background** (matches `#F8F7F4` base)
- **Single product, centered** — let the coffee speak
- **Minimal props** — perhaps a small ceramic cup, a spoon
- If lifestyle context: natural light, ceramic or linen surfaces

**What to avoid:** Busy compositions, multiple products, colored backgrounds that clash with the palette, stock-generic "coffee beans on burlap" shots.

---

## Stock Source Recommendations

| Need | Source | Search terms |
|------|--------|--------------|
| Product mockup bags | Smartmockups / Placeit | "coffee bag mockup kraft" |
| Origin landscapes | Unsplash | "ethiopia coffee farm", "colombia coffee mountain" |
| Brewing ritual | Unsplash | "pour over coffee morning", "specialty coffee barista" |
| Minimal cafe | Unsplash | "minimal coffee white", "japanese coffee aesthetic" |

---

## Technical Specs

**Product main image:**
- Display: 800×800px (square, standard for e-commerce)
- Source: 1600×1600px minimum
- Background: White `#FFFFFF`

**Product thumbnails:**
- Display: 200×200px
- Same source, Next.js auto-resizes

**Hero:**
- Display: full-width, 2:1 ratio
- Source: 2400×1200px minimum

---

## Optimization

- Use Next.js `next/image` for automatic WebP conversion
- Product images: target < 50KB WebP
- Hero: target < 150KB WebP
- All images need descriptive alt text in Spanish
- Add `loading="lazy"` on product grid images, `priority` on hero

---

## Favicon

- Logo on `#1A1209` (dark espresso) background
- Required: favicon.ico (32×32), apple-touch-icon.png (180×180)
- OG image: 1200×630px, product on brand background
