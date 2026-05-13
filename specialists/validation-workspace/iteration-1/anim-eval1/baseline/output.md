# La Nonna — Scroll Animations with Framer Motion

**Date:** 2026-05-13  
**Stack:** Next.js 15 + App Router, React 19, TypeScript  
**Brand feel:** Warm, Mediterranean, family-run Italian — subtle and inviting, never aggressive

---

## 1. Install Framer Motion

```bash
npm install framer-motion
```

Framer Motion 11+ ships React 19 support. No additional peer-dep changes needed.

---

## 2. Shared Animation Primitives

Create **`src/lib/motion.ts`** — single source of truth for all timing, easing, and variant definitions used site-wide.

```typescript
// src/lib/motion.ts
import type { Variants, Transition } from "framer-motion";

// ─── Easing curves ──────────────────────────────────────────────────────────
// "Mediterranean ease" — slow start, gentle deceleration. Feels unhurried, warm.
export const ease = {
  /** Default content entry: leisurely but decisive */
  out:    [0.16, 1, 0.3, 1]  as const,
  /** Staggered children: slightly snappier so each card feels crisp */
  outSnap:[0.22, 1, 0.36, 1] as const,
  /** For scale reveals (dish cards, CTA banner) */
  spring: { type: "spring", stiffness: 60, damping: 18, mass: 0.8 } as const,
} satisfies Record<string, readonly number[] | object>;

// ─── Shared transitions ──────────────────────────────────────────────────────
export const transition = {
  /** Single block fading in */
  fadeUp: {
    duration: 0.72,
    ease: ease.out,
  } satisfies Transition,

  /** Section header (eyebrow + title) */
  heading: {
    duration: 0.80,
    ease: ease.out,
  } satisfies Transition,

  /** Card / stagger parent — controls stagger timing */
  staggerContainer: {
    staggerChildren: 0.14,
    delayChildren: 0.10,
  } satisfies Transition,

  /** Individual card within a stagger group */
  card: {
    duration: 0.60,
    ease: ease.outSnap,
  } satisfies Transition,

  /** CTA banner — slightly longer for drama */
  banner: {
    duration: 0.90,
    ease: ease.out,
  } satisfies Transition,

  /** Footer — long and soft, no need to rush */
  footer: {
    duration: 0.80,
    ease: ease.out,
    staggerChildren: 0.10,
    delayChildren: 0.05,
  } satisfies Transition,
} satisfies Record<string, Transition>;

// ─── Reusable Variants ───────────────────────────────────────────────────────

/** Generic fade + slide up — used on most text blocks */
export const fadeUpVariants: Variants = {
  hidden: { opacity: 0, y: 28 },
  visible: {
    opacity: 1,
    y: 0,
    transition: transition.fadeUp,
  },
};

/** Section heading (eyebrow + h2 together) — slightly more y travel */
export const headingVariants: Variants = {
  hidden: { opacity: 0, y: 36 },
  visible: {
    opacity: 1,
    y: 0,
    transition: transition.heading,
  },
};

/** Stagger container — wraps a group of cards */
export const staggerContainerVariants: Variants = {
  hidden: {},
  visible: {
    transition: transition.staggerContainer,
  },
};

/** Individual card in a stagger group */
export const cardVariants: Variants = {
  hidden: { opacity: 0, y: 24, scale: 0.97 },
  visible: {
    opacity: 1,
    y: 0,
    scale: 1,
    transition: transition.card,
  },
};

/** Fade + slide from left (image side of 2-col layout) */
export const slideFromLeftVariants: Variants = {
  hidden: { opacity: 0, x: -40 },
  visible: {
    opacity: 1,
    x: 0,
    transition: { duration: 0.84, ease: ease.out },
  },
};

/** Fade + slide from right (text side of 2-col layout) */
export const slideFromRightVariants: Variants = {
  hidden: { opacity: 0, x: 40 },
  visible: {
    opacity: 1,
    x: 0,
    transition: { duration: 0.84, ease: ease.out, delay: 0.12 },
  },
};

/** Scale + fade reveal — used for full-width banners */
export const bannerVariants: Variants = {
  hidden: { opacity: 0, scale: 0.985 },
  visible: {
    opacity: 1,
    scale: 1,
    transition: transition.banner,
  },
};

/** Footer columns — subtle bottom-up */
export const footerItemVariants: Variants = {
  hidden: { opacity: 0, y: 16 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.60, ease: ease.out },
  },
};
```

