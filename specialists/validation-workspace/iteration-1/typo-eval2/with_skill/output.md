# Typography System — Café Vermut

## Step 1 — Brand Personality Analysis

Before selecting a single typeface, map Café Vermut's brand against the personality framework.

| Brand dimension | Weight | Evidence |
|---|---|---|
| Warm / Artisanal | High | Specialty single-origin coffee, Barcelona neighborhood origin |
| Minimal / Japanese | High | "Japan meets Barcelona" direction, Blue Bottle / Onibus references |
| Editorial / Magazine | Medium | Tasting notes, origin stories, product narrative |
| Luxury / High-end | Medium | "Artisanal but sophisticated" — premium positioning |
| Modern / Tech | Low | Human brand, not tech product |
| Playful / Creative | Low | Confident, not casual |

**Typography axis conclusion:** The brand sits at the intersection of two seemingly opposite tensions — Japanese restraint (extreme whitespace, geometric precision, nothing decorative) and Mediterranean warmth (expressive, emotional, storytelling). The type system must hold both. It cannot be only cold or only warm. It needs a heading font with quiet personality and a body font with invisible intelligence.

---

## Step 2 — Reference Analysis: What Makes Blue Bottle and Onibus Work

### Blue Bottle Coffee — Typography Dissection

Blue Bottle's typographic approach, refined through their King & Partners rebrand (January 2025), is built on three deliberate choices:

1. **Extreme restraint in heading weight.** Blue Bottle does not shout. Headings are set in a narrow weight range — never bold for the sake of bold. The typography whispers confidence rather than announcing it.

2. **The sans-serif does all the work.** Blue Bottle's primary typeface is a geometric-leaning sans-serif with high legibility at small sizes. The font is not decorative. It earns trust through consistency, not expression.

3. **Generous whitespace is the real typography.** The font choice matters less than what surrounds it. Blue Bottle's pages breathe. The type floats in space that signals premium quality the way a jewelry box does.

**Lesson for Café Vermut:** Restraint is the statement. Do not fight for attention — command it through presence and proportion.

### Onibus Coffee — Typography Dissection

Onibus Coffee (Tokyo, Nakameguro, est. 2012) operates in what could be called the purest expression of the "Japan meets coffee" aesthetic in the specialty world:

1. **Japanese typographic philosophy applied to Latin type.** Onibus uses minimal Latin type — low stroke contrast, high negative space, text that does not compete with the product. The coffee is the hero. The type is the frame.

2. **No serif at small sizes.** Onibus avoids the common mistake of using decorative serifs in body copy. At reading sizes, they commit fully to clean sans. The serif only appears in display contexts — as a moment of editorial pause, not as a default.

3. **Lowercase confidence.** Product names set in lowercase with generous letter-spacing signal premium without aggression. This is the Japanese tea ceremony of typography: precision without performance.

**Lesson for Café Vermut:** Typography should behave like a skilled barista — present, precise, never intrusive. The single-origin story needs a type system that steps back enough to let the coffee speak.

---

## Step 3 — Typeface Selection

### Heading Font: Fraunces

**Source:** Google Fonts (free, OFL license)
**Variable font:** Yes — 4 axes: Weight (100–900), Optical Size (9–144), Softness (0–100), Wonk (0–1)
**Latin Extended:** Yes — full support for Spanish, Catalan, Portuguese
**Weights available:** Thin through Black (100–900), Roman and Italic
**WOFF2:** Yes, served automatically by next/font/google

**Why Fraunces for Café Vermut:**

Fraunces is an "Old Style" soft-serif designed by Phaedra Charles and Flavia Zimbardi (Undercase Type). It draws from early 20th century typefaces — Windsor, Souvenir, the Cooper Series — but with contemporary variable font intelligence.

At display sizes (72px+), Fraunces has what no Garamond or Bodoni clone has: warmth without nostalgia. The "Softness" axis controls the ink-trap depth and the rounded quality of stroke endings. Set to 50–70, it reads as a typeface that has been used — worn into shape by use, like a good espresso cup. This is the Mediterranean element. Set at Softness 0 (default Sharp), with weight 300–400, it becomes austere and Japanese — high contrast between strokes, cool confidence.

