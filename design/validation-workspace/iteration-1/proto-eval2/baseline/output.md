# Development Briefing — Ana López Photography Landing Page
**Prepared by:** Design & Handoff Team
**Date:** 2026-05-13
**For:** Frontend Developer
**Status:** Ready for Development (Iteration 1 — Baseline)

---

## 1. Project Overview & Tech Stack

### Client
Ana López — Freelance photographer based in Barcelona. Specializes in weddings, portraits, and corporate photography. The landing page is the primary commercial touchpoint: it must convey premium quality, emotional resonance, and convert visitors into booking inquiries.

### Goal
A single-page portfolio site that loads fast, looks stunning on every device, and guides the visitor from first impression (Hero) to conversion (Contact Form) without friction.

### Confirmed Tech Stack

| Layer | Technology | Version | Notes |
|---|---|---|---|
| Framework | Next.js | 14 (App Router) | SSG preferred; no dynamic server routes needed |
| Styling | Tailwind CSS | 3.x | Use JIT mode; custom design tokens required |
| Animation | Framer Motion | 11.x | Page-level + scroll-triggered + hover animations |
| Email / Contact | Resend | Latest | Server Action in Next.js App Router |
| Language | TypeScript | 5.x | Strict mode on |
| Image optimization | next/image | Built-in | Critical for photography site performance |
| Fonts | next/font | Built-in | Google Fonts via next/font/google |
| Deployment | Vercel | — | Assumed; aligns with Next.js defaults |

### Repository Conventions
- `src/app/` — App Router pages and layouts
- `src/components/` — Reusable UI components (one file per component)
- `src/sections/` — Full-page sections (Hero, Portfolio, About, etc.)
- `src/lib/` — Utility functions, Resend config, constants
- `src/types/` — Shared TypeScript types
- `public/images/` — Static photography assets (optimized before commit)
- `src/styles/globals.css` — Tailwind base + custom CSS variables

---

## 2. Build Order & Rationale

Build in this exact sequence. Each step creates a stable foundation for the next.

### Phase 0 — Project Scaffolding (Day 1, Morning)
1. `npx create-next-app@14 ana-lopez --typescript --tailwind --app --src-dir --import-alias "@/*"`
2. Install dependencies: `framer-motion resend react-hook-form zod @hookform/resolvers`
3. Configure `tailwind.config.ts`: extend theme with brand colors, custom font families, and spacing tokens (see Section 3).
4. Set up `next.config.ts`: enable `images.remotePatterns` if using external CDN, configure `images.formats: ['image/avif', 'image/webp']`.
5. Create `globals.css` with CSS custom properties for brand palette.
6. Add font configuration in `src/app/layout.tsx` using `next/font/google`.
7. Create `.env.local` with `RESEND_API_KEY` placeholder.

**Why first:** All subsequent work imports from this baseline. Getting fonts, colors, and image config right here avoids cascading refactors.

### Phase 1 — Layout Shell & Navigation (Day 1, Afternoon)
1. Build `src/app/layout.tsx` — root layout with `<html>`, `<body>`, font class injection.
2. Build `src/app/page.tsx` — single page that renders all 6 section components in order.
3. Build `<Header>` component — sticky/transparent-on-hero navbar with logo, anchor links, and mobile hamburger. Use Framer Motion for show/hide on scroll direction.
4. Build `<Footer>` component — minimal: copyright, social links (Instagram, LinkedIn).

**Why second:** Header/footer wrap every section. Building the shell first means sections can be dropped in without layout rework.

### Phase 2 — Hero Section (Day 2)
Full-screen background image, headline, sub-headline, CTA button.

**Why third:** It is the most visually complex single component. Resolving image loading, overlay gradients, and viewport height behavior (especially on mobile Safari) early prevents surprises later.

### Phase 3 — Portfolio Grid Section (Day 2–3)
12-photo grid, 3 columns on desktop, hover overlay with title.

**Why fourth:** The grid is the core product demonstration. It uses `next/image`, Framer Motion hover states, and a lightbox consideration — resolving these interaction patterns here establishes reusable patterns for the rest of the site.

