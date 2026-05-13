# Typography System — La Nonna

**Project:** La Nonna Italian Restaurant — Gràcia, Barcelona
**Stack:** Next.js 16 + Tailwind CSS v4
**Languages:** Spanish / Catalan
**Brand:** Authentic, warm, Italian-classic + modern

---

## 1. Typeface Pairing Decision

### Heading Font — Playfair Display
- **Source:** Google Fonts
- **Weights loaded:** 400 (Regular), 500 (Medium), 700 (Bold), 400 Italic
- **Rationale:** Playfair Display carries strong editorial personality rooted in classical serif tradition — high-contrast strokes, prominent serifs, and a slightly dramatic italic that evokes Italian signage and old-world menus. It is immediately warm and trustworthy at display sizes without feeling formal or cold. The existing project already uses it, confirming brand fit.

### Body Font — Lato
- **Source:** Google Fonts
- **Weights loaded:** 300 (Light), 400 (Regular), 700 (Bold)
- **Rationale:** Lato is a humanist sans-serif designed with classical proportions that echo a warm, handcrafted quality. Its open apertures and legible letter-spacing handle Spanish and Catalan diacritics cleanly (á, é, í, ó, ú, à, è, ï, ü, ç, l·l). It pairs with Playfair Display without competing — the sans carries body copy, UI labels, prices, and legal text at small sizes without fatigue.

### Pairing Logic
```
Playfair Display  →  Hero headlines, section titles, menu item names, pull quotes
Lato              →  Body copy, menu descriptions, prices, form labels, nav, legal
```

---

## 2. Full Type Scale

The scale follows a **Major Third (1.250)** ratio anchored at 16px base. Display sizes break the ratio for dramatic effect.

| Token        | rem      | px (ref) | Usage                                    | Font              | Weight | Line-height | Letter-spacing |
|--------------|----------|----------|------------------------------------------|-------------------|--------|-------------|----------------|
| `--fs-2xs`   | 0.625rem | 10px     | Legal fine print, footnotes              | Lato              | 400    | 1.6         | 0.03em         |
| `--fs-xs`    | 0.75rem  | 12px     | Form helper text, labels small           | Lato              | 400    | 1.5         | 0.02em         |
| `--fs-sm`    | 0.875rem | 14px     | Form labels, nav items, price tags       | Lato              | 400/700| 1.5         | 0.01em         |
| `--fs-base`  | 1rem     | 16px     | Body copy, menu descriptions             | Lato              | 400    | 1.7         | 0              |
| `--fs-md`    | 1.125rem | 18px     | Lead paragraphs, featured descriptions   | Lato              | 300    | 1.7         | 0              |
| `--fs-lg`    | 1.25rem  | 20px     | Card titles, menu item names (small)     | Lato 700 / PD 400 | 700    | 1.4         | −0.01em        |
| `--fs-xl`    | 1.5rem   | 24px     | Section sub-headings (h3)                | Playfair Display  | 400    | 1.3         | −0.01em        |
| `--fs-2xl`   | 1.875rem | 30px     | Section headings (h2)                    | Playfair Display  | 700    | 1.25        | −0.02em        |
| `--fs-3xl`   | 2.25rem  | 36px     | Page headings (h1 interior pages)        | Playfair Display  | 700    | 1.2         | −0.02em        |
| `--fs-4xl`   | 3rem     | 48px     | Hero sub-headline                        | Playfair Display  | 500i   | 1.15        | −0.02em        |
| `--fs-5xl`   | 3.75rem  | 60px     | Hero headline (desktop)                  | Playfair Display  | 700    | 1.1         | −0.03em        |
| `--fs-6xl`   | 4.5rem   | 72px     | Hero headline (large screens ≥1280px)    | Playfair Display  | 700    | 1.05        | −0.03em        |

