# Development Briefing — Ana López Photography

**Prepared by:** Prototype Designer
**Date:** 2026-05-13
**Status:** Ready for Development

---

## 1. Project Overview

Ana López Photography is a single-page landing site for a professional photographer based in Barcelona. The primary goal is to convert visitors into booking inquiries by presenting Ana's portfolio with impact, establishing credibility through testimonials, and making contact frictionless. The target audience includes couples planning weddings, individuals seeking portrait sessions, and corporate clients — all reached via the single `/` route.

---

## 2. Tech Stack Confirmation

| Layer | Technology | Version / Notes |
|---|---|---|
| Framework | Next.js | 14 (App Router) |
| Styling | Tailwind CSS | v3 — utility-first, no CSS modules |
| Animation | Framer Motion | v11+ — `motion` components, `useInView` |
| Email / Contact | Resend | Server Action integration |
| Image Optimization | Next.js `<Image>` | Built-in, mandatory for all photos |
| Hosting target | Vercel | Assumed — Edge runtime compatible |
| Language | TypeScript | Strict mode enabled |

---

## 3. Design System Location

- Token file: `styles/globals.css` (CSS custom properties)
- Component documentation: `~/.claude/agencia-web-v2/design/design-system-manager.md`
- UI specs: `~/.claude/agencia-web-v2/design/ui-designer.md`

---

## 4. Page Build Order

Build in this sequence to unblock parallel work and validate the design system early.

1. **`app/layout.tsx` — Root layout shell**
   Reason: Everything depends on it. Includes `<header>`, `<main>`, `<footer>`, global font loading (next/font), Tailwind base, and `prefers-reduced-motion` CSS block in `globals.css`. Do not start any section until this is in place.

2. **Section 1 — Hero**
   Reason: It is the first thing the client sees in staging and the visual benchmark for all remaining sections. Validating the full-bleed image + typography overlay early catches any contrast or font-loading issues before they propagate.

3. **Section 4 — Services (3 cards)**
   Reason: The card component is the most reused pattern. Build it as a generic `<ServiceCard>` component with props, then reuse for Testimonials. Validates the design system tokens (spacing, radius, shadow) early.

4. **Section 2 — Portfolio Grid**
   Reason: The most technically complex section (12 images, hover overlay, lazy loading). Isolate it second-to-none in order to address `<Image>` layout shifts and hover animation performance before the rest of the page is wired up.

5. **Section 3 — About**
   Reason: Static layout, low complexity. Builds in a single sitting once the grid system is proven.

6. **Section 5 — Testimonials**
   Reason: Reuses the card component from Services. Quick build.

7. **Section 6 — Contact Form**
   Reason: Last section built, but third-party integration (Resend) must be wired here. Requires env vars configured in Vercel before testing. See integration notes below.

8. **`not-found.tsx` and `error.tsx`**
   Reason: Required before any client handoff or QA pass.

---

## 5. Critical Implementation Notes

### 5.1 — Portfolio Grid: hover overlay must not block keyboard navigation

The hover overlay on each portfolio photo (that reveals a caption or "View" affordance) must be implemented with CSS `opacity` transitions, not `display:none` toggling. The overlay and its focusable contents must always be present in the DOM and reachable via keyboard (`Tab`). Use `:focus-within` on the grid item to trigger the same visible state as `:hover`, so keyboard users get the same experience as mouse users. Do NOT use pointer-events tricks that make the overlay unreachable by keyboard.

```tsx
// Correct pattern
<div className="group relative overflow-hidden">
  <Image ... />
  <div className="absolute inset-0 bg-black/50 opacity-0 transition-opacity
                  group-hover:opacity-100 group-focus-within:opacity-100
                  flex items-end p-4">
    <p className="text-white text-sm">Wedding — Barcelona 2024</p>
  </div>
</div>
```

### 5.2 — Hero image: LCP optimisation is non-negotiable

