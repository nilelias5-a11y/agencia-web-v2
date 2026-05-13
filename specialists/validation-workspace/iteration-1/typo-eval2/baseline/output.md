# Typography System — Café Vermut
## Specialty Coffee Shop | "Japan meets Barcelona" | Minimal, Premium

---

## 1. Why Blue Bottle and Onibus Coffee's Typography Works

**Blue Bottle Coffee** uses a restrained editorial system: a high-contrast serif with tight tracking for display, paired with a clean grotesque for UI and body. The key insight is *tension through contrast* — the serif carries warmth, craft, and story; the sans-serif handles function with zero friction. White space is aggressive. Type sizes jump dramatically between hierarchy levels. Nothing is "medium" — things are either very large or very small.

**Onibus Coffee** leans more Japanese-minimal: ultra-thin sans-serif at large sizes creates an airy, gallery-like feel. Body copy is compact and dense but highly legible. The brand uses type as texture — long columns of small text create visual weight without imagery. Caption-level type carries as much design intention as the hero headline.

**What both share:**
- A single expressive typeface (usually serif or high-contrast) reserved exclusively for headlines and product names — never for UI
- A neutral, highly-legible sans-serif that disappears into function
- Dramatic size contrast (ratio of ~6:1 between display and body)
- Tight letter-spacing on display sizes, normal on body
- Generous line height in body copy (editorial rhythm)
- Deliberate use of uppercase/lowercase — never mixed arbitrarily
- Language-specific considerations (ligatures, special characters for Spanish)

---

## 2. Font Pairing — Selection and Rationale

### Display / Editorial: **Cormorant Garamond** (Variable)
**Source:** Google Fonts — variable font, free, open license
**Axes:** Weight (300–700), optical size available

**Why Cormorant:**
- High-contrast thin/thick strokes evoke Japanese calligraphic tradition while referencing European editorial typography (Barcelona newspaper culture)
- The extreme contrast at large sizes (60px+) is visually arresting — exactly what Blue Bottle achieves with their display serif
- Italic variant is exceptionally expressive for tasting notes and origin stories — reads as handwritten expertise
- Spanish-language support is native: ñ, tildes (á/é/í/ó/ú), ¿/¡ are beautifully drawn
- Variable font reduces HTTP requests; a single file covers all weights
- At small sizes it degrades gracefully — use weight 400+ below 18px
- Optical-size variation: the thin strokes remain crisp at display sizes, avoiding the "spider web" problem of other high-contrast serifs

**Used for:** Hero headlines, product names, origin story pull quotes, section titles, editorial callouts

### Body / UI: **Plus Jakarta Sans** (Variable)
**Source:** Google Fonts — variable font, free, open license
**Axes:** Weight (200–800)

**Why Plus Jakarta Sans:**
- Geometric grotesque with humanist corrections: more warmth than Helvetica, more precision than Gill Sans
- The slightly wider apertures improve legibility for long Spanish body copy
- The "Plus" variant adds a subtle optical quality — slightly more refined than the base Jakarta Sans
- Works excellently at 13–16px for navigation and labels without appearing clinical
- Variable weight axis (200–800) gives full control: ExtraLight (200) for captions and tags, Regular (400) for body, SemiBold (600) for navigation and UI
- Strong multilingual support for Spanish
- Pairs with Cormorant by providing maximum contrast: where Cormorant is expressive and historical, Jakarta Sans is contemporary and systematic

**Used for:** Navigation, body copy, tasting notes body, product descriptions, labels, captions, buttons, metadata

### Why this pairing over alternatives:

| Alternative | Rejected because |
|---|---|
| Playfair Display + Inter | Playfair is too British-editorial; lacks the Japanese thinness. Inter is excellent but generic |
| Shippori Mincho + Noto Sans | Japanese but loses the Mediterranean warmth; Noto Sans lacks character at small sizes |
| Freight Display + Aktiv Grotesk | Excellent but both are paid; contradicts lean stack for early-stage project |
| EB Garamond + DM Sans | EB Garamond has inconsistent weight rendering across OS; DM Sans too rounded |

**Cormorant + Plus Jakarta Sans** is the specific pairing that achieves "Japan meets Barcelona" without forcing it: Cormorant's calligraphic heritage reads as both Japanese brush stroke and Spanish literary tradition; Plus Jakarta Sans is the contemporary global standard for premium DTC brands.

---

## 3. Type Scale

Based on a **Major Third (1.25)** modular scale anchored at 16px base. Display sizes break from the modular scale to achieve the dramatic contrast seen in Blue Bottle.

