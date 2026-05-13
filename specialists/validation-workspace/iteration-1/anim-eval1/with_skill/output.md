# La Nonna — Scroll Animation System (Framer Motion)

**Skill applied:** animation-premium  
**Standard:** Four Laws of Premium Motion  
**Brand feel:** Warm, Mediterranean, family-run Italian. Subtle and inviting — motion that serves, never shouts.

---

## Motion Design Decisions (per section)

| Section | Pattern | Rationale |
|---|---|---|
| Hero | Already exists — integrate with page load sequence via `staggerChildren` | Entry point: atmosphere, not distraction |
| Menu destacado | Staggered card fade-up | 3 cards reveal in sequence — eye tracks left→right naturally |
| Historia de la familia | Split entrance: image slides from left, text fades from right | Mirrors 2-column layout; creates directional tension |
| Testimonios | Staggered cards with subtle scale | Quotes feel "presented" one at a time |
| Reservas CTA | Single bold fade-up with slight scale | Full-width banner needs weight — one clean motion |
| Footer | Gentle fade-in, no transform | Footer is utility — motion should be almost invisible |

---

## Easing Reference

```typescript
// The agency standard curve — exponential deceleration, premium feel
export const EASE_OUT_EXPO = [0.16, 1, 0.3, 1] as const

// Softer curve for large elements (banner, full-width blocks)
export const EASE_OUT_QUART = [0.25, 1, 0.5, 1] as const

// Micro-interactions (hover feedback, focus states)
export const EASE_STANDARD = [0.4, 0, 0.2, 1] as const
```

---

## File 1: `lib/animation/variants.ts`

Animation variants for every section. All variants respect a single source of truth.

```typescript
// lib/animation/variants.ts
import type { Variants } from "framer-motion"

// ---------------------------------------------------------------------------
// Shared easing curves
// ---------------------------------------------------------------------------
export const EASE_OUT_EXPO = [0.16, 1, 0.3, 1] as const
export const EASE_OUT_QUART = [0.25, 1, 0.5, 1] as const
export const EASE_STANDARD  = [0.4, 0, 0.2, 1] as const

// ---------------------------------------------------------------------------
// Reusable base variant — fade up (agency standard)
// Used as building block; sections customise duration/distance
// ---------------------------------------------------------------------------
export function makeFadeUp(
  yOffset = 28,
  duration = 0.65,
  ease: number[] = [...EASE_OUT_EXPO]
): Variants {
  return {
    hidden: { opacity: 0, y: yOffset },
    visible: {
      opacity: 1,
      y: 0,
      transition: { duration, ease },
    },
  }
}

// ---------------------------------------------------------------------------
// Container variant — orchestrates stagger for child elements
// ---------------------------------------------------------------------------
export function makeStaggerContainer(
  staggerChildren = 0.12,
  delayChildren = 0
): Variants {
  return {
    hidden: {},
    visible: {
      transition: { staggerChildren, delayChildren },
    },
  }
}

// ---------------------------------------------------------------------------
// Section: Menu destacado
// 3 dish cards, stagger 0.14s, moderate fade-up
// ---------------------------------------------------------------------------
export const menuCardVariants: Variants = makeFadeUp(32, 0.7, [...EASE_OUT_EXPO])

export const menuContainerVariants: Variants = makeStaggerContainer(0.14)

// ---------------------------------------------------------------------------
// Section: Historia de la familia (2-column split)
// ---------------------------------------------------------------------------
export const historiaImageVariants: Variants = {
  hidden: { opacity: 0, x: -40 },
  visible: {
    opacity: 1,
    x: 0,
    transition: { duration: 0.8, ease: EASE_OUT_EXPO },
  },
}

export const historiaTextVariants: Variants = {
  hidden: { opacity: 0, x: 40 },
  visible: {
    opacity: 1,
    x: 0,
    transition: { duration: 0.8, ease: EASE_OUT_EXPO, delay: 0.15 },
  },
}

// Container for the historia text block — staggers internal paragraphs
export const historiaTextBlockVariants: Variants = makeStaggerContainer(0.10, 0.25)

export const historiaParagraphVariants: Variants = makeFadeUp(16, 0.55, [...EASE_OUT_EXPO])

// ---------------------------------------------------------------------------
// Section: Testimonios
// 3 quote cards — stagger + subtle scale for "presented" feel
// ---------------------------------------------------------------------------
export const testimonioCardVariants: Variants = {
  hidden: { opacity: 0, y: 24, scale: 0.97 },
  visible: {
    opacity: 1,
    y: 0,
    scale: 1,
    transition: { duration: 0.65, ease: EASE_OUT_EXPO },
  },
}

export const testimonioContainerVariants: Variants = makeStaggerContainer(0.16)

// ---------------------------------------------------------------------------
// Section: Reservas CTA (full-width banner)
// Single clean motion — headline first, then button
// ---------------------------------------------------------------------------
export const ctaBannerVariants: Variants = {
  hidden: { opacity: 0, y: 20 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.7, ease: EASE_OUT_QUART },
  },
}

export const ctaHeadlineVariants: Variants = {
  hidden: { opacity: 0, y: 18 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.65, ease: EASE_OUT_EXPO },
  },
}

export const ctaButtonVariants: Variants = {
  hidden: { opacity: 0, y: 12 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.55, ease: EASE_OUT_EXPO, delay: 0.18 },
  },
}

export const ctaContainerVariants: Variants = makeStaggerContainer(0.18, 0)

// ---------------------------------------------------------------------------
// Section: Footer
// Nearly invisible fade — utility section, motion is background noise
// ---------------------------------------------------------------------------
export const footerVariants: Variants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { duration: 0.5, ease: EASE_STANDARD },
  },
}
```

