# Prototype — Café Vermut · Homepage Hero Section (/)

**Last updated:** 2026-05-13
**Status:** Draft
**Prepared by:** Prototype Designer
**Phase coverage:** Phase 1 (Section Prototype) · Phase 2 (Scroll Animation Map — Hero) · Phase 3 (Interactive States — Hero CTAs) · Phase 4 (Development Briefing — Hero scope)

---

## Section 1 — Hero (Full-Screen)

---

### Layout

#### Desktop (≥ 1280px)

- **Container:** Full viewport width × 100dvh (minimum 640px, maximum uncapped)
- **Background layer:** `<figure>` with `<img>` — object-fit: cover, object-position: center 40%, fills 100% × 100% of section. Overlay: `linear-gradient(to bottom, rgba(10,8,6,0.55) 0%, rgba(10,8,6,0.35) 50%, rgba(10,8,6,0.65) 100%)` — token: `--overlay-hero`
- **Content column:** centered, max-width: 720px, horizontal padding: `var(--space-6)` (24px each side), positioned at vertical center with `display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center`
- **Vertical rhythm:** content block starts at 50% viewport height (transform: translateY(-50%) not used — use flexbox centering). Minimum top padding: `var(--space-20)` (80px) to clear fixed navigation.
- **CTA row:** `display: flex; flex-direction: row; gap: var(--space-4)` (16px between buttons), centered
- **Bottom anchor hint (optional scroll indicator):** absolutely positioned, bottom: `var(--space-8)` (32px), centered horizontally

#### Tablet (768px–1279px)

- Same full-screen layout. Max-width of content column: 580px
- Horizontal padding: `var(--space-5)` (20px each side)
- CTA row: same row layout, gap: `var(--space-3)` (12px)
- Background image object-position: center 35%

#### Mobile (< 768px)

- Full viewport: 100dvh (minimum 580px)
- Content column: max-width: 100%, horizontal padding: `var(--space-4)` (16px each side)
- CTA row: `flex-direction: column; align-items: stretch` — buttons stack vertically, gap: `var(--space-3)` (12px)
- Background image object-position: center 30% (tighter crop to show counter/bar area)
- Minimum top padding: `var(--space-16)` (64px) to clear mobile navigation bar

---

### Content Inventory

Visual order (top → bottom, left → right within each row):

1. **Eyebrow label**
   - Text: `Café de Especialidad · Barcelona`
   - Token: `text-overline` — font-size: `var(--text-xs)` (12px), font-weight: 600, letter-spacing: `var(--tracking-widest)` (0.15em), text-transform: uppercase
   - Color token: `var(--color-text-inverse-secondary)` (white at 75% opacity, approx. `rgba(255,255,255,0.75)`)
   - Margin-bottom: `var(--space-4)` (16px)

2. **Headline (H1)**
   - Text: `El café que cambia cómo ves el café`
   - Token: `text-h1` — font-size: `var(--text-5xl)` (clamp(2.5rem, 5vw, 3.75rem)), font-weight: 700, line-height: `var(--leading-tight)` (1.15), font-family: `var(--font-display)`
   - Color token: `var(--color-text-inverse)` (white, `#FFFFFF`)
   - Max-width: 680px (constrain line length to ~2 lines on desktop)
   - Margin-bottom: `var(--space-5)` (20px)

3. **Subheadline / supporting copy**
   - Text: [2 lines — example: `Granos de origen único, tostados en Barcelona. Cada taza cuenta la historia del lugar donde nació.`]
   - Token: `text-body-lg` — font-size: `var(--text-lg)` (18px), font-weight: 400, line-height: `var(--leading-relaxed)` (1.65)
   - Color token: `var(--color-text-inverse-secondary)` (`rgba(255,255,255,0.75)`)
   - Max-width: 560px
   - Margin-bottom: `var(--space-8)` (32px)