| Token | Size | Line Height | Letter Spacing | Weight | Use |
|---|---|---|---|---|---|
| `--text-display-xl` | 96px | 0.95 | -0.04em | 300 | Hero single word |
| `--text-display-lg` | 72px | 1.0 | -0.03em | 300 | Hero headline |
| `--text-display-md` | 56px | 1.05 | -0.025em | 300 | Section hero |
| `--text-display-sm` | 40px | 1.1 | -0.02em | 400 | Product name large |
| `--text-heading-xl` | 32px | 1.15 | -0.015em | 400 | H2 editorial |
| `--text-heading-lg` | 24px | 1.25 | -0.01em | 400 | H3 / product card title |
| `--text-heading-md` | 20px | 1.3 | -0.005em | 400 | H4 / sidebar heading |
| `--text-body-xl` | 18px | 1.7 | 0 | 400 | Lead paragraph |
| `--text-body-lg` | 16px | 1.65 | 0 | 400 | Body copy (default) |
| `--text-body-md` | 15px | 1.6 | 0 | 400 | Secondary body |
| `--text-body-sm` | 14px | 1.5 | 0.01em | 400 | UI text, descriptions |
| `--text-label-lg` | 13px | 1.4 | 0.08em | 500 | Navigation (uppercase) |
| `--text-label-md` | 12px | 1.4 | 0.1em | 500 | Tags, badges, categories |
| `--text-label-sm` | 11px | 1.3 | 0.12em | 500 | Micro-captions, metadata |
| `--text-tasting` | 14px italic | 1.8 | 0.02em | 300 | Tasting notes (Cormorant italic) |

**Notes on the scale:**
- Display tokens use Cormorant Garamond
- Heading tokens use Cormorant Garamond
- Body, label, and tasting tokens use Plus Jakarta Sans
- Exception: `--text-tasting` uses Cormorant Italic — the one case where the display font descends to small size, justified by the editorial nature of tasting notes
- Letter spacing is negative on display (optical correction for high-contrast serifs at large size) and positive on labels (readability enhancement for uppercase/small)

---

## 4. Variable Font Loading Strategy for Next.js

```typescript
// app/fonts.ts
import { Cormorant_Garamond, Plus_Jakarta_Sans } from 'next/font/google'

export const cormorant = Cormorant_Garamond({
  subsets: ['latin'],
  // Request only the weights actually used in the type scale
  // Variable font: Next.js will request the variable version when weight array > 1
  weight: ['300', '400', '500', '600'],
  style: ['normal', 'italic'],
  variable: '--font-cormorant',
  display: 'swap',
  preload: true,
  // Adjust for the high-contrast nature — reduces layout shift on swap
  adjustFontFallback: true,
})

export const plusJakartaSans = Plus_Jakarta_Sans({
  subsets: ['latin'],
  weight: ['200', '300', '400', '500', '600', '700'],
  style: ['normal', 'italic'],
  variable: '--font-jakarta',
  display: 'swap',
  preload: true,
  adjustFontFallback: true,
})
```

```typescript
// app/layout.tsx
import { cormorant, plusJakartaSans } from './fonts'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html
      lang="es"
      className={`${cormorant.variable} ${plusJakartaSans.variable}`}
    >
      <body>{children}</body>
    </html>
  )
}
```

**Loading strategy rationale:**
- `next/font/google` downloads fonts at build time and self-hosts them — zero runtime dependency on Google CDN, no CORS issues, no privacy regulation concerns (GDPR-relevant for EU)
- `display: 'swap'` ensures text is visible immediately with system fallback — critical for LCP metrics on product pages
- `adjustFontFallback: true` generates a synthetic fallback font with matching metrics, minimizing Cumulative Layout Shift (CLS) when the real font loads
- `preload: true` (default) injects `<link rel="preload">` for the primary font files — Cormorant is preloaded because it's visible above the fold in the hero
- Variable font files are automatically selected by Next.js when multiple weights are requested — one HTTP request instead of N
- Both fonts served from `/_next/static/media/` — cached aggressively by CDN

**Performance targets this achieves:**
- LCP: <2.5s (fonts preloaded, no CLS from swap because of adjustFontFallback)
- No flash of invisible text (FOIT)
- Minimal CLS: synthetic fallback metrics match actual font metrics closely

---

## 5. CSS Custom Properties

