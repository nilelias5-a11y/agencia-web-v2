# Café Vermut — Hero Section Prototype
## Developer Handoff Specification · Iteration 1 · Eval 1

**Date:** 2026-05-13
**Status:** Approved for development
**Target:** Full-screen homepage Hero section

---

## 1. LAYOUT SPEC — Desktop / Tablet / Mobile

---

### 1.1 Global Structure

The Hero is a single full-viewport-height (`100dvh`) section. It uses a stacking model: background image layer → gradient overlay layer → content layer → optional noise/texture layer (top).

```
┌─────────────────────────────────────────────────────────────┐
│  [z-index: 0]  Background image (cover, fixed on desktop)   │
│  [z-index: 1]  Gradient overlay                             │
│  [z-index: 2]  Content container                            │
│  [z-index: 3]  Noise texture (optional, multiply/overlay)   │
└─────────────────────────────────────────────────────────────┘
```

---

### 1.2 Desktop (≥ 1280px)

**Viewport:** `100vw × 100dvh` (minimum 700px height enforced)

**Content area:**
- Max-width container: `1200px`
- Horizontal padding: `80px` each side (so usable inner width: `1040px`)
- Content is positioned in the **lower-left quadrant** of the viewport
- Vertical alignment: content block starts at `58vh` from the top (visually bottom-weighted)
- Content block max-width: `640px`

**Background image:**
- `object-fit: cover`, `object-position: center 40%` (keeps upper café atmosphere visible)
- No `background-attachment: fixed` — use CSS `transform: translateY()` via JS scroll parallax instead (avoids iOS repaint issues)
- Aspect hint to art direction team: crop focal point at horizontal center, vertical 35–40% from top

**Gradient overlay:**
- Two-layer gradient stacked:
  1. Radial gradient from bottom-left: `radial-gradient(ellipse 80% 60% at 0% 100%, rgba(0,0,0,0.72) 0%, transparent 70%)`
  2. Linear gradient top: `linear-gradient(to bottom, rgba(0,0,0,0.15) 0%, transparent 40%)`
- Combined via CSS `background` shorthand on the overlay `<div>` with two background-image values

**Layout grid reference (desktop):**

```
|←——— 80px ———|←————————————— Content block (max 640px) —————————————→|
                                                                         |←— 480px+ —→|
                Eyebrow text                                             (image breathes)
                Headline H1
                Subheadline
                [CTA Group]
|←——————————————————————————————————— 1200px max-width ——————————————————————————————————→|
```

**Vertical rhythm (desktop):**

| Element          | Distance from top of content block |
|------------------|-------------------------------------|
| Eyebrow          | 0 (anchor)                         |
| Gap: eyebrow→H1  | 16px                               |
| H1               | —                                  |
| Gap: H1→sub      | 24px                               |
| Subheadline      | —                                  |
| Gap: sub→CTAs    | 40px                               |
| CTA group        | —                                  |

---

### 1.3 Tablet (768px – 1279px)

**Viewport:** `100dvh` (min-height: `600px`)

**Content area:**
- Horizontal padding: `48px` each side
- Content max-width: `560px` (still left-aligned)
- Vertical position: content block starts at `54vh`

**Background image:**
- `object-position: 35% 40%` — shifts focal point slightly toward center to compensate for narrower viewport
- Parallax disabled on tablet (motion-sensitivity + performance). Image stays `fixed: none`

**Gradient overlay:**
- Same two-layer gradient as desktop. Radial ellipse adjusted to `100% 55%` at bottom-left

**Vertical rhythm (tablet):**

| Element          | Distance from top of content block |
|------------------|-------------------------------------|
| Eyebrow          | 0                                  |
| Gap: eyebrow→H1  | 12px                               |
| H1               | —                                  |
| Gap: H1→sub      | 20px                               |
| Subheadline      | —                                  |
| Gap: sub→CTAs    | 32px                               |
| CTA group        | —                                  |

---

### 1.4 Mobile (< 768px)

**Viewport:** `100dvh` (min-height: `580px`)