---

## 3. `useReducedMotion` Hook Wrapper

Framer Motion exposes `useReducedMotion()` natively. We centralise the pattern in one small helper so every animated component uses it consistently.

```typescript
// src/hooks/useMotionSafe.ts
"use client";

import { useReducedMotion } from "framer-motion";

/**
 * Returns `"visible"` immediately (skipping all animations) when the user
 * has requested reduced motion, otherwise returns `"hidden"` so Framer
 * Motion runs the normal entrance sequence.
 */
export function useInitialMotionState(): "hidden" | "visible" {
  const prefersReduced = useReducedMotion();
  return prefersReduced ? "visible" : "hidden";
}
```

> **Why this approach:**  
> Setting `initial="visible"` means the element is already in its final state — no transform, no opacity change, no layout shift. All downstream variants run as no-ops because they animate from `visible` → `visible`. It is safer than trying to `animate={false}`, and plays nicely with SSR hydration.

---

## 4. `ScrollReveal` Utility Component

A thin wrapper around Framer Motion's `motion.div` + `whileInView` that every section can consume without boilerplate.

```tsx
// src/components/ui/ScrollReveal.tsx
"use client";

import { motion, type Variants } from "framer-motion";
import { useInitialMotionState } from "@/hooks/useMotionSafe";
import { fadeUpVariants } from "@/lib/motion";

interface ScrollRevealProps {
  children: React.ReactNode;
  variants?: Variants;
  className?: string;
  /** Override the intersection threshold (default 0.12) */
  threshold?: number;
  /** Extra rootMargin to shift trigger point (default "-40px") */
  margin?: string;
  /** Render as a different HTML element */
  as?: keyof JSX.IntrinsicElements;
}

export default function ScrollReveal({
  children,
  variants = fadeUpVariants,
  className,
  threshold = 0.12,
  margin = "-40px",
  as = "div",
}: ScrollRevealProps) {
  const initial = useInitialMotionState();

  const Component = motion[as as "div"] as typeof motion.div;

  return (
    <Component
      className={className}
      variants={variants}
      initial={initial}
      whileInView="visible"
      viewport={{ once: true, amount: threshold, margin }}
    >
      {children}
    </Component>
  );
}
```

---

## 5. Section Animations — Full Implementation

### 5.1 Hero

The Hero is already on-screen at load — **no scroll trigger needed**. We use a mount-time entrance (`animate` not `whileInView`) with a staggered sequence: supertitle → title → subtitle → CTAs.

The existing `AnimateIn.tsx` handles hero parallax via a raw `scroll` listener. We keep that untouched and only add entrance motion.

```tsx
// src/components/home/Hero.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import Image from "next/image";
import { useInitialMotionState } from "@/hooks/useMotionSafe";

// Hero-specific variants — staggered from the bottom, soft & warm
const heroContainerVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.18,
      delayChildren: 0.20,  // slight pause so image settles first
    },
  },
} as const;

const heroItemVariants = {
  hidden: { opacity: 0, y: 22 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.80, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

const heroCtaVariants = {
  hidden: { opacity: 0, y: 16 },
  visible: {
    opacity: 1,
    y: 0,
    transition: {
      duration: 0.70,
      ease: [0.16, 1, 0.3, 1],
      staggerChildren: 0.12,
      delayChildren: 0,
    },
  },
} as const;

const heroCtaItemVariants = {
  hidden: { opacity: 0, y: 12, scale: 0.97 },
  visible: {
    opacity: 1,
    y: 0,
    scale: 1,
    transition: { duration: 0.55, ease: [0.22, 1, 0.36, 1] },
  },
} as const;

export default function Hero() {
  const t = useTranslations("home.hero");
  const initial = useInitialMotionState();

  return (
    <section className="hero" aria-label="Hero">
      <div className="hero__bg" aria-hidden="true">
        <Image
          src="https://images.unsplash.com/photo-1414235077428-338989a2e8c0?auto=format&fit=crop&w=1920&q=80"
          alt=""
          fill
          priority
          style={{ objectFit: "cover", objectPosition: "center 40%" }}
        />
      </div>
      <div className="hero__overlay" aria-hidden="true" />

      {/* Animated content */}
      <motion.div
        className="hero__content"
        variants={heroContainerVariants}
        initial={initial}
        animate="visible"
      >
        <motion.p className="hero__supertitle" variants={heroItemVariants}>
          {t("supertitle")}
        </motion.p>

        <motion.h1 className="hero__title" variants={heroItemVariants}>
          {t("title")}
        </motion.h1>

        <motion.p className="hero__subtitle" variants={heroItemVariants}>
          {t("subtitle")}
        </motion.p>

        <motion.div className="hero__ctas" variants={heroCtaVariants}>
          <motion.div variants={heroCtaItemVariants}>
            <Link href="/reservar" className="btn btn--primary btn--lg">
              {t("cta_primary")}
            </Link>
          </motion.div>
          <motion.div variants={heroCtaItemVariants}>
            <Link href="/carta" className="btn btn--inverse-ghost btn--lg">
              {t("cta_secondary")}
            </Link>
          </motion.div>
        </motion.div>
      </motion.div>
    </section>
  );
}
```