4. **Primary CTA button**
   - Label: `Descubre nuestros cafés`
   - Component: `Button` — variant: `primary`, size: `lg`
   - href: `/cafes` (destination TBD by client — placeholder)
   - Token overrides on dark background: background `var(--color-brand-primary)`, color `var(--color-text-on-brand)`, hover: `var(--color-brand-primary-dark)`
   - Min-width: 200px on desktop, 100% width on mobile (see Mobile Notes)
   - Touch target: minimum 48px height (exceeds 44px minimum)

5. **Secondary CTA button**
   - Label: `Visítanos`
   - Component: `Button` — variant: `ghost`, size: `lg`
   - href: `/contacto` (or anchor `#location` if map section exists on page)
   - On dark background: border: 1.5px solid `rgba(255,255,255,0.6)`, color: `var(--color-text-inverse)`, hover: background `rgba(255,255,255,0.1)`
   - Min-width: 140px on desktop, 100% width on mobile
   - Touch target: minimum 48px height

6. **Background image**
   - Role: decorative (full-bleed behind all content)
   - Aspect ratio: 16:9 (intrinsic), displayed as cover fill
   - Alt text: `""` (empty — decorative background, content conveyed by headline)
   - Object-fit: cover, object-position: center 40%
   - Loading: `eager` (above the fold, LCP candidate)
   - Fetchpriority: `high`
   - Width/height attributes: match viewport (1920×1080 baseline, responsive `srcset`)
   - Overlay `<div>` sits on top (see Layout — overlay gradient)

7. **Scroll indicator (bottom anchor hint)** — optional, confirm with director
   - Component: animated chevron-down icon or animated line
   - Color: `var(--color-text-inverse)` at 50% opacity
   - Position: absolute, bottom: 32px, centered
   - Animation: continuous float (see Phase 2 — Continuous Animations)
   - aria-hidden: `true` (decorative)

---

### Component Usage

| Component | Variant | Size | Notes |
|---|---|---|---|
| `Button` | `primary` | `lg` | Dark-background override tokens apply |
| `Button` | `ghost` | `lg` | Border and text use inverse color tokens |
| `Image` (or `<img>`) | — | Full-bleed | `object-fit: cover`, `fetchpriority="high"`, `loading="eager"` |
| `Section` wrapper | `hero` | — | `<section aria-labelledby="hero-heading">` |
| Overlay `<div>` | — | — | Gradient via CSS, `aria-hidden="true"` |

---

### Animation Specification

**Entry trigger:** Page load (not IntersectionObserver — hero is always in viewport on load). Animations begin after DOMContentLoaded + 100ms delay to allow background image paint.

**Background image:**
- Effect: fade in from `opacity: 0` → `opacity: 1` + subtle scale `scale(1.04)` → `scale(1.0)`
- Duration: 1200ms
- Delay: 0ms
- Easing: `cubic-bezier(0.25, 0.46, 0.45, 0.94)` (ease-out-quad)
- Note: scale creates gentle "breathing in" entrance that establishes atmosphere

**Overlay gradient:**
- Effect: fade in `opacity: 0` → `opacity: 1`
- Duration: 800ms
- Delay: 200ms
- Easing: `cubic-bezier(0.25, 0.46, 0.45, 0.94)`

**Animation sequence (content — staggered):**

| # | Element | Effect | Duration | Delay | Easing |
|---|---|---|---|---|---|
| 1 | Eyebrow | Fade in `opacity: 0→1` + translateY `12px→0` | 500ms | 300ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| 2 | Headline (H1) | Fade up `opacity: 0→1` + translateY `28px→0` | 700ms | 450ms | `cubic-bezier(0.16, 1, 0.3, 1)` (ease-out-expo) |
| 3 | Subheadline | Fade up `opacity: 0→1` + translateY `20px→0` | 600ms | 620ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| 4 | CTA group (both buttons) | Fade up `opacity: 0→1` + translateY `16px→0` | 500ms | 780ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| 5 | Scroll indicator | Fade in `opacity: 0→0.5` | 400ms | 1100ms | `ease-out` |

**Total sequence duration:** ~1500ms from page load to full visibility (all elements settled)

**Button hover states (interactive, not scroll-triggered):**

