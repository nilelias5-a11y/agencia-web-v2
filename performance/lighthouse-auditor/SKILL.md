---
name: lighthouse-auditor
description: "Run, interpret and fix Lighthouse audits for Next.js apps on Vercel. Use this skill whenever the user mentions: Lighthouse scores, Lighthouse CI, LHCI, audit results, Performance/Accessibility/SEO/Best Practices scores, 'my Lighthouse score is X', TTI, TBT, speed index, accessibility violations, contrast ratio, missing aria labels, or wants to set up automated Lighthouse in CI. Also trigger when the user pastes a Lighthouse report or mentions specific audit failures. Always use this skill for any Lighthouse-related question in a Next.js + Vercel context."
---

# Lighthouse Auditor

You are a Lighthouse audit specialist for a web agency. Your role: help set up Lighthouse CI, interpret audit results, and fix specific audit failures in Next.js apps deployed on Vercel.

## Lighthouse fundamentals — know this before diagnosing

### Lab data vs Field data
- **Lab (Lighthouse)**: synthetic test from a controlled environment. Fast feedback, reproducible, but doesn't reflect real users.
- **Field (CrUX)**: real user data from Chrome. Found in Google Search Console → Core Web Vitals. This is what Google uses for rankings.
- **Rule**: Fix Lighthouse audits, but validate with CrUX. A perfect Lighthouse score with bad CrUX means the fix hasn't shipped to users yet.

### Score categories
| Category | What it measures | Main signals |
|---|---|---|
| Performance | Loading speed and interactivity | LCP, TBT, CLS, Speed Index, FCP, TTI |
| Accessibility | WCAG compliance | ARIA, contrast, keyboard nav, semantic HTML |
| Best Practices | Security and modern web practices | HTTPS, no mixed content, deprecated APIs |
| SEO | Search engine discoverability | Meta tags, canonical, robots, structured data |

**Performance score weights (2024):**
- LCP: 25%
- TBT: 30% (proxy for INP in lab)
- CLS: 15%
- FCP: 10%
- Speed Index: 10%
- TTI: 10%

TBT (Total Blocking Time) is the highest-weight metric and the most fixable.

## Setting up Lighthouse CI

### Install and configure

```bash
npm install --save-dev @lhci/cli
```

```js
// lighthouserc.js (root of project)
module.exports = {
  ci: {
    collect: {
      staticDistDir: './.next',         // or startServerCommand for SSR
      startServerCommand: 'npm start',
      url: ['http://localhost:3000', 'http://localhost:3000/about'],
      numberOfRuns: 3,                  // average of 3 runs for stability
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        // Thresholds — adjust to your baseline
        'categories:performance': ['warn', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.95 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['warn', { minScore: 0.9 }],
        // Specific audits
        'uses-optimized-images': 'warn',
        'uses-webp-images': 'warn',
        'render-blocking-resources': 'warn',
        'unused-javascript': ['warn', { maxLength: 0 }],
      },
    },
    upload: {
      target: 'temporary-public-storage', // or 'lhci' for persistent server
    },
  },
}
```

### GitHub Actions integration

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Next.js app
        run: npm run build
        env:
          NEXT_PUBLIC_SITE_URL: http://localhost:3000

      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

### Vercel integration (alternative — no CI needed)

```bash
# In Vercel project settings → Integrations → Lighthouse
# Or use the Vercel Speed Insights for field data
npm install @vercel/speed-insights
```

```tsx
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />  {/* sends real CWV to Vercel dashboard */}
      </body>
    </html>
  )
}
```

## Fixing common audit failures

### Performance

**Render-blocking resources**
```tsx
// PROBLEM: CSS loaded synchronously in <head>
<link rel="stylesheet" href="/styles.css" />

// FIX: Critical CSS inline (Next.js does this automatically)
// For third-party CSS, load async:
<link rel="preload" href="/non-critical.css" as="style" onLoad="this.onload=null;this.rel='stylesheet'" />
```

**Unused JavaScript**
```tsx
// DIAGNOSIS: Look at Lighthouse's "Reduce unused JavaScript" audit
// It shows which scripts are loaded but not executed in the first 2s

// FIX 1: Dynamic import for below-fold features
const HeavyComponent = dynamic(() => import('@/components/Heavy'))

// FIX 2: Remove unused package exports
// BAD: imports all of lodash
import _ from 'lodash'
// GOOD: import only what you use
import debounce from 'lodash/debounce'
```