---

## File 2: `lib/animation/useReducedMotion.ts`

Single hook — consumed everywhere. Never duplicate this check.

```typescript
// lib/animation/useReducedMotion.ts
"use client"

import { useReducedMotion as useFramerReducedMotion } from "framer-motion"

/**
 * Returns true if the user has requested reduced motion.
 * Wraps Framer Motion's built-in hook for consistency.
 * When true, all animated components must show final state immediately.
 */
export function useReducedMotion(): boolean {
  // Framer Motion's hook reads the OS/browser prefers-reduced-motion media query
  // and re-renders if the user changes it at runtime.
  return useFramerReducedMotion() ?? false
}
```

---

## File 3: `components/animation/ScrollReveal.tsx`

The core reusable primitive. Every section wraps its content in this.

```typescript
// components/animation/ScrollReveal.tsx
"use client"

import { useRef } from "react"
import { motion, useInView } from "framer-motion"
import type { Variants, HTMLMotionProps } from "framer-motion"
import { useReducedMotion } from "@/lib/animation/useReducedMotion"

interface ScrollRevealProps extends HTMLMotionProps<"div"> {
  /**
   * Framer Motion variants. Must define "hidden" and "visible" keys.
   * Defaults to a standard fade-up if omitted.
   */
  variants?: Variants
  /**
   * Fraction of element visible before animation triggers.
   * 0.15 is the sweet spot for most sections — triggers before
   * the element is fully in view so motion feels natural, not late.
   */
  threshold?: number
  /**
   * Once: true = animates only on first scroll-into-view (recommended for
   * content sections). False = resets and replays on scroll-out (avoid for
   * content — it feels janky on scroll-back).
   */
  once?: boolean
  /** Extra delay in seconds before the animation starts. */
  delay?: number
  children: React.ReactNode
  className?: string
}

const defaultVariants: Variants = {
  hidden: { opacity: 0, y: 28 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.65, ease: [0.16, 1, 0.3, 1] },
  },
}

export function ScrollReveal({
  variants = defaultVariants,
  threshold = 0.15,
  once = true,
  delay = 0,
  children,
  className,
  ...rest
}: ScrollRevealProps) {
  const ref = useRef<HTMLDivElement>(null)
  const prefersReduced = useReducedMotion()

  // useInView from Framer Motion — backed by IntersectionObserver
  // `amount` maps directly to IntersectionObserver threshold
  // `once` prevents replay on scroll-back
  const isInView = useInView(ref, {
    amount: threshold,
    once,
  })

  // Accessibility: when reduced motion is preferred, skip to final state
  if (prefersReduced) {
    return (
      <div ref={ref} className={className}>
        {children}
      </div>
    )
  }

  return (
    <motion.div
      ref={ref}
      className={className}
      variants={variants}
      initial="hidden"
      animate={isInView ? "visible" : "hidden"}
      // Per-instance delay override (applied via custom prop, not variant)
      // We inject it here so callers don't need to clone variants
      transition={delay > 0 ? { delay } : undefined}
      {...rest}
    >
      {children}
    </motion.div>
  )
}
```