- Primary button hover:
  - Background: `var(--color-brand-primary)` → `var(--color-brand-primary-dark)`
  - Transform: `translateY(-1px)`
  - Box-shadow: `0 4px 16px rgba(0,0,0,0.25)`
  - Duration: 180ms
  - Easing: `cubic-bezier(0.25, 0.46, 0.45, 0.94)`

- Primary button active (press):
  - Transform: `translateY(0px)`
  - Box-shadow: none
  - Duration: 80ms
  - Easing: `ease-in`

- Ghost button hover:
  - Background: `rgba(255,255,255,0.10)`
  - Border-color: `rgba(255,255,255,0.85)`
  - Duration: 180ms
  - Easing: `cubic-bezier(0.25, 0.46, 0.45, 0.94)`

- Ghost button active:
  - Background: `rgba(255,255,255,0.06)`
  - Duration: 80ms

**Scroll indicator animation (continuous loop):**
- Effect: translateY `0px` → `8px` → `0px`
- Duration: 1600ms per cycle
- Easing: `cubic-bezier(0.45, 0, 0.55, 1)` (ease-in-out-sine)
- Iteration: infinite
- Paused when: `prefers-reduced-motion: reduce` is active

**Exit animation:** None — hero elements remain visible as user scrolls (background parallax optional, see Phase 2)

**`prefers-reduced-motion` behavior:**
- All `translateY` values set to `0` (no movement)
- All `scale` effects removed (no scale on background image)
- `opacity` transitions retained at 50% of specified duration
- Continuous scroll indicator animation: `animation-play-state: paused`

---

### Mobile Notes

**Content retained on mobile (all elements visible):**
- Eyebrow label: visible, font-size scales down to `var(--text-xxs)` (10px) if needed, or remains at 12px
- Headline: visible — font-size: `clamp(2rem, 6vw, 2.5rem)` on mobile (overrides desktop token)
- Subheadline: visible — font-size: `var(--text-base)` (16px) on mobile
- Both CTA buttons: visible, stacked vertically (column direction), full width

**Hidden on mobile:** None — all content retained

**Layout reordering:** No reordering — vertical stack is identical, buttons simply change from row to column direction

**Touch targets:**
- Primary CTA: height 52px on mobile (exceeds 44px minimum) — `padding: 14px 24px`
- Secondary CTA: height 52px on mobile — `padding: 14px 24px`
- Scroll indicator: `width: 44px; height: 44px` centered (even if icon is smaller, tap zone is padded)

**Background image on mobile:**
- `object-position: center 30%` — adjusted to keep the most atmospheric part of the image visible on portrait crop
- Same `fetchpriority="high"` and `loading="eager"`

**Horizontal scroll:** Never allowed. All content respects `padding: 0 var(--space-4)` and no element overflows.

**Viewport height on mobile:** Use `100dvh` (dynamic viewport height) to account for mobile browser chrome. Fallback: `100vh` for older browsers.

**Font size floor:** Headline must never drop below 2rem (32px) on any screen size. Subheadline floor: 1rem (16px).

---

### Accessibility Notes

**Heading hierarchy:**
- The `El café que cambia cómo ves el café` headline is the page's primary `<h1>`
- This is the only `<h1>` on the homepage
- `id="hero-heading"` attribute required for `aria-labelledby` on the section wrapper

**ARIA landmark:**
```html
<section aria-labelledby="hero-heading" id="hero">
```
This section is wrapped in `<section>` (not `<div>`) so it registers as a landmark region in screen readers.

**Background image alt text:**
```html
<img src="..." alt="" role="presentation" />
```
Alt is empty string because the image is decorative — the content of the section is fully conveyed by the headline and copy. Do not describe the image; screen readers skip decorative images.

**Overlay div:**
```html
<div aria-hidden="true" class="hero-overlay" />
```
Gradient overlay is purely visual; `aria-hidden="true"` prevents it from being read.

**Eyebrow text:** Plain `<p>` element (not a heading). Announce as supporting context, not a navigational landmark.

**Subheadline:** `<p>` element. No heading level — it is descriptive body text supporting the H1.