```css
/* styles/tokens/typography.css */

:root {
  /* Font families */
  --font-display: var(--font-cormorant), 'Georgia', 'Times New Roman', serif;
  --font-body: var(--font-jakarta), 'Helvetica Neue', 'Arial', sans-serif;

  /* Font weights — semantic naming */
  --font-weight-thin: 200;
  --font-weight-light: 300;
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  /* Type scale — display (Cormorant) */
  --text-display-xl: 6rem;        /* 96px */
  --text-display-lg: 4.5rem;      /* 72px */
  --text-display-md: 3.5rem;      /* 56px */
  --text-display-sm: 2.5rem;      /* 40px */
  --text-heading-xl: 2rem;        /* 32px */
  --text-heading-lg: 1.5rem;      /* 24px */
  --text-heading-md: 1.25rem;     /* 20px */

  /* Type scale — body (Plus Jakarta Sans) */
  --text-body-xl: 1.125rem;       /* 18px */
  --text-body-lg: 1rem;           /* 16px */
  --text-body-md: 0.9375rem;      /* 15px */
  --text-body-sm: 0.875rem;       /* 14px */
  --text-label-lg: 0.8125rem;     /* 13px */
  --text-label-md: 0.75rem;       /* 12px */
  --text-label-sm: 0.6875rem;     /* 11px */

  /* Line heights */
  --leading-tightest: 0.95;
  --leading-tight: 1.0;
  --leading-snug: 1.1;
  --leading-normal-display: 1.25;
  --leading-relaxed: 1.5;
  --leading-loose: 1.65;
  --leading-editorial: 1.7;
  --leading-tasting: 1.8;

  /* Letter spacing */
  --tracking-display-xl: -0.04em;
  --tracking-display-lg: -0.03em;
  --tracking-display-md: -0.025em;
  --tracking-display-sm: -0.02em;
  --tracking-heading: -0.01em;
  --tracking-normal: 0;
  --tracking-label: 0.08em;
  --tracking-label-wide: 0.1em;
  --tracking-label-widest: 0.12em;

  /* Composite text styles */
  --ts-hero: var(--font-weight-light) var(--text-display-lg) / var(--leading-tight) var(--font-display);
  --ts-product-name: var(--font-weight-regular) var(--text-display-sm) / var(--leading-snug) var(--font-display);
  --ts-origin-lead: var(--font-weight-regular) var(--text-body-xl) / var(--leading-editorial) var(--font-body);
  --ts-body: var(--font-weight-regular) var(--text-body-lg) / var(--leading-loose) var(--font-body);
  --ts-tasting: var(--font-weight-light) italic var(--text-body-sm) / var(--leading-tasting) var(--font-display);
  --ts-nav: var(--font-weight-medium) var(--text-label-lg) / var(--leading-relaxed) var(--font-body);
  --ts-caption: var(--font-weight-medium) var(--text-label-md) / var(--leading-relaxed) var(--font-body);
}

/* Responsive scale adjustments */
@media (max-width: 768px) {
  :root {
    --text-display-xl: 3.5rem;   /* 56px on mobile */
    --text-display-lg: 2.75rem;  /* 44px on mobile */
    --text-display-md: 2.25rem;  /* 36px on mobile */
    --text-display-sm: 2rem;     /* 32px on mobile */
    --text-heading-xl: 1.75rem;  /* 28px on mobile */
  }
}

/* Utility classes for semantic use */
.text-display { font-family: var(--font-display); }
.text-body { font-family: var(--font-body); }

.text-tasting-note {
  font: var(--ts-tasting);
  letter-spacing: var(--tracking-label);
}

.text-nav-item {
  font: var(--ts-nav);
  letter-spacing: var(--tracking-label);
  text-transform: uppercase;
}

.text-origin-body {
  font: var(--ts-body);
  font-feature-settings: 'liga' 1, 'kern' 1;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}
```

---