**Content area:**
- Horizontal padding: `24px` each side (full-bleed content)
- Content is **centered horizontally** (switch from left-align to center-align on all text)
- Content block spans full available width (no max-width constraint on mobile)
- Vertical position: content block starts at `50vh` (more centered feel due to small screen)
- Content block is `position: absolute; bottom: 64px; left: 24px; right: 24px` — anchored to bottom

**Background image:**
- `object-position: 50% 30%` — tighter crop on the atmospheric top portion of the image
- Parallax: **disabled**. Static image only.
- Recommendation to art direction: provide a **2:3 portrait crop** version for `srcset` at `< 768px`

**Gradient overlay (mobile-specific):**
- Heavier gradient required due to centered text over mid-image:
  1. Linear: `linear-gradient(to top, rgba(0,0,0,0.80) 0%, rgba(0,0,0,0.30) 55%, rgba(0,0,0,0.10) 100%)`
- Simplify to single layer (performance + legibility)

**CTA Group (mobile):**
- Stack CTAs vertically (column flex)
- Both CTAs: full-width (`width: 100%`)
- Gap between CTAs: `12px`

**Vertical rhythm (mobile):**

| Element          | Distance from top of content block |
|------------------|-------------------------------------|
| Eyebrow          | 0                                  |
| Gap: eyebrow→H1  | 10px                               |
| H1               | —                                  |
| Gap: H1→sub      | 16px                               |
| Subheadline      | —                                  |
| Gap: sub→CTAs    | 28px                               |
| CTA group        | —                                  |

---

### 1.5 Typography Scale

| Element         | Desktop              | Tablet               | Mobile               | Weight   | Line-height | Letter-spacing |
|-----------------|----------------------|----------------------|----------------------|----------|-------------|----------------|
| Eyebrow         | 12px / `font-size`   | 11px                 | 11px                 | 500      | 1.2         | `0.12em`       |
| H1 Headline     | 64px                 | 52px                 | 40px                 | 700      | 1.05        | `-0.02em`      |
| Subheadline     | 18px                 | 17px                 | 16px                 | 400      | 1.6         | `0`            |
| CTA Primary     | 15px                 | 15px                 | 15px                 | 600      | 1           | `0.04em`       |
| CTA Secondary   | 14px                 | 14px                 | 14px                 | 500      | 1           | `0.04em`       |

**Font family:** System serif stack as placeholder — confirm with brand guidelines.
Suggestion: `'Playfair Display', Georgia, serif` for H1; `'Inter', system-ui, sans-serif` for all other elements.

**Color (text on dark overlay):**
- Eyebrow: `rgba(255,255,255,0.65)` — muted, supporting role
- H1: `#FFFFFF`
- Subheadline: `rgba(255,255,255,0.80)`
- CTA Primary (text): `#1A0A00` (dark espresso — on light button background)
- CTA Secondary (text): `#FFFFFF`

---

### 1.6 CTA Button Specs

**CTA Primary — "Descubre nuestros cafés"**
- Background: `#E8D5A3` (warm parchment/cream — café brand warm tone)
- Text color: `#1A0A00`
- Border: none
- Border-radius: `4px`
- Padding: `16px 32px` (desktop/tablet) / `18px 24px` (mobile, full-width)
- Hover state: background `#D4BC88`, slight scale `transform: scale(1.02)`, transition `200ms ease`
- Active state: scale `0.98`, background `#C4AC78`
- Focus-visible: `outline: 2px solid #FFFFFF; outline-offset: 3px`

**CTA Secondary — "Visítanos"**
- Background: transparent
- Text color: `#FFFFFF`
- Border: `1.5px solid rgba(255,255,255,0.60)`
- Border-radius: `4px`
- Padding: `15px 28px` (desktop/tablet) / `17px 24px` (mobile, full-width)
- Hover state: background `rgba(255,255,255,0.10)`, border-color `rgba(255,255,255,0.90)`, transition `200ms ease`
- Active state: background `rgba(255,255,255,0.18)`
- Focus-visible: `outline: 2px solid #FFFFFF; outline-offset: 3px`

