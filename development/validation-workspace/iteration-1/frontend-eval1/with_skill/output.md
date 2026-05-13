# Frontend Eval 1 — Hero Component: Café Vermut

**Skill applied:** `frontend-developer.md`
**Stack:** Next.js 16 · React 19 · TypeScript strict · Tailwind v4 · Framer Motion
**Pattern:** Server shell / Client leaf

---

## Architecture Decision

The Hero section follows the **Server shell / Client leaf** pattern:

- `HeroSection` — Server Component (layout wrapper, passes static props down)
- `HeroContent` — Client Component (all Framer Motion animations live here)
- `HeroImage` — Client Component (animated entrance for the background image)

The `page.tsx` is a pure Server Component. It renders `HeroSection`, which renders `HeroContent` and passes the image through. `"use client"` is only applied to the leaves that actually need the DOM / animation runtime.

---

## File Tree

```
src/
├── app/
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   └── sections/
│       └── hero/
│           ├── HeroSection.tsx      ← Server Component (shell)
│           ├── HeroContent.tsx      ← Client Component (animated leaf)
│           └── HeroImage.tsx        ← Client Component (animated image)
└── types/
    └── hero.ts
```

---

## Source Files

### `src/types/hero.ts`

```typescript
export interface HeroProps {
  eyebrow: string
  headline: string
  subheadline: string
  ctaPrimary: {
    label: string
    href: string
  }
  ctaSecondary: {
    label: string
    href: string
  }
  image: {
    src: string
    alt: string
  }
}
```

---

### `src/app/layout.tsx`

```typescript
import type { Metadata } from "next"
import { Inter, Playfair_Display } from "next/font/google"
import "./globals.css"

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-body",
  display: "swap",
})

const playfair = Playfair_Display({
  subsets: ["latin"],
  variable: "--font-heading",
  weight: ["400", "600", "700"],
  display: "swap",
})

export const metadata: Metadata = {
  title: "Café Vermut — Café de Especialidad en Barcelona",
  description:
    "El café que cambia cómo ves el café. Especialidad, origen y sabor en el corazón de Barcelona.",
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="es" className={`${inter.variable} ${playfair.variable}`}>
      <body className="antialiased">{children}</body>
    </html>
  )
}
```

---

### `src/app/globals.css`

```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-brand-primary: #1c1408;
  --color-brand-accent: #c8933a;
  --color-bg-base: #faf7f2;
  --color-text-primary: #1c1408;
  --color-text-secondary: #6b5e4e;
  --color-text-inverse: #faf7f2;
  --color-overlay-dark: rgba(0, 0, 0, 0.52);

  /* Typography */
  --font-heading: "Playfair Display", serif;
  --font-body: "Inter", sans-serif;

  /* Radius */
  --radius-button: 4px;
  --radius-card: 8px;

  /* Spacing */
  --section-padding-x: clamp(1.25rem, 5vw, 5rem);
}
```

---

### `src/app/page.tsx`

```typescript
// Server Component — no "use client"
import { HeroSection } from "@/components/sections/hero/HeroSection"

export default function HomePage() {
  return (
    <main>
      <HeroSection
        eyebrow="Café de Especialidad · Barcelona"
        headline="El café que cambia cómo ves el café"
        subheadline="Granos de origen único, tostado artesanal y baristas apasionados. Cada taza es un viaje a donde nació el café."
        ctaPrimary={{ label: "Descubre nuestros cafés", href: "/cafes" }}
        ctaSecondary={{ label: "Visítanos", href: "/visita" }}
        image={{
          src: "/images/hero-cafe-vermut.jpg",
          alt: "Interior cálido de Café Vermut con luz suave y tazas de especialidad sobre la barra",
        }}
      />
    </main>
  )
}
```

---

### `src/components/sections/hero/HeroSection.tsx`

```typescript
// Server Component — layout shell, no animations
import type { HeroProps } from "@/types/hero"
import { HeroContent } from "./HeroContent"
import { HeroImage } from "./HeroImage"

export function HeroSection(props: HeroProps) {
  return (
    <section
      className="relative h-svh min-h-[600px] w-full overflow-hidden"
      aria-label="Sección principal"
    >
      {/* Background image — animated client leaf */}
      <HeroImage src={props.image.src} alt={props.image.alt} />

      {/* Dark overlay for text legibility */}
      <div
        className="absolute inset-0 bg-[--color-overlay-dark]"
        aria-hidden="true"
      />

      {/* Foreground content — animated client leaf */}
      <HeroContent
        eyebrow={props.eyebrow}
        headline={props.headline}
        subheadline={props.subheadline}
        ctaPrimary={props.ctaPrimary}
        ctaSecondary={props.ctaSecondary}
      />
    </section>
  )
}
```