**Timing breakdown:**
| Element | Delay from section mount | Duration |
|---|---|---|
| Supertitle | 200ms | 800ms |
| Title (h1) | 380ms | 800ms |
| Subtitle | 560ms | 800ms |
| CTA button 1 | 740ms | 550ms |
| CTA button 2 | 860ms | 550ms |

---

### 5.2 Menu Destacado — 3 dish cards with stagger

The section header fades up first, then the three cards stagger in with a scale+fade reveal that feels like dishes being set on the table.

```tsx
// src/components/home/MenuHighlight.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import Image from "next/image";
import type { DishItem } from "@/types";
import { useInitialMotionState } from "@/hooks/useMotionSafe";
import {
  headingVariants,
  staggerContainerVariants,
  cardVariants,
  fadeUpVariants,
} from "@/lib/motion";

const DISH_IMAGES = [
  "https://images.unsplash.com/photo-1621996346565-e3dbc646d9a9?auto=format&fit=crop&w=600&q=80",
  "https://images.unsplash.com/photo-1476124369491-e7addf5db371?auto=format&fit=crop&w=600&q=80",
  "https://images.unsplash.com/photo-1571877227200-a0d98ea607e9?auto=format&fit=crop&w=600&q=80",
];

function DishCard({
  name,
  description,
  price,
  imageUrl,
}: DishItem & { imageUrl: string }) {
  return (
    // Each card is a motion.article — driven by cardVariants from parent stagger
    <motion.article className="dish-card" variants={cardVariants}>
      <div className="dish-card__image">
        <Image src={imageUrl} alt={name} fill style={{ objectFit: "cover" }} />
      </div>
      <div className="dish-card__body">
        <h3 className="dish-card__name">{name}</h3>
        {description && (
          <p className="dish-card__description">{description}</p>
        )}
        <div className="dish-card__footer">
          <p className="dish-card__price">{price} €</p>
        </div>
      </div>
    </motion.article>
  );
}

export default function MenuHighlight() {
  const t = useTranslations("home.menu_highlight");
  const dishes = t.raw("dishes") as DishItem[];
  const initial = useInitialMotionState();

  return (
    <section className="menu-highlight section">
      <div className="container">

        {/* Header: eyebrow + title fade up together */}
        <motion.div
          className="menu-highlight__header"
          variants={headingVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.5 }}
        >
          <p className="eyebrow">{t("eyebrow")}</p>
          <h2>{t("title")}</h2>
        </motion.div>

        {/* Cards grid: stagger container */}
        <motion.div
          className="menu-highlight__grid"
          variants={staggerContainerVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.12, margin: "-32px" }}
        >
          {dishes.map((d, i) => (
            <DishCard
              key={d.name}
              {...d}
              imageUrl={DISH_IMAGES[i] ?? DISH_IMAGES[0]}
            />
          ))}
        </motion.div>

        {/* Footer CTA */}
        <motion.div
          className="menu-highlight__footer"
          variants={fadeUpVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.8 }}
        >
          <Link href="/carta" className="btn btn--ghost">
            {t("cta")}
          </Link>
        </motion.div>
      </div>
    </section>
  );
}
```

**Timing breakdown (cards):**
| Card | Delay after grid enters viewport | Duration |
|---|---|---|
| Card 1 | 100ms | 600ms |
| Card 2 | 240ms | 600ms |
| Card 3 | 380ms | 600ms |

Stagger: 140ms between each card. The `delayChildren: 0.10` gives the grid a 100ms grace period before the first card starts, so the container itself is invisible just long enough that all three cards arrive as a group, not one by one.

---

### 5.3 Historia de la Familia — 2-column image + text