**CTA Group container:**
- Desktop/Tablet: `display: flex; flex-direction: row; gap: 16px; align-items: center`
- Mobile: `display: flex; flex-direction: column; gap: 12px; align-items: stretch`

---

## 2. CONTENT INVENTORY (Visual Order)

The following represents elements in the exact visual reading order (top to bottom), with their semantic role, copy, and attributes.

---

### Element 1 — Eyebrow / Kicker

| Attribute       | Value                                          |
|-----------------|------------------------------------------------|
| Visual order    | 1 (topmost text element)                       |
| Semantic tag    | `<p>` with `aria-hidden="false"`               |
| Role            | Supporting context / category label            |
| Copy            | `Café de Especialidad · Barcelona`             |
| Separator char  | `·` (U+00B7 MIDDLE DOT) — not a bullet (`•`)  |
| Case            | Mixed case as written (not `text-transform`)   |
| Max length      | 40 chars (do not wrap — single line always)    |
| Truncation      | N/A — copy is fixed                            |

---

### Element 2 — Headline H1

| Attribute       | Value                                          |
|-----------------|------------------------------------------------|
| Visual order    | 2                                              |
| Semantic tag    | `<h1>`                                         |
| Role            | Primary page title / brand statement           |
| Copy            | `El café que cambia cómo ves el café`          |
| Max lines       | 2 lines at desktop, up to 3 lines at mobile    |
| Special chars   | `ó` (U+00F3), `é` (U+00E9) — confirm font support |
| Hyphenation     | `hyphens: none` — no auto-hyphenation          |
| Orphan control  | CSS `orphans: 2; widows: 2`; consider `<wbr>` hints if needed |

---

### Element 3 — Subheadline

| Attribute       | Value                                                                                        |
|-----------------|----------------------------------------------------------------------------------------------|
| Visual order    | 3                                                                                            |
| Semantic tag    | `<p>`                                                                                        |
| Role            | Secondary supporting copy — origin and process                                               |
| Copy (line 1)   | `Cultivado en los mejores orígenes del mundo,`                                               |
| Copy (line 2)   | `tostado con precisión artesanal en el corazón de Barcelona.`                               |
| Note            | Two semantic lines, rendered as a single `<p>`. No `<br>` — let natural reflow handle it.   |
| Max lines       | 2 lines desktop/tablet, up to 3 lines mobile                                                 |
| Max chars       | ~120 chars total (current copy: ~104 chars — within budget)                                  |

> **Copy note for client:** Subheadline copy above is a suggested fill based on the approved concept ("origen y proceso"). Client must confirm or provide final copy before development.

---

### Element 4 — CTA Primary

| Attribute       | Value                            |
|-----------------|----------------------------------|
| Visual order    | 4 (left on desktop, top on mobile) |
| Semantic tag    | `<a href="/cafes">` (or `<button>` if SPA routing) |
| Role            | Primary call to action           |
| Copy            | `Descubre nuestros cafés`        |
| Icon            | None (clean text-only button)    |
| Destination     | `/cafes` (catalog page)          |
| `aria-label`    | Not needed — copy is descriptive |

---

### Element 5 — CTA Secondary

| Attribute       | Value                                |
|-----------------|--------------------------------------|
| Visual order    | 5 (right on desktop, bottom on mobile) |
| Semantic tag    | `<a href="/visita">`                  |
| Role            | Secondary call to action             |
| Copy            | `Visítanos`                          |
| Icon            | None                                 |
| Destination     | `/visita` (visit/location page)      |
| `aria-label`    | Not needed — copy is descriptive     |

---

### Element 6 — Background Image (non-content layer)

| Attribute       | Value                                                          |
|-----------------|----------------------------------------------------------------|
| Visual order    | Behind all content (z-index: 0)                               |
| Semantic tag    | `<img>` with `role="presentation"` and `alt=""`               |
| Role            | Atmospheric / decorative — no informational content           |
| Recommended src | `hero-cafeteria-atmosferica.jpg`                              |
| Formats         | AVIF (primary), WebP (fallback), JPEG (legacy)                |
| Sizes attribute | `100vw` (always full viewport width)                          |
| Srcset hints    | 640w, 1280w, 1920w, 2560w (for HiDPI desktop)                 |
| `loading`       | `eager` (above the fold — never lazy-load hero image)         |
| `fetchpriority` | `high`                                                         |
| `decoding`      | `async`                                                        |