### Phase 4 — About Section (Day 3)
Photo of Ana + biography text, side-by-side layout.

**Why fifth:** Straightforward layout; no new interaction patterns. A palette-cleanser between the complex grid and cards.

### Phase 5 — Services Section (Day 3–4)
3 cards: Weddings, Portraits, Corporate. Each has icon/image, title, short description, optional price range or CTA.

**Why sixth:** Cards use consistent component pattern. Build a single `<ServiceCard>` component and render it 3 times from a data array — this tests the component abstraction strategy.

### Phase 6 — Testimonials Section (Day 4)
3 quote cards with client name, role, and star rating or decorative quote mark.

**Why seventh:** Similar card pattern to Services; implementation should be fast given Phase 5 patterns.

### Phase 7 — Contact Form Section (Day 4–5)
Name, email, service interest (select), message, submit button. Integrated with Resend via Next.js Server Action.

**Why last:** Requires `.env.local` setup, a Server Action, form validation with Zod, and UI feedback states (loading, success, error). Keeping it last means it does not block visual progress while API credentials are confirmed.

### Phase 8 — Polish & QA (Day 5)
- Scroll-triggered animations on all sections (Framer Motion `useInView` / `whileInView`).
- Responsive audit: 375px, 768px, 1280px, 1440px breakpoints.
- Accessibility audit (see Section 5).
- Lighthouse run and performance optimization (see Section 6).
- Cross-browser check: Chrome, Safari, Firefox, Safari iOS.

---

## 3. Critical Implementation Notes

### 3.1 Tailwind Configuration — Brand Tokens

```ts
// tailwind.config.ts — extend section
theme: {
  extend: {
    colors: {
      brand: {
        cream:    '#F5F0EB',  // primary background
        charcoal: '#1C1C1C',  // primary text
        gold:     '#C9A96E',  // accent / CTA
        muted:    '#6B6B6B',  // secondary text
      },
    },
    fontFamily: {
      serif:  ['var(--font-cormorant)', 'Georgia', 'serif'],  // headings
      sans:   ['var(--font-inter)',     'system-ui', 'sans-serif'], // body
    },
    spacing: {
      section: '6rem',   // vertical padding between sections on desktop
      'section-sm': '3rem', // vertical padding on mobile
    },
  },
}
```

### 3.2 Hero Section — Critical Gotchas

- Use `height: 100svh` (small viewport height) NOT `100vh` to avoid the mobile Safari address bar overlap. Implement via a Tailwind arbitrary value: `h-[100svh]` or add `svh` to the Tailwind config.
- The background image MUST use `<Image>` with `fill` prop and `priority` attribute (prevents LCP penalty).
- Overlay: use a CSS gradient `from-black/60 via-black/30 to-transparent` rather than a flat color — it preserves mid-image detail.
- CTA button should be an `<a href="#contact">` anchor, NOT a Next.js `<Link>` (same-page scroll, not navigation).
- Add `fetchPriority="high"` via the `Image` component's `priority` prop.

```tsx
// Hero image — correct pattern
<Image
  src="/images/hero.jpg"
  alt="Ana López — fotografía profesional en Barcelona"
  fill
  priority
  className="object-cover object-center"
  sizes="100vw"
  quality={85}
/>
```

### 3.3 Portfolio Grid — Hover Overlay

- Grid layout: `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3` — never hardcode 3 columns for mobile.
- Each photo cell is `position: relative; overflow: hidden` to contain the overlay.
- Hover overlay uses Framer Motion `variants` with `opacity: 0` → `opacity: 1` and a subtle `scale` on the image (1.0 → 1.05) for depth.
- All 12 images need `sizes` prop tuned to grid column width: `sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"`.
- If a lightbox is desired in v2, plan component interface now: `onOpenLightbox(index: number)` prop on each card. Do not build it now but leave the hook.
- Photo data should come from a typed array in `src/lib/portfolio-data.ts` — never hardcode in JSX.