This section uses a directional split: the image slides in from the left, the text content from the right. They share the same viewport trigger so they arrive together but from opposing directions, reinforcing the visual balance.

The existing `HistoriaSnippet` is a full-width overlay, not a 2-column layout. The 2-column variant is `FeaturedSection`. Both get animations below.

**`FeaturedSection` (the 2-col image+text panel):**

```tsx
// src/components/home/FeaturedSection.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import Image from "next/image";
import { useInitialMotionState } from "@/hooks/useMotionSafe";
import { slideFromLeftVariants, slideFromRightVariants } from "@/lib/motion";

// Text items stagger inside the right column
const textContainerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.12, delayChildren: 0.14 },
  },
} as const;

const textItemVariants = {
  hidden: { opacity: 0, y: 18 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.68, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

export default function FeaturedSection() {
  const t = useTranslations("home.featured");
  const initial = useInitialMotionState();

  return (
    <section className="split split--50-50">

      {/* Image slides in from the left */}
      <motion.div
        className="split__image"
        variants={slideFromLeftVariants}
        initial={initial}
        whileInView="visible"
        viewport={{ once: true, amount: 0.20 }}
      >
        <Image
          src="https://images.unsplash.com/photo-1551183053-bf91798d047a?w=800&q=80"
          alt={t("image_alt")}
          width={800}
          height={600}
        />
      </motion.div>

      {/* Text content slides from the right, items stagger */}
      <motion.div
        className="split__content"
        variants={textContainerVariants}
        initial={initial}
        whileInView="visible"
        viewport={{ once: true, amount: 0.20 }}
      >
        <motion.p className="eyebrow" variants={slideFromRightVariants}>
          {t("eyebrow")}
        </motion.p>
        <motion.h2 variants={slideFromRightVariants}>{t("title")}</motion.h2>
        <motion.p variants={textItemVariants}>{t("body_1")}</motion.p>
        <motion.p variants={textItemVariants}>{t("body_2")}</motion.p>
        <motion.div variants={textItemVariants}>
          <Link href="/carta" className="btn btn--primary">
            {t("cta")}
          </Link>
        </motion.div>
      </motion.div>
    </section>
  );
}
```

**`HistoriaSnippet` (full-width terracotta overlay — this is the "Historia de la familia" section on the homepage):**

```tsx
// src/components/home/HistoriaSnippet.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import Image from "next/image";
import { useInitialMotionState } from "@/hooks/useMotionSafe";

// Content items inside the overlay stagger upward
const overlayContainerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.16, delayChildren: 0.22 },
  },
} as const;

const overlayItemVariants = {
  hidden: { opacity: 0, y: 24 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.76, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

// The background image scales in subtly — a warm "welcome" gesture
const bgRevealVariants = {
  hidden: { opacity: 0, scale: 1.04 },
  visible: {
    opacity: 1,
    scale: 1,
    transition: { duration: 1.10, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

export default function HistoriaSnippet() {
  const t = useTranslations("home.historia_snippet");
  const initial = useInitialMotionState();

  return (
    <section className="historia-snippet">
      {/* Background image — slower reveal, like a curtain rising */}
      <motion.div
        className="historia-snippet__bg"
        aria-hidden="true"
        variants={bgRevealVariants}
        initial={initial}
        whileInView="visible"
        viewport={{ once: true, amount: 0.15 }}
      >
        <Image
          src="https://images.unsplash.com/photo-1556910103-1c02745aae4d?auto=format&fit=crop&w=1920&q=80"
          alt=""
          fill
          style={{ objectFit: "cover", objectPosition: "center" }}
        />
      </motion.div>

      <div className="historia-snippet__overlay" aria-hidden="true" />

      {/* Staggered text content */}
      <motion.div
        className="historia-snippet__content"
        variants={overlayContainerVariants}
        initial={initial}
        whileInView="visible"
        viewport={{ once: true, amount: 0.30 }}
      >
        <motion.h2 variants={overlayItemVariants}>{t("title")}</motion.h2>
        <motion.p variants={overlayItemVariants}>{t("body")}</motion.p>
        <motion.div variants={overlayItemVariants}>
          <Link href="/historia" className="btn btn--inverse">
            {t("cta")}
          </Link>
        </motion.div>
      </motion.div>
    </section>
  );
}
```

---

### 5.4 Testimonios — 3 quote cards with stagger

Same pattern as dish cards but the stagger is slightly faster (120ms instead of 140ms) — testimonials are lighter and should feel more conversational.