---

## 3. ANIMATION SPECIFICATION

All animations must respect `prefers-reduced-motion`. See Section 5 (Accessibility) for the reduced-motion variant.

---

### 3.1 Entry Animation — Sequence Overview

The Hero content enters via a **staggered reveal** on initial page load. The animation sequence begins once the background image has loaded (listen for `img.complete` or `load` event on the hero `<img>`). If image load takes > 800ms, begin animation sequence anyway to avoid perceived delay.

**Total entry sequence duration:** ~1400ms from trigger

---

### 3.2 Background Image Fade-in

| Property        | Value                                                  |
|-----------------|--------------------------------------------------------|
| Element         | Background image layer (`<img>`)                       |
| Effect          | Opacity fade + very subtle scale                       |
| Start state     | `opacity: 0; transform: scale(1.04)`                   |
| End state       | `opacity: 1; transform: scale(1.00)`                   |
| Duration        | `1200ms`                                               |
| Delay           | `0ms` (starts immediately)                             |
| Easing          | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` — ease-out     |
| CSS property    | `transition: opacity 1200ms, transform 1200ms`         |

---

### 3.3 Gradient Overlay Fade-in

| Property        | Value                                    |
|-----------------|------------------------------------------|
| Element         | Gradient overlay `<div>`                 |
| Effect          | Opacity fade                             |
| Start state     | `opacity: 0`                             |
| End state       | `opacity: 1`                             |
| Duration        | `800ms`                                  |
| Delay           | `200ms` (slight lag behind image)        |
| Easing          | `ease-out`                               |

---

### 3.4 Content Stagger Sequence

Each text element and the CTA group enters individually with a **fade-up** motion.

**Fade-up definition:**
- Start: `opacity: 0; transform: translateY(20px)`
- End: `opacity: 1; transform: translateY(0)`
- Easing for all: `cubic-bezier(0.16, 1, 0.3, 1)` — spring-like ease-out (snappy arrival)
- Duration for all: `700ms`

| Order | Element            | Delay from trigger | Notes                              |
|-------|--------------------|--------------------|------------------------------------|
| 1     | Eyebrow            | `300ms`            | First text to appear               |
| 2     | H1 Headline        | `450ms`            | +150ms after eyebrow               |
| 3     | Subheadline        | `600ms`            | +150ms after H1                    |
| 4     | CTA group          | `780ms`            | +180ms — group animates as a unit  |

**CSS implementation note:** Use CSS custom properties + `animation-delay` on each element. Do not use JS-driven GSAP for this sequence — pure CSS `@keyframes` is sufficient and reduces JS payload.

```css
/* Example keyframe */
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-eyebrow  { animation: fadeUp 700ms cubic-bezier(0.16, 1, 0.3, 1) 300ms both; }
.hero-h1       { animation: fadeUp 700ms cubic-bezier(0.16, 1, 0.3, 1) 450ms both; }
.hero-sub      { animation: fadeUp 700ms cubic-bezier(0.16, 1, 0.3, 1) 600ms both; }
.hero-ctas     { animation: fadeUp 700ms cubic-bezier(0.16, 1, 0.3, 1) 780ms both; }
```

**`animation-fill-mode: both`** is critical — prevents flash of visible content before animation starts.

---

### 3.5 Parallax Scroll Effect (Desktop only)

| Property        | Value                                              |
|-----------------|-----------------------------------------------------|
| Applies to      | Background image layer only                         |
| Technique       | JS `requestAnimationFrame` reading `window.scrollY` |
| Movement ratio  | `0.35` (image moves 35% of scroll distance)         |
| Direction       | Image moves UP as user scrolls DOWN                |
| Transform       | `translateY(scrollY * 0.35 * -1)px`               |
| Max offset      | Cap at `-80px` to prevent image from leaving frame  |
| Trigger         | Only active while `scrollY < heroHeight`            |
| Disable on      | Tablet and mobile (see 3.7)                         |
| Performance     | `will-change: transform` on the image element       |

**Implementation note:** Wrap parallax in a `matchMedia('(min-width: 1280px)')` listener. Remove the `will-change` property when hero scrolls out of view (intersection observer).

---

### 3.6 CTA Hover / Interaction States

**Primary CTA hover:**
- `transform: scale(1.02)` + `background-color` shift
- Duration: `180ms`
- Easing: `ease-out`
- No bounce — linear scale arrival

**Secondary CTA hover:**
- `background-color` fade-in + `border-color` shift
- Duration: `180ms`
- Easing: `ease-out`

**Both CTAs active (mousedown/touchstart):**
- `transform: scale(0.97)`
- Duration: `80ms`
- Easing: `ease-in`

---

### 3.7 Disabled / Reduced Motion Variants

When `prefers-reduced-motion: reduce` is detected:
- Remove all `transform` animations (translateY, scale on image)
- Entry sequence becomes **opacity-only** fades:
  - Image: `opacity 600ms ease`
  - Each text element: `opacity 400ms ease`, same stagger delays preserved
- Parallax: **disabled entirely**
- Hover states: `background-color` + `border-color` transitions only, no `transform`

---

## 4. MOBILE NOTES

---

### 4.1 Image & Performance

- **Provide a dedicated portrait crop** (`2:3` ratio, ~828×1242px) for mobile breakpoint. The landscape desktop image at `object-position` is an acceptable fallback, but a purpose-shot portrait crop dramatically improves perceived quality.
- Use `<picture>` + `<source media="(max-width: 767px)">` for the portrait srcset.
- Compress AVIF mobile image to ≤ 120KB at 1x resolution target.
- Do **not** use CSS `background-image` for the hero photo — use `<img>` for LCP optimization (browser preload scanner can't preload CSS background-image).

### 4.2 Tap Targets

- Both CTAs must be a minimum of `48px` in height on mobile (WCAG 2.5.5 AAA, practically enforced at AA-equivalent 44px minimum).
- CTA full-width layout ensures tap target always spans full content width.
- Add `min-height: 48px` explicitly.

### 4.3 Viewport Height (The DVH Problem)

- Use `height: 100dvh` (dynamic viewport height) — this is critical on iOS Safari where `100vh` includes the browser chrome and causes the hero to overflow.
- Provide `height: 100svh` as a `@supports` fallback for browsers that support `svh` but not `dvh` (minor edge case, worth noting).
- For very old browsers: `height: 100vh` as the base, then override with `dvh`.

```css
.hero {
  height: 100vh; /* fallback */
  height: 100svh; /* intermediate fallback */
  height: 100dvh; /* preferred */
  min-height: 580px; /* absolute floor */
}
```

### 4.4 Text & Contrast on Mobile

- Mobile gradient is heavier by design (single strong bottom-up gradient vs. desktop's radial + top).
- Verify contrast ratio of `rgba(255,255,255,0.80)` subheadline text against the mobile gradient at the content block's vertical position. Target: WCAG AA minimum 4.5:1 for body text.
- If gradient isn't sufficient, add a `backdrop-filter: blur(0px)` — don't blur, just use it as a base for a `background: rgba(0,0,0,0.10)` semi-transparent panel behind the content block (soft vignette approach, not a hard card).

### 4.5 Safe Area Insets

- The bottom content anchor (`bottom: 64px`) must account for notched devices.
- Use: `bottom: calc(64px + env(safe-area-inset-bottom))`
- Apply `padding-bottom: env(safe-area-inset-bottom)` on the hero container as well.

### 4.6 Scroll Indicator (Optional Enhancement)

- On mobile, consider a subtle animated chevron-down or scroll line at the bottom center of the hero.
- Fade out on first scroll event (`window.addEventListener('scroll', handler, { once: true })`).
- Mark as `aria-hidden="true"` — purely decorative.

### 4.7 Long-Press & Text Selection

- Prevent text selection on the headline + eyebrow (accidental long-press on mobile selects text, disrupting the premium feel):
  ```css
  .hero-content { user-select: none; -webkit-user-select: none; }
  ```
- Preserve text selectability for subheadline if content team wants copy-pasteable text (tradeoff — discuss with client).

---

## 5. ACCESSIBILITY NOTES

---

### 5.1 Heading Hierarchy

- The `<h1>` must be the **first and only H1 on the page**.
- Eyebrow text is a `<p>` — do not use `<h2>` or `<span aria-label>` on it. It is purely visual context.
- Confirm with the rest of page architecture that no other `<h1>` exists (nav logo/brand mark sometimes incorrectly uses `<h1>`).

### 5.2 Background Image

- The background photo is **decorative** — it sets atmosphere but conveys no information not available in the text.
- Implementation: `<img alt="" role="presentation">` placed in a `<div aria-hidden="true">` wrapper.
- Alternative if using CSS `background-image`: add `aria-hidden="true"` to the pseudo-element container.

### 5.3 Color Contrast

| Text element    | Text color                  | Background (approximate) | Target ratio | Notes                          |
|-----------------|-----------------------------|---------------------------|--------------|--------------------------------|
| Eyebrow         | `rgba(255,255,255,0.65)`    | Dark overlay ~`#1a1a1a`   | ≥ 4.5:1      | May fail at 0.65 opacity — verify with overlay darkest area |
| H1              | `#FFFFFF`                   | Dark overlay ~`#1a1a1a`   | ≥ 4.5:1      | Passes comfortably             |
| Subheadline     | `rgba(255,255,255,0.80)`    | Dark overlay ~`#1a1a1a`   | ≥ 4.5:1      | Likely passes — verify         |
| CTA Primary     | `#1A0A00` on `#E8D5A3`      | N/A (solid button)        | ≥ 4.5:1      | Verify: ~8.5:1 estimated — passes |
| CTA Secondary   | `#FFFFFF` on transparent    | Dark overlay              | ≥ 4.5:1      | Verify with actual image pixel sampling |