```ts
// src/lib/portfolio-data.ts
export interface PortfolioPhoto {
  id: string;
  src: string;
  alt: string;
  category: 'wedding' | 'portrait' | 'corporate';
  title: string;
}

export const portfolioPhotos: PortfolioPhoto[] = [
  // 12 entries
];
```

### 3.4 Framer Motion — Scroll Animations

Use `whileInView` for section entry animations. Apply to the section wrapper, not individual elements, to keep the animation budget manageable.

```tsx
// Standard section entry pattern — reuse across all sections
<motion.section
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, amount: 0.2 }}
  transition={{ duration: 0.6, ease: 'easeOut' }}
>
```

- `viewport={{ once: true }}` is mandatory — re-triggering animations on scroll-up is visually noisy and hurts perceived performance.
- For staggered children (Services cards, Testimonial cards), use `staggerChildren: 0.15` on the parent `variants` object.
- Respect `prefers-reduced-motion`. Wrap all motion components:

```tsx
// src/hooks/useReducedMotion.ts
import { useReducedMotion } from 'framer-motion';
// Use this hook in animated components; if true, skip transition props
```

### 3.5 Contact Form — Server Action with Resend

The form must NOT require a full page reload. Use a React Server Action called from a Client Component.

```tsx
// src/app/actions/send-email.ts  (Server Action)
'use server';
import { Resend } from 'resend';
import { z } from 'zod';

const resend = new Resend(process.env.RESEND_API_KEY);

const ContactSchema = z.object({
  name:    z.string().min(2).max(100),
  email:   z.string().email(),
  service: z.enum(['bodas', 'retratos', 'corporativo', 'otro']),
  message: z.string().min(10).max(2000),
});

export async function sendContactEmail(formData: FormData) {
  const parsed = ContactSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    return { success: false, error: 'Datos del formulario inválidos.' };
  }
  try {
    await resend.emails.send({
      from:    'web@analopez.com',        // must be verified domain in Resend
      to:      'ana@analopez.com',        // Ana's inbox
      replyTo: parsed.data.email,
      subject: `Nueva consulta: ${parsed.data.service}`,
      html: `<p><strong>Nombre:</strong> ${parsed.data.name}</p>
             <p><strong>Email:</strong> ${parsed.data.email}</p>
             <p><strong>Servicio:</strong> ${parsed.data.service}</p>
             <p><strong>Mensaje:</strong> ${parsed.data.message}</p>`,
    });
    return { success: true };
  } catch {
    return { success: false, error: 'Error al enviar. Inténtalo de nuevo.' };
  }
}
```

- The Client Component (`<ContactForm>`) uses `react-hook-form` + `zodResolver` for real-time client-side validation.
- Use `useFormState` (React 19 / Next.js 14) or `useTransition` to handle the async Server Action call.
- UI states required: **idle**, **loading** (disable button, show spinner), **success** (replace form with thank-you message), **error** (show inline error, re-enable form).
- Add honeypot field (`<input name="hp" style={{ display: 'none' }} />`) as basic spam prevention.
- Rate limiting is out of scope for v1 but document the hook point in code comments.

### 3.6 next/image Configuration

```ts
// next.config.ts
const nextConfig = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [375, 640, 768, 1024, 1280, 1440, 1920],
    imageSizes: [16, 32, 64, 128, 256],
    minimumCacheTTL: 31536000, // 1 year for photography assets
  },
};
```

All photography source files should be delivered as high-quality JPEGs (≥ 2000px on longest edge). `next/image` will handle AVIF/WebP conversion and responsive serving.

### 3.7 Typography Scale

```css
/* Design token reference — implement via Tailwind utilities */
/* Display: Cormorant Garamond, 600 weight, tracking -0.02em */
/* H1: 4.5rem / 72px desktop → 2.5rem / 40px mobile */
/* H2: 3rem / 48px desktop → 2rem / 32px mobile */
/* H3: 1.5rem / 24px */
/* Body: Inter, 400 weight, 1rem / 16px, line-height 1.7 */
/* Caption: Inter, 400 weight, 0.875rem / 14px */
/* CTA Label: Inter, 600 weight, 0.875rem, letter-spacing 0.1em, UPPERCASE */
```