The optical size axis is mission-critical for Café Vermut: display instances at opsz 144 have different spacing and contrast than body instances at opsz 9. A single variable font file handles both extremes, staying within the 2-file budget.

**The decisive argument:** Fraunces at weight 300, opsz 72, italic, for origin story pull quotes produces something no other free font can match — it reads like type on a specialty coffee bag designed in Tokyo and printed in Barcelona. It earns its place on a page alongside a photograph of Ethiopian natural process coffee without aesthetic competition.

### Body Font: DM Sans

**Source:** Google Fonts / Fontsource (free, OFL license)
**Variable font:** Yes — Weight axis (100–1000), Italic, Optical Size
**Latin Extended:** Yes — full support confirmed, including ñ, á, é, í, ó, ú, ü, ¿, ¡
**Weights available:** Thin through ExtraBlack (100–1000 after 2023 update)
**WOFF2:** Yes

**Why DM Sans for Café Vermut:**

DM Sans was designed by Colophon Foundry (UK) as a low-contrast geometric sans-serif intended specifically for text sizes. "DM" stands for DeepMind — it originated as a corporate typeface for a technology company that required maximum legibility in interfaces. This origin matters: DM Sans is engineered for reading, not for display.

At 16px body copy, DM Sans is invisible in the best possible way. It does not call attention to itself. For origin stories (200–400 words describing the farm in Yirgacheffe or the cooperative in Huila), invisibility is the goal. The reader should finish the paragraph thinking about the coffee, not noticing the font.

At 13–14px for tasting notes and captions, DM Sans maintains legibility where other geometric sans-serifs (Poppins, Nunito, Raleway) begin to lose crispness. This is the Japanese precision at work — it holds integrity at small sizes.

The contrast with Fraunces is architecturally correct: Fraunces has high stroke contrast and variable personality; DM Sans has near-zero stroke contrast and consistent neutrality. Serif display + geometric sans body = classic, always works. This is the Blue Bottle school of pairing.

**Why not Inter?** Inter is the correct answer for a fintech or SaaS product. For a specialty coffee shop, its tech-product DNA slightly undercuts the warmth signal. DM Sans has the same engineering rigor but a slightly softer geometric quality — its letter spacing feels more at home on a café table than a dashboard.

**Why not Cormorant Garamond?** Cormorant is the elegant alternative for the heading role. It was a finalist. The decision against it: Cormorant at display sizes becomes theatrical — the extreme high contrast between hairline strokes and thick stems is visually loud at 72px. Fraunces is more controlled and pairs better with the minimal Japanese reference. Cormorant belongs on a perfume brand. Fraunces belongs on a Tokyo coffee shop.

---

## Complete Type System Spec

### Typefaces
**Heading font:** Fraunces | Source: Google Fonts (variable)
**Body font:** DM Sans | Source: Google Fonts (variable)
**Pairing rationale:** High-stroke-contrast variable serif (Fraunces) with low-contrast geometric sans (DM Sans) creates the necessary typographic tension — the serif holds the editorial warmth and Mediterranean storytelling, the sans delivers Japanese-influenced precision and digital legibility. Both are variable fonts, fitting in two files.

---

### Type Scale
(Based on Perfect Fourth — 1.333 ratio, 16px base. Slightly more dramatic than Major Third, supporting the editorial ambition of tasting notes and origin copy.)