**CTA keyboard behavior:**
- Both CTA buttons are standard `<a>` tags (or `<button>` if triggering JS behavior)
- If `<a>`: `href` must be populated (not `#`) — screen readers announce destination
- Tab order: Primary CTA receives focus first (DOM order), then Secondary CTA
- Focus ring: `outline: 3px solid var(--color-focus-ring)` with `outline-offset: 2px` — must be visible against the dark hero background. Recommended: white focus ring `#FFFFFF` or brand-color ring with sufficient contrast

**Color contrast on dark background:**
- Eyebrow: `rgba(255,255,255,0.75)` on dark overlay — must verify ≥ 4.5:1 against the darkest expected overlay color. At `rgba(10,8,6,0.55)` overlay, effective background is approximately `#2A2826`. White at 75% opacity ≈ `#BFBEBC` — contrast ratio: ~5.4:1 (passes AA)
- Headline: `#FFFFFF` on same background — contrast ratio: ~10.1:1 (passes AAA)
- Subheadline: same as eyebrow — ~5.4:1 (passes AA)

**Skip navigation:** The site's skip link (`<a href="#main-content" class="skip-link">Saltar al contenido principal</a>`) must be the first focusable element in the DOM, appearing before the `<header>`. Hero section should be inside `<main id="main-content">`.

**Scroll indicator:** `aria-hidden="true"` — purely decorative. If it is an `<a>` anchor link to the next section, add `aria-label="Desplázate hacia abajo"` and ensure it has a visible focus state.

---

---

## Phase 2 — Scroll Animation Map (Hero scope)

### Animation Philosophy
Café Vermut's motion language is unhurried and deliberate — like the pause before the first sip. Animations build presence rather than show off speed. Entrances are smooth and slightly editorial; nothing bounces or overshoots. The hero sets the tone: the world fades in, then the words arrive with intention.

### Global Rules
- All scroll-triggered animations (below the hero) use IntersectionObserver (no scroll event listeners)
- Default threshold: 0.15 (element is 15% visible before triggering)
- Once-only: animations do not re-trigger on scroll-up (add/remove class pattern, not toggle)
- Reduced motion: all `translateY`/`translateX`/`scale` effects removed; `opacity` transitions retained at 60% of full duration
- Hero animations: triggered on page load (not scroll), DOMContentLoaded + 100ms

### Home (/) — Hero Section Animation Registry

| Section | Element | Effect | Duration | Delay | Easing |
|---|---|---|---|---|---|
| Hero | Background image | Fade in + scale 1.04→1.0 | 1200ms | 0ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Hero | Overlay gradient | Fade in | 800ms | 200ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Hero | Eyebrow label | Fade up 12px | 500ms | 300ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Hero | Headline (H1) | Fade up 28px | 700ms | 450ms | `cubic-bezier(0.16, 1, 0.3, 1)` |
| Hero | Subheadline | Fade up 20px | 600ms | 620ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Hero | CTA group | Fade up 16px | 500ms | 780ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Hero | Scroll indicator | Fade in (opacity only) | 400ms | 1100ms | `ease-out` |

### Special Animations

#### Parallax Effects
- **Hero background image:** On desktop only (≥ 1024px), moves at 0.3x scroll rate as user scrolls past the hero. Implementation: `transform: translateY(calc(var(--scroll-y) * 0.3px))` via CSS custom property updated on scroll (passive listener). Disabled on mobile (`@media (max-width: 1023px)`) and when `prefers-reduced-motion: reduce` is active.

#### Continuous Animations (loops)
- **Scroll indicator chevron:** Float animation — translateY 0→8px→0, 1600ms, `cubic-bezier(0.45, 0, 0.55, 1)`, infinite. Paused on `prefers-reduced-motion: reduce`.

#### Page Transition
- Type: fade (opacity 1→0 on exit, 0→1 on enter)
- Duration: 200ms exit / 300ms enter
- Triggered on: navigation link click (exclude external links and anchor links)

---

## Phase 3 — Interactive State Definitions (Hero CTAs)