### 3.8 Section IDs for Anchor Navigation

The header links must resolve. Ensure every section root element has the correct ID:

| Section | ID |
|---|---|
| Hero | `hero` |
| Portfolio | `portfolio` |
| About | `sobre-mi` |
| Services | `servicios` |
| Testimonials | `testimonios` |
| Contact | `contacto` |

Use Spanish IDs to match the language of the site.

### 3.9 Mobile Navigation

- Hamburger menu triggers a full-screen overlay nav (not a sidebar drawer) — this pattern works better for portrait photography sites.
- Use Framer Motion `AnimatePresence` for mount/unmount of the overlay.
- Close the overlay on any anchor link click (add `onClick` handler to each nav item).
- Trap focus inside the overlay when open (ARIA requirement — see Section 5).

---

## 4. Third-Party Integrations

### 4.1 Resend (Email)

| Item | Detail |
|---|---|
| Package | `resend` (official Node.js SDK) |
| Env var | `RESEND_API_KEY` in `.env.local` and Vercel environment |
| Sender domain | Must be verified in Resend dashboard BEFORE launch (DNS TXT + MX records) |
| From address | `web@analopez.com` — confirm with client |
| To address | `ana@analopez.com` — confirm with client |
| Test mode | Use `onboarding@resend.dev` as from address during development |
| Rate limit | Resend free tier: 100 emails/day, 3,000/month — sufficient for initial launch |
| Error logging | Log failures server-side; do not expose Resend error details to the client |

**Development workflow:** During local development, use `onboarding@resend.dev` as the sender (Resend allows this without domain verification). Switch to the client's verified domain before deploying to production.

### 4.2 Vercel (Deployment)

| Item | Detail |
|---|---|
| Framework preset | Next.js (auto-detected) |
| Build command | `next build` (default) |
| Node.js version | 20.x (set in Vercel dashboard) |
| Environment vars | `RESEND_API_KEY` — add to Production, Preview, and Development environments |
| Analytics | Enable Vercel Analytics (free tier) — one line in `layout.tsx` |
| Image optimization | Provided by Vercel's built-in Image Optimization API |

### 4.3 Google Fonts (via next/font)

Using `next/font/google` means fonts are self-hosted by Next.js — no third-party DNS lookup at runtime, zero GDPR concerns.

```tsx
// src/app/layout.tsx
import { Cormorant_Garamond, Inter } from 'next/font/google';

const cormorant = Cormorant_Garamond({
  subsets: ['latin'],
  weight: ['300', '400', '600'],
  variable: '--font-cormorant',
  display: 'swap',
});

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
});
```

### 4.4 Optional: Analytics (Post-Launch)

If the client wants visitor analytics, recommend **Vercel Analytics** (privacy-friendly, GDPR-compliant out of the box, no cookie banner required) over Google Analytics for a photography portfolio. Do not implement unless client confirms.

---

## 5. Accessibility Checklist

Every item below must pass before launch. Mark as done in code review.

### 5.1 Images & Media
- [ ] Every `<Image>` has a meaningful, descriptive `alt` attribute (describe the photo subject and context — not "photo 1")
- [ ] Hero image `alt` describes the scene: e.g., `"Pareja de novios en el Port Olímpic de Barcelona, fotografiados por Ana López"`
- [ ] Decorative images (e.g., background textures) have `alt=""` and `role="presentation"`
- [ ] No image conveys information that is not also available as text