The Hero background photo is the Largest Contentful Paint element. It MUST have:
- `priority` prop on `<Image>` (disables lazy loading, injects `<link rel="preload">`)
- `sizes="100vw"` to serve correct responsive variant
- `quality={85}` — do not use default (75); hero photos show compression artifacts at default
- The image file must be provided in WebP format at minimum 2400px wide
- Do NOT wrap the hero image in a Framer Motion `motion.div` with an entry animation — the image itself must be visible immediately; only the text overlay animates in

```tsx
<Image
  src="/photos/hero.webp"
  alt="Ana López fotografiando en la playa de Barcelona durante la hora dorada"
  fill
  priority
  quality={85}
  sizes="100vw"
  className="object-cover object-center"
/>
```

### 5.3 — Framer Motion: always gate with `useReducedMotion`

Every `motion.*` component that uses `x`, `y`, or `scale` transforms must check `useReducedMotion()`. If `true`, pass `initial={false}` or supply a no-transform variant. Opacity transitions are permitted even under reduced motion.

```tsx
const prefersReducedMotion = useReducedMotion()

const variants = {
  hidden: { opacity: 0, y: prefersReducedMotion ? 0 : 24 },
  visible: { opacity: 1, y: 0 },
}
```

Place this hook at the section level, not per-element, to avoid multiple `useReducedMotion` calls.

### 5.4 — Contact Form: Server Action, not API route

Use Next.js 14 Server Actions for form submission (not `fetch` to `/api/contact`). The form must work with and without JavaScript (progressive enhancement). Implement with `useFormState` and `useFormStatus` from `react-dom`.

- The Server Action calls Resend inside `try/catch`
- On success: return `{ success: true, message: "Mensaje enviado. Ana te contactará en menos de 24 horas." }`
- On Resend error: return `{ success: false, message: "Ha ocurrido un error. Por favor escríbenos a hola@analopez.es" }` — never expose the raw error
- The `RESEND_API_KEY` and `RESEND_FROM_EMAIL` env vars must be set in `.env.local` and in Vercel project settings before any form test
- Add `"use server"` directive at the top of `app/actions/contact.ts`

### 5.5 — Portfolio Grid: Prevent Cumulative Layout Shift (CLS)

All 12 portfolio images must have explicit aspect ratio reservations. Do NOT let images determine their own height. Use Tailwind's `aspect-[3/4]` (portrait) or `aspect-square` class on the wrapper, with `<Image fill>` inside. This ensures the grid does not reflow as images load.

```tsx
<div className="relative aspect-[3/4] overflow-hidden rounded-sm">
  <Image src={photo.src} alt={photo.alt} fill className="object-cover" />
</div>
```

Never use `width` and `height` props with pixel values on grid images — `fill` + aspect-ratio wrapper is the correct pattern here.

### 5.6 — Scroll animations: use `whileInView`, not manual IntersectionObserver

With Framer Motion in the stack, use `whileInView` and `viewport={{ once: true, amount: 0.15 }}` instead of a custom `useIntersectionObserver` hook. This reduces boilerplate and is already tested for SSR compatibility. Set `once: true` on every section-entry animation — elements must not re-animate on scroll-up.

```tsx
<motion.div
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true, amount: 0.15 }}
  variants={fadeUpVariants}
>
```

### 5.7 — Navigation: sticky header must not obscure section anchors

If the header is sticky, all in-page anchor links (e.g., `#contacto`) will scroll to a position that hides behind the header. Fix with CSS scroll-margin-top on every section:

```css
/* In globals.css */
section[id] {
  scroll-margin-top: 80px; /* match header height */
}
```

Adjust the value to match the actual rendered header height. Use a CSS custom property `--header-height` so it only needs to change in one place.

---

## 6. Third-Party Integrations

### Resend (Email delivery for Contact Form)