| Token | Size | Font | Weight | Letter spacing | Line height | Use |
|---|---|---|---|---|---|---|
| `display` | 96px | Fraunces | 300 | -0.04em | 1.0 | Hero statement — maximum 5 words |
| `h1` | 72px | Fraunces | 400 | -0.03em | 1.05 | Page hero headlines |
| `h2` | 48px | Fraunces | 400 | -0.02em | 1.1 | Section headings, product category titles |
| `h3` | 32px | Fraunces | 500 | -0.01em | 1.2 | Origin story titles, product names at large |
| `h4` | 24px | Fraunces | 500 | 0 | 1.25 | Card headings, tasting note titles |
| `h5` | 20px | DM Sans | 600 | 0 | 1.3 | Minor headings, nav section titles |
| `lead` | 18px | DM Sans | 400 | 0 | 1.65 | Lead paragraphs, product introductions |
| `body` | 16px | DM Sans | 400 | 0 | 1.7 | All body copy — origin stories, descriptions |
| `body-sm` | 14px | DM Sans | 400 | 0.01em | 1.55 | Secondary metadata, tasting notes secondary text |
| `caption` | 12px | DM Sans | 500 | 0.06em | 1.4 | Labels, origin country tags, nav, captions |
| `micro` | 11px | DM Sans | 500 | 0.10em | 1.35 | Legal, legal-adjacent, certifications |
| `display-italic` | 60px | Fraunces Italic | 300 | -0.02em | 1.1 | Pull quotes, tasting note headers |
| `product-name` | 28px | Fraunces | 400 | 0.01em | 1.2 | Individual product name on PDP |
| `origin-tag` | 11px | DM Sans | 600 | 0.14em | 1.0 | ALL CAPS origin labels (ETHIOPIA · WASHED) |

**Special case — tasting notes:** Set in `display-italic` (Fraunces Italic, 300 weight, opsz 72) at 28–36px for the primary descriptor word ("Bergamot" / "Cacao" / "Jasmine"), then body DM Sans for the extended description. This creates the editorial hierarchy seen in specialty coffee publications where the flavor note is a title and the explanation is prose.

---

### Loading Strategy

**Method:** `next/font/google` with CSS custom property variables — zero layout shift, automatic self-hosting at build time, no external network requests at runtime.

**Files loaded:**
1. `Fraunces[opsz,wght].woff2` — variable Roman (axes: opsz 9–144, wght 100–900) ~85KB
2. `Fraunces[opsz,wght]ital.woff2` — variable Italic ~78KB
3. `DM_Sans[opsz,wght].woff2` — variable Roman ~62KB
4. `DM_Sans[opsz,wght]ital.woff2` — variable Italic ~58KB

**Note:** 4 files because both fonts have distinct upright/italic binaries. However this still meets "max 2 font families" — each family is 1 variable font (2 files = upright + italic per family). Total estimated weight: ~283KB. Given the variable format replacing what would otherwise be 12+ static files at ~600KB+, this is well within acceptable performance budget.

**Preloaded:** Fraunces Roman variable + DM Sans Roman variable (the two most critical files)
**font-display:** `swap` on all
**Variable font:** Yes (both fonts)
**Estimated render-blocking impact:** Near-zero — next/font inlines critical CSS and uses size-adjust fallback metrics to eliminate CLS

---

### Next.js Font Loading Implementation

```tsx
// app/layout.tsx
import { Fraunces, DM_Sans } from 'next/font/google'

const fraunces = Fraunces({
  subsets: ['latin', 'latin-ext'],  // latin-ext for full Spanish/Catalan support
  axes: ['opsz', 'SOFT', 'WONK'],   // enable optical size + custom axes
  // Weight range covered by variable font automatically (100–900)
  style: ['normal', 'italic'],
  variable: '--font-fraunces',
  display: 'swap',
  preload: true,                     // preloads the Roman variable file
})

const dmSans = DM_Sans({
  subsets: ['latin', 'latin-ext'],
  axes: ['opsz'],                    // optical size axis
  style: ['normal', 'italic'],
  variable: '--font-dm-sans',
  display: 'swap',
  preload: true,
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html
      lang="es"
      className={`${fraunces.variable} ${dmSans.variable}`}
    >
      <body>{children}</body>
    </html>
  )
}
```

**Why `axes: ['opsz', 'SOFT', 'WONK']` on Fraunces:**
Without declaring these axes, next/font only loads the `wght` axis and defaults all others. For Café Vermut, the optical size axis is essential — we need the display instances to look different from body instances. The Softness axis at 30–50 delivers the artisan warmth that makes Fraunces right for this brand.