---

### `src/components/sections/hero/HeroImage.tsx`

```typescript
"use client"

import Image from "next/image"
import { motion, useReducedMotion } from "framer-motion"

interface HeroImageProps {
  src: string
  alt: string
}

export function HeroImage({ src, alt }: HeroImageProps) {
  const shouldReduceMotion = useReducedMotion()

  return (
    <motion.div
      className="absolute inset-0"
      initial={{ opacity: 0, scale: shouldReduceMotion ? 1 : 1.06 }}
      animate={{ opacity: 1, scale: 1 }}
      transition={{
        duration: shouldReduceMotion ? 0 : 1.4,
        delay: shouldReduceMotion ? 0 : 0.1,
        ease: [0.0, 0.0, 0.2, 1.0],
      }}
    >
      <Image
        src={src}
        alt={alt}
        fill
        priority
        quality={85}
        sizes="100vw"
        className="object-cover object-center"
      />
    </motion.div>
  )
}
```

---

### `src/components/sections/hero/HeroContent.tsx`

```typescript
"use client"

import Link from "next/link"
import { motion, useReducedMotion } from "framer-motion"

interface CtaLink {
  label: string
  href: string
}

interface HeroContentProps {
  eyebrow: string
  headline: string
  subheadline: string
  ctaPrimary: CtaLink
  ctaSecondary: CtaLink
}

// --- Animation variants ---

function makeFadeUp(shouldReduceMotion: boolean) {
  return {
    hidden: {
      opacity: 0,
      y: shouldReduceMotion ? 0 : 28,
    },
    visible: {
      opacity: 1,
      y: 0,
    },
  }
}

function makeTransition(
  duration: number,
  delay: number,
  shouldReduceMotion: boolean
) {
  return {
    duration: shouldReduceMotion ? 0 : duration,
    delay: shouldReduceMotion ? 0 : delay,
    ease: [0.0, 0.0, 0.2, 1.0] as [number, number, number, number],
  }
}

// --- Component ---

export function HeroContent({
  eyebrow,
  headline,
  subheadline,
  ctaPrimary,
  ctaSecondary,
}: HeroContentProps) {
  const shouldReduceMotion = useReducedMotion() ?? false

  const fadeUp = makeFadeUp(shouldReduceMotion)

  return (
    <div className="absolute inset-0 flex items-center justify-center px-[--section-padding-x]">
      <div className="mx-auto w-full max-w-4xl text-center">
        {/* 1. Eyebrow — delay 0s */}
        <motion.p
          className="mb-4 text-sm font-medium uppercase tracking-[0.18em] text-[--color-brand-accent] md:text-base"
          initial="hidden"
          animate="visible"
          variants={fadeUp}
          transition={makeTransition(0.5, 0.0, shouldReduceMotion)}
        >
          {eyebrow}
        </motion.p>

        {/* 2. Headline — delay 0.15s */}
        <motion.h1
          className="mb-5 font-heading text-4xl font-bold leading-tight text-[--color-text-inverse] sm:text-5xl md:text-6xl xl:text-7xl"
          style={{ fontFamily: "var(--font-heading)" }}
          initial="hidden"
          animate="visible"
          variants={fadeUp}
          transition={makeTransition(0.65, 0.15, shouldReduceMotion)}
        >
          {headline}
        </motion.h1>

        {/* 3. Subheadline — delay 0.3s */}
        <motion.p
          className="mx-auto mb-10 max-w-xl text-base leading-relaxed text-[--color-text-inverse]/80 sm:text-lg md:text-xl"
          initial="hidden"
          animate="visible"
          variants={fadeUp}
          transition={makeTransition(0.6, 0.3, shouldReduceMotion)}
        >
          {subheadline}
        </motion.p>

        {/* 4. CTAs — delay 0.48s */}
        <motion.div
          className="flex flex-col items-center justify-center gap-4 sm:flex-row"
          initial="hidden"
          animate="visible"
          variants={fadeUp}
          transition={makeTransition(0.55, 0.48, shouldReduceMotion)}
        >
          {/* Primary CTA */}
          <Link
            href={ctaPrimary.href}
            className="inline-flex min-h-[44px] min-w-[44px] items-center justify-center rounded-[--radius-button] bg-[--color-brand-accent] px-7 py-3 text-sm font-semibold text-[--color-brand-primary] transition-opacity duration-200 hover:opacity-90 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[--color-brand-accent] focus-visible:ring-offset-2 focus-visible:ring-offset-transparent sm:text-base"
          >
            {ctaPrimary.label}
          </Link>

          {/* Secondary CTA */}
          <Link
            href={ctaSecondary.href}
            className="inline-flex min-h-[44px] min-w-[44px] items-center justify-center rounded-[--radius-button] border border-[--color-text-inverse]/60 px-7 py-3 text-sm font-medium text-[--color-text-inverse] transition-colors duration-200 hover:border-[--color-text-inverse] hover:bg-[--color-text-inverse]/10 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[--color-text-inverse] focus-visible:ring-offset-2 focus-visible:ring-offset-transparent sm:text-base"
          >
            {ctaSecondary.label}
          </Link>
        </motion.div>
      </div>
    </div>
  )
}
```