**Menu item name:** `--fs-lg` with Playfair Display 400 — sized for scanability on a dense menu card.
**Menu item description:** `--fs-base` with Lato 400, color muted — reads as supporting content.
**Menu item price:** `--fs-sm` with Lato 700, color accent — visually separated, concise.

---

## 3. Next.js Font Loading Strategy

Use `next/font/google` with `display: swap` and `preload: true`. Declare fonts once in the root layout and pass them as CSS variables so Tailwind and the CSS design system can both consume them.

### `/app/fonts.ts`

```ts
import { Playfair_Display, Lato } from 'next/font/google';

export const playfairDisplay = Playfair_Display({
  subsets: ['latin', 'latin-ext'],   // latin-ext covers á é í ó ú à è ï ü ç
  weight: ['400', '500', '700'],
  style: ['normal', 'italic'],
  display: 'swap',
  preload: true,
  variable: '--font-heading',
});

export const lato = Lato({
  subsets: ['latin', 'latin-ext'],
  weight: ['300', '400', '700'],
  style: ['normal', 'italic'],
  display: 'swap',
  preload: true,
  variable: '--font-body',
});
```

### `/app/layout.tsx` (root layout)

```tsx
import { playfairDisplay, lato } from './fonts';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html
      lang="es"
      className={`${playfairDisplay.variable} ${lato.variable}`}
    >
      <body>{children}</body>
    </html>
  );
}
```

This pattern:
- Generates self-hosted font files at build time (no third-party runtime requests)
- Injects `--font-heading` and `--font-body` CSS variables on `<html>`
- Zero layout shift because `display: swap` + preloaded critical weights
- `latin-ext` subset ensures full coverage of Spanish and Catalan characters

---

## 4. CSS Custom Properties

Add to `/app/globals.css` (or the existing design system file):

```css
:root {
  /* ─── Font families ─── */
  --font-heading: var(--font-heading, 'Playfair Display', Georgia, 'Times New Roman', serif);
  --font-body:    var(--font-body, 'Lato', system-ui, -apple-system, sans-serif);

  /* ─── Type scale (rem) ─── */
  --fs-2xs:  0.625rem;   /* 10px — legal fine print */
  --fs-xs:   0.75rem;    /* 12px — helper text */
  --fs-sm:   0.875rem;   /* 14px — labels, prices */
  --fs-base: 1rem;       /* 16px — body copy */
  --fs-md:   1.125rem;   /* 18px — lead text */
  --fs-lg:   1.25rem;    /* 20px — card/menu item titles */
  --fs-xl:   1.5rem;     /* 24px — h3 */
  --fs-2xl:  1.875rem;   /* 30px — h2 */
  --fs-3xl:  2.25rem;    /* 36px — h1 interior */
  --fs-4xl:  3rem;       /* 48px — hero sub */
  --fs-5xl:  3.75rem;    /* 60px — hero headline */
  --fs-6xl:  4.5rem;     /* 72px — hero headline lg */

  /* ─── Line heights ─── */
  --lh-tight:   1.1;
  --lh-snug:    1.25;
  --lh-normal:  1.5;
  --lh-relaxed: 1.7;

  /* ─── Letter spacing ─── */
  --ls-tight:  -0.03em;
  --ls-snug:   -0.02em;
  --ls-normal:  0;
  --ls-wide:    0.02em;
  --ls-wider:   0.03em;

  /* ─── Semantic type roles ─── */
  --type-hero-size:        var(--fs-5xl);
  --type-hero-lh:          var(--lh-tight);
  --type-hero-ls:          var(--ls-tight);

  --type-h1-size:          var(--fs-3xl);
  --type-h1-lh:            1.2;
  --type-h1-ls:            var(--ls-snug);

  --type-h2-size:          var(--fs-2xl);
  --type-h2-lh:            1.25;
  --type-h2-ls:            var(--ls-snug);

  --type-h3-size:          var(--fs-xl);
  --type-h3-lh:            1.3;
  --type-h3-ls:            -0.01em;

  --type-menu-name-size:   var(--fs-lg);
  --type-menu-name-lh:     1.4;

  --type-menu-desc-size:   var(--fs-base);
  --type-menu-desc-lh:     var(--lh-relaxed);

  --type-menu-price-size:  var(--fs-sm);
  --type-menu-price-lh:    var(--lh-normal);

  --type-body-size:        var(--fs-base);
  --type-body-lh:          var(--lh-relaxed);

  --type-lead-size:        var(--fs-md);
  --type-lead-lh:          var(--lh-relaxed);

  --type-label-size:       var(--fs-sm);
  --type-label-lh:         var(--lh-normal);

  --type-legal-size:       var(--fs-xs);
  --type-legal-lh:         1.6;
}

/* ─── Fluid hero headline: scales between 48px (mobile) and 72px (xl) ─── */
@media (min-width: 1280px) {
  :root {
    --type-hero-size: var(--fs-6xl);
  }
}

/* ─── Base typographic reset ─── */
body {
  font-family: var(--font-body);
  font-size:   var(--type-body-size);
  line-height: var(--type-body-lh);
  font-weight: 400;
  -webkit-font-smoothing:  antialiased;
  -moz-osx-font-smoothing: grayscale;
}

h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
}

h1 {
  font-size:      var(--type-h1-size);
  line-height:    var(--type-h1-lh);
  letter-spacing: var(--type-h1-ls);
  font-weight:    700;
}

h2 {
  font-size:      var(--type-h2-size);
  line-height:    var(--type-h2-lh);
  letter-spacing: var(--type-h2-ls);
  font-weight:    700;
}

h3 {
  font-size:      var(--type-h3-size);
  line-height:    var(--type-h3-lh);
  letter-spacing: -0.01em;
  font-weight:    400;
}
```

