# Ana López Portfolio — UI Specs (Eval 2)
**Brand palette:** Off-white `#F8F6F3` · Charcoal `#2C2C2C` · Gold accent `#C9A96E`
**Typography:** Cormorant Garamond (headings) · Inter (body/UI)
**Date:** 2026-05-13

---

## 1. NAVBAR — Desktop & Mobile (All Element States)

### Token Foundation

| Token | Value |
|---|---|
| `nav-height-desktop` | 80px |
| `nav-height-mobile` | 64px |
| `nav-bg` | `#F8F6F3` (opaque on scroll: `rgba(248,246,243,0.96)` + `backdrop-filter: blur(8px)`) |
| `nav-border-bottom` | `1px solid rgba(44,44,44,0.08)` (appears on scroll / sticky) |
| `nav-z-index` | 100 |
| `nav-px-desktop` | 64px |
| `nav-px-tablet` | 32px |
| `nav-px-mobile` | 20px |

---

### 1.1 Desktop Navbar (≥ 1024px)

**Layout:** Single horizontal row, `display: flex; align-items: center; justify-content: space-between;`

```
[ LOGO ]          [ Trabajo ]  [ Sobre mí ]  [ Contacto ]          [ Hablemos → ]
```

**Logo**
- Font: Cormorant Garamond, 500 weight, 22px, letter-spacing: 0.06em
- Color: `#2C2C2C`
- Text: "Ana López" (or monogram mark if SVG logo)
- Cursor: pointer
- Transition: `opacity 200ms ease`
- Hover: `opacity: 0.65`
- Active/current page: `opacity: 1` (no change — logo always full)

**Nav Links (×3)**
- Font: Inter, 400 weight, 13px, letter-spacing: 0.08em, text-transform: uppercase
- Color default: `#2C2C2C` at `opacity: 0.55`
- Gap between links: 40px
- Padding: `8px 0` (vertical hit area, no horizontal padding — underline below text)
- Underline: `text-decoration: none`; custom underline via `::after` pseudo-element:
  - `content: ''`, `display: block`, `height: 1px`, `background: #C9A96E`
  - `width: 0`, `transition: width 280ms cubic-bezier(0.25, 0.1, 0.25, 1)`
  - `transform-origin: left center`

| State | Color | Underline width | Opacity |
|---|---|---|---|
| Default | `#2C2C2C` | 0% | 0.55 |
| Hover | `#2C2C2C` | 100% | 1.0 |
| Active (current page) | `#2C2C2C` | 100% | 1.0 |
| Focus-visible | `#2C2C2C` + `outline: 2px solid #C9A96E; outline-offset: 4px` | 100% | 1.0 |
| Pressed (:active) | `#2C2C2C` | 100% | 0.75 |

**CTA Button — "Hablemos"**
- Font: Inter, 500 weight, 12px, letter-spacing: 0.12em, text-transform: uppercase
- Padding: `12px 28px`
- Border-radius: `0` (sharp corners — editorial feel)
- Transition: `all 250ms cubic-bezier(0.25, 0.1, 0.25, 1)`
- Arrow icon: `→` inline, margin-left: 8px, `display: inline-block`, transition: `transform 250ms ease`

| State | Background | Border | Text color | Arrow offset | Shadow |
|---|---|---|---|---|---|
| Default | transparent | `1.5px solid #2C2C2C` | `#2C2C2C` | translateX(0) | none |
| Hover | `#2C2C2C` | `1.5px solid #2C2C2C` | `#F8F6F3` | translateX(4px) | none |
| Focus-visible | transparent | `1.5px solid #C9A96E` | `#2C2C2C` | translateX(0) | `0 0 0 3px rgba(201,169,110,0.25)` |
| Active (:active) | `#1a1a1a` | `1.5px solid #1a1a1a` | `#F8F6F3` | translateX(2px) | none |
| Disabled | transparent | `1.5px solid rgba(44,44,44,0.25)` | `rgba(44,44,44,0.35)` | translateX(0) | none |

---

### 1.2 Mobile Navbar (< 768px)

**Layout:** `display: flex; align-items: center; justify-content: space-between;`
- Height: 64px
- Left: Logo (same as desktop, 20px size)
- Right: Hamburger icon button

**Hamburger Icon Button**
- Hit area: 44px × 44px (WCAG minimum)
- Visible icon area: 24px × 18px (3 bars)
- Padding: `10px` (centers 24px icon within 44px)
- Background: transparent
- Border: none
- Color: `#2C2C2C`
- `aria-label`: "Abrir menú" / "Cerrar menú" (toggled)
- `aria-expanded`: false / true (toggled)