---

## File 4: `components/animation/ScrollRevealGroup.tsx`

Orchestrates stagger for groups of children (cards, lists).

```typescript
// components/animation/ScrollRevealGroup.tsx
"use client"

import { useRef } from "react"
import { motion, useInView } from "framer-motion"
import type { Variants } from "framer-motion"
import { useReducedMotion } from "@/lib/animation/useReducedMotion"

interface ScrollRevealGroupProps {
  /** Container variants (must define staggerChildren in transition) */
  containerVariants: Variants
  /** Shared variants applied to each direct child */
  itemVariants: Variants
  /** Children must be renderable as motion.* elements */
  children: React.ReactNode[]
  /** Tag for the wrapper element */
  as?: keyof JSX.IntrinsicElements
  className?: string
  threshold?: number
  once?: boolean
}

export function ScrollRevealGroup({
  containerVariants,
  itemVariants,
  children,
  as: Tag = "div",
  className,
  threshold = 0.10,
  once = true,
}: ScrollRevealGroupProps) {
  const ref = useRef<HTMLDivElement>(null)
  const prefersReduced = useReducedMotion()
  const isInView = useInView(ref, { amount: threshold, once })

  // Accessibility: render without animation
  if (prefersReduced) {
    return (
      // @ts-expect-error — dynamic tag typing
      <Tag ref={ref} className={className}>
        {children}
      </Tag>
    )
  }

  const MotionTag = motion[Tag as "div"] ?? motion.div

  return (
    <MotionTag
      ref={ref}
      className={className}
      variants={containerVariants}
      initial="hidden"
      animate={isInView ? "visible" : "hidden"}
    >
      {(children as React.ReactNode[]).map((child, i) => (
        <motion.div key={i} variants={itemVariants}>
          {child}
        </motion.div>
      ))}
    </MotionTag>
  )
}
```

---

## File 5: Per-section implementation

### 5a. `MenuDestacado` section