---

## 5. Tailwind Theme Extension

### `tailwind.config.ts`

```ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        heading: ['var(--font-heading)', 'Georgia', '"Times New Roman"', 'serif'],
        body:    ['var(--font-body)',    'system-ui', '-apple-system', 'sans-serif'],
      },
      fontSize: {
        '2xs':  ['0.625rem',  { lineHeight: '1.6',  letterSpacing: '0.03em' }],
        'xs':   ['0.75rem',   { lineHeight: '1.5',  letterSpacing: '0.02em' }],
        'sm':   ['0.875rem',  { lineHeight: '1.5',  letterSpacing: '0.01em' }],
        'base': ['1rem',      { lineHeight: '1.7'  }],
        'md':   ['1.125rem',  { lineHeight: '1.7'  }],
        'lg':   ['1.25rem',   { lineHeight: '1.4',  letterSpacing: '-0.01em' }],
        'xl':   ['1.5rem',    { lineHeight: '1.3',  letterSpacing: '-0.01em' }],
        '2xl':  ['1.875rem',  { lineHeight: '1.25', letterSpacing: '-0.02em' }],
        '3xl':  ['2.25rem',   { lineHeight: '1.2',  letterSpacing: '-0.02em' }],
        '4xl':  ['3rem',      { lineHeight: '1.15', letterSpacing: '-0.02em' }],
        '5xl':  ['3.75rem',   { lineHeight: '1.1',  letterSpacing: '-0.03em' }],
        '6xl':  ['4.5rem',    { lineHeight: '1.05', letterSpacing: '-0.03em' }],
      },
      lineHeight: {
        tight:   '1.1',
        snug:    '1.25',
        normal:  '1.5',
        relaxed: '1.7',
      },
      letterSpacing: {
        tighter: '-0.03em',
        tight:   '-0.02em',
        snug:    '-0.01em',
        normal:  '0',
        wide:    '0.02em',
        wider:   '0.03em',
      },
    },
  },
  plugins: [],
};

export default config;
```

---

## 6. Utility Classes for Recurring Patterns

Add to `globals.css` for consistent semantic re-use across components:

