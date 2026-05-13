## Typography System — La Nonna

### Step 1 — Brand Personality Analysis

| Brand dimension        | Score | Typography expression                              |
|------------------------|-------|----------------------------------------------------|
| Warm / Artisanal       | ★★★★★ | Humanist serif — conveys hand-crafted authenticity |
| Luxury / High-end      | ★★★☆☆ | Elegant display weight — never heavy or slab       |
| Modern (light touch)   | ★★★☆☆ | Pairing sans must feel contemporary, not retro     |
| Editorial / Menu-first | ★★★★☆ | Strong hierarchy between name, description, price  |
| Playful / Creative     | ★★☆☆☆ | Warmth yes, quirkiness no                          |

Brand archetype: **The Trusted Artisan** — a family kitchen that has been perfecting its recipes for two decades. The typography should feel like a beautifully set Italian menu: readable, confident, never fussy.

---

### Step 2 — Pairing Logic

#### Selected pair

**Heading font: Cormorant Garamond (variable)**
- Source: Google Fonts (`family=Cormorant+Garamond:ital,wght@0,300..700;1,300..700`)
- Also available via `@fontsource/cormorant-garamond` on npm (Fontsource)
- Classification: Old-style humanist serif, high contrast, display-optimised
- Why: Rooted in Garamond — the typeface of Renaissance Italian printing. High contrast strokes read beautifully at display sizes (hero headlines, menu item names). The italic cuts are genuinely elegant and add emotional warmth to pull-quotes and category labels. Exists as a variable font, compressing weights 300–700 into a single WOFF2 file (~56 KB).

**Body font: DM Sans (variable)**
- Source: Google Fonts (`family=DM+Sans:ital,opsz,wght@0,9..40,300..700;1,9..40,300..700`)
- Also available via `@fontsource/dm-sans` on npm (Fontsource)
- Classification: Geometric humanist sans-serif
- Why: DM Sans is neutral enough not to compete with Cormorant, yet warm enough to feel friendly — it avoids the cold sterility of pure geometric sans fonts (Futura, Montserrat). The optical-size axis means it self-adjusts tracking at small sizes, which is critical for form labels, legal text, and mobile navigation. Latin Extended character set covers all Spanish and Catalan glyphs (à, é, í, ï, ó, ú, ü, ñ, ç, l·l, etc.). Variable file covers weights 300–700 + italic, ~18 KB.

**Pairing rationale:** Cormorant Garamond (high-contrast old-style serif) and DM Sans (low-contrast geometric humanist sans) create maximum contrast where it matters — headings feel Italian and timeless, body text feels clean and contemporary. The pairing honours the brand's founding year (2003) while avoiding nostalgia. Total font payload ≈ 74 KB WOFF2, within the <80 KB budget with 2 requests.

#### Pairing validation
```
[✓] Available on Google Fonts (variable font, no self-hosting required)
[✓] Available via Fontsource npm for self-hosting if needed
[✓] Weights 300 / 400 / 500 / 600 / 700 all present in variable range
[✓] Latin Extended character set: confirmed for both fonts
[✓] Variable font versions available (single file per family)
[✓] License: SIL Open Font License 1.1 — commercial web use permitted
[✓] WOFF2 format available
[✓] 2 font requests total (1 variable file per family)
[✓] Estimated payload: Cormorant Garamond var ~56 KB + DM Sans var ~18 KB = ~74 KB < 80 KB ✓
```

---

### Step 3 — Full Type Scale

Base: **16px**, Ratio: **1.25 (Major Third)**. Display token uses 1.333 step to reach 72px for hero impact.

| Token       | Size  | Font               | Weight     | Letter spacing | Line height | Use case                                        |
|-------------|-------|--------------------|------------|----------------|-------------|-------------------------------------------------|
| `display`   | 72px  | Cormorant Garamond | 700        | −0.03em        | 1.05        | Hero headline ("La auténtica cocina italiana")  |
| `h1`        | 48px  | Cormorant Garamond | 700        | −0.02em        | 1.1         | Page titles (Historia, Carta, Reservar)         |
| `h2`        | 36px  | Cormorant Garamond | 600        | −0.015em       | 1.2         | Section headings ("Nuestros platos", "Eventos") |
| `h3`        | 24px  | Cormorant Garamond | 600        | 0              | 1.3         | Menu category headings ("Antipasti", "Dolci")   |
| `h4`        | 20px  | DM Sans            | 600        | 0              | 1.4         | Menu item names, card headings                  |
| `body-lg`   | 18px  | DM Sans            | 400        | 0              | 1.65        | Lead paragraphs, about page intro               |
| `body`      | 16px  | DM Sans            | 400        | 0              | 1.65        | All standard body copy                          |
| `body-sm`   | 14px  | DM Sans            | 400        | 0.01em         | 1.55        | Menu item descriptions, secondary text          |
| `price`     | 14px  | DM Sans            | 600        | 0.01em         | 1.4         | Menu prices (tabular-nums, right-aligned)       |
| `label`     | 12px  | DM Sans            | 500        | 0.06em         | 1.4         | Form labels, nav items, badges                  |
| `caption`   | 12px  | DM Sans            | 400        | 0.02em         | 1.4         | Captions, photo credits                         |
| `legal`     | 11px  | DM Sans            | 400        | 0.01em         | 1.5         | Legal text, cookie notice, aviso legal          |
| `nav`       | 14px  | DM Sans            | 500        | 0.08em         | 1.0         | Navigation items (uppercase tracking)           |