**Eyebrow contrast flag:** `rgba(255,255,255,0.65)` at `12px / 500 weight` is small text. WCAG AA requires 4.5:1. Consider increasing to `rgba(255,255,255,0.75)` or bumping font-size to `13px` (large text threshold eases requirement to 3:1). Use Figma's contrast plugin or the [APCA calculator](https://www.myndex.com/APCA/) for the actual overlay composite value.

### 5.4 Focus Management

- On page load, **do not** programmatically move focus to the Hero unless it is the result of a navigation action (e.g., user navigated to `/` from another page via SPA routing).
- Tab order: Eyebrow and subheadline are non-interactive — they are naturally skipped. Tab hits the two CTA buttons in visual order (left/top first).
- Focus style: Both CTAs must show `outline: 2px solid #FFFFFF; outline-offset: 3px` — never suppress `outline: none` without a replacement.
- Ensure focus ring is visible against the dark image background (white outline works; avoid dark or colored outlines here).

### 5.5 Keyboard Navigation

- Both CTAs must be natively keyboard-activatable via `Enter` and `Space` (using `<a>` or `<button>` — do not use `<div onClick>`).
- If using `<a>`, `Space` does not natively trigger navigation — this is acceptable browser behavior, but confirm with dev if SPA routing needs a `keydown` handler.