**Bar specs (3 lines):**
- Bar: `width: 24px; height: 1.5px; background: #2C2C2C; border-radius: 0`
- Gap between bars: 6px
- Animation to X: bars 1 & 3 rotate ±45deg, bar 2 fades out — `transition: 300ms cubic-bezier(0.23, 1, 0.32, 1)`

| State | Top bar | Mid bar | Bottom bar | Container opacity |
|---|---|---|---|---|
| Closed (default) | `translateY(0) rotate(0)` | `opacity: 1` | `translateY(0) rotate(0)` | 1.0 |
| Hover (closed) | — | — | — | 0.65 |
| Open (X) | `translateY(7.5px) rotate(45deg)` | `opacity: 0, scaleX(0)` | `translateY(-7.5px) rotate(-45deg)` | 1.0 |
| Hover (open) | — | — | — | 0.65 |
| Focus-visible | `outline: 2px solid #C9A96E; outline-offset: 4px; border-radius: 2px` | — | — | 1.0 |

**Mobile Menu Overlay (full-screen drawer)**
- Trigger: hamburger click
- Type: full-viewport overlay (not sidebar) — covers everything below nav
- Background: `#F8F6F3`
- Entry animation: `opacity: 0 → 1` + `translateY(-12px) → translateY(0)`, `duration: 350ms, cubic-bezier(0.23, 1, 0.32, 1)`
- Exit animation: reverse, `duration: 250ms`
- `z-index`: 99 (below nav bar at 100)
- `padding-top`: 64px (nav height) + 48px = 112px total from top

**Mobile Menu Links**
- Font: Cormorant Garamond, 400 weight, 42px, letter-spacing: 0.02em
- Color: `#2C2C2C`
- Display: block, stacked vertically
- Gap: 8px between items
- Padding per item: `16px 20px`
- Staggered entry: each link delays by 60ms × index (`0ms, 60ms, 120ms`)
- Entry per link: `opacity: 0 → 1` + `translateX(-16px) → translateX(0)`, `duration: 300ms ease-out`

| State | Color | Transform | Opacity |
|---|---|---|---|
| Default | `#2C2C2C` | none | 0.7 |
| Hover | `#2C2C2C` | `translateX(8px)` | 1.0 |
| Active (current page) | `#C9A96E` | none | 1.0 |
| Focus-visible | `#2C2C2C` + gold outline | none | 1.0 |
| Pressed | `#C9A96E` | none | 0.8 |

**Mobile CTA — "Hablemos"**
- Position: bottom of overlay, above safe area
- `margin-top: auto` (pushes to bottom via flex column layout)
- `margin-bottom: 40px + env(safe-area-inset-bottom)`
- `margin-left: 20px; margin-right: 20px`
- Full width: `width: 100%`
- Height: 52px
- Same states as desktop CTA (adapted full-width)
- Font: Inter, 500 weight, 13px, letter-spacing: 0.1em

---

## 2. PORTFOLIO GRID CARD

### 2.1 Grid Container — Desktop (≥ 1024px)

- Columns: 3 equal columns
- `display: grid; grid-template-columns: repeat(3, 1fr)`
- Column gap: 24px
- Row gap: 32px
- Container max-width: 1280px
- Container padding: `0 64px`

### 2.2 Card Anatomy

**Card shell**
- `position: relative`
- `overflow: hidden`
- `border-radius: 0` (editorial — no rounding)
- `aspect-ratio: 3/4` (portrait, for consistency across variable photo sizes)
- `background: #2C2C2C` (visible during image load)
- `cursor: pointer`

**Image**
- `object-fit: cover; width: 100%; height: 100%`
- `transition: transform 600ms cubic-bezier(0.25, 0.46, 0.45, 0.94)`
- Default: `transform: scale(1.0)`
- Hover: `transform: scale(1.06)` — slow Ken Burns pull-back

**Overlay (hover reveal)**
- `position: absolute; inset: 0`
- `background: linear-gradient(to top, rgba(44,44,44,0.85) 0%, rgba(44,44,44,0.4) 50%, rgba(44,44,44,0.0) 100%)`
- `opacity: 0`
- `transition: opacity 400ms cubic-bezier(0.25, 0.1, 0.25, 1)`
- Hover: `opacity: 1`