```tsx
// src/components/home/ReviewsSection.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { useInitialMotionState } from "@/hooks/useMotionSafe";
import { headingVariants, staggerContainerVariants } from "@/lib/motion";

interface ReviewItem {
  text: string;
  author: string;
  stars: number;
}

// Review card variant — slightly faster than dish cards, more intimate
const reviewCardVariants = {
  hidden: { opacity: 0, y: 20, scale: 0.98 },
  visible: {
    opacity: 1,
    y: 0,
    scale: 1,
    transition: { duration: 0.55, ease: [0.22, 1, 0.36, 1] },
  },
} as const;

// Stagger container for reviews — tighter spacing (120ms)
const reviewsGridVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.12, delayChildren: 0.08 },
  },
} as const;

function ReviewCard({ text, author, stars }: ReviewItem) {
  return (
    <motion.article className="review-card" variants={reviewCardVariants}>
      <div className="review-card__stars" aria-label={`${stars} estrellas`}>
        {"★".repeat(stars)}
      </div>
      <blockquote className="review-card__text">{text}</blockquote>
      <div className="review-card__author">
        <span className="review-card__name">{author}</span>
      </div>
    </motion.article>
  );
}

export default function ReviewsSection() {
  const t = useTranslations("home.reviews");
  const items = t.raw("items") as ReviewItem[];
  const initial = useInitialMotionState();

  return (
    <section className="reviews section">
      <div className="container">

        {/* Section header */}
        <motion.div
          className="reviews__header"
          variants={headingVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.5 }}
        >
          <p className="eyebrow">{t("eyebrow")}</p>
          <p className="reviews__rating">{t("rating")}</p>
        </motion.div>

        {/* Cards grid with stagger */}
        <motion.div
          className="reviews__grid"
          variants={reviewsGridVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.10, margin: "-32px" }}
        >
          {items.map((r) => (
            <ReviewCard
              key={r.author}
              text={r.text}
              author={r.author}
              stars={r.stars}
            />
          ))}
        </motion.div>
      </div>
    </section>
  );
}
```

**Timing breakdown (review cards):**
| Card | Delay | Duration |
|---|---|---|
| Card 1 | 80ms | 550ms |
| Card 2 | 200ms | 550ms |
| Card 3 | 320ms | 550ms |

---

### 5.5 Reservas CTA — Full-width banner

The terracotta banner is the most emotionally weighted section — it's the conversion moment. Use a scale-from-slightly-smaller reveal so it feels like it emerges from the page, then stagger the h2 → paragraph → button inside.

```tsx
// src/components/home/ReservarCta.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import { useInitialMotionState } from "@/hooks/useMotionSafe";

// Banner wrapper: scale+fade — the section itself breathes open
const bannerWrapperVariants = {
  hidden: { opacity: 0, scale: 0.984 },
  visible: {
    opacity: 1,
    scale: 1,
    transition: { duration: 0.90, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

// Content items stagger inside the banner
const ctaContainerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.14, delayChildren: 0.28 },
  },
} as const;

const ctaItemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.68, ease: [0.16, 1, 0.3, 1] },
  },
} as const;

const ctaButtonVariants = {
  hidden: { opacity: 0, y: 12, scale: 0.96 },
  visible: {
    opacity: 1,
    y: 0,
    scale: 1,
    transition: { duration: 0.60, ease: [0.22, 1, 0.36, 1] },
  },
} as const;

export default function ReservarCta() {
  const t = useTranslations("home.reservar_cta");
  const initial = useInitialMotionState();

  return (
    <motion.section
      className="reservar-cta"
      variants={bannerWrapperVariants}
      initial={initial}
      whileInView="visible"
      viewport={{ once: true, amount: 0.25 }}
    >
      <motion.div
        className="container"
        variants={ctaContainerVariants}
        // No initial/whileInView here — inherits from parent section
      >
        <motion.h2 variants={ctaItemVariants}>{t("title")}</motion.h2>
        <motion.p variants={ctaItemVariants}>{t("body")}</motion.p>
        <motion.div className="reservar-cta__actions" variants={ctaButtonVariants}>
          <Link href="/reservar" className="btn btn--inverse btn--lg">
            {t("cta")}
          </Link>
        </motion.div>
      </motion.div>
    </motion.section>
  );
}
```

**Timing breakdown:**
| Element | Delay from section trigger | Duration |
|---|---|---|
| Section scale-in | 0ms | 900ms |
| H2 | 280ms | 680ms |
| Paragraph | 420ms | 680ms |
| CTA button | 560ms | 600ms |