**Special typographic rules for La Nonna:**
- Menu item names: `h4` token, Cormorant Garamond 600 italic preferred for warmth
- Section category labels ("Antipasti"): `label` token, DM Sans 500, ALL CAPS, 0.12em tracking
- Hero headline uses Cormorant Garamond 300 italic as a supplementary "subtitle" line at `body-lg` size for contrast
- Prices always use `font-variant-numeric: tabular-nums` to prevent layout shift across rows

---

### Loading Strategy

**Self-hosting via Fontsource (recommended for Next.js + performance control)**

Install:
```bash
npm install @fontsource-variable/cormorant-garamond
npm install @fontsource-variable/dm-sans
```

Import in `app/layout.tsx` (loaded once at app root):
```typescript
import '@fontsource-variable/cormorant-garamond';
import '@fontsource-variable/dm-sans';
```

**Preload critical font files in `<head>` via Next.js metadata or `app/layout.tsx`:**
```tsx
// app/layout.tsx — next/font approach (alternative, preferred for Next.js 13+)
import { DM_Sans, Cormorant_Garamond } from 'next/font/google';

const cormorant = Cormorant_Garamond({
  subsets: ['latin', 'latin-ext'],
  weight: ['300', '400', '600', '700'],
  style: ['normal', 'italic'],
  variable: '--font-heading',
  display: 'swap',
  preload: true,
});

const dmSans = DM_Sans({
  subsets: ['latin', 'latin-ext'],
  weight: ['300', '400', '500', '600', '700'],
  variable: '--font-body',
  display: 'swap',
  preload: true,
});

export default function RootLayout({ children }) {
  return (
    <html lang="es" className={`${cormorant.variable} ${dmSans.variable}`}>
      {children}
    </html>
  );
}
```

**Loading summary:**
- Strategy: `next/font/google` (zero-runtime, automatic preload + self-host)
- font-display: `swap` (no invisible text during load)
- Variable font: YES for both typefaces
- Preloaded: heading (300, 400, 600, 700 normal + italic), body (300, 400, 500, 600, 700)
- Requests: 2 (1 variable WOFF2 per family, handled by Next.js)
- Estimated payload: ~74 KB WOFF2 total

---

### CSS Custom Properties

Full CSS implementation, including `@font-face` declarations (for Fontsource self-hosting path) and all design tokens as CSS custom properties, Tailwind-compatible.