### Button States — Primary CTA ("Descubre nuestros cafés")

#### Default
- Background: `var(--color-brand-primary)`
- Color: `var(--color-text-on-brand)`
- Border: none
- Border-radius: `var(--radius-md)` (8px)
- Padding: `var(--space-3)` `var(--space-6)` (12px 24px desktop), `14px 24px` mobile
- Font: `text-button-lg` — font-size: `var(--text-base)` (16px), font-weight: 600, letter-spacing: `var(--tracking-wide)` (0.025em)
- Box-shadow: `var(--shadow-sm)`

#### Hover
- Background: `var(--color-brand-primary-dark)`
- Transform: `translateY(-1px)`
- Box-shadow: `0 4px 16px rgba(0,0,0,0.25)`
- Transition: `background 180ms cubic-bezier(0.25,0.46,0.45,0.94), transform 180ms cubic-bezier(0.25,0.46,0.45,0.94), box-shadow 180ms cubic-bezier(0.25,0.46,0.45,0.94)`

#### Active (pressed)
- Background: `var(--color-brand-primary-dark)`
- Transform: `translateY(0px)`
- Box-shadow: `var(--shadow-xs)`
- Transition: `transform 80ms ease-in, box-shadow 80ms ease-in`

#### Focus-visible
- Outline: `3px solid #FFFFFF`
- Outline-offset: `3px`
- (White ring ensures visibility on dark hero background)

#### Disabled (not applicable for hero CTA — link always enabled)

---

### Button States — Secondary CTA ("Visítanos")

#### Default
- Background: `transparent`
- Color: `var(--color-text-inverse)` (`#FFFFFF`)
- Border: `1.5px solid rgba(255,255,255,0.60)`
- Border-radius: `var(--radius-md)` (8px)
- Padding: `var(--space-3)` `var(--space-6)` (same as primary)
- Font: same as primary CTA

#### Hover
- Background: `rgba(255,255,255,0.10)`
- Border-color: `rgba(255,255,255,0.85)`
- Transform: `translateY(-1px)`
- Transition: `background 180ms cubic-bezier(0.25,0.46,0.45,0.94), border-color 180ms cubic-bezier(0.25,0.46,0.45,0.94), transform 180ms cubic-bezier(0.25,0.46,0.45,0.94)`

#### Active (pressed)
- Background: `rgba(255,255,255,0.06)`
- Transform: `translateY(0px)`
- Transition: `transform 80ms ease-in`

#### Focus-visible
- Outline: `3px solid #FFFFFF`
- Outline-offset: `3px`

---

## Phase 4 — Development Briefing (Hero Section)

### Project Overview
Café Vermut is a specialty coffee venue in Barcelona. The homepage hero section must immediately convey atmosphere, identity, and a dual call-to-action: product discovery and physical visit. The section is the primary LCP candidate and must load with high performance priority.

### Tech Stack Confirmation
- Framework: Next.js (App Router)
- Styling: CSS Modules or Tailwind CSS with design token layer (confirm with director)
- Image optimization: `next/image` with `priority` prop for hero image
- Animation: CSS transitions + CSS custom properties for parallax; no heavy animation libraries required for this section
- Accessibility: WCAG 2.1 AA minimum

### Critical Implementation Notes

1. **LCP optimization is mandatory for the hero image.** Use `<Image>` from `next/image` with `priority` prop (equivalent to `fetchpriority="high"` + `loading="eager"`). Provide a complete `sizes` attribute: `sizes="100vw"`. Supply `srcset` with at least 3 breakpoints: 768w, 1280w, 1920w. Do not lazy-load.

2. **Use `100dvh` for the section height**, with a `height: 100vh` fallback for older browsers. Do NOT use `min-height: 100vh` alone — this causes content overflow issues on short mobile viewports where the browser chrome is visible.

3. **CTA group flex behavior:** On mobile, the CTA container switches to `flex-direction: column`. Each button gets `width: 100%`. This must be implemented as a responsive style, not JavaScript.

