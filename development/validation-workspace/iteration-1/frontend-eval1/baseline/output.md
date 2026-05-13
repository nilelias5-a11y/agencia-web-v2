# Hero Component — Café Vermut

**Stack:** Next.js 16 · React 19 · TypeScript strict · Tailwind v4 · Framer Motion  
**Pattern:** Server shell / Client leaf  
**Date:** 2026-05-13

---

## Architecture

```
app/
├── page.tsx                          ← Server Component (shell)
└── components/
    └── hero/
        ├── Hero.tsx                  ← Server Component (shell, public API)
        └── HeroClient.tsx            ← Client Component (leaf, all animation logic)
```

---

## File 1 — `app/components/hero/HeroClient.tsx`

```tsx
"use client";

import Image from "next/image";
import Link from "next/link";
import {
  motion,
  useReducedMotion,
  type Variants,
} from "framer-motion";

// ---------------------------------------------------------------------------
// Types
// ---------------------------------------------------------------------------

export interface HeroClientProps {
  imageSrc: string;
  imageAlt: string;
  eyebrow: string;
  headline: string;
  subheadline: string;
  ctaPrimary: { label: string; href: string };
  ctaSecondary: { label: string; href: string };
}

// ---------------------------------------------------------------------------
// Animation variants
// ---------------------------------------------------------------------------

const DURATION = 0.6;
const EASE = [0.22, 1, 0.36, 1] as const; // custom ease-out expo

function buildVariants(reduced: boolean): Variants {
  if (reduced) {
    // Respect prefers-reduced-motion: no movement, just instant visibility
    return {
      hidden: { opacity: 0 },
      visible: { opacity: 1, transition: { duration: 0 } },
    };
  }
  return {
    hidden: { opacity: 0, y: 24 },
    visible: (delay: number) => ({
      opacity: 1,
      y: 0,
      transition: {
        duration: DURATION,
        delay,
        ease: EASE,
      },
    }),
  };
}

function buildImageVariants(reduced: boolean): Variants {
  if (reduced) {
    return {
      hidden: { opacity: 0 },
      visible: { opacity: 1, transition: { duration: 0 } },
    };
  }
  return {
    hidden: { opacity: 0, scale: 1.04 },
    visible: {
      opacity: 1,
      scale: 1,
      transition: {
        duration: 1.2,
        delay: 0.2, // image starts early but finishes later
        ease: EASE,
      },
    },
  };
}

// ---------------------------------------------------------------------------
// Delays (seconds): eyebrow → headline → subheadline → CTAs → (image runs in parallel from 0.2)
// ---------------------------------------------------------------------------
const DELAYS = {
  eyebrow: 0.0,
  headline: 0.15,
  subheadline: 0.3,
  ctas: 0.45,
} as const;

// ---------------------------------------------------------------------------
// Component
// ---------------------------------------------------------------------------

export default function HeroClient({
  imageSrc,
  imageAlt,
  eyebrow,
  headline,
  subheadline,
  ctaPrimary,
  ctaSecondary,
}: HeroClientProps) {
  const shouldReduceMotion = useReducedMotion();
  const contentVariants = buildVariants(shouldReduceMotion ?? false);
  const imageVariants = buildImageVariants(shouldReduceMotion ?? false);

  return (
    <section
      aria-label="Hero"
      className="relative flex h-svh min-h-[600px] w-full items-center justify-center overflow-hidden"
    >
      {/* ------------------------------------------------------------------ */}
      {/* Background image                                                    */}
      {/* ------------------------------------------------------------------ */}
      <motion.div
        className="absolute inset-0 -z-10"
        variants={imageVariants}
        initial="hidden"
        animate="visible"
      >
        <Image
          src={imageSrc}
          alt={imageAlt}
          fill
          priority
          sizes="100vw"
          className="object-cover object-center"
          quality={90}
        />
        {/* Gradient overlay for text legibility */}
        <div
          aria-hidden="true"
          className="absolute inset-0 bg-gradient-to-b from-black/60 via-black/40 to-black/70"
        />
      </motion.div>

      {/* ------------------------------------------------------------------ */}
      {/* Content                                                             */}
      {/* ------------------------------------------------------------------ */}
      <div className="relative z-10 mx-auto max-w-4xl px-6 text-center text-white">
        {/* Eyebrow */}
        <motion.p
          className="mb-4 text-sm font-semibold uppercase tracking-[0.2em] text-amber-300"
          variants={contentVariants}
          custom={DELAYS.eyebrow}
          initial="hidden"
          animate="visible"
        >
          {eyebrow}
        </motion.p>

        {/* Headline */}
        <motion.h1
          className="mb-6 text-4xl font-bold leading-tight tracking-tight text-white sm:text-5xl md:text-6xl lg:text-7xl"
          variants={contentVariants}
          custom={DELAYS.headline}
          initial="hidden"
          animate="visible"
        >
          {headline}
        </motion.h1>

        {/* Subheadline (2 lines via max-width + balanced wrapping) */}
        <motion.p
          className="mx-auto mb-10 max-w-2xl text-balance text-lg leading-relaxed text-white/80 sm:text-xl"
          variants={contentVariants}
          custom={DELAYS.subheadline}
          initial="hidden"
          animate="visible"
        >
          Descubre granos de origen único, tostados con precisión y servidos
          con pasión.{" "}
          <span className="whitespace-nowrap">
            Porque el buen café no necesita explicaciones.
          </span>
        </motion.p>

        {/* CTAs */}
        <motion.div
          className="flex flex-col items-center justify-center gap-4 sm:flex-row"
          variants={contentVariants}
          custom={DELAYS.ctas}
          initial="hidden"
          animate="visible"
        >
          {/* Primary CTA */}
          <Link
            href={ctaPrimary.href}
            className={[
              "inline-flex items-center justify-center",
              "rounded-full bg-amber-400 px-8 py-3.5",
              "text-sm font-semibold text-neutral-900",
              "shadow-lg shadow-amber-500/30",
              "transition-all duration-200",
              "hover:bg-amber-300 hover:shadow-amber-400/40",
              "focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-amber-300",
              "active:scale-[0.97]",
            ].join(" ")}
          >
            {ctaPrimary.label}
          </Link>

          {/* Secondary CTA */}
          <Link
            href={ctaSecondary.href}
            className={[
              "inline-flex items-center justify-center",
              "rounded-full border border-white/40 bg-white/10 px-8 py-3.5",
              "text-sm font-semibold text-white backdrop-blur-sm",
              "transition-all duration-200",
              "hover:bg-white/20 hover:border-white/60",
              "focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-white",
              "active:scale-[0.97]",
            ].join(" ")}
          >
            {ctaSecondary.label}
          </Link>
        </motion.div>
      </div>

      {/* ------------------------------------------------------------------ */}
      {/* Scroll indicator                                                    */}
      {/* ------------------------------------------------------------------ */}
      <motion.div
        aria-hidden="true"
        className="absolute bottom-8 left-1/2 -translate-x-1/2"
        initial={{ opacity: 0, y: -8 }}
        animate={
          shouldReduceMotion
            ? { opacity: 0.6 }
            : {
                opacity: [0, 0.6, 0.6, 0],
                y: [0, 8, 8, 0],
              }
        }
        transition={
          shouldReduceMotion
            ? { delay: 1.2, duration: 0 }
            : { delay: 1.2, duration: 2, repeat: Infinity, ease: "easeInOut" }
        }
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          strokeWidth="2"
          strokeLinecap="round"
          strokeLinejoin="round"
          className="text-white/70"
        >
          <path d="M12 5v14M5 12l7 7 7-7" />
        </svg>
      </motion.div>
    </section>
  );
}
```