**Reveal Effect — "venetian blind" wipe**
- Overlay uses a `clip-path` wipe instead of plain opacity for the "revelado" effect:
  - Default: `clip-path: inset(100% 0 0 0)` (hidden — clipped from bottom)
  - Hover: `clip-path: inset(0% 0 0 0)` (fully revealed)
  - `transition: clip-path 500ms cubic-bezier(0.77, 0, 0.175, 1), opacity 200ms ease`
  - Opacity also goes `0 → 1` simultaneously for softer edge

**Card text content (inside overlay)**
- `position: absolute; bottom: 0; left: 0; right: 0; padding: 28px 24px`
- `transform: translateY(12px)` default → `translateY(0)` on hover
- `transition: transform 400ms cubic-bezier(0.25, 0.46, 0.45, 0.94) 80ms` (80ms delay — after overlay starts)
- `opacity: 0` default → `opacity: 1` on hover (same timing)

  **Series/category tag**
  - Font: Inter, 400 weight, 10px, letter-spacing: 0.14em, text-transform: uppercase
  - Color: `#C9A96E`
  - Margin-bottom: 6px

  **Title**
  - Font: Cormorant Garamond, 400 weight, 22px, letter-spacing: 0.01em
  - Color: `#F8F6F3`
  - Line-height: 1.2

  **View link**
  - Font: Inter, 400 weight, 11px, letter-spacing: 0.1em, text-transform: uppercase
  - Color: `rgba(248,246,243,0.65)`
  - Margin-top: 10px
  - Arrow: ` →` inline

**Card border accent (optional gold line)**
- `::after` pseudo-element
- `position: absolute; bottom: 0; left: 0`
- `width: 0; height: 2px; background: #C9A96E`
- `transition: width 500ms cubic-bezier(0.77, 0, 0.175, 1)`
- Hover: `width: 100%` — sweeps left to right synchronized with overlay wipe

### 2.3 Card States Summary

| State | Image scale | Overlay clip | Overlay opacity | Text transform | Border accent |
|---|---|---|---|---|---|
| Default | scale(1.0) | inset(100% 0 0 0) | 0 | translateY(12px) / opacity:0 | width: 0 |
| Hover | scale(1.06) | inset(0% 0 0 0) | 1 | translateY(0) / opacity:1 | width: 100% |
| Focus-visible | scale(1.0) | inset(0% 0 0 0) | 1 | translateY(0) / opacity:1 | width: 100% |
| Active (:active) | scale(1.03) | inset(0% 0 0 0) | 0.9 | translateY(0) / opacity:1 | width: 100% |

### 2.4 Mobile Cards (< 768px)

- Grid: `grid-template-columns: 1fr` (single column stack)
- Column gap: n/a
- Row gap: 16px
- Card aspect ratio: `4/3` (landscape — wider, shorter, better on narrow screens)
- Container padding: `0 20px`
- Hover states: **disabled on touch** (`@media (hover: none)` — overlay removed)
- Touch: overlay always partially visible:
  - `clip-path: inset(0)`, `opacity: 0.7` — always-on gradient overlay
  - Text always visible (no reveal animation)
  - `transform: none` always

### 2.5 Tablet Cards (768px–1023px)

- Grid: `grid-template-columns: repeat(2, 1fr)`
- Column gap: 20px
- Row gap: 24px
- Container padding: `0 32px`
- Aspect ratio: `3/4` (same as desktop)
- Hover states: active (tablet with mouse/trackpad)

---

## 3. SPACING SYSTEM & GRID SPEC

### 3.1 Base Spacing Scale (8px base unit)

| Token | Value | Usage |
|---|---|---|
| `space-1` | 4px | Micro gaps (icon-to-label, tag margins) |
| `space-2` | 8px | Tight internal padding |
| `space-3` | 12px | Input padding-y, small component padding |
| `space-4` | 16px | Default element spacing, mobile card gap |
| `space-5` | 20px | Mobile container padding-x |
| `space-6` | 24px | Desktop card column-gap, section-internal |
| `space-8` | 32px | Tablet container padding-x, card row gap |
| `space-10` | 40px | Section padding-y (mobile), nav element groups |
| `space-12` | 48px | Section padding-y (tablet) |
| `space-16` | 64px | Desktop container padding-x, section padding-y (desktop) |
| `space-20` | 80px | Large section separators (desktop) |
| `space-24` | 96px | Hero top padding |
| `space-32` | 128px | XL section gaps, above-fold hero height contribution |