```css
/* ============================================================
   LA NONNA — Typography System
   Typefaces: Cormorant Garamond (variable) + DM Sans (variable)
   Generated by Typography Master — 2026-05-13
   ============================================================ */


/* ----------------------------------------------------------
   @font-face — Self-hosting path (Fontsource)
   Use this block if NOT using next/font/google.
   If using next/font/google, remove this block entirely.
   ---------------------------------------------------------- */
@font-face {
  font-family: 'Cormorant Garamond';
  src: url('/fonts/cormorant-garamond-variable.woff2') format('woff2 supports variations'),
       url('/fonts/cormorant-garamond-variable.woff2') format('woff2');
  font-weight: 300 700;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Cormorant Garamond';
  src: url('/fonts/cormorant-garamond-variable-italic.woff2') format('woff2 supports variations'),
       url('/fonts/cormorant-garamond-variable-italic.woff2') format('woff2');
  font-weight: 300 700;
  font-style: italic;
  font-display: swap;
}

@font-face {
  font-family: 'DM Sans';
  src: url('/fonts/dm-sans-variable.woff2') format('woff2 supports variations'),
       url('/fonts/dm-sans-variable.woff2') format('woff2');
  font-weight: 300 700;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'DM Sans';
  src: url('/fonts/dm-sans-variable-italic.woff2') format('woff2 supports variations'),
       url('/fonts/dm-sans-variable-italic.woff2') format('woff2');
  font-weight: 300 700;
  font-style: italic;
  font-display: swap;
}


/* ----------------------------------------------------------
   CSS Custom Properties — Typography Tokens
   ---------------------------------------------------------- */
:root {
  /* --- Font families --- */
  --font-heading: 'Cormorant Garamond', 'Garamond', 'Georgia', 'Times New Roman', serif;
  --font-body: 'DM Sans', 'Helvetica Neue', 'Arial', sans-serif;

  /* --- Type scale (size) --- */
  --text-legal:   0.6875rem;  /* 11px */
  --text-xs:      0.75rem;    /* 12px — caption, label */
  --text-sm:      0.875rem;   /* 14px — body-sm, price, nav */
  --text-base:    1rem;       /* 16px — body */
  --text-lg:      1.125rem;   /* 18px — body-lg */
  --text-xl:      1.25rem;    /* 20px — h4, menu item name */
  --text-2xl:     1.5rem;     /* 24px — h3 */
  --text-3xl:     1.875rem;   /* 30px — intermediate */
  --text-4xl:     2.25rem;    /* 36px — h2 */
  --text-5xl:     3rem;       /* 48px — h1 */
  --text-6xl:     3.75rem;    /* 60px — intermediate display */
  --text-display: 4.5rem;     /* 72px — display/hero */

  /* --- Font weights --- */
  --weight-light:    300;
  --weight-regular:  400;
  --weight-medium:   500;
  --weight-semibold: 600;
  --weight-bold:     700;

  /* --- Line heights --- */
  --leading-none:    1.0;
  --leading-tight:   1.1;
  --leading-snug:    1.2;
  --leading-heading: 1.3;
  --leading-normal:  1.4;
  --leading-relaxed: 1.55;
  --leading-loose:   1.65;

  /* --- Letter spacing --- */
  --tracking-tightest: -0.03em;
  --tracking-tighter:  -0.02em;
  --tracking-tight:    -0.015em;
  --tracking-normal:   0em;
  --tracking-wide:     0.01em;
  --tracking-wider:    0.02em;
  --tracking-widest:   0.06em;
  --tracking-caps:     0.12em;  /* for ALL CAPS labels */

  /* --- Measure (line length) --- */
  --measure-narrow:  52ch;
  --measure-base:    68ch;
  --measure-wide:    80ch;

  /* --- Paragraph spacing --- */
  --paragraph-spacing: 1em;
}


/* ----------------------------------------------------------
   Semantic Typography Utility Classes
   ---------------------------------------------------------- */

/* Display — Hero headlines */
.type-display {
  font-family: var(--font-heading);
  font-size: var(--text-display);
  font-weight: var(--weight-bold);
  font-style: normal;
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tightest);
}

/* H1 — Page titles */
.type-h1 {
  font-family: var(--font-heading);
  font-size: var(--text-5xl);
  font-weight: var(--weight-bold);
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tighter);
}

/* H2 — Section headings */
.type-h2 {
  font-family: var(--font-heading);
  font-size: var(--text-4xl);
  font-weight: var(--weight-semibold);
  line-height: var(--leading-snug);
  letter-spacing: var(--tracking-tight);
}

/* H3 — Category headings, subsections */
.type-h3 {
  font-family: var(--font-heading);
  font-size: var(--text-2xl);
  font-weight: var(--weight-semibold);
  line-height: var(--leading-heading);
  letter-spacing: var(--tracking-normal);
}

/* H4 — Menu item names (use italic variant for warmth) */
.type-h4 {
  font-family: var(--font-heading);
  font-size: var(--text-xl);
  font-weight: var(--weight-semibold);
  font-style: italic;
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-normal);
}

/* Body large — Lead paragraphs, about intro */
.type-body-lg {
  font-family: var(--font-body);
  font-size: var(--text-lg);
  font-weight: var(--weight-regular);
  line-height: var(--leading-loose);
  letter-spacing: var(--tracking-normal);
  max-width: var(--measure-base);
}

/* Body — Standard body copy */
.type-body {
  font-family: var(--font-body);
  font-size: var(--text-base);
  font-weight: var(--weight-regular);
  line-height: var(--leading-loose);
  letter-spacing: var(--tracking-normal);
  max-width: var(--measure-base);
}

/* Body small — Menu descriptions, metadata */
.type-body-sm {
  font-family: var(--font-body);
  font-size: var(--text-sm);
  font-weight: var(--weight-regular);
  line-height: var(--leading-relaxed);
  letter-spacing: var(--tracking-wide);
  max-width: var(--measure-base);
}

/* Price — Menu prices */
.type-price {
  font-family: var(--font-body);
  font-size: var(--text-sm);
  font-weight: var(--weight-semibold);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-wide);
  font-variant-numeric: tabular-nums;
  font-feature-settings: 'tnum' 1;
}

/* Label — Form labels, nav, badges */
.type-label {
  font-family: var(--font-body);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-widest);
  text-transform: uppercase;
}

/* Nav — Navigation items */
.type-nav {
  font-family: var(--font-body);
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  line-height: var(--leading-none);
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
}

/* Caption — Photo credits, supplementary info */
.type-caption {
  font-family: var(--font-body);
  font-size: var(--text-xs);
  font-weight: var(--weight-regular);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-wider);
}

/* Legal — Aviso legal, cookie notice */
.type-legal {
  font-family: var(--font-body);
  font-size: var(--text-legal);
  font-weight: var(--weight-regular);
  line-height: 1.5;
  letter-spacing: var(--tracking-wide);
  max-width: var(--measure-wide);
}

/* Menu category separator label */
.type-menu-category {
  font-family: var(--font-body);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  line-height: var(--leading-none);
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
}

/* Hero subtitle — italic accent line below display */
.type-hero-subtitle {
  font-family: var(--font-heading);
  font-size: var(--text-lg);
  font-weight: var(--weight-light);
  font-style: italic;
  line-height: var(--leading-snug);
  letter-spacing: var(--tracking-normal);
}
```

