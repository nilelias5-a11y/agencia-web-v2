# La Nonna — Hero Section: Complete Spec & React Component

## 1. Layout Choice & Rationale

**Layout: Full-viewport split — left text panel (45%) + right full-bleed image (55%), stacked on mobile**

Why this layout for La Nonna:

- **Authenticity over spectacle.** A full-screen background image with overlay text is a tourist-trap cliché. The split layout lets the food/atmosphere image breathe without drowning the message, and gives the brand voice room to lead.
- **CTA hierarchy.** The reservation CTA is the primary business goal. A dedicated text panel (no overlay blur, no contrast hacks) gives the CTA maximum legibility and click target area, especially on mobile.
- **Neighborhood warmth.** The cream panel on the left anchors the page in brand color, evoking a warm kitchen wall. The image on the right is a secondary witness, not the hero.
- **Mobile stacked behavior.** On small screens the image collapses to a 40vh cinematic strip above the text block, keeping the food visible without sacrificing CTA accessibility.

---

## 2. Headline Strategy

**Primary headline (H1):**
> La cocina de la nonna,  
> en el corazón de Gràcia.

**Why:**
- Bilingual — Spanish/Italian hybrid mirrors how Barcelonans actually speak about their barrio. Creates instant neighbourhood belonging.
- "La nonna" is the name and the concept — it doesn't need explanation.
- "Corazón de Gràcia" is geographic, emotional, and positioning in one shot. Not "authentic Italian" (tourist bait) — just where this family cooks.

**Subheadline (body text):**
> Cocina italiana de familia desde 2003. Sin prisas. Reserva tu mesa.

- Three micro-sentences. Each earns its place.
- "Sin prisas" (no rush) is the counter-positioning against tourist restaurants. It signals: you can stay, you belong here.
- Ends with a soft CTA echo before the button.

---

## 3. CTA Strategy

**Primary CTA:** "Reservar mesa" — terracotta button, full-width on mobile, prominent on desktop.

**Secondary CTA:** "Ver la carta" — ghost/outline button. Lower visual weight. Serves visitors who aren't ready to commit but want to build appetite before deciding.

**CTA logic:**
- Primary drives the reservation funnel (the business goal).
- Secondary reduces bounce by giving undecided users a next action. Viewing the menu increases conversion intent — so it's a soft funnel, not a distraction.
- No third CTA. Two is the maximum before choice paralysis sets in.

**Trust signal below CTAs:**
> Abierto martes–domingo · Carrer de Verdi, 34 · Tel. 932 XX XX XX

Low-prominence inline text. Answers the "is this real?" question for first-time visitors without visual noise.

---

## 4. Animation Plan (Framer Motion)

**Philosophy:** Slow, warm, unhurried — matches "sin prisas" brand voice. No bounce, no spring, no fast snap. Everything eases in like bread coming out of an oven.

| Element | Animation | Timing |
|---|---|---|
| Left panel background | Fade in from `opacity: 0` | 0ms delay, 800ms duration |
| H1 (line 1) | Slide up 24px + fade in | 200ms delay, 900ms |
| H1 (line 2) | Slide up 24px + fade in | 400ms delay, 900ms |
| Subheadline | Fade in | 700ms delay, 700ms |
| Primary CTA | Fade in + slight scale from 0.97 | 900ms delay, 600ms |
| Secondary CTA | Fade in | 1000ms delay, 600ms |
| Trust signal | Fade in | 1100ms delay, 500ms |
| Right image | Fade in + subtle scale from 1.03→1.0 | 100ms delay, 1200ms |

**Hover states:**
- Primary CTA: background darkens 10%, slight lift (translateY -2px), 200ms ease
- Secondary CTA: border + text shifts to terracotta, 200ms ease
- Image panel: very subtle scale to 1.02 over 800ms (Ken Burns lite, loop)

**Reduced motion:** All animations respect `prefers-reduced-motion` via Framer Motion's `useReducedMotion()`.

---

## 5. Mobile Behavior

- **Breakpoint:** `md` (768px) is the split point
- Below `md`:
  - Image becomes a 40vh strip at the top, `object-fit: cover`, `object-position: center 30%` (keeps faces/food in frame)
  - Text panel fills remaining viewport
  - CTAs stack vertically, both full-width
  - H1 drops from `text-5xl` to `text-3xl`
  - Trust signal moves below secondary CTA in smaller font
- Sticky mobile CTA bar considered but rejected — the viewport is short enough that the CTA is always reachable on first scroll.

---