4. **Parallax must be scroll-passive.** The scroll event listener updating `--scroll-y` must use `{ passive: true }`. Wrap in `requestAnimationFrame` to avoid janky updates. Scope the parallax transform to the `<figure>` containing the background image, not the `<section>` itself.

5. **Entry animations must not cause CLS.** The initial state (opacity: 0, transform) must be set via CSS class applied at render time (server-side), not added by JavaScript on mount. Use a pattern like `data-animate="pending"` removed by JS after load, or use CSS `@keyframes` with `animation-fill-mode: both` and `animation-play-state: paused` until JS sets it to `running`.

6. **Focus ring visibility on dark background.** The default browser focus ring or a brand-color focus ring may be invisible against the dark hero overlay. Override with a white focus ring specifically for elements inside the hero: `.hero a:focus-visible, .hero button:focus-visible { outline: 3px solid #fff; outline-offset: 3px; }`.

7. **`prefers-reduced-motion` implementation** must be in `globals.css` as a single `@media (prefers-reduced-motion: reduce)` block that zeros out all hero animation translate values and pauses the scroll indicator loop. Do not scatter individual overrides across component files.

### Performance Targets
- LCP (hero image): < 2.5s
- CLS: < 0.1 (hero must not cause layout shift — reserve space for image with explicit width/height or aspect-ratio)
- INP: < 200ms
- PageSpeed mobile: ≥ 90

### Browser Support
- Chrome, Firefox, Safari, Edge — latest 2 versions
- Safari iOS — latest 2 versions (test `100dvh` behavior specifically)
- Chrome Android — latest 2 versions
- No IE11 required

### Accessibility Checklist — Hero Section

**Semantic HTML**
- [ ] `<section aria-labelledby="hero-heading">` wraps the entire hero
- [ ] Hero is inside `<main id="main-content">` (target for skip link)
- [ ] Headline is `<h1 id="hero-heading">` — only one `<h1>` on the page
- [ ] Eyebrow is `<p>` (not a heading)
- [ ] Subheadline is `<p>` (not a heading)
- [ ] Background `<img>` has `alt=""` and `role="presentation"`
- [ ] Overlay `<div>` has `aria-hidden="true"`
- [ ] Scroll indicator has `aria-hidden="true"` (if decorative) or `aria-label="Desplázate hacia abajo"` (if interactive link)

**Keyboard Navigation**
- [ ] Primary CTA (`<a>` or `<button>`) is reachable via Tab
- [ ] Secondary CTA is reachable via Tab after primary CTA (DOM order matches visual order)
- [ ] Both CTAs have visible focus ring on dark background (white outline, 3px, offset 3px)
- [ ] Scroll indicator (if interactive) is reachable and has focus state

**Color & Contrast**
- [ ] Eyebrow text (`rgba(255,255,255,0.75)` on dark overlay): verify ≥ 4.5:1 against actual background
- [ ] Headline (`#FFFFFF` on dark overlay): verify ≥ 3:1 (large text) — expected pass
- [ ] Subheadline: same as eyebrow — verify ≥ 4.5:1
- [ ] Primary CTA button: verify brand color background vs. white text ≥ 4.5:1
- [ ] Ghost CTA: verify white text on transparent/dark background ≥ 4.5:1
- [ ] All focus rings: verify white ring against dark overlay — expected ~21:1

**Motion**
- [ ] `@media (prefers-reduced-motion: reduce)` block in globals.css removes all translateY/scale from hero animations
- [ ] Scroll indicator loop paused on reduced-motion
- [ ] Parallax disabled on reduced-motion
- [ ] All opacity transitions retained (reduced to 60% duration) on reduced-motion

**Images & Media**
- [ ] Hero image loaded with `priority` prop (no lazy-loading)
- [ ] Hero image has `width` and `height` attributes set (prevents CLS)
- [ ] `srcset` provided for at least 3 sizes (768, 1280, 1920)
- [ ] No autoplay video in hero (if video variant is added later, add `muted`, `playsinline`, and a pause control)

---

*End of Hero Section Prototype — Café Vermut · Phase 1–4*