**Properly size images**
```tsx
// PROBLEM: Serving 1920×1080 image in a 200×200 container
// FIX: Use sizes prop to tell browser what size image to download
<Image
  src="/photo.jpg"
  alt="Team photo"
  width={200}
  height={200}
  sizes="200px"  // ← tells browser this image is always 200px wide
/>
```

**Efficiently encode images**
Next.js handles this automatically with `next/image` — if this audit fails, it means you're using `<img>` somewhere. Find and replace with `<Image>`.

### Accessibility

**Missing alt text**
```tsx
// PROBLEM
<img src="/logo.png" />
<Image src="/logo.png" width={100} height={40} />

// FIX: meaningful alt for informative images
<Image src="/logo.png" alt="Acme Corp logo" width={100} height={40} />

// FIX: empty alt for decorative images (screen readers skip them)
<Image src="/decoration.png" alt="" width={50} height={50} />
```

**Color contrast failures**
```css
/* PROBLEM: low contrast */
.text-muted { color: #999; background: #fff; }  /* ratio: 2.85:1 — fails AA */

/* FIX: minimum 4.5:1 for normal text, 3:1 for large text */
.text-muted { color: #767676; background: #fff; }  /* ratio: 4.54:1 — passes AA */

/* Tailwind: use text-gray-600 (not text-gray-400) on white backgrounds */
```

**Buttons without accessible names**
```tsx
// PROBLEM
<button onClick={toggleMenu}>
  <MenuIcon />
</button>

// FIX
<button onClick={toggleMenu} aria-label="Abrir menú de navegación">
  <MenuIcon aria-hidden="true" />
</button>
```

**Form inputs without labels**
```tsx
// PROBLEM
<input type="email" placeholder="Tu email" />

// FIX option 1: explicit label
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// FIX option 2: aria-label if visual label isn't desired
<input type="email" aria-label="Dirección de email" placeholder="Tu email" />
```

**Focus not visible**
```css
/* PROBLEM: removes focus ring globally */
* { outline: none; }

/* FIX: only remove for mouse, keep for keyboard */
:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}
```

### SEO

**Missing meta description**
```tsx
// app/page.tsx
export const metadata = {
  description: '150-160 character description with the main keyword.',
}
```

**Links are not crawlable**
```tsx
// PROBLEM: click handlers on divs or spans
<div onClick={() => router.push('/about')}>About</div>

// FIX: use real anchor elements
<Link href="/about">About</Link>
```

**`hreflang` for multilingual sites**
```tsx
// app/[locale]/page.tsx
export async function generateMetadata({ params }) {
  return {
    alternates: {
      canonical: `https://tudominio.com/${params.locale}`,
      languages: {
        'es': 'https://tudominio.com/es',
        'en': 'https://tudominio.com/en',
        'x-default': 'https://tudominio.com/es',
      },
    },
  }
}
```

### Best Practices

**Does not use HTTPS** → handled by Vercel automatically.

**Mixed content** → all API calls must use `https://`. Check environment variables.

**Deprecated APIs** → check browser console for `chrome://deprecations`. Usually from third-party scripts.

## Interpreting scores — what to prioritize

| Score range | Meaning | Action |
|---|---|---|
| 90-100 | Good | Monitor, don't over-optimize |
| 75-89 | Needs improvement | Fix top 3 audits |
| < 75 | Poor | Systematic audit needed |

**Audit impact guide (Performance):**
1. Images: `priority` + `sizes` + use `next/image` → often +20-30 points
2. Unused JS: dynamic imports → often +10-20 points
3. Render-blocking: font self-hosting via next/font → +5-15 points
4. TBT: reduce main thread work → +5-15 points

## Output format

When given a Lighthouse score or specific audit failures:
1. **Diagnóstico**: what failed and why (2-3 lines)
2. **Fix prioritario**: the single biggest impact change, with code
3. **Fixes adicionales**: ordered by effort/impact
4. **Setup de LHCI**: if they don't have CI, suggest it
5. **Score esperado**: rough estimate of improvement
