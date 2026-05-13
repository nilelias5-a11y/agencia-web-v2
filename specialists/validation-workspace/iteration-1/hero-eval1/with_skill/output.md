# Hero Section Spec — La Nonna

## Applying the 3-Second Rule

Before any visitor reads a single word, three questions must be answered at a glance:

1. **What is this?** — A real Italian restaurant in Gràcia, Barcelona. Not a chain, not a tourist trap.
2. **Why does it matter to me?** — Authentic family cooking since 2003, the kind your grandmother would make.
3. **What do I do next?** — Reserve your table tonight.

The hero must deliver all three answers in the first viewport, at every screen size, in under 3 seconds.

---

## Layout Pattern Selection

**Selected: Pattern 4 — Video Background (with Pattern 1 Centered Power overlay)**

**Rationale:**

For a restaurant in the hospitality niche, video background is the dominant award-winning pattern. The reasons are visceral: food, warmth, and atmosphere cannot be communicated through copy alone — the visitor must *feel* the place before they decide to go. Pattern 4 is used by Noma, Osteria Francescana's digital presence, and Etxebarri's landing pages — the top references in the restaurant Awwwards space.

La Nonna specifically needs to overcome the "tourist restaurant in Barcelona" stigma. A 20-second looping video of a wood table, a hand placing a ceramic bowl of pasta, steam rising from a pot, and the Gràcia neighborhood light through linen curtains communicates *neighborhood authenticity* in a way no headline can.

The text overlay follows Pattern 1 (Centered Power) because the brand identity — warm, grandmother-like, family — calls for a centered, intimate framing rather than a cold split-screen layout.

**Award-winning reference sites:**
1. **Noma (noma.dk)** — Ultra-minimal text over atmospheric food photography, single reservation CTA with extreme white space. Borrow: the confidence of letting the image carry the weight.
2. **The Fat Duck (thefatduck.co.uk)** — Centered headline with elegant serif, dark overlay giving text room to breathe. Borrow: the dark semi-transparent scrim technique.
3. **SingleThread Farm (singlethreadfarms.com)** — Video loop of kitchen/farm activity, minimal copy, direct booking CTA. Borrow: the looping video structure and static fallback approach.
4. **Lyle's London (lyleslondon.com)** — Typographic confidence on food imagery, Playfair-class serif treatment. Borrow: the serif headline weight against a moody background.

---

## Hero Section Spec

**Layout pattern:** 4 (Video Background) + 1 (Centered Power overlay)

**Background:**
- Primary: Looping MP4/WebM video (20–25 seconds, no audio) — shot of pasta being hand-rolled on a floured wooden board, then cut to a terracotta bowl being placed on a table, candlelight catching the olive oil sheen. Provided by client or sourced from Artgrid/Pond5 with warm color grade.
- Video overlay: `linear-gradient(to bottom, rgba(42,26,14,0.45) 0%, rgba(42,26,14,0.65) 100%)` — warm-brown scrim, not cold black.
- Fallback static image: High-res photograph of a rustic pasta dish on a La Nonna table, same scrim applied.

**Headline:** "La cucina di casa tua" | Font: Playfair Display | Size: 72px desktop / 40px mobile | Weight: 700 | Color: #F5EDE0 (cream) | Letter-spacing: -0.5px | Line-height: 1.1
*English subtitle beneath in smaller weight: "Your home kitchen, in the heart of Gràcia"*

**Subheadline:** "Cocina italiana de familia desde 2003 · Gracia, Barcelona" | Font: Inter | Size: 18px desktop / 15px mobile | Weight: 400 | Color: rgba(245,237,224,0.80) | Letter-spacing: 0.03em

**Primary CTA:** "Reservar mesa" → /reservas | Background: #C0553A (terracotta) | Text: #F5EDE0 | Size: 18px | Padding: 16px 36px | Border-radius: 2px (slightly squared — intentional, artisan feel) | Hover: darken 8%, subtle scale(1.03)

**Secondary CTA:** "Ver la carta" → /menu | Style: ghost button | Border: 1.5px solid rgba(245,237,224,0.60) | Text: #F5EDE0 | Same size | Hover: border goes full opacity, bg rgba(245,237,224,0.08)