```tsx
// components/sections/MenuDestacado.tsx
"use client"

import Image from "next/image"
import { ScrollRevealGroup } from "@/components/animation/ScrollRevealGroup"
import { ScrollReveal } from "@/components/animation/ScrollReveal"
import {
  menuCardVariants,
  menuContainerVariants,
  makeFadeUp,
  EASE_OUT_EXPO,
} from "@/lib/animation/variants"

const sectionTitleVariants = makeFadeUp(20, 0.6, [...EASE_OUT_EXPO])

interface Dish {
  name: string
  price: string
  image: string
  description: string
}

interface MenuDestacadoProps {
  dishes: Dish[]
}

export function MenuDestacado({ dishes }: MenuDestacadoProps) {
  return (
    <section className="menu-destacado">
      {/* Section title animates independently before cards */}
      <ScrollReveal variants={sectionTitleVariants} threshold={0.2}>
        <h2 className="section-title">Nuestra cocina</h2>
        <p className="section-subtitle">Los favoritos de siempre</p>
      </ScrollReveal>

      {/* Cards stagger in sequence */}
      <ScrollRevealGroup
        containerVariants={menuContainerVariants}
        itemVariants={menuCardVariants}
        as="ul"
        className="menu-grid"
        threshold={0.10}
      >
        {dishes.map((dish) => (
          // Each dish card — the motion.div wrapper is injected by ScrollRevealGroup
          <li key={dish.name} className="dish-card">
            <div className="dish-card__image-wrap">
              <Image
                src={dish.image}
                alt={dish.name}
                fill
                className="dish-card__image"
              />
            </div>
            <div className="dish-card__body">
              <h3 className="dish-card__name">{dish.name}</h3>
              <p className="dish-card__desc">{dish.description}</p>
              <span className="dish-card__price">{dish.price}</span>
            </div>
          </li>
        ))}
      </ScrollRevealGroup>
    </section>
  )
}
```

### 5b. `HistoriaFamilia` section

```tsx
// components/sections/HistoriaFamilia.tsx
"use client"

import { useRef } from "react"
import Image from "next/image"
import { motion, useInView } from "framer-motion"
import { useReducedMotion } from "@/lib/animation/useReducedMotion"
import {
  historiaImageVariants,
  historiaTextVariants,
  historiaTextBlockVariants,
  historiaParagraphVariants,
} from "@/lib/animation/variants"

interface HistoriaFamiliaProps {
  image: string
  paragraphs: string[]
}

export function HistoriaFamilia({ image, paragraphs }: HistoriaFamiliaProps) {
  const sectionRef = useRef<HTMLElement>(null)
  const prefersReduced = useReducedMotion()

  // Single IntersectionObserver for the whole section
  // threshold: 0.12 — triggers when ~12% of section is visible
  const isInView = useInView(sectionRef, { amount: 0.12, once: true })

  if (prefersReduced) {
    return (
      <section ref={sectionRef} className="historia">
        <div className="historia__image-col">
          <Image src={image} alt="La famiglia Conti" fill className="historia__image" />
        </div>
        <div className="historia__text-col">
          {paragraphs.map((p, i) => (
            <p key={i}>{p}</p>
          ))}
        </div>
      </section>
    )
  }

  return (
    <section ref={sectionRef} className="historia">
      {/* Image slides in from left */}
      <motion.div
        className="historia__image-col"
        variants={historiaImageVariants}
        initial="hidden"
        animate={isInView ? "visible" : "hidden"}
      >
        <Image src={image} alt="La famiglia Conti" fill className="historia__image" />
      </motion.div>

      {/* Text block fades from right; paragraphs stagger internally */}
      <motion.div
        className="historia__text-col"
        variants={historiaTextVariants}
        initial="hidden"
        animate={isInView ? "visible" : "hidden"}
      >
        <motion.div
          variants={historiaTextBlockVariants}
          initial="hidden"
          animate={isInView ? "visible" : "hidden"}
        >
          {paragraphs.map((p, i) => (
            <motion.p key={i} variants={historiaParagraphVariants}>
              {p}
            </motion.p>
          ))}
        </motion.div>
      </motion.div>
    </section>
  )
}
```

### 5c. `Testimonios` section