---

## File 2 — `app/components/hero/Hero.tsx`

```tsx
import type { HeroClientProps } from "./HeroClient";
import HeroClient from "./HeroClient";

// ---------------------------------------------------------------------------
// Default props — override via page-level data fetching if needed
// ---------------------------------------------------------------------------

const DEFAULT_PROPS: HeroClientProps = {
  imageSrc: "/images/hero-cafe-vermut.jpg",
  imageAlt:
    "Interior del Café Vermut: tazas de café de especialidad sobre una barra de madera cálida en Gràcia, Barcelona",
  eyebrow: "Café de Especialidad · Barcelona",
  headline: "El café que cambia cómo ves el café",
  subheadline:
    "Descubre granos de origen único, tostados con precisión y servidos con pasión. Porque el buen café no necesita explicaciones.",
  ctaPrimary: {
    label: "Descubre nuestros cafés",
    href: "/menu",
  },
  ctaSecondary: {
    label: "Visítanos",
    href: "/visita",
  },
};

// ---------------------------------------------------------------------------
// Server Component shell — accepts optional prop overrides
// ---------------------------------------------------------------------------

interface HeroProps extends Partial<HeroClientProps> {}

export default function Hero(overrides: HeroProps = {}) {
  const props: HeroClientProps = { ...DEFAULT_PROPS, ...overrides };

  // The Server Component does no rendering beyond delegating to the
  // Client leaf. This keeps RSC boundaries clean and allows server-side
  // data fetching to be wired here without touching animation logic.
  return <HeroClient {...props} />;
}
```