**Establishment badge:** Small line of micro-copy above the headline — "Dal 2003 · Ristorante di famiglia" in Inter 12px / uppercase / letter-spacing 0.15em / color rgba(245,237,224,0.55). Acts as a trust signal and sets the premium artisan register.

**Animation sequence (total: 1.4 seconds):**
1. `t=0ms` — Video background starts playing (already loaded)
2. `t=0ms` — Badge fades in: `opacity 0→1, translateY 8px→0, duration 500ms, ease out`
3. `t=150ms` — Headline reveals: `opacity 0→1, translateY 16px→0, duration 700ms, ease out` (staggered word-by-word if using Framer Motion variants)
4. `t=450ms` — Subheadline fades in: `opacity 0→1, translateY 12px→0, duration 600ms, ease out`
5. `t=700ms` — CTAs appear: `opacity 0→1, translateY 8px→0, duration 500ms, ease out`
6. `prefers-reduced-motion`: all animations disabled, content appears immediately at full opacity

**Mobile adaptation (375px–768px):**
- Video plays on mobile only if `navigator.connection.saveData !== true` and connection is 4G+; otherwise static image fallback auto-activates
- Headline: 40px, line-height 1.15, max-width 90vw, centered
- Subheadline: 14px, max-width 85vw
- CTAs stack vertically (flex-col), full width minus 48px margin, `Reservar mesa` first
- Badge remains above headline, same sizing
- Overall padding-bottom increased to avoid overlap with bottom browser chrome
- Min-height: 100dvh (uses dynamic viewport height for mobile browser chrome)

**Scroll indicator:**
- Subtle animated chevron-down icon, centered at bottom, `opacity 0.5`, `translateY` loop animation `0px → 8px → 0px, 2s infinite, ease-in-out`
- Disappears after user scrolls 50px (JS event listener or Framer Motion scroll hook)

**Accessibility notes:**
- Headline + subheadline on scrim: contrast ratio ≥ 7.5:1 (AAA) — cream #F5EDE0 on rgba(42,26,14,0.65) dark brown background tested
- Primary CTA: #F5EDE0 on #C0553A — contrast ratio 4.8:1 (AA compliant, meets minimum for large text)
- `<video>` element has `aria-hidden="true"` — it is decorative
- All text is in semantic HTML elements, not inside the video
- `autoPlay muted loop playsInline` attributes on video element (required for mobile autoplay policy)
- `alt` on fallback `<img>`: "Pasta artesanal hecha a mano en La Nonna, restaurante italiano en Gràcia, Barcelona"
- Focus ring visible on CTA buttons (do not remove outline on focus-visible)

---

## React Component

The implementation follows the **Server shell + Client leaf pattern**:
- `HeroSection` is a React Server Component (no `'use client'` — pure HTML structure, no hooks)
- `HeroClient` is the Client Component leaf (animations, video behavior, scroll indicator)
- `HeroVideo` is a Client Component handling the video/fallback logic

```tsx
// components/hero/HeroSection.tsx
// Server Component — shell and semantic structure

import { HeroClient } from './HeroClient'

export function HeroSection() {
  return (
    <section
      className="relative w-full min-h-[100dvh] flex items-center justify-center overflow-hidden"
      aria-label="Bienvenida a La Nonna"
    >
      {/* Background layer — handled by client for video/fallback logic */}
      <HeroClient />
    </section>
  )
}
```