## 6. Image Guidance

Recommended photo: close-mid shot of a pasta dish or family-style spread on a rustic wooden table, warm tungsten lighting, shallow depth of field. Avoid: stock-photo perfect plating, white tablecloths, empty restaurant shots at dusk (all tourist clichés).

For this component, a placeholder from Unsplash is used. Replace `src` with the actual asset path.

---

## 7. Full React/TypeScript Component

```tsx
// components/HeroSection.tsx
"use client";

import { useReducedMotion, motion } from "framer-motion";
import Link from "next/link";
import Image from "next/image";

// ─── Animation variants ────────────────────────────────────────────────────

const fadeUp = (delay = 0) => ({
  hidden: { opacity: 0, y: 24 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.9, delay, ease: [0.22, 1, 0.36, 1] },
  },
});

const fadeIn = (delay = 0, duration = 0.7) => ({
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { duration, delay, ease: "easeOut" },
  },
});

const imageVariant = {
  hidden: { opacity: 0, scale: 1.03 },
  visible: {
    opacity: 1,
    scale: 1,
    transition: { duration: 1.2, delay: 0.1, ease: [0.22, 1, 0.36, 1] },
  },
};

// ─── Component ─────────────────────────────────────────────────────────────

export default function HeroSection() {
  const shouldReduceMotion = useReducedMotion();

  // When reduced motion is preferred, skip all animation variants
  const animate = shouldReduceMotion ? "visible" : undefined;
  const initial = shouldReduceMotion ? "visible" : "hidden";
  const whileInView = shouldReduceMotion ? undefined : "visible";
  const viewport = { once: true, amount: 0.1 };

  return (
    <section
      aria-label="Bienvenida a La Nonna"
      className="relative flex min-h-screen w-full flex-col md:flex-row overflow-hidden bg-[#F5EDE0]"
    >
      {/* ── Mobile image strip (visible below md) ─────────────────────────── */}
      <motion.div
        className="relative h-[40vh] w-full flex-shrink-0 md:hidden"
        variants={imageVariant}
        initial={initial}
        animate={animate}
        whileInView={whileInView}
        viewport={viewport}
      >
        <Image
          src="https://images.unsplash.com/photo-1555396273-367ea4eb4db5?w=800&q=80"
          alt="Pasta artesanal en La Nonna, Gràcia"
          fill
          priority
          className="object-cover object-[center_30%]"
          sizes="100vw"
        />
        {/* Subtle gradient at bottom to blend into cream panel */}
        <div className="absolute inset-x-0 bottom-0 h-16 bg-gradient-to-t from-[#F5EDE0] to-transparent" />
      </motion.div>

      {/* ── Left: text panel ───────────────────────────────────────────────── */}
      <div className="relative z-10 flex flex-1 flex-col justify-center px-8 py-16 md:px-16 md:py-24 lg:px-24 xl:pl-32 xl:pr-16 md:max-w-[55%]">
        
        {/* Established badge */}
        <motion.p
          variants={fadeIn(0, 0.8)}
          initial={initial}
          animate={animate}
          whileInView={whileInView}
          viewport={viewport}
          className="mb-6 font-['Inter'] text-xs font-semibold uppercase tracking-[0.2em] text-[#3D4C2A]"
        >
          Desde 2003 · Gràcia, Barcelona
        </motion.p>

        {/* H1 — two lines, staggered */}
        <h1 className="font-['Playfair_Display'] text-[2.6rem] font-bold leading-[1.1] text-[#2A1A0E] md:text-5xl lg:text-6xl">
          <motion.span
            className="block"
            variants={fadeUp(0.2)}
            initial={initial}
            animate={animate}
            whileInView={whileInView}
            viewport={viewport}
          >
            La cocina de la nonna,
          </motion.span>
          <motion.span
            className="block text-[#C0553A]"
            variants={fadeUp(0.4)}
            initial={initial}
            animate={animate}
            whileInView={whileInView}
            viewport={viewport}
          >
            en el corazón de Gràcia.
          </motion.span>
        </h1>

        {/* Subheadline */}
        <motion.p
          variants={fadeIn(0.7)}
          initial={initial}
          animate={animate}
          whileInView={whileInView}
          viewport={viewport}
          className="mt-6 max-w-sm font-['Inter'] text-base leading-relaxed text-[#2A1A0E]/70 md:text-lg"
        >
          Cocina italiana de familia desde 2003.
          <br />
          Sin prisas. Reserva tu mesa.
        </motion.p>

        {/* CTA group */}
        <motion.div
          variants={fadeIn(0.9, 0.6)}
          initial={initial}
          animate={animate}
          whileInView={whileInView}
          viewport={viewport}
          className="mt-10 flex w-full flex-col gap-4 sm:flex-row md:w-auto"
        >
          {/* Primary CTA */}
          <Link
            href="/reservas"
            className="
              group inline-flex items-center justify-center
              rounded-sm bg-[#C0553A] px-8 py-4
              font-['Inter'] text-sm font-semibold uppercase tracking-widest text-[#F5EDE0]
              transition-all duration-200
              hover:-translate-y-0.5 hover:bg-[#A8472E] hover:shadow-lg hover:shadow-[#C0553A]/30
              focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#C0553A]
              w-full sm:w-auto
            "
          >
            Reservar mesa
          </Link>

          {/* Secondary CTA */}
          <Link
            href="/carta"
            className="
              inline-flex items-center justify-center
              rounded-sm border border-[#2A1A0E]/30 px-8 py-4
              font-['Inter'] text-sm font-semibold uppercase tracking-widest text-[#2A1A0E]/70
              transition-all duration-200
              hover:border-[#C0553A] hover:text-[#C0553A]
              focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#C0553A]
              w-full sm:w-auto
            "
          >
            Ver la carta
          </Link>
        </motion.div>

        {/* Trust signal */}
        <motion.p
          variants={fadeIn(1.1, 0.5)}
          initial={initial}
          animate={animate}
          whileInView={whileInView}
          viewport={viewport}
          className="mt-8 font-['Inter'] text-xs text-[#2A1A0E]/45"
        >
          Abierto mar–dom · Carrer de Verdi, 34 · 932 XX XX XX
        </motion.p>
      </div>

      {/* ── Right: image panel (desktop only) ─────────────────────────────── */}
      <motion.div
        className="relative hidden md:block md:w-[55%] flex-shrink-0"
        variants={imageVariant}
        initial={initial}
        animate={animate}
        whileInView={whileInView}
        viewport={viewport}
      >
        <Image
          src="https://images.unsplash.com/photo-1555396273-367ea4eb4db5?w=1200&q=85"
          alt="Pasta artesanal en La Nonna, Gràcia"
          fill
          priority
          className="object-cover object-center transition-transform duration-[800ms] ease-out hover:scale-[1.02]"
          sizes="55vw"
        />
        {/* Left-side gradient: blends image into cream panel */}
        <div className="absolute inset-y-0 left-0 w-24 bg-gradient-to-r from-[#F5EDE0] to-transparent" />
      </motion.div>
    </section>
  );
}
```