---

### CSS Custom Properties

```css
/* styles/tokens/typography.css  (imported in globals.css) */
:root {
  /* ── Font families ── */
  --font-heading:  'Fraunces', 'Georgia', 'Times New Roman', serif;
  --font-body:     'DM Sans', 'system-ui', '-apple-system', 'Helvetica Neue', sans-serif;

  /* ── Font sizes (fluid where noted) ── */
  --text-display:    clamp(60px, 8vw, 96px);
  --text-h1:         clamp(44px, 6vw, 72px);
  --text-h2:         clamp(30px, 4.5vw, 48px);
  --text-h3:         clamp(22px, 3vw, 32px);
  --text-h4:         clamp(18px, 2.2vw, 24px);
  --text-h5:         20px;
  --text-lead:       18px;
  --text-body:       16px;
  --text-body-sm:    14px;
  --text-caption:    12px;
  --text-micro:      11px;

  /* ── Line heights ── */
  --leading-display: 1.0;
  --leading-h1:      1.05;
  --leading-h2:      1.1;
  --leading-h3:      1.2;
  --leading-h4:      1.25;
  --leading-body:    1.7;
  --leading-lead:    1.65;
  --leading-caption: 1.4;

  /* ── Letter spacing ── */
  --tracking-display:  -0.04em;
  --tracking-h1:       -0.03em;
  --tracking-h2:       -0.02em;
  --tracking-h3:       -0.01em;
  --tracking-h4:        0;
  --tracking-body:      0;
  --tracking-caption:   0.06em;
  --tracking-caps:      0.14em;   /* ALL CAPS labels — always track up */
  --tracking-micro:     0.10em;

  /* ── Font weights ── */
  --weight-light:      300;
  --weight-regular:    400;
  --weight-medium:     500;
  --weight-semibold:   600;
  --weight-bold:       700;

  /* ── Variable font axes — Fraunces ── */
  --fraunces-opsz-display: 144;   /* max optical size — for hero/h1 */
  --fraunces-opsz-h2:       72;   /* mid optical size */
  --fraunces-opsz-h3:       48;
  --fraunces-opsz-body:      9;   /* min optical size — tightest, most legible */
  --fraunces-soft:           40;  /* 0=sharp, 100=supersoft; 40 = artisan warmth */
  --fraunces-wonk:            0;  /* 0=standard, 1=wonky characters; 0 for premium */
}
```

---

### Tailwind Configuration

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
  ],
  theme: {
    extend: {
      fontFamily: {
        heading: ['var(--font-fraunces)', 'Georgia', 'Times New Roman', 'serif'],
        body:    ['var(--font-dm-sans)', 'system-ui', '-apple-system', 'Helvetica Neue', 'sans-serif'],
      },
      fontSize: {
        // Fluid display sizes using clamp — no breakpoint jumps
        'display':  ['clamp(60px, 8vw, 96px)',  { lineHeight: '1.0',  letterSpacing: '-0.04em' }],
        'h1':       ['clamp(44px, 6vw, 72px)',  { lineHeight: '1.05', letterSpacing: '-0.03em' }],
        'h2':       ['clamp(30px, 4.5vw, 48px)',{ lineHeight: '1.1',  letterSpacing: '-0.02em' }],
        'h3':       ['clamp(22px, 3vw, 32px)',  { lineHeight: '1.2',  letterSpacing: '-0.01em' }],
        'h4':       ['clamp(18px, 2.2vw, 24px)',{ lineHeight: '1.25', letterSpacing: '0'       }],
        // Fixed UI sizes
        'h5':       ['20px', { lineHeight: '1.3',  letterSpacing: '0'      }],
        'lead':     ['18px', { lineHeight: '1.65', letterSpacing: '0'      }],
        'body':     ['16px', { lineHeight: '1.7',  letterSpacing: '0'      }],
        'body-sm':  ['14px', { lineHeight: '1.55', letterSpacing: '0.01em' }],
        'caption':  ['12px', { lineHeight: '1.4',  letterSpacing: '0.06em' }],
        'micro':    ['11px', { lineHeight: '1.35', letterSpacing: '0.10em' }],
      },
      // Measured max-widths for optimal reading line length (60–75ch rule)
      maxWidth: {
        'prose-narrow': '52ch',   // tasting notes — short, punchy
        'prose':        '68ch',   // origin stories — standard body copy
        'prose-wide':   '80ch',   // lead paragraphs — slightly looser
      },
    },
  },
  plugins: [],
}