**Non-standard values (brand-specific):**
- `28px` — Card overlay inner padding-y (between `space-6` and `space-8`)
- `40px` — Mobile CTA bottom margin

### 3.2 Grid System — Desktop (≥ 1280px, large)

| Property | Value |
|---|---|
| Max content width | 1280px |
| Container padding-x | 64px each side |
| Usable width | 1152px |
| Columns | 12 |
| Column width | (1152 − 11 × 24) / 12 = 73.5px |
| Gutter (column gap) | 24px |
| Row gutter | 32px |

**Portfolio section within 12-col grid:**
- Photo grid: spans cols 1–12 (full bleed within container)
- Each photo card: 4 cols wide = `(4 × 73.5) + (3 × 24)` = 366px per card

### 3.3 Grid System — Desktop medium (1024px–1279px)

| Property | Value |
|---|---|
| Container padding-x | 48px each side |
| Usable width | 928px–1183px |
| Columns | 12 |
| Gutter | 20px |
| Row gutter | 28px |

### 3.4 Grid System — Tablet (768px–1023px)

| Property | Value |
|---|---|
| Container padding-x | 32px each side |
| Usable width | 704px–959px |
| Columns | 8 |
| Gutter | 20px |
| Row gutter | 24px |

**Portfolio section within 8-col grid:**
- Each photo card: 4 cols wide (2-column layout = 2 × 4cols)

### 3.5 Grid System — Mobile (< 768px)

| Property | Value |
|---|---|
| Container padding-x | 20px each side |
| Usable width | calc(100vw − 40px) |
| Columns | 4 |
| Gutter | 16px |
| Row gutter | 16px |

**Portfolio section within 4-col grid:**
- Each photo card: 4 cols wide (full width, single-column stack)

### 3.6 Breakpoints

| Name | Min width | Target devices |
|---|---|---|
| `xs` | 0px | Small phones |
| `sm` | 480px | Large phones |
| `md` | 768px | Tablets portrait |
| `lg` | 1024px | Tablets landscape, small laptops |
| `xl` | 1280px | Desktop |
| `2xl` | 1536px | Large desktop / wide monitors |

### 3.7 Vertical Rhythm — Section Spacing

| Section boundary | Desktop | Tablet | Mobile |
|---|---|---|---|
| Navbar height | 80px | 80px | 64px |
| Hero top padding-top | 96px | 72px | 56px |
| Section padding-y (default) | 96px | 64px | 48px |
| Section padding-y (large) | 128px | 80px | 64px |
| Inter-section divider (thin) | 1px solid `rgba(44,44,44,0.08)` | — | — |

### 3.8 Typography Scale

| Role | Font | Weight | Size desktop | Size mobile | Line-height | Letter-spacing |
|---|---|---|---|---|---|---|
| H1 — Hero | Cormorant Garamond | 300 | 72px | 42px | 1.0 | -0.01em |
| H2 — Section title | Cormorant Garamond | 400 | 48px | 34px | 1.1 | 0 |
| H3 — Card title | Cormorant Garamond | 400 | 22px | 20px | 1.2 | 0.01em |
| Label / Eyebrow | Inter | 400 | 10px | 10px | 1.4 | 0.14em (uppercase) |
| Body — default | Inter | 400 | 15px | 15px | 1.65 | 0 |
| Body — small | Inter | 400 | 13px | 13px | 1.6 | 0 |
| Nav links | Inter | 400 | 13px | 42px (mobile menu) | 1.0 | 0.08em (uppercase) |
| CTA button | Inter | 500 | 12px | 13px | 1.0 | 0.12em (uppercase) |

### 3.9 Color Usage Rules

| Token | Hex | Usage |
|---|---|---|
| `color-bg` | `#F8F6F3` | Page background, nav background, mobile menu |
| `color-fg` | `#2C2C2C` | All body text, headings, borders, icons |
| `color-accent` | `#C9A96E` | Underline hover, active nav link, card gold bar, category tags, CTA focus ring |
| `color-fg-dim` | `rgba(44,44,44,0.55)` | Default nav link state |
| `color-fg-subtle` | `rgba(44,44,44,0.08)` | Dividers, nav border on scroll |
| `color-overlay` | `rgba(44,44,44,0.85)` | Card overlay bottom (gradient start) |
| `color-on-dark` | `#F8F6F3` | Text on dark overlays (card titles, CTA hover) |

---

*End of UI Specs — Ana López Portfolio (Eval 2 · Iteration 1 baseline)*