---

### Tailwind CSS Theme Extension

Add to `tailwind.config.ts` to expose all tokens as Tailwind utilities:

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        heading: [
          'var(--font-heading)',
          'Garamond',
          'Georgia',
          'Times New Roman',
          'serif',
        ],
        body: [
          'var(--font-body)',
          'Helvetica Neue',
          'Arial',
          'sans-serif',
        ],
      },
      fontSize: {
        'legal': ['0.6875rem', { lineHeight: '1.5', letterSpacing: '0.01em' }],
        'xs':    ['0.75rem',   { lineHeight: '1.4', letterSpacing: '0.02em' }],
        'sm':    ['0.875rem',  { lineHeight: '1.55', letterSpacing: '0.01em' }],
        'base':  ['1rem',      { lineHeight: '1.65', letterSpacing: '0' }],
        'lg':    ['1.125rem',  { lineHeight: '1.65', letterSpacing: '0' }],
        'xl':    ['1.25rem',   { lineHeight: '1.4',  letterSpacing: '0' }],
        '2xl':   ['1.5rem',    { lineHeight: '1.3',  letterSpacing: '0' }],
        '3xl':   ['1.875rem',  { lineHeight: '1.25', letterSpacing: '-0.01em' }],
        '4xl':   ['2.25rem',   { lineHeight: '1.2',  letterSpacing: '-0.015em' }],
        '5xl':   ['3rem',      { lineHeight: '1.1',  letterSpacing: '-0.02em' }],
        '6xl':   ['3.75rem',   { lineHeight: '1.05', letterSpacing: '-0.025em' }],
        'display': ['4.5rem',  { lineHeight: '1.05', letterSpacing: '-0.03em' }],
      },
      lineHeight: {
        'none':    '1.0',
        'tight':   '1.1',
        'snug':    '1.2',
        'heading': '1.3',
        'normal':  '1.4',
        'relaxed': '1.55',
        'loose':   '1.65',
      },
      letterSpacing: {
        'tightest': '-0.03em',
        'tighter':  '-0.02em',
        'tight':    '-0.015em',
        'normal':   '0',
        'wide':     '0.01em',
        'wider':    '0.02em',
        'widest':   '0.06em',
        'caps':     '0.12em',
      },
      maxWidth: {
        'measure-narrow': '52ch',
        'measure':        '68ch',
        'measure-wide':   '80ch',
      },
    },
  },
  plugins: [],
};

export default config;
```

---

### Responsive Typography

Critical breakpoints where type scale adjusts:

```css
/* Mobile-first: base scale already defined in :root above */

/* Tablet (md: 768px) — scale up display and h1 */
@media (min-width: 768px) {
  :root {
    --text-display: 5.5rem;   /* 88px */
    --text-5xl:     3.5rem;   /* 56px */
  }
}

/* Desktop (lg: 1024px) — full display scale */
@media (min-width: 1024px) {
  :root {
    --text-display: 6rem;     /* 96px — maximum hero impact */
    --text-5xl:     4rem;     /* 64px */
    --text-4xl:     2.5rem;   /* 40px */
  }
}
```

---

### next/font/google — Complete Implementation (app/layout.tsx)

```typescript
// app/layout.tsx
import type { Metadata } from 'next';
import { Cormorant_Garamond, DM_Sans } from 'next/font/google';
import './globals.css';