```tsx
// components/sections/Testimonios.tsx
"use client"

import { ScrollRevealGroup } from "@/components/animation/ScrollRevealGroup"
import { ScrollReveal } from "@/components/animation/ScrollReveal"
import {
  testimonioCardVariants,
  testimonioContainerVariants,
  makeFadeUp,
  EASE_OUT_EXPO,
} from "@/lib/animation/variants"

const sectionTitleVariants = makeFadeUp(18, 0.6, [...EASE_OUT_EXPO])

interface Testimonio {
  quote: string
  author: string
  location: string
}

interface TestimoniosProps {
  items: Testimonio[]
}

export function Testimonios({ items }: TestimoniosProps) {
  return (
    <section className="testimonios">
      <ScrollReveal variants={sectionTitleVariants} threshold={0.2}>
        <h2 className="section-title">Lo que dicen nuestros comensales</h2>
      </ScrollReveal>

      <ScrollRevealGroup
        containerVariants={testimonioContainerVariants}
        itemVariants={testimonioCardVariants}
        as="ul"
        className="testimonios__grid"
        threshold={0.10}
      >
        {items.map((t) => (
          <li key={t.author} className="testimonio-card">
            <blockquote className="testimonio-card__quote">
              &ldquo;{t.quote}&rdquo;
            </blockquote>
            <footer className="testimonio-card__author">
              <strong>{t.author}</strong>
              <span>{t.location}</span>
            </footer>
          </li>
        ))}
      </ScrollRevealGroup>
    </section>
  )
}
```

### 5d. `ReservasCTA` section

```tsx
// components/sections/ReservasCTA.tsx
"use client"

import { useRef } from "react"
import Link from "next/link"
import { motion, useInView } from "framer-motion"
import { useReducedMotion } from "@/lib/animation/useReducedMotion"
import {
  ctaHeadlineVariants,
  ctaButtonVariants,
  ctaContainerVariants,
} from "@/lib/animation/variants"

interface ReservasCTAProps {
  headline: string
  subtext?: string
  buttonLabel: string
  href: string
}

export function ReservasCTA({ headline, subtext, buttonLabel, href }: ReservasCTAProps) {
  const ref = useRef<HTMLElement>(null)
  const prefersReduced = useReducedMotion()

  // Full-width banner: trigger at 20% visibility — it's large, fires a bit later
  const isInView = useInView(ref, { amount: 0.20, once: true })

  if (prefersReduced) {
    return (
      <section ref={ref} className="reservas-cta">
        <h2>{headline}</h2>
        {subtext && <p>{subtext}</p>}
        <Link href={href} className="btn btn--primary btn--lg">
          {buttonLabel}
        </Link>
      </section>
    )
  }

  return (
    <section ref={ref} className="reservas-cta">
      <motion.div
        className="reservas-cta__inner"
        variants={ctaContainerVariants}
        initial="hidden"
        animate={isInView ? "visible" : "hidden"}
      >
        <motion.h2 variants={ctaHeadlineVariants} className="reservas-cta__headline">
          {headline}
        </motion.h2>

        {subtext && (
          <motion.p
            variants={ctaButtonVariants}  // reuse same subtle fade-up, stagger handles timing
            className="reservas-cta__subtext"
          >
            {subtext}
          </motion.p>
        )}

        <motion.div variants={ctaButtonVariants}>
          <Link href={href} className="btn btn--primary btn--lg">
            {buttonLabel}
          </Link>
        </motion.div>
      </motion.div>
    </section>
  )
}
```

### 5e. `Footer` section

```tsx
// components/layout/Footer.tsx  (add animation to existing footer)
"use client"

import { useRef } from "react"
import { motion, useInView } from "framer-motion"
import { useReducedMotion } from "@/lib/animation/useReducedMotion"
import { footerVariants } from "@/lib/animation/variants"

// Wrap your existing footer JSX content inside this shell.
// Replace the outer <footer> tag with this component.

interface FooterWrapperProps {
  children: React.ReactNode
  className?: string
}

export function FooterWrapper({ children, className }: FooterWrapperProps) {
  const ref = useRef<HTMLElement>(null)
  const prefersReduced = useReducedMotion()

  // Footer: very low threshold (0.05) — it's at page bottom, fires as soon
  // as the very top edge enters the viewport
  const isInView = useInView(ref, { amount: 0.05, once: true })

  if (prefersReduced) {
    return (
      <footer ref={ref} className={className}>
        {children}
      </footer>
    )
  }

  return (
    <motion.footer
      ref={ref}
      className={className}
      variants={footerVariants}
      initial="hidden"
      animate={isInView ? "visible" : "hidden"}
    >
      {children}
    </motion.footer>
  )
}
```