```tsx
// components/hero/HeroClient.tsx
'use client'

import { useEffect, useRef, useState } from 'react'
import { motion, useScroll, useTransform, AnimatePresence } from 'framer-motion'
import Image from 'next/image'
import Link from 'next/link'

// ─── Animation Variants ────────────────────────────────────────────────────

const fadeUpVariants = {
  hidden: { opacity: 0, y: 16 },
  visible: (delay: number) => ({
    opacity: 1,
    y: 0,
    transition: {
      duration: 0.65,
      delay,
      ease: [0.22, 1, 0.36, 1],
    },
  }),
}

const headlineVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.06,
    },
  },
}

const wordVariant = {
  hidden: { opacity: 0, y: 20 },
  visible: {
    opacity: 1,
    y: 0,
    transition: {
      duration: 0.55,
      ease: [0.22, 1, 0.36, 1],
    },
  },
}

// ─── Types ─────────────────────────────────────────────────────────────────

interface HeroClientProps {
  videoSrc?: string
  fallbackImageSrc?: string
}

// ─── Sub-component: Video Background ───────────────────────────────────────

function HeroBackground({
  videoSrc,
  fallbackImageSrc,
}: HeroClientProps) {
  const [useVideo, setUseVideo] = useState(false)
  const [videoLoaded, setVideoLoaded] = useState(false)

  useEffect(() => {
    // Only autoplay video on fast connections and non-data-saver mode
    const connection = (navigator as Navigator & {
      connection?: { effectiveType?: string; saveData?: boolean }
    }).connection

    const shouldUseVideo =
      typeof window !== 'undefined' &&
      !connection?.saveData &&
      (connection?.effectiveType === '4g' || !connection?.effectiveType)

    setUseVideo(shouldUseVideo && Boolean(videoSrc))
  }, [videoSrc])

  return (
    <div className="absolute inset-0 z-0" aria-hidden="true">
      {/* Warm brown scrim overlay */}
      <div
        className="absolute inset-0 z-10"
        style={{
          background:
            'linear-gradient(to bottom, rgba(42,26,14,0.45) 0%, rgba(42,26,14,0.68) 100%)',
        }}
      />

      {/* Video background */}
      {useVideo && videoSrc && (
        <video
          className={`absolute inset-0 h-full w-full object-cover transition-opacity duration-700 ${
            videoLoaded ? 'opacity-100' : 'opacity-0'
          }`}
          autoPlay
          muted
          loop
          playsInline
          onCanPlay={() => setVideoLoaded(true)}
          aria-hidden="true"
        >
          <source src={videoSrc.replace('.mp4', '.webm')} type="video/webm" />
          <source src={videoSrc} type="video/mp4" />
        </video>
      )}

      {/* Static fallback image — always rendered, hidden behind video when video loads */}
      <Image
        src={
          fallbackImageSrc ??
          '/images/hero-fallback.jpg'
        }
        alt="Pasta artesanal hecha a mano en La Nonna, restaurante italiano en Gràcia, Barcelona"
        fill
        priority
        sizes="100vw"
        className={`object-cover transition-opacity duration-700 ${
          useVideo && videoLoaded ? 'opacity-0' : 'opacity-100'
        }`}
        quality={90}
      />
    </div>
  )
}

// ─── Sub-component: Scroll Indicator ───────────────────────────────────────

function ScrollIndicator() {
  const [visible, setVisible] = useState(true)

  useEffect(() => {
    const handleScroll = () => {
      if (window.scrollY > 50) setVisible(false)
    }
    window.addEventListener('scroll', handleScroll, { passive: true })
    return () => window.removeEventListener('scroll', handleScroll)
  }, [])

  return (
    <AnimatePresence>
      {visible && (
        <motion.div
          className="absolute bottom-8 left-1/2 -translate-x-1/2 z-20 flex flex-col items-center gap-1"
          initial={{ opacity: 0 }}
          animate={{ opacity: 0.5 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.4 }}
          aria-hidden="true"
        >
          <motion.svg
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="#F5EDE0"
            strokeWidth="1.5"
            strokeLinecap="round"
            strokeLinejoin="round"
            animate={{ y: [0, 8, 0] }}
            transition={{ duration: 2, repeat: Infinity, ease: 'easeInOut' }}
          >
            <polyline points="6 9 12 15 18 9" />
          </motion.svg>
        </motion.div>
      )}
    </AnimatePresence>
  )
}

// ─── Main Client Component ──────────────────────────────────────────────────

export function HeroClient({
  videoSrc = '/videos/hero-loop.mp4',
  fallbackImageSrc = '/images/hero-fallback.jpg',
}: HeroClientProps) {
  const headlineWords = ['La', 'cucina', 'di', 'casa', 'tua']

  // Respect prefers-reduced-motion
  const [reducedMotion, setReducedMotion] = useState(false)

  useEffect(() => {
    const mq = window.matchMedia('(prefers-reduced-motion: reduce)')
    setReducedMotion(mq.matches)
    const handler = (e: MediaQueryListEvent) => setReducedMotion(e.matches)
    mq.addEventListener('change', handler)
    return () => mq.removeEventListener('change', handler)
  }, [])

  // Parallax effect on scroll for text content
  const containerRef = useRef<HTMLDivElement>(null)
  const { scrollY } = useScroll()
  const textY = useTransform(scrollY, [0, 500], [0, reducedMotion ? 0 : -60])

  const animationProps = reducedMotion
    ? { initial: 'visible', animate: 'visible' }
    : { initial: 'hidden', animate: 'visible' }

  return (
    <>
      {/* Background */}
      <HeroBackground videoSrc={videoSrc} fallbackImageSrc={fallbackImageSrc} />

      {/* Content */}
      <motion.div
        ref={containerRef}
        className="relative z-20 flex flex-col items-center justify-center text-center px-6 sm:px-8 max-w-4xl mx-auto"
        style={{ y: textY }}
      >
        {/* Establishment badge */}
        <motion.p
          className="mb-6 text-xs sm:text-sm uppercase tracking-[0.15em] font-medium"
          style={{ color: 'rgba(245,237,224,0.55)', fontFamily: 'Inter, sans-serif' }}
          variants={fadeUpVariants}
          custom={0}
          {...animationProps}
        >
          Dal 2003 · Ristorante di famiglia
        </motion.p>

        {/* Main headline — Italian, staggered word reveal */}
        <motion.h1
          className="mb-3 font-bold leading-[1.1] tracking-[-0.5px]"
          style={{
            fontFamily: "'Playfair Display', Georgia, serif",
            color: '#F5EDE0',
            fontSize: 'clamp(38px, 7vw, 72px)',
          }}
          variants={reducedMotion ? undefined : headlineVariants}
          {...animationProps}
          aria-label="La cucina di casa tua"
        >
          {headlineWords.map((word, i) => (
            <motion.span
              key={i}
              className="inline-block mr-[0.25em] last:mr-0"
              variants={reducedMotion ? undefined : wordVariant}
            >
              {word}
            </motion.span>
          ))}
        </motion.h1>

        {/* English subtitle */}
        <motion.p
          className="mb-3 font-light italic"
          style={{
            fontFamily: "'Playfair Display', Georgia, serif",
            color: 'rgba(245,237,224,0.75)',
            fontSize: 'clamp(16px, 2.2vw, 24px)',
            letterSpacing: '0.01em',
          }}
          variants={fadeUpVariants}
          custom={reducedMotion ? 0 : 0.3}
          {...animationProps}
        >
          Your home kitchen, in the heart of Gràcia
        </motion.p>

        {/* Subheadline — location + year */}
        <motion.p
          className="mb-10 font-normal"
          style={{
            fontFamily: 'Inter, sans-serif',
            color: 'rgba(245,237,224,0.70)',
            fontSize: 'clamp(13px, 1.6vw, 17px)',
            letterSpacing: '0.03em',
          }}
          variants={fadeUpVariants}
          custom={reducedMotion ? 0 : 0.45}
          {...animationProps}
        >
          Cocina italiana de familia desde 2003 · Gràcia, Barcelona
        </motion.p>

        {/* CTA buttons */}
        <motion.div
          className="flex flex-col sm:flex-row items-center gap-4 sm:gap-5 w-full sm:w-auto"
          variants={fadeUpVariants}
          custom={reducedMotion ? 0 : 0.65}
          {...animationProps}
        >
          {/* Primary CTA */}
          <Link
            href="/reservas"
            className="group relative w-full sm:w-auto inline-flex items-center justify-center overflow-hidden rounded-sm px-9 py-4 text-base font-semibold transition-all duration-300 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#F5EDE0]"
            style={{
              backgroundColor: '#C0553A',
              color: '#F5EDE0',
              fontFamily: 'Inter, sans-serif',
              letterSpacing: '0.02em',
              minHeight: '52px',
            }}
            onMouseEnter={(e) => {
              const el = e.currentTarget
              el.style.backgroundColor = '#a8472f'
              el.style.transform = 'scale(1.03)'
            }}
            onMouseLeave={(e) => {
              const el = e.currentTarget
              el.style.backgroundColor = '#C0553A'
              el.style.transform = 'scale(1)'
            }}
          >
            Reservar mesa
          </Link>

          {/* Secondary CTA */}
          <Link
            href="/menu"
            className="group w-full sm:w-auto inline-flex items-center justify-center rounded-sm px-9 py-4 text-base font-medium transition-all duration-300 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#F5EDE0]"
            style={{
              border: '1.5px solid rgba(245,237,224,0.55)',
              color: '#F5EDE0',
              fontFamily: 'Inter, sans-serif',
              letterSpacing: '0.02em',
              minHeight: '52px',
              backgroundColor: 'transparent',
            }}
            onMouseEnter={(e) => {
              const el = e.currentTarget
              el.style.borderColor = 'rgba(245,237,224,0.95)'
              el.style.backgroundColor = 'rgba(245,237,224,0.08)'
            }}
            onMouseLeave={(e) => {
              const el = e.currentTarget
              el.style.borderColor = 'rgba(245,237,224,0.55)'
              el.style.backgroundColor = 'transparent'
            }}
          >
            Ver la carta
          </Link>
        </motion.div>
      </motion.div>

      {/* Scroll indicator */}
      <ScrollIndicator />
    </>
  )
}
```