---

## 8. Usage Notes

### Dependencies
```bash
npm install framer-motion
```

### Google Fonts (layout.tsx or globals.css)
```tsx
// app/layout.tsx
import { Playfair_Display, Inter } from "next/font/google";

const playfair = Playfair_Display({
  subsets: ["latin"],
  variable: "--font-playfair",
  weight: ["400", "700", "900"],
});

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
});
```

Or via Tailwind config to reference as `font-['Playfair_Display']` (already used inline in the component for portability).

### Next.js Image domains
If using the Unsplash placeholder, add to `next.config.js`:
```js
images: {
  remotePatterns: [{ hostname: "images.unsplash.com" }],
}
```
Replace with local `/public/hero.jpg` for production.

### Tailwind config — custom colors (optional but recommended)
```js
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      terracotta: "#C0553A",
      cream: "#F5EDE0",
      olive: "#3D4C2A",
      espresso: "#2A1A0E",
    },
  },
},
```

---

## 9. Design Decisions Summary

| Decision | Choice | Rationale |
|---|---|---|
| Layout | Split panel (text left, image right) | Avoids tourist-cliché full-bleed overlay; gives CTA maximum visibility |
| Headline language | Spanish (with Italian restaurant name) | Neighborhood-first positioning; locals, not tourists |
| Primary CTA | "Reservar mesa" | Plain, direct, action-first; no friction word like "hacer" |
| Animation speed | Slow ease (800–1200ms) | "Sin prisas" brand voice in motion |
| Mobile image | 40vh strip, not hidden | Keeps the food visible — appetite drives reservations |
| Color use | Terracotta for H1 accent + CTA | Brand color does double duty: identity + conversion |
| Est. badge | "Desde 2003" above H1 | Builds trust without cluttering the headline |
| Trust signal | Below CTAs, low contrast | Present for skeptics, invisible to decided visitors |