## 6. Tailwind Configuration

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      fontFamily: {
        display: ['var(--font-cormorant)', 'Georgia', 'Times New Roman', 'serif'],
        body: ['var(--font-jakarta)', 'Helvetica Neue', 'Arial', 'sans-serif'],
      },

      fontSize: {
        // Display scale (Cormorant)
        'display-xl': ['6rem', { lineHeight: '0.95', letterSpacing: '-0.04em' }],
        'display-lg': ['4.5rem', { lineHeight: '1.0', letterSpacing: '-0.03em' }],
        'display-md': ['3.5rem', { lineHeight: '1.05', letterSpacing: '-0.025em' }],
        'display-sm': ['2.5rem', { lineHeight: '1.1', letterSpacing: '-0.02em' }],
        'heading-xl': ['2rem', { lineHeight: '1.15', letterSpacing: '-0.015em' }],
        'heading-lg': ['1.5rem', { lineHeight: '1.25', letterSpacing: '-0.01em' }],
        'heading-md': ['1.25rem', { lineHeight: '1.3', letterSpacing: '-0.005em' }],

        // Body scale (Plus Jakarta Sans)
        'body-xl': ['1.125rem', { lineHeight: '1.7', letterSpacing: '0' }],
        'body-lg': ['1rem', { lineHeight: '1.65', letterSpacing: '0' }],
        'body-md': ['0.9375rem', { lineHeight: '1.6', letterSpacing: '0' }],
        'body-sm': ['0.875rem', { lineHeight: '1.5', letterSpacing: '0.01em' }],
        'label-lg': ['0.8125rem', { lineHeight: '1.4', letterSpacing: '0.08em' }],
        'label-md': ['0.75rem', { lineHeight: '1.4', letterSpacing: '0.1em' }],
        'label-sm': ['0.6875rem', { lineHeight: '1.3', letterSpacing: '0.12em' }],

        // Special: tasting notes (Cormorant italic at small size)
        'tasting': ['0.875rem', { lineHeight: '1.8', letterSpacing: '0.02em' }],
      },

      fontWeight: {
        thin: '200',
        light: '300',
        regular: '400',
        medium: '500',
        semibold: '600',
        bold: '700',
      },

      lineHeight: {
        tightest: '0.95',
        tight: '1.0',
        snug: '1.1',
        'normal-display': '1.25',
        relaxed: '1.5',
        loose: '1.65',
        editorial: '1.7',
        tasting: '1.8',
      },

      letterSpacing: {
        'display-xl': '-0.04em',
        'display-lg': '-0.03em',
        'display-md': '-0.025em',
        'display-sm': '-0.02em',
        heading: '-0.01em',
        label: '0.08em',
        'label-wide': '0.1em',
        'label-widest': '0.12em',
      },
    },
  },
  plugins: [],
}

export default config
```

---

## 7. Usage Patterns by Content Type

### Hero Display Text
```tsx
<h1 className="font-display text-display-lg font-light tracking-display-lg leading-tight">
  Café del fin<br />del mundo
</h1>
```

### Product Name (PDP)
```tsx
<h1 className="font-display text-display-sm font-regular tracking-display-sm leading-snug">
  Yirgacheffe Natural
</h1>
<p className="font-body text-label-lg font-medium tracking-label uppercase text-stone-500 mt-2">
  Etiopía · Lavado · Altura 1.900m
</p>
```

### Origin Story (Body Copy)
```tsx
<article className="font-body text-body-lg leading-loose max-w-prose">
  <p className="text-body-xl leading-editorial mb-6">
    En las laderas del Rift Valley etíope, los cafetos crecen salvajes...
  </p>
  <p>
    Lorem ipsum body copy continues here...
  </p>
</article>
```

### Tasting Notes (Editorial)
```tsx
<blockquote className="font-display text-tasting italic font-light leading-tasting tracking-normal border-l border-stone-300 pl-4">
  Frambuesa fresca, chocolate con leche, final largo de mandarina.
</blockquote>
```

### Navigation
```tsx
<nav>
  <a className="font-body text-label-lg font-medium tracking-label uppercase hover:opacity-60 transition-opacity">
    Tienda
  </a>
</nav>
```

### Product Labels / Captions
```tsx
<span className="font-body text-label-md font-medium tracking-label-wide uppercase">
  Origen único · 250g
</span>
```

---

## 8. Design Decisions Specific to Spanish Language

1. **Ligatures enabled** (`font-feature-settings: 'liga' 1`) — Cormorant's ligatures (fi, fl, ff) are beautiful in Spanish long-form text
2. **Tildes (á, é, í, ó, ú, ñ)** — both fonts have excellent diacritic design; confirmed in Google Fonts Latin subset
3. **Opening punctuation (¿, ¡)** — both fonts include these; Cormorant's version is particularly elegant at display sizes
4. **Line length** — Spanish words average slightly longer than English; max-w-prose (65ch) remains appropriate
5. **Uppercase labels** — Spanish navigation items (Tienda, Origen, Contacto) are short enough that the uppercase label treatment works without readability loss

---

## 9. Summary

| Role | Typeface | Weight | Style |
|---|---|---|---|
| Hero headline | Cormorant Garamond | 300 | Normal |
| Product names | Cormorant Garamond | 400 | Normal |
| Origin story lead | Plus Jakarta Sans | 400 | Normal |
| Body copy | Plus Jakarta Sans | 400 | Normal |
| Tasting notes | Cormorant Garamond | 300 | Italic |
| Section headings | Cormorant Garamond | 400 | Normal |
| Navigation | Plus Jakarta Sans | 500 | Normal, uppercase |
| Labels / captions | Plus Jakarta Sans | 500 | Normal, uppercase |
| Metadata / tags | Plus Jakarta Sans | 400–500 | Normal |

**The core discipline:** Cormorant is expressive and never used for UI. Plus Jakarta Sans is functional and never used for display. The only deliberate rule break is tasting notes — Cormorant italic at 14px — because tasting notes are editorial, not functional, and the italic gives them a sensory, handwritten quality that reinforces the premium coffee narrative.