| Item | Detail |
|---|---|
| Purpose | Deliver contact form submissions to Ana López's inbox |
| Integration phase | Section 6 (Contact Form) — last section |
| Package | `resend` npm package (`npm install resend`) |
| Required env vars | `RESEND_API_KEY` (from Resend dashboard), `RESEND_FROM_EMAIL` (e.g. `noreply@analopez.es`) |
| Required env vars (local) | Add to `.env.local` — never commit to git |
| Required env vars (production) | Add to Vercel → Settings → Environment Variables |
| Domain verification | Ana's domain must be verified in Resend dashboard (DNS TXT + MX records). Coordinate with client before staging handoff. Without this, emails land in spam or bounce. |
| Rate limiting | Resend free tier: 100 emails/day, 3,000/month. Sufficient for a photography landing page. |
| Spam protection | Add honeypot field (hidden via CSS, not `display:none` — use `position:absolute; left:-9999px`). If honeypot field is filled on submission, return success silently without calling Resend. |
| Test mode | Use Resend test API key during development. Switch to production key in Vercel env vars only. |
| Email template | Plain-text email is sufficient for MVP. Subject: `"Nueva consulta de [nombre] — Ana López Web"`. Body: name, email, message, timestamp. |

### Next.js `<Image>` (Built-in — no extra package)

| Item | Detail |
|---|---|
| Purpose | Optimized image serving, automatic WebP conversion, lazy loading |
| Config required | Add `remotePatterns` in `next.config.js` if any images are served from external CDN. If all images are local (`/public`), no config needed. |
| Photo handoff | Client must deliver all 12 portfolio photos + hero + Ana portrait at minimum 1600px wide in JPG or WebP. The developer must not resize photos manually. |

### Vercel Analytics (Recommended — zero-config)

| Item | Detail |
|---|---|
| Purpose | Privacy-friendly page view tracking |
| Package | `@vercel/analytics` |
| Setup | Add `<Analytics />` to `app/layout.tsx`. No env vars needed on Vercel. |
| Phase | Add at the end, just before first client review |

---

## 7. Accessibility Checklist

### Semantic HTML
- [ ] One `<h1>` per page — in the Hero section only: Ana's name or primary tagline
- [ ] Heading hierarchy: Hero `<h1>` → Section titles `<h2>` → Card titles `<h3>`
- [ ] Landmark regions present: `<header>`, `<main>`, `<footer>`, `<nav>`, each content section wrapped in `<section aria-labelledby="[section-id]">`
- [ ] Skip navigation link as the first focusable element in `<body>`: `<a href="#main-content" className="sr-only focus:not-sr-only">Ir al contenido principal</a>`
- [ ] `<main id="main-content">` receives skip link target
- [ ] All 12 portfolio images have descriptive `alt` text (e.g. `"Pareja abrazada en la playa de Barceloneta, reportaje de boda"`) — NOT filenames or empty strings
- [ ] Hero background image: `alt` describes the scene, not "hero image" or similar
- [ ] Ana's portrait in About section: `alt="Retrato de Ana López, fotógrafa profesional en Barcelona"`
- [ ] Decorative dividers or background shapes: `alt=""` and `aria-hidden="true"`
- [ ] All icon-only buttons (e.g. social media icons in footer, close button if any): `aria-label` with meaningful text
- [ ] All form inputs have associated `<label>` elements (visible, not placeholder-only)
- [ ] No content conveyed exclusively through an image without alt text

### Keyboard Navigation
- [ ] All interactive elements (links, buttons, form fields) reachable via `Tab` in logical visual order
- [ ] Tab order matches left-to-right, top-to-bottom reading order
- [ ] Portfolio grid items: each photo card is a focusable element (`<a>` wrapper or `tabIndex={0}` with `role="button"` if not a link); hover overlay visible on `:focus-within`
- [ ] Visible focus ring on ALL interactive elements — do not suppress `outline: none` without a custom replacement; use `focus-visible:ring-2 focus-visible:ring-[brand-color]` in Tailwind
- [ ] Form submit button: `type="submit"`, not `type="button"`
- [ ] Navigation menu links: `<a>` elements, not `<div>` or `<span>` with click handlers
- [ ] Escape key: if any modal or lightbox is added in the future, it must close on Escape; document the hook even if unused at MVP