---

## File 3 — `app/page.tsx` (homepage, Server Component)

```tsx
import Hero from "@/components/hero/Hero";

// This is a React Server Component.
// Add async data fetching here when needed (e.g., CMS hero data).

export default function HomePage() {
  return (
    <main>
      <Hero />
      {/* Additional homepage sections go here */}
    </main>
  );
}
```

---

## Notes on decisions

### Server shell / Client leaf pattern

- `Hero.tsx` is a **Server Component** (no `"use client"` directive). It owns
  default props, could `await` CMS data, and delegates rendering to the leaf.
- `HeroClient.tsx` is a **Client Component** (`"use client"` at the top). It
  holds all Framer Motion logic, `useReducedMotion`, and interactive elements.
- The boundary is explicit: the server never imports `framer-motion`; the
  client never does async I/O.

### Orchestrated entrance animation (5 elements)

| # | Element       | Delay   | Effect                |
|---|---------------|---------|-----------------------|
| 1 | Eyebrow       | 0.00 s  | fade + slide up 24 px |
| 2 | Headline      | 0.15 s  | fade + slide up 24 px |
| 3 | Subheadline   | 0.30 s  | fade + slide up 24 px |
| 4 | CTAs (group)  | 0.45 s  | fade + slide up 24 px |
| 5 | Hero image    | 0.20 s  | fade + scale 1.04→1   |

The image runs in parallel with a gentle scale-in to feel cinematic while
text arrives staggered on top.

### `prefers-reduced-motion`

`useReducedMotion()` from Framer Motion reads the OS/browser media query at
runtime. When `true`:
- All `y` / `scale` transforms are removed.
- `duration` is set to `0` (instant opacity change — respects user intent
  without a jarring pop by keeping `opacity` transitions conceptually present).
- The scroll indicator loop is replaced with a static fade-in.

### `next/image` with `priority`

`priority` is set on the hero image so Next.js emits a `<link rel="preload">`
in `<head>`, eliminating LCP delay. `fill` + `sizes="100vw"` ensures
responsive srcset generation. `quality={90}` balances fidelity vs. transfer
size for a hero photograph.

### Tailwind v4 notes

- Uses `h-svh` (small viewport height unit) instead of `h-screen` to handle
  mobile browser chrome correctly.
- `text-balance` on the subheadline uses the CSS `text-wrap: balance` property,
  natively available in modern browsers and forwarded by Tailwind v4.
- No `@apply` — all utility classes are inlined per Tailwind v4 conventions.

### TypeScript strict

- All props are typed via explicit interfaces.
- `useReducedMotion()` returns `boolean | null`; the `?? false` coercion is
  explicit.
- `as const` on the easing array ensures the tuple type is inferred correctly
  by Framer Motion's `Easing` type.

### Required dependencies

```jsonc
// package.json (relevant additions)
{
  "dependencies": {
    "framer-motion": "^11.0.0",   // React 19 compatible
    "next": "^16.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  }
}
```

### Recommended `next.config.ts` image domain

```ts
import type { NextConfig } from "next";

const config: NextConfig = {
  images: {
    // Add remote patterns here if imageSrc is a remote URL
    // remotePatterns: [{ protocol: "https", hostname: "cdn.cafevermut.com" }],
  },
};

export default config;
```