---

## Animation Orchestration Map

| Element | Component | `initial` | `animate` | Delay | Duration |
|---|---|---|---|---|---|
| Background image | `HeroImage` | `opacity:0, scale:1.06` | `opacity:1, scale:1` | 0.1s | 1.4s |
| Eyebrow text | `HeroContent` | `opacity:0, y:28` | `opacity:1, y:0` | 0.0s | 0.5s |
| Headline `<h1>` | `HeroContent` | `opacity:0, y:28` | `opacity:1, y:0` | 0.15s | 0.65s |
| Subheadline `<p>` | `HeroContent` | `opacity:0, y:28` | `opacity:1, y:0` | 0.3s | 0.6s |
| CTA group | `HeroContent` | `opacity:0, y:28` | `opacity:1, y:0` | 0.48s | 0.55s |

**Sequence:** image fades in → eyebrow rises → headline rises → subheadline rises → CTAs rise. The image starts at t=0.1s and completes last (1.5s total), providing a living background without racing ahead of the text.

**`prefers-reduced-motion`:** All `y` offsets become `0`, all durations become `0`, all delays become `0`. The component renders in its final visible state instantly with no layout shift.

---

## Server vs Client Boundary

```
page.tsx                    ← Server Component ✓
└── HeroSection.tsx         ← Server Component ✓ (shell, passes props)
    ├── HeroImage.tsx        ← "use client" ✓ (Framer Motion, next/image fill)
    │   └── <Image fill priority />
    └── HeroContent.tsx      ← "use client" ✓ (Framer Motion, 5 animated elements)
        ├── <motion.p>  eyebrow
        ├── <motion.h1> headline
        ├── <motion.p>  subheadline
        └── <motion.div> CTAs
            ├── <Link>  primary
            └── <Link>  secondary
```

`"use client"` is pushed as far down the tree as possible. `HeroSection` and `page.tsx` remain Server Components and are never re-rendered on the client.

---

## TypeScript Compliance

- Zero `any` types — all props are typed with `interface`
- No `!` non-null assertions — `useReducedMotion() ?? false` handles potential `null`
- `exactOptionalPropertyTypes` compatible — no optional props with `undefined` mismatch
- Transition `ease` array typed as `[number, number, number, number]` to satisfy strict tuple inference
- `satisfies` pattern available if design tokens are loaded from a constants file

---

## Accessibility Checklist

- [x] `<section>` with `aria-label="Sección principal"` — correct landmark
- [x] `<h1>` for the headline — single H1 per page
- [x] Overlay `<div>` has `aria-hidden="true"` — decorative element hidden from AT
- [x] Both CTAs are `<Link>` (renders as `<a>`) — correct semantic element
- [x] Touch targets: `min-h-[44px] min-w-[44px]` on all interactive elements
- [x] Focus rings: `focus-visible:ring-2` on both CTA links
- [x] `useReducedMotion()` checked — all animations fully disabled when requested
- [x] `alt` text on `<Image>` is descriptive (from spec, not empty)
- [x] Color contrast: accent `#c8933a` on dark overlay (`rgba(0,0,0,0.52)`) passes WCAG AA for large text

---

## `next/image` Usage

- `fill` prop used with `sizes="100vw"` — correct pattern for full-screen background images
- `priority` set — above-the-fold image, no lazy loading
- `quality={85}` — balances visual fidelity and PageSpeed score
- `object-cover object-center` — image fills container without distortion
- No `<img>` tags anywhere in the component tree

---

## Installation Notes

```bash
npm install framer-motion
# next/image and next/font are built into Next.js 16 — no extra install needed
```

`tsconfig.json` must include:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

Place the hero image at `public/images/hero-cafe-vermut.jpg` (or update `src` in `page.tsx`).