### Color & Contrast
- [ ] All body text on dark backgrounds: minimum 4.5:1 contrast ratio (WCAG AA)
- [ ] All body text on light backgrounds: minimum 4.5:1 contrast ratio
- [ ] Hero text overlay on photo: the overlay `bg-black/50` must be verified to achieve 4.5:1 against white text — test with the actual hero photo (darkest and lightest areas). If ratio fails on lighter photo regions, increase overlay opacity or add a gradient
- [ ] Service card text: verify brand color buttons pass 3:1 against button background (large text / UI component threshold)
- [ ] Testimonials quotes: light text on potentially light background — verify
- [ ] Focus rings: must contrast at 3:1 against both the element background and the page background
- [ ] Interactive elements are not differentiated by color alone (e.g., links have underline or other non-color indicator in body text)

### Motion
- [ ] `prefers-reduced-motion` media query added to `globals.css`:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
    }
  }
  ```
- [ ] All Framer Motion `motion.*` components use `useReducedMotion()` hook (see Critical Implementation Note 5.3)
- [ ] No auto-playing animations that move content (e.g. no auto-advancing carousel, no looping parallax on load)
- [ ] If a scroll-parallax effect is added to the Hero, it must be disabled under `prefers-reduced-motion` and on all mobile breakpoints

### Forms (Contact Form)
- [ ] All three fields (name, email, message) have visible `<label>` elements positioned above the field
- [ ] Required fields: label text includes `"*"` or `"(obligatorio)"` — never indicated by color alone
- [ ] `required` attribute on all required fields
- [ ] `type="email"` on the email field (enables native validation and appropriate mobile keyboard)
- [ ] Error messages rendered as text adjacent to the field, not as toast-only
- [ ] Error messages associated with their field via `aria-describedby="[field-id]-error"`
- [ ] `aria-invalid="true"` applied to fields with validation errors
- [ ] On form submission with errors: focus programmatically moved to the first field with an error
- [ ] Submission result (success or failure) announced to screen readers: use `aria-live="polite"` region that is always in the DOM, updated on response
- [ ] Submit button shows loading state with `aria-busy="true"` during submission
- [ ] Fields are disabled (`disabled` attribute) during submission to prevent double-submit
- [ ] Success message includes next-step guidance, not just "Message sent"

### Images & Media
- [ ] All 12 portfolio images below the fold: no `priority` prop (lazy loaded by default with Next.js `<Image>`)
- [ ] Hero image: `priority` prop (NOT lazy loaded — see Critical Note 5.2)
- [ ] Ana's portrait (About section): assess fold position on desktop vs mobile; if below fold on mobile, omit `priority`
- [ ] No autoplaying video with audio anywhere on the page
- [ ] If a video background is ever added to Hero in future iterations: must have `muted`, `playsInline`, and a visible pause/play control accessible by keyboard

### Language & Internationalisation
- [ ] `<html lang="es">` set in `app/layout.tsx`
- [ ] If any English phrases appear (e.g. "Contact", "Portfolio"): wrap in `lang="en"` attribute or use Spanish equivalents per copy brief

---

## 8. Performance Targets

| Metric | Target | How to achieve |
|---|---|---|
| Largest Contentful Paint (LCP) | < 2.5s | `priority` on hero image, preload WebP, no render-blocking JS in `<head>` |
| Cumulative Layout Shift (CLS) | < 0.1 | `aspect-ratio` wrappers on all images (see Note 5.5), no font FOUT (use `next/font` with `display: swap` only if fallback metrics match) |
| Interaction to Next Paint (INP) | < 200ms | Server Actions are fast; keep contact form JS bundle lean; no heavy client-side libraries except Framer Motion |
| First Contentful Paint (FCP) | < 1.8s | Inline critical CSS (Tailwind purges unused), defer non-critical scripts |
| Total Blocking Time (TBT) | < 200ms | Avoid synchronous third-party scripts; Vercel Analytics is async by default |
| Google PageSpeed (mobile) | ≥ 90 | Meet all above + serve correctly sized images via `sizes` prop on every `<Image>` |
| Google PageSpeed (desktop) | ≥ 95 | |
| Page weight (initial JS bundle) | < 200 kB gzipped | Audit with `next build --debug`; tree-shake Framer Motion (import only used components) |
| Portfolio image weight (each) | < 120 kB at 800px wide | Next.js handles compression; ensure source files are not PNG for photographs |

### Performance checklist before handoff to client
- [ ] Run `next build` and confirm no build errors
- [ ] Run Lighthouse in Chrome DevTools on production URL (not localhost) — mobile preset
- [ ] Verify all 12 portfolio images are lazy-loaded (check Network tab: images load as you scroll)
- [ ] Verify hero image is preloaded (check `<head>` for `<link rel="preload" as="image">`)
- [ ] Verify zero CLS in Layout Shift regions (Lighthouse CLS detail or `PerformanceObserver`)
- [ ] Verify Resend call does not block page load (it is server-side, so it should not)

---

## 9. Browser Support

| Browser | Versions |
|---|---|
| Chrome | Latest 2 versions |
| Firefox | Latest 2 versions |
| Safari macOS | Latest 2 versions |
| Edge | Latest 2 versions |
| Safari iOS | Latest 2 versions (iPhone 12+) |
| Chrome Android | Latest 2 versions |
| Internet Explorer | Not supported |

**CSS features to verify cross-browser:**
- `aspect-ratio` property: supported in all targets above; no polyfill needed
- `dvh` units (for Hero full-screen height): use `min-h-screen` as fallback → `min-h-[100dvh]` with `@supports`
- CSS Grid (Portfolio): fully supported; no fallback needed
- `backdrop-filter` (if used for nav blur): NOT supported in Firefox without flag — use solid background fallback
- CSS custom properties: fully supported

---

## 10. Section-by-Section Quick Reference

This table gives the developer the non-negotiable decisions for each section so no design question is ever needed.

| Section | Height | Key layout | Primary animation | Notes |
|---|---|---|---|---|
| Hero | 100dvh (min 600px) | Full-bleed image + centered text overlay | Text stack fades up on load (no scroll trigger) | `priority` on image, overlay bg-black/50 |
| Portfolio Grid | Auto | 3-col CSS Grid on desktop, 2-col tablet, 1-col mobile | Cards fade up stagger on scroll (80ms between) | `aspect-[3/4]` wrapper, hover overlay with `:focus-within` |
| About | Auto (min 480px) | 2-col: image left, text right on desktop; stacked on mobile | Image fades in, text fades up, simultaneous | Ana portrait on right on mobile (image below text) |
| Services | Auto | 3-col flex/grid on desktop, 1-col on mobile | Cards fade up stagger on scroll (100ms between) | Cards: title `<h3>`, icon at top, body text, optional CTA |
| Testimonials | Auto | 3-col on desktop, 1-col on mobile | Quotes fade in (no translate — subtle) | `<blockquote>` element, `<cite>` for attribution |
| Contact Form | Auto (min 480px) | Single-column centered, max-width 560px | Form fades up on scroll | Server Action, Resend, honeypot, full a11y |

---

## 11. Copy Placeholders (to fill with client content before dev starts)

The developer must NOT invent copy. If final copy is not yet provided, use these explicit placeholders — never Lorem Ipsum:

- `[ANA_TAGLINE]` — Hero headline (e.g. "Fotografía que cuenta tu historia")
- `[ANA_SUBHEADLINE]` — Hero subheadline (1 sentence, ≤ 12 words)
- `[ANA_BIO]` — About section body (2–3 paragraphs)
- `[SERVICE_1_TITLE]`, `[SERVICE_1_DESC]` — Bodas card
- `[SERVICE_2_TITLE]`, `[SERVICE_2_DESC]` — Retratos card
- `[SERVICE_3_TITLE]`, `[SERVICE_3_DESC]` — Corporativo card
- `[TESTIMONIAL_1_QUOTE]`, `[TESTIMONIAL_1_AUTHOR]` × 3
- `[CONTACT_EMAIL_RECIPIENT]` — Where Resend sends the email (Ana's inbox address)
- `[ANA_EMAIL_PUBLIC]` — Displayed in footer / contact section fallback

---

## 12. Sign-off Required From

- [ ] Director approval of this briefing before development begins
- [ ] Client (Ana López) approval of copy and photo assets before Section 2 (Portfolio Grid) is built
- [ ] Client approval on staging before launch
- [ ] Director final sign-off before production deployment

---

*End of Development Briefing — Ana López Photography*
*Prepared by Prototype Designer · 2026-05-13*