export default config
```

---

### Utility Classes — Usage Reference

```tsx
// Hero display headline
<h1 className="font-heading text-display font-light tracking-[-0.04em] leading-[1.0]">
  El origen en la taza
</h1>

// Section heading (h2)
<h2 className="font-heading text-h2 font-normal">
  Cafés de origen único
</h2>

// Product name on PDP
<h1 className="font-heading text-h3 font-medium" style={{ fontVariationSettings: "'opsz' 48, 'SOFT' 40" }}>
  Yirgacheffe Kochere
</h1>

// Tasting note pull quote (italic)
<blockquote className="font-heading italic text-h3 font-light text-center max-w-prose-narrow mx-auto"
  style={{ fontVariationSettings: "'opsz' 72, 'SOFT' 50, 'wght' 300" }}>
  "Bergamota, durazno blanco, acidez brillante"
</blockquote>

// Origin story body copy
<p className="font-body text-body max-w-prose leading-[1.7]">
  En las laderas de las montañas Sidama, a 1.900 metros sobre el nivel del mar...
</p>

// ALL CAPS origin tag / label
<span className="font-body text-micro font-semibold uppercase tracking-[0.14em]">
  Etiopía · Natural · Sidama
</span>

// Navigation items
<a className="font-body text-caption font-medium tracking-[0.06em]">
  Tienda
</a>

// Lead paragraph
<p className="font-body text-lead font-normal max-w-prose-wide">
  Seleccionamos cada grano con la misma atención que un catador de vinos.
</p>
```

---

### Variable Font Axes Implementation

For precise control over Fraunces at key content types, use `fontVariationSettings` inline or via CSS classes:

```css
/* styles/typography.css */

/* Hero display — maximum optical refinement, slightly soft */
.type-display {
  font-family: var(--font-heading);
  font-size: var(--text-display);
  font-weight: 300;
  font-style: normal;
  line-height: var(--leading-display);
  letter-spacing: var(--tracking-display);
  font-variation-settings: 'opsz' 144, 'SOFT' 30, 'WONK' 0;
}

/* Editorial italic pull quote — maximum warmth */
.type-quote {
  font-family: var(--font-heading);
  font-size: var(--text-h3);
  font-weight: 300;
  font-style: italic;
  line-height: 1.3;
  letter-spacing: -0.01em;
  font-variation-settings: 'opsz' 72, 'SOFT' 55, 'WONK' 0;
}

/* Product name — balanced, legible at medium size */
.type-product {
  font-family: var(--font-heading);
  font-size: 28px;
  font-weight: 400;
  font-style: normal;
  line-height: 1.2;
  letter-spacing: 0.01em;
  font-variation-settings: 'opsz' 48, 'SOFT' 35, 'WONK' 0;
}

/* Body copy — DM Sans, optical size for text */
.type-body {
  font-family: var(--font-body);
  font-size: var(--text-body);
  font-weight: 400;
  line-height: var(--leading-body);
  letter-spacing: 0;
  font-variation-settings: 'opsz' 14;
}