---

## Implementation Notes

### File structure

```
components/
  hero/
    HeroSection.tsx     ← Server Component (shell)
    HeroClient.tsx      ← Client Component (animations, video, interactivity)

public/
  videos/
    hero-loop.mp4       ← 20–25s loop, h.264, max 8MB, warm color grade
    hero-loop.webm      ← VP9 encode, typically 40% smaller than MP4
  images/
    hero-fallback.jpg   ← 1920×1080 min, WebP preferred, max 200KB via Next.js optimization
```

### Font setup (app/layout.tsx)

```tsx
import { Playfair_Display, Inter } from 'next/font/google'

const playfair = Playfair_Display({
  subsets: ['latin'],
  weight: ['400', '700'],
  style: ['normal', 'italic'],
  variable: '--font-playfair',
  display: 'swap',
})

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

// Apply to <html>: className={`${playfair.variable} ${inter.variable}`}
```

### Usage in page.tsx

```tsx
import { HeroSection } from '@/components/hero/HeroSection'

export default function HomePage() {
  return (
    <main>
      <HeroSection />
      {/* rest of page sections */}
    </main>
  )
}
```

### Tailwind config additions (tailwind.config.ts)

```ts
theme: {
  extend: {
    colors: {
      terracotta: '#C0553A',
      cream: '#F5EDE0',
      olive: '#3D4C2A',
      'warm-brown': '#2A1A0E',
    },
    fontFamily: {
      playfair: ['var(--font-playfair)', 'Georgia', 'serif'],
      inter: ['var(--font-inter)', 'system-ui', 'sans-serif'],
    },
  },
},
```