The `delayChildren: 0.28` ensures the section has had 280ms to reveal itself before the text starts appearing, so the content never races ahead of its container.

---

### 5.6 Footer

The footer doesn't need drama — it's the end of the journey. A gentle stagger of the three columns sliding up, then the bottom bar fading in last.

```tsx
// src/components/layout/Footer.tsx
"use client";

import { motion } from "framer-motion";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";
import { useInitialMotionState } from "@/hooks/useMotionSafe";
import { footerItemVariants } from "@/lib/motion";

const footerGridVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.10, delayChildren: 0.05 },
  },
} as const;

const footerBottomVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { duration: 0.60, ease: [0.16, 1, 0.3, 1], delay: 0.40 },
  },
} as const;

export default function Footer() {
  const t = useTranslations("footer");
  const year = new Date().getFullYear();
  const initial = useInitialMotionState();

  return (
    <footer className="footer" role="contentinfo">
      <div className="container">

        {/* 3-column grid — columns stagger in */}
        <motion.div
          className="footer__grid"
          variants={footerGridVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.15 }}
        >
          {/* Col 1 — Brand */}
          <motion.div variants={footerItemVariants}>
            <p className="footer__col-title">La Nonna</p>
            <p className="footer__address">{t("tagline")}</p>
          </motion.div>

          {/* Col 2 — Address */}
          <motion.div variants={footerItemVariants}>
            <p className="footer__col-title">{t("col_visit")}</p>
            <address className="footer__address">
              <span>Carrer de Verdi, 42</span><br />
              <span>08012 Gràcia, Barcelona</span><br />
              <a href={`tel:${t("phone").replace(/\s/g, "")}`}>{t("phone")}</a><br />
              <a href={`mailto:${t("email")}`}>{t("email")}</a>
            </address>
          </motion.div>

          {/* Col 3 — Hours */}
          <motion.div variants={footerItemVariants}>
            <p className="footer__col-title">{t("hours_label")}</p>
            <address className="footer__address">
              {(t.raw("hours") as string[]).map((line: string) => (
                <span key={line} style={{ display: "block" }}>{line}</span>
              ))}
            </address>
          </motion.div>
        </motion.div>

        {/* Bottom bar — fades in after columns */}
        <motion.div
          className="footer__bottom"
          variants={footerBottomVariants}
          initial={initial}
          whileInView="visible"
          viewport={{ once: true, amount: 0.8 }}
        >
          <p className="footer__copy">© {year} La Nonna</p>
          <div className="footer__legal">
            <Link href="/aviso-legal" className="footer__legal-link">
              {t("legal_aviso")}
            </Link>
          </div>
        </motion.div>
      </div>
    </footer>
  );
}
```

**Timing breakdown (footer columns):**
| Column | Delay | Duration |
|---|---|---|
| Brand (col 1) | 50ms | 600ms |
| Address (col 2) | 150ms | 600ms |
| Hours (col 3) | 250ms | 600ms |
| Bottom bar | 400ms | 600ms |

---

## 6. Remove Legacy `AnimateIn` CSS Classes

With Framer Motion managing all entrance animations, the `.animate-in` and `.animate-stagger` CSS classes in `components.css` are no longer needed. Remove them from JSX attributes.

The `AnimateIn` client component can be stripped down to **only the hero parallax logic**:

```tsx
// src/components/ui/AnimateIn.tsx
"use client";

import { useEffect } from "react";

/**
 * Hero parallax only — scroll animations are now handled by Framer Motion
 * in each individual section component.
 */
export default function AnimateIn() {
  useEffect(() => {
    const hero = document.querySelector(".hero") as HTMLElement | null;
    const heroBg = document.querySelector(".hero__bg") as HTMLElement | null;

    function onScroll() {
      if (!hero || !heroBg) return;
      const rect = hero.getBoundingClientRect();
      if (rect.bottom <= 0) return;
      const scrolled = Math.max(0, -rect.top);
      heroBg.style.transform = `translateY(${scrolled * 0.3}px)`;
    }

    if (heroBg) {
      heroBg.style.willChange = "transform";
      window.addEventListener("scroll", onScroll, { passive: true });
    }

    return () => {
      window.removeEventListener("scroll", onScroll);
    };
  }, []);

  return null;
}
```