### 5.6 Motion & Vestibular Disorders

- Full `prefers-reduced-motion` implementation is required, not optional (see Section 3.7).
- The parallax scroll effect is specifically flagged as a vestibular trigger — it must be unconditionally disabled under reduced-motion preference, regardless of `matchMedia` screen size check.
- Test with macOS > Accessibility > Reduce Motion = ON and iOS Settings > Accessibility > Motion > Reduce Motion = ON.

### 5.7 Screen Reader Experience

The screen reader will encounter the following reading order (after `aria-hidden` background image is excluded):

1. (landmark) `<header>` / `<nav>` — site navigation (above hero in DOM, but likely position:absolute overlay)
2. `<section aria-label="Hero">` or `<section aria-labelledby="hero-heading">`
3. Eyebrow paragraph: "Café de Especialidad · Barcelona"
4. H1: "El café que cambia cómo ves el café"
5. Subheadline paragraph (2-line copy)
6. Link: "Descubre nuestros cafés"
7. Link: "Visítanos"

Add `aria-label="Inicio"` or `aria-labelledby="hero-heading"` to the `<section>` wrapping the Hero to create a named landmark region. This helps screen reader users navigate by landmarks.

### 5.8 Autoplay / Motion Trigger

- There is no video or autoplay content in this spec.
- If a video background version is considered in future iterations, it must: be muted, have no audio, be pauseable via a visible control, and be disabled under `prefers-reduced-motion`.