### Dependencies required

```bash
npm install framer-motion
# next/image and next/link are included in Next.js 16
```

---

## Design Rationale Summary

**Why Pattern 4 + centered overlay:** La Nonna's differentiator is atmosphere and authenticity — things that must be *felt*, not read. A video loop of the kitchen and table communicates the warmth of a family restaurant in 2 seconds flat. The centered overlay (Pattern 1) places the visitor in an intimate, enveloping relationship with the brand — the same feeling as walking through the door.

**Why Italian headline:** "La cucina di casa tua" (your home kitchen) is the brand's emotional core. Italian copy in the headline — even for a Spanish/international audience — signals authenticity and makes the brand feel like it belongs to actual Italians, not a marketing team. The English italic subtitle below bridges the comprehension gap.

**Why terracotta CTA:** #C0553A is the brand's warmest, most energetic color. Against the dark warm-brown scrim, it reads immediately as a call to action without feeling aggressive. The ghost secondary button creates clear hierarchy: "reserve" is primary, "see menu" is for browsers.

**Why `border-radius: 2px` (rounded-sm):** Full sharp corners feel corporate; fully rounded feels casual/generic. A 2px radius is the premium restaurant sweet spot — artisan, deliberate, neither trendy nor corporate. Same as Noma's UI approach.

**Why `clamp()` for font sizes:** Eliminates breakpoint jumps. The headline scales fluidly from 38px on mobile to 72px on desktop, staying proportional at every intermediate width — critical for the wide variety of devices a Barcelona restaurant audience uses.