> **Note:** The hero parallax runs on the `.hero__bg` div which is wrapped in a `motion.div` (the image itself). Framer Motion and the raw `transform` style don't conflict here — Framer Motion sets the initial opacity/scale, then the scroll handler takes over `transform` independently once the hero is visible.
>
> If conflict is observed, move the parallax target to a wrapper div outside the `motion.div` or use Framer Motion's `useScroll` + `useTransform` instead (see Addendum A).

---

## 7. Updated `layout.tsx` — No Changes Required

`AnimateIn` is already imported and rendered in `[locale]/layout.tsx`. No changes needed there since the component is preserved (just simplified internally).

---

## 8. `prefers-reduced-motion` — Full Audit

Three layers of protection:

### Layer 1 — CSS (already in `globals.css`)
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```
This kills the `ken-burns` keyframe animation on the hero bg image. Good.

### Layer 2 — React hook (`useInitialMotionState`)
All `initial` props use our hook. When `prefers-reduced-motion: reduce` is active:
- `initial="visible"` → element renders already at its final state
- `whileInView="visible"` → no-op, `visible` → `visible`
- Result: zero animation, zero layout shift, zero CLS

### Layer 3 — Framer Motion global (belt-and-suspenders)
Add this to your root layout or a global `Providers` component:

```tsx
// src/components/ui/MotionConfig.tsx
"use client";

import { MotionConfig } from "framer-motion";
import { useReducedMotion } from "framer-motion";

export default function ReducedMotionProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  const reduced = useReducedMotion();
  return (
    <MotionConfig reducedMotion={reduced ? "always" : "never"}>
      {children}
    </MotionConfig>
  );
}
```

Add to `[locale]/layout.tsx`:
```tsx
// wrap <main> (and ideally the whole body) with <ReducedMotionProvider>
import ReducedMotionProvider from "@/components/ui/MotionConfig";

// inside the JSX:
<ReducedMotionProvider>
  <Header />
  <main id="main-content">{children}</main>
  <Footer />
  <AnimateIn />
</ReducedMotionProvider>
```

With `reducedMotion="always"`, Framer Motion internally suppresses all animation math — no spring calculations, no RAF loop entries — which is more performant than the hook-only approach.

---

## 9. Complete Timing + Easing Reference Table

| Section | Trigger | Delay | Duration | Easing | Stagger |
|---|---|---|---|---|---|
| **Hero supertitle** | mount | 200ms | 800ms | [0.16, 1, 0.3, 1] | — |
| **Hero title** | mount | 380ms | 800ms | [0.16, 1, 0.3, 1] | — |
| **Hero subtitle** | mount | 560ms | 800ms | [0.16, 1, 0.3, 1] | — |
| **Hero CTA 1** | mount | 740ms | 550ms | [0.22, 1, 0.36, 1] | — |
| **Hero CTA 2** | mount | 860ms | 550ms | [0.22, 1, 0.36, 1] | — |
| **FeaturedSection image** | scroll 20% | 0ms | 840ms | [0.16, 1, 0.3, 1] | — |
| **FeaturedSection text** | scroll 20% | 120ms | 680ms each | [0.16, 1, 0.3, 1] | 120ms |
| **MenuHighlight header** | scroll 50% | 0ms | 800ms | [0.16, 1, 0.3, 1] | — |
| **DishCard 1** | scroll 12% | 100ms | 600ms | [0.22, 1, 0.36, 1] | — |
| **DishCard 2** | scroll 12% | 240ms | 600ms | [0.22, 1, 0.36, 1] | — |
| **DishCard 3** | scroll 12% | 380ms | 600ms | [0.22, 1, 0.36, 1] | — |
| **MenuHighlight CTA** | scroll 80% | 0ms | 720ms | [0.16, 1, 0.3, 1] | — |
| **HistoriaSnippet bg** | scroll 15% | 0ms | 1100ms | [0.16, 1, 0.3, 1] | — |
| **HistoriaSnippet h2** | scroll 30% | 220ms | 760ms | [0.16, 1, 0.3, 1] | — |
| **HistoriaSnippet p** | scroll 30% | 380ms | 760ms | [0.16, 1, 0.3, 1] | — |
| **HistoriaSnippet CTA** | scroll 30% | 540ms | 760ms | [0.16, 1, 0.3, 1] | — |
| **Reviews header** | scroll 50% | 0ms | 800ms | [0.16, 1, 0.3, 1] | — |
| **ReviewCard 1** | scroll 10% | 80ms | 550ms | [0.22, 1, 0.36, 1] | — |
| **ReviewCard 2** | scroll 10% | 200ms | 550ms | [0.22, 1, 0.36, 1] | — |
| **ReviewCard 3** | scroll 10% | 320ms | 550ms | [0.22, 1, 0.36, 1] | — |
| **ReservarCta section** | scroll 25% | 0ms | 900ms | [0.16, 1, 0.3, 1] | — |
| **ReservarCta h2** | scroll 25% | 280ms | 680ms | [0.16, 1, 0.3, 1] | — |
| **ReservarCta p** | scroll 25% | 420ms | 680ms | [0.16, 1, 0.3, 1] | — |
| **ReservarCta button** | scroll 25% | 560ms | 600ms | [0.22, 1, 0.36, 1] | — |
| **Footer col 1** | scroll 15% | 50ms | 600ms | [0.16, 1, 0.3, 1] | — |
| **Footer col 2** | scroll 15% | 150ms | 600ms | [0.16, 1, 0.3, 1] | — |
| **Footer col 3** | scroll 15% | 250ms | 600ms | [0.16, 1, 0.3, 1] | — |
| **Footer bottom bar** | scroll 80% | 400ms | 600ms | [0.16, 1, 0.3, 1] | — |

---

## 10. New Files to Create

```
src/
  lib/
    motion.ts                    ← variant + timing primitives
  hooks/
    useMotionSafe.ts             ← useInitialMotionState hook
  components/
    ui/
      ScrollReveal.tsx           ← optional utility wrapper
      MotionConfig.tsx           ← ReducedMotionProvider