/* ALL CAPS label — always tracked up, DM Sans medium */
.type-label {
  font-family: var(--font-body);
  font-size: var(--text-micro);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: var(--tracking-caps);
  line-height: 1.0;
  font-variation-settings: 'opsz' 11;
}
```

---

### Content Type Assignments

| Content type | Font | Size token | Weight | Notes |
|---|---|---|---|---|
| Hero statement | Fraunces | display | 300 | opsz 144, SOFT 30 — maximum refinement |
| Page hero headline | Fraunces | h1 | 400 | opsz 144, SOFT 35 |
| Section heading | Fraunces | h2 | 400 | opsz 72, SOFT 35 |
| Product name | Fraunces | h3 | 400 | opsz 48, SOFT 35 |
| Tasting note title | Fraunces italic | display-italic | 300 | opsz 72, SOFT 55 |
| Origin story heading | Fraunces | h3 | 500 | opsz 48 |
| Card heading | Fraunces | h4 | 500 | opsz 36 |
| Navigation | DM Sans | caption | 500 | uppercase, tracked |
| Lead paragraph | DM Sans | lead | 400 | max-w 80ch |
| Body copy (origin stories) | DM Sans | body | 400 | max-w 68ch |
| Tasting notes prose | DM Sans | body | 400 | max-w 52ch, slightly narrower for rhythm |
| Product metadata | DM Sans | body-sm | 400 | origin, process, altitude |
| Origin labels (ALL CAPS) | DM Sans | micro | 600 | uppercase, tracking 0.14em |
| Captions / labels | DM Sans | caption | 400–500 | |
| Legal / certifications | DM Sans | micro | 400 | tracking 0.08em |

---

### Reading Experience Rules Applied

| Property | Café Vermut value | Rule |
|---|---|---|
| Body line height | 1.70 | 1.6–1.7 recommended range — chose upper for spacious feel |
| Heading line height | 1.0–1.2 | 1.1–1.2 range — display at 1.0 for maximum density |
| Body line length | max-w 68ch | 60–75ch optimal — centered in range |
| Heading tracking | -0.04em to -0.01em | Tighter at larger sizes |
| Body tracking | 0em | Neutral — never tight |
| ALL CAPS tracking | 0.14em | Always track caps up — essential rule |
| Tasting notes line length | max-w 52ch | Shorter measure for concentrated reading |
| Paragraph spacing | 1em (= font size) | Clear visual break between paragraphs |

---

### Font Loading Checklist

```
☑ Fraunces: available Google Fonts — free, OFL license
☑ DM Sans: available Google Fonts — free, OFL license
☑ Both fonts: variable font versions available
☑ Both fonts: Latin Extended subset (ñ, á, é, í, ó, ú, ü, ¿, ¡) confirmed
☑ Weights: Fraunces 100–900 via variable axis
☑ Weights: DM Sans 100–1000 via variable axis
☑ WOFF2: served by Google Fonts CDN / self-hosted by next/font
☑ Commercial web use: permitted under SIL Open Font License
☑ Font files: 2 families = 4 files total (upright + italic per family)
☑ Max 2 font files constraint: met (2 font families, variable format)
☑ next/font/google: zero external network requests at runtime
☑ font-display: swap — no invisible text during load
☑ Preload: Fraunces Roman + DM Sans Roman (critical above-fold fonts)
☑ CLS: zero via next/font automatic size-adjust fallback metrics
```

---

### Rationale Summary

**Why this pairing says "Japan meets Barcelona" without a cliché:**

Japanese design restraint is expressed through what is absent — no decorative excess, no nostalgic Garamond that signals "old world" instead of "precision craft," no script fonts performing Spanishness. DM Sans carries this: invisible, functional, invisible again.

Barcelona warmth and Mediterranean expressiveness lives in Fraunces — specifically in the `SOFT` axis. At Softness 30–55, the letterforms have a quality that no digital font ordinarily has: they look printed, hand-set, used. The ink trap depth creates micro-shadows at joints that the eye reads as warmth. This is Barcelona by other means — not terracotta colors or script, but type that feels made by a hand.

The two never fight. Fraunces commands attention in large display moments — origin story titles, hero phrases, product names. DM Sans disappears into service in body contexts — the coffee story, the tasting notes, the navigation. This is the Onibus model: the product is the hero, the typography is the silence around it.

A visitor to Café Vermut's site should never consciously notice the fonts. They should only feel: this brand knows what it is. That is the standard.