### 5.2 Semantic HTML
- [ ] Page has exactly one `<h1>` (in the Hero section — Ana's name or tagline)
- [ ] Sections use proper heading hierarchy: `<h1>` → `<h2>` (section titles) → `<h3>` (card titles)
- [ ] Every section is wrapped in `<section>` with an `aria-label` or `aria-labelledby` pointing to its heading
- [ ] Navigation uses `<nav aria-label="Menú principal">`
- [ ] Footer uses `<footer>`
- [ ] Contact form uses `<form>` with `<fieldset>` and `<legend>` if grouping related fields

### 5.3 Keyboard Navigation
- [ ] All interactive elements (links, buttons, form inputs, select) are reachable via Tab key
- [ ] Tab order follows visual reading order (left-to-right, top-to-bottom)
- [ ] Mobile nav overlay traps focus when open — focus cannot escape to content behind the overlay
- [ ] Mobile nav overlay returns focus to the hamburger button when closed
- [ ] Portfolio grid items are focusable and show a visible focus ring (not just hover state)
- [ ] CTA button in Hero has a visible focus ring with sufficient contrast

### 5.4 Color & Contrast
- [ ] All body text (Inter, cream background) meets WCAG AA: minimum 4.5:1 contrast ratio
- [ ] Gold accent (`#C9A96E`) on dark background meets 3:1 for large text / UI components
- [ ] Error states in the contact form use both color AND an icon/text label (never color alone)
- [ ] Focus rings are visible: use `outline: 2px solid #C9A96E` or equivalent with `outline-offset: 2px`

### 5.5 Forms
- [ ] Every input has an associated `<label>` element (not just placeholder text)
- [ ] Required fields are marked with `aria-required="true"` AND a visible indicator
- [ ] Validation errors are announced to screen readers via `aria-live="polite"` or `role="alert"`
- [ ] Form success message is focused or announced after submission
- [ ] The submit button text is descriptive: "Enviar consulta" (not just "Enviar")

### 5.6 Motion & Animation
- [ ] All Framer Motion animations respect `prefers-reduced-motion` media query
- [ ] No content flashes or blinks more than 3 times per second (seizure safety)
- [ ] Parallax effects (if any) are disabled at `prefers-reduced-motion: reduce`

### 5.7 Language & Structure
- [ ] `<html lang="es">` is set in root layout
- [ ] Page has a descriptive `<title>` tag: "Ana López | Fotógrafa profesional en Barcelona"
- [ ] Meta description is set (≤ 160 characters) via Next.js `generateMetadata`
- [ ] Skip-to-content link is the first focusable element on the page (visually hidden until focused)

```tsx
// Skip link — place at top of layout.tsx body content
<a
  href="#hero"
  className="sr-only focus:not-sr-only focus:fixed focus:top-4 focus:left-4 focus:z-50 focus:px-4 focus:py-2 focus:bg-brand-gold focus:text-brand-charcoal focus:rounded"
>
  Saltar al contenido principal
</a>
```

### 5.8 Testing Tools
- Run **axe DevTools** (browser extension) — aim for 0 critical / 0 serious violations.
- Tab through the entire page manually in Chrome without a mouse.
- Test with VoiceOver (macOS/iOS) or NVDA (Windows) for screen reader smoke test.

---

## 6. Performance Targets

Ana López's audience will include clients browsing on mobile, often from Instagram links. Performance is a direct conversion factor.

### 6.1 Core Web Vitals — Required Targets

| Metric | Target | Threshold (Fail) |
|---|---|---|
| LCP (Largest Contentful Paint) | < 2.0s | > 2.5s |
| FID / INP (Interaction to Next Paint) | < 100ms | > 200ms |
| CLS (Cumulative Layout Shift) | < 0.05 | > 0.1 |
| FCP (First Contentful Paint) | < 1.2s | > 1.8s |
| TTFB (Time to First Byte) | < 200ms | > 600ms |

**Lighthouse Score Targets:** Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 95, SEO ≥ 95.

### 6.2 Image Optimization Strategy

Photography sites live or die by image performance. Follow this checklist:

- [ ] Hero image: provide a pre-optimized source at 2560px wide, ≤ 500KB as JPEG 85. `next/image` will serve AVIF/WebP.
- [ ] Portfolio grid images: source at 1200px wide, ≤ 200KB each. 12 images × 200KB = 2.4MB source; Next.js will serve appropriately sized variants per device.
- [ ] About section photo: 800px wide, ≤ 150KB.
- [ ] Use `placeholder="blur"` on all `<Image>` components — provide a `blurDataURL` (base64 tiny JPEG) for instant low-quality image placeholder (LQIP) while the full image loads.
- [ ] Set `loading="lazy"` on all images below the fold (everything except Hero). `next/image` does this by default; ensure `priority` is NOT set on non-hero images.
- [ ] Never use `<img>` tags — always `next/image`.

### 6.3 JavaScript Bundle Strategy

- [ ] All section components that use Framer Motion should be Client Components (`'use client'`). Keep Server Components for static sections where no interactivity is needed (About, Testimonials).
- [ ] Import Framer Motion with tree-shaking: `import { motion } from 'framer-motion'` — do not import the entire library.
- [ ] Use `dynamic(() => import(...), { ssr: false })` for any heavy third-party widget added in the future (not current scope).
- [ ] Analyze bundle with `next build` output and `@next/bundle-analyzer` if bundle > 200KB gzipped.

### 6.4 Font Loading Strategy

- [ ] `next/font` with `display: 'swap'` ensures text is visible during font load.
- [ ] Only load the font weights actually used (see Section 4.3: 300, 400, 600 for Cormorant; Inter is variable, so a range is fine).
- [ ] Preload is handled automatically by `next/font` — no manual `<link rel="preload">` needed.

### 6.5 Static Generation

- [ ] The entire site should be statically generated (`next build` produces static HTML). No `getServerSideProps` or dynamic routes are needed.
- [ ] Confirm `output: 'export'` is NOT set if using Vercel (Vercel handles static serving automatically; `output: 'export'` would break Server Actions).
- [ ] The contact form Server Action requires the Node.js runtime — this is compatible with Vercel's serverless functions and works without any special configuration.

### 6.6 CSS Performance

- [ ] Tailwind JIT generates only used CSS classes — production bundle should be ≤ 15KB gzipped.
- [ ] Avoid `@layer utilities` with complex selectors that generate large CSS output.
- [ ] No inline `style` props for values that can be expressed as Tailwind utilities.

### 6.7 Monitoring (Post-Launch)

- Enable **Vercel Speed Insights** (one line of code, free tier, real-user Core Web Vitals).
- Set up a monthly Lighthouse CI run if the project budget allows.

---

## Appendix A — Environment Variables Reference

| Variable | Where | Description |
|---|---|---|
| `RESEND_API_KEY` | `.env.local` + Vercel | Resend API key for sending contact form emails |

No other server-side secrets are required for v1.

## Appendix B — File / Folder Structure (Target)

```
src/
  app/
    layout.tsx          ← Root layout, fonts, metadata
    page.tsx            ← Renders all 6 sections in order
    actions/
      send-email.ts     ← Resend Server Action
  components/
    Header.tsx
    Footer.tsx
    ServiceCard.tsx
    TestimonialCard.tsx
    PortfolioCard.tsx
  sections/
    Hero.tsx
    Portfolio.tsx
    About.tsx
    Services.tsx
    Testimonials.tsx
    Contact.tsx
  hooks/
    useReducedMotion.ts
  lib/
    portfolio-data.ts
    services-data.ts
    testimonials-data.ts
  types/
    index.ts
public/
  images/
    hero.jpg
    portfolio/
      01.jpg … 12.jpg
    about-ana.jpg
```

## Appendix C — Pre-Launch Checklist

- [ ] Resend sender domain verified (DNS records applied and propagated)
- [ ] `RESEND_API_KEY` added to Vercel Production environment
- [ ] All 12 portfolio photos have meaningful alt text reviewed and approved by Ana
- [ ] `<html lang="es">` confirmed
- [ ] Meta title and description reviewed and approved by Ana
- [ ] Mobile nav tested on real iPhone (Safari) and Android (Chrome)
- [ ] Contact form end-to-end tested: submit → email arrives in Ana's inbox
- [ ] Lighthouse audit run on deployed Vercel URL (not localhost)
- [ ] `100svh` hero height tested on iPhone Safari
- [ ] All anchor links tested (smooth scroll to correct section)

---

*Development Briefing v1.0 — Ana López Photography*
*This document is the single source of truth for the initial development sprint. Update it if scope changes.*