```css
/* Hero headline */
.text-hero {
  font-family:    var(--font-heading);
  font-size:      var(--type-hero-size);
  line-height:    var(--type-hero-lh);
  letter-spacing: var(--type-hero-ls);
  font-weight:    700;
}

/* Menu item name (Playfair at lg size) */
.text-menu-name {
  font-family:  var(--font-heading);
  font-size:    var(--type-menu-name-size);
  line-height:  var(--type-menu-name-lh);
  font-weight:  400;
}

/* Menu item description */
.text-menu-desc {
  font-family: var(--font-body);
  font-size:   var(--type-menu-desc-size);
  line-height: var(--type-menu-desc-lh);
  font-weight: 300;
}

/* Menu price */
.text-menu-price {
  font-family:    var(--font-body);
  font-size:      var(--type-menu-price-size);
  line-height:    var(--type-menu-price-lh);
  font-weight:    700;
  letter-spacing: var(--ls-wide);
}

/* Form label */
.text-label {
  font-family:    var(--font-body);
  font-size:      var(--type-label-size);
  line-height:    var(--type-label-lh);
  font-weight:    700;
  letter-spacing: var(--ls-wide);
  text-transform: uppercase;
}

/* Legal / fine print */
.text-legal {
  font-family: var(--font-body);
  font-size:   var(--type-legal-size);
  line-height: var(--type-legal-lh);
  font-weight: 400;
}
```

---

## 7. Tailwind Class Reference (Quick Cheatsheet)

| Content role          | Tailwind classes                                                  |
|-----------------------|-------------------------------------------------------------------|
| Hero headline         | `font-heading text-5xl lg:text-6xl font-bold tracking-tighter`   |
| Hero sub-headline     | `font-heading text-4xl font-medium italic tracking-tight`         |
| Interior page h1      | `font-heading text-3xl font-bold tracking-tight`                  |
| Section h2            | `font-heading text-2xl font-bold tracking-tight`                  |
| Section h3            | `font-heading text-xl font-normal tracking-snug`                  |
| Menu item name        | `font-heading text-lg font-normal`                                |
| Menu item description | `font-body text-base font-light leading-relaxed`                  |
| Menu item price       | `font-body text-sm font-bold tracking-wide`                       |
| Body copy             | `font-body text-base leading-relaxed`                             |
| Lead paragraph        | `font-body text-md font-light leading-relaxed`                    |
| Form label            | `font-body text-sm font-bold tracking-wide uppercase`             |
| Form helper text      | `font-body text-xs`                                               |
| Legal text            | `font-body text-xs leading-relaxed`                               |

---

## 8. Notes on Spanish/Catalan Compatibility

- `latin-ext` subset covers all required characters: á é í ó ú ü à è ï ç ñ l·l (Catalan interpunct via `l·l` ligature).
- Playfair Display renders Catalan accentuated vowels (à, è, ï) with the same proportional elegance as base Latin — confirmed in Google Fonts specimen.
- Lato 400 handles dense Spanish text (long compound nouns, polysyllabic words) without crowding at `--fs-base` with `line-height: 1.7`.
- At `--fs-xs` (legal text), Lato Light is avoided — Regular 400 keeps legal text legible on screen even at 12px.

---

## 9. Implementation Checklist

- [ ] Create `/app/fonts.ts` with the `next/font/google` declarations above
- [ ] Add `playfairDisplay.variable` + `lato.variable` to `<html>` className in root layout
- [ ] Paste CSS custom properties block into `globals.css`
- [ ] Update `tailwind.config.ts` with the theme extension
- [ ] Add utility classes (`.text-hero`, `.text-menu-name`, etc.) to `globals.css`
- [ ] Remove any existing `@import url('https://fonts.googleapis.com/...')` calls — `next/font` self-hosts
- [ ] Verify build: `next build` — confirm 0 font-related warnings
- [ ] Test on mobile (375px) that hero headline does not overflow at `text-5xl`