---

## File 6: Global CSS — Reduced Motion Safety Net

Add to `globals.css` as a hard fallback (supplements the JS hook):

```css
/* globals.css — add at bottom of animation section */

/* -----------------------------------------------------------------------
   Reduced Motion — Hard CSS fallback
   The JS hook (useReducedMotion) handles Framer Motion.
   This CSS rule catches any CSS transitions/keyframes that exist outside
   of Framer Motion (hover states, loading spinners, etc.)
   ----------------------------------------------------------------------- */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* -----------------------------------------------------------------------
   GPU Acceleration hints — applied to all animated sections
   Remove will-change after animation is complete via JS if needed
   ----------------------------------------------------------------------- */
.menu-destacado,
.historia,
.testimonios,
.reservas-cta {
  /* contain: layout prevents scroll-triggered animations from
     causing layout recalculations in the rest of the page */
  contain: layout;
}
```

---

## File 7: `components/animation/index.ts` (barrel export)

```typescript
// components/animation/index.ts
export { ScrollReveal } from "./ScrollReveal"
export { ScrollRevealGroup } from "./ScrollRevealGroup"
```

---

## Integration Checklist

```
□ framer-motion installed (npm install framer-motion)
□ ScrollReveal + ScrollRevealGroup added to components/animation/
□ variants.ts added to lib/animation/
□ useReducedMotion.ts added to lib/animation/
□ Each section component is a "use client" component
□ FooterWrapper wraps existing footer JSX
□ CSS reduced-motion fallback added to globals.css
□ No width/height animations (transform + opacity only — CLS safe)
□ Tested: prefers-reduced-motion: reduce → all final states visible immediately
□ Tested: 3 dish cards stagger at 0.14s intervals
□ Tested: Historia image/text enter from opposite sides
□ Tested: Footer fades in without transform (pure opacity)
```

---

## Animation Timing Summary

| Section | y offset | Duration | Ease | Stagger | Threshold |
|---|---|---|---|---|---|
| Menu — title | 20px | 0.60s | EXPO | — | 0.20 |
| Menu — cards | 32px | 0.70s | EXPO | 0.14s | 0.10 |
| Historia — image | x –40px | 0.80s | EXPO | — | 0.12 |
| Historia — text | x +40px | 0.80s | EXPO | 0.15s delay | 0.12 |
| Historia — paragraphs | 16px | 0.55s | EXPO | 0.10s | inherited |
| Testimonios — title | 18px | 0.60s | EXPO | — | 0.20 |
| Testimonios — cards | 24px + scale 0.97 | 0.65s | EXPO | 0.16s | 0.10 |
| CTA headline | 18px | 0.65s | EXPO | — | 0.20 |
| CTA button | 12px | 0.55s | EXPO | 0.18s delay | 0.20 |
| Footer | opacity only | 0.50s | STANDARD | — | 0.05 |

---

## Four Laws Applied

1. **Purpose over decoration** — Every animation communicates arrival. Cards stagger because the eye should sweep left-to-right over the menu. The historia split entrance mirrors the physical layout. Nothing moves without reason.

2. **Subtlety signals confidence** — Maximum y-offset is 40px (historia image). No bouncy springs, no overshoot. `EASE_OUT_EXPO` decelerates sharply and settles — confident, warm, Italian.

3. **Performance is non-negotiable** — Only `opacity` and `transform` are animated. `contain: layout` on section containers. No `width`, `height`, `top`, `left`. All transitions run on the compositor thread.

4. **Accessibility first** — `useReducedMotion()` wraps every animated component. When reduced motion is preferred: all components render final state immediately with no wrapper overhead. The CSS `@media` rule is a hard fallback for CSS-based effects outside Framer Motion.