### 5.9 Semantic HTML Recommended Structure

```html
<section class="hero" aria-labelledby="hero-heading">
  <!-- Background image layer — decorative -->
  <div class="hero-bg" aria-hidden="true">
    <picture>
      <source media="(max-width: 767px)" srcset="hero-portrait-828w.avif 1x, hero-portrait-1656w.avif 2x" type="image/avif">
      <source media="(max-width: 767px)" srcset="hero-portrait-828w.webp 1x, hero-portrait-1656w.webp 2x" type="image/webp">
      <source srcset="hero-landscape-1920w.avif 1x, hero-landscape-3840w.avif 2x" type="image/avif">
      <source srcset="hero-landscape-1920w.webp 1x, hero-landscape-3840w.webp 2x" type="image/webp">
      <img
        src="hero-landscape-1920w.jpg"
        alt=""
        role="presentation"
        loading="eager"
        fetchpriority="high"
        decoding="async"
        class="hero-bg-image"
      >
    </picture>
  </div>

  <!-- Gradient overlay layer — decorative -->
  <div class="hero-overlay" aria-hidden="true"></div>

  <!-- Content layer -->
  <div class="hero-content">
    <p class="hero-eyebrow">Café de Especialidad · Barcelona</p>
    <h1 class="hero-h1" id="hero-heading">El café que cambia cómo ves el café</h1>
    <p class="hero-sub">
      Cultivado en los mejores orígenes del mundo,
      tostado con precisión artesanal en el corazón de Barcelona.
    </p>
    <div class="hero-ctas">
      <a href="/cafes" class="btn btn-primary">Descubre nuestros cafés</a>
      <a href="/visita" class="btn btn-secondary">Visítanos</a>
    </div>
  </div>
</section>
```

---

## Appendix: Breakpoint Summary Reference

| Breakpoint     | Range          | Layout shift                     | Text align   | CTA direction |
|----------------|----------------|----------------------------------|--------------|---------------|
| Desktop        | ≥ 1280px       | Lower-left anchored content      | Left         | Row           |
| Tablet         | 768px–1279px   | Lower-left anchored content      | Left         | Row           |
| Mobile         | < 768px        | Bottom anchored, full-bleed      | Center       | Column        |

## Appendix: Animation Timing Summary

| Element            | Duration | Delay  | Easing                          | Effect           |
|--------------------|----------|--------|---------------------------------|------------------|
| BG image           | 1200ms   | 0ms    | `cubic-bezier(0.25,.46,.45,.94)` | Fade + scale(1.04→1) |
| Gradient overlay   | 800ms    | 200ms  | `ease-out`                      | Fade             |
| Eyebrow            | 700ms    | 300ms  | `cubic-bezier(0.16, 1, 0.3, 1)` | Fade-up 20px     |
| H1 Headline        | 700ms    | 450ms  | `cubic-bezier(0.16, 1, 0.3, 1)` | Fade-up 20px     |
| Subheadline        | 700ms    | 600ms  | `cubic-bezier(0.16, 1, 0.3, 1)` | Fade-up 20px     |
| CTA group          | 700ms    | 780ms  | `cubic-bezier(0.16, 1, 0.3, 1)` | Fade-up 20px     |

---

*End of Hero Section Prototype Specification — Café Vermut · Iteration 1*