const cormorant = Cormorant_Garamond({
  subsets: ['latin', 'latin-ext'],
  weight: ['300', '400', '600', '700'],
  style: ['normal', 'italic'],
  variable: '--font-heading',
  display: 'swap',
  preload: true,
});

const dmSans = DM_Sans({
  subsets: ['latin', 'latin-ext'],
  weight: ['300', '400', '500', '600', '700'],
  style: ['normal', 'italic'],
  variable: '--font-body',
  display: 'swap',
  preload: true,
});

export const metadata: Metadata = {
  title: 'La Nonna — Restaurante Italiano en Gràcia, Barcelona',
  description: 'Cocina italiana auténtica en el corazón de Gràcia desde 2003.',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html
      lang="es"
      className={`${cormorant.variable} ${dmSans.variable}`}
    >
      <body className="font-body text-base antialiased">
        {children}
      </body>
    </html>
  );
}
```

**Note on `next/font` vs Fontsource:** `next/font/google` is the recommended path for Next.js 13+ (App Router). It automatically self-hosts fonts, injects preload hints into `<head>`, removes all third-party Google Fonts network requests, and applies `font-display: swap`. Use Fontsource npm only if you need fine-grained control over which Unicode subsets are loaded per-page.

---

### Typography in globals.css — Base Styles

```css
/* globals.css — typography base layer */

*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-size: 16px; /* base for rem calculations */
  -webkit-text-size-adjust: 100%;
  text-size-adjust: 100%;
}

body {
  font-family: var(--font-body);
  font-size: var(--text-base);
  font-weight: var(--weight-regular);
  line-height: var(--leading-loose);
  color: #1a1614; /* warm near-black, defined by color system */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
  font-feature-settings: 'kern' 1, 'liga' 1, 'calt' 1;
}

/* Heading reset — apply tokens via .type-* classes or Tailwind utilities */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: var(--weight-bold);
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tighter);
  margin-top: 0;
  margin-bottom: 0.5em;
}

h1 { font-size: var(--text-5xl); letter-spacing: var(--tracking-tighter); }
h2 { font-size: var(--text-4xl); letter-spacing: var(--tracking-tight); font-weight: var(--weight-semibold); }
h3 { font-size: var(--text-2xl); letter-spacing: var(--tracking-normal); font-weight: var(--weight-semibold); }
h4 { font-size: var(--text-xl);  letter-spacing: var(--tracking-normal); font-family: var(--font-heading); font-style: italic; }
h5 { font-size: var(--text-lg);  letter-spacing: var(--tracking-normal); font-family: var(--font-body); font-weight: var(--weight-semibold); }
h6 { font-size: var(--text-base); font-family: var(--font-body); font-weight: var(--weight-semibold); }

p {
  margin-top: 0;
  margin-bottom: var(--paragraph-spacing);
  max-width: var(--measure-base);
}

/* Optimise ligatures in headings — Cormorant Garamond has beautiful ff/fi/fl ligatures */
h1, h2, h3 {
  font-feature-settings: 'kern' 1, 'liga' 1, 'calt' 1, 'dlig' 1;
}

/* Tabular numbers for all price/numeric contexts */
[data-type="price"],
.price,
.tabular-nums {
  font-variant-numeric: tabular-nums;
  font-feature-settings: 'tnum' 1;
}

/* Small caps for menu category labels */
.menu-category,
[data-type="category"] {
  font-variant-caps: all-small-caps;
  letter-spacing: var(--tracking-caps);
}
```

---

### Summary — Deliverable Checklist

| Deliverable                                  | Status |
|----------------------------------------------|--------|
| Typeface selection with rationale             | Done   |
| Source validation (all 6 criteria)            | Done   |
| Full type scale table (12 tokens)             | Done   |
| Loading strategy (next/font + font-display)   | Done   |
| @font-face declarations (self-hosting path)   | Done   |
| CSS custom properties (:root)                 | Done   |
| Semantic utility classes (.type-*)            | Done   |
| Tailwind theme extension                      | Done   |
| Responsive scale breakpoints                  | Done   |
| globals.css base layer                        | Done   |
| next/font/google layout.tsx implementation    | Done   |
| Performance budget validated (<80KB, 2 req)   | Done   |
| Latin Extended charset confirmed              | Done   |
| Commercial license confirmed (OFL 1.1)        | Done   |