```

## Files to Modify

```
src/components/home/Hero.tsx           ← add "use client", motion wrappers
src/components/home/MenuHighlight.tsx  ← add "use client", stagger grid
src/components/home/FeaturedSection.tsx← add "use client", split reveal
src/components/home/HistoriaSnippet.tsx← add "use client", overlay stagger
src/components/home/ReviewsSection.tsx ← add "use client", stagger grid
src/components/home/ReservarCta.tsx    ← add "use client", banner scale
src/components/layout/Footer.tsx       ← add "use client", column stagger
src/components/ui/AnimateIn.tsx        ← strip IntersectionObserver, keep parallax
src/app/[locale]/layout.tsx            ← wrap with ReducedMotionProvider
```

---

## 11. SSR / Hydration Notes

- All motion components require `"use client"`. Currently Hero, MenuHighlight, FeaturedSection, HistoriaSnippet, ReviewsSection, ReservarCta, and Footer are **server components** — adding `"use client"` at the top makes them client components. This is fine for homepage sections (they are leaf components, no data-fetching happens inside them).
- `useTranslations` works in client components when wrapped in `NextIntlClientProvider`, which is already in `[locale]/layout.tsx`.
- `viewport={{ once: true }}` ensures animations fire only once — no re-triggering on scroll-up, which would feel jarring and un-Mediterranean.
- `initial` is set from the hook (not hardcoded to `"hidden"`) so SSR renders elements in their final visible state, preventing any flash-of-invisible-content during hydration.

---

## Addendum A — Optional: Hero Parallax via Framer Motion `useScroll`

If the dual-transform approach (Framer Motion + raw scroll listener) proves problematic, replace `AnimateIn`'s parallax with this:

```tsx
// Inside Hero.tsx, replace the static hero__bg div with:
"use client";
import { motion, useScroll, useTransform, useReducedMotion } from "framer-motion";
import { useRef } from "react";

// Inside Hero component:
const heroRef = useRef<HTMLElement>(null);
const prefersReduced = useReducedMotion();

const { scrollYProgress } = useScroll({
  target: heroRef,
  offset: ["start start", "end start"],
});

// Move bg 30% of the hero's scroll distance — matches original behavior
const bgY = useTransform(
  scrollYProgress,
  [0, 1],
  prefersReduced ? ["0%", "0%"] : ["0%", "30%"]
);

// In JSX:
<section className="hero" ref={heroRef}>
  <motion.div
    className="hero__bg"
    aria-hidden="true"
    style={{ y: bgY }}
  >
    {/* Image */}
  </motion.div>
  ...
</section>
```

Then remove the parallax logic from `AnimateIn.tsx` entirely, letting the file become a no-op (or be deleted from `layout.tsx`).

---

*Generated for: La Nonna — Fase 3 development*  
*Target: homepage scroll animations, production-ready*
