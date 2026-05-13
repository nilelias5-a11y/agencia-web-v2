# UI Specification — Ana López Photography Portfolio
**Skill:** ui-designer | **Phase:** 1–6 complete  
**Client:** Ana López — photographer, Estilo editorial / fine-art  
**Date:** 2026-05-13

---

## PHASE 1 — Inspiration Research

### Reference 1 — Raphaël Bourdin Portfolio

**URL:** https://www.awwwards.com/sites/raphael-bourdin-portfolio-1  
**Award:** Awwwards SOTD  
**What makes it exceptional:**
- Strict 2-token palette (near-black #050709 / near-white #FCFCFC) — zero visual noise, image is absolute king
- Nav floats as a barely-visible strip; logo + links share the same optical weight; no borders, no backgrounds
- Portfolio reveals use a clip-path wipe: a dark rectangle sweeps left-to-right as the cursor enters the card, exposing detail meta-text beneath

**Elements to adapt:**
- Clip-path reveal on card hover → adapt as a bottom-to-top overlay wipe in Ana's warm palette
- Equal optical weight across all nav items — achieved with letter-spacing rather than bold

---

### Reference 2 — Cyd Stumpel Portfolio 2025

**URL:** https://www.awwwards.com/sites/cyd-stumpel-portfolio-2025  
**Award:** Awwwards SOTD 2025  
**What makes it exceptional:**
- Cormorant-class serif display type contrasts with utilitarian body — same strategy as Ana's type brief
- Grid does NOT use equal-height rows — row height is dictated by tallest image in each row, preserving aspect ratios
- Hover state: desaturate all other cards to 20% opacity; hovered card remains full color → focus effect

**Elements to adapt:**
- Sibling-dimming hover: when one card is hovered all other cards drop to opacity 0.35 over 250 ms
- Aspect ratio preservation over forced equal-height boxes

---

### Reference 3 — Framer "Hover Reveal" Editorial Pattern (Framer Marketplace)

**URL:** https://www.framer.com/marketplace/components/simple-hover-reveal/  
**Award:** Widely referenced Awwwards-nominated interaction pattern  
**What makes it exceptional:**
- Typography paired with image reveal: user hovers a project title and the related image materialises from behind a mask
- 0-height clip-rect expands to full height in 400 ms with ease-out — cinematic, not cheap
- Works at full viewport width; no card border or shadow required

**Elements to adapt:**
- Clip-mask expand as the primary reveal mechanic for the portfolio overlay (category label + project name)

---

### Reference 4 — Lara Jade Photography (larajade.com)

**URL:** https://www.larajade.com/  
**Award:** Frequently cited in Siteinspire editorial photography category  
**What makes it exceptional:**
- Ultra-minimal white navbar; logo is simply the name in a refined serif; zero decoration
- 3-column masonry: images are never cropped to equal boxes — height is intrinsic
- Mobile: single-column stack, full-bleed images edge to edge with 0 side padding

**Elements to adapt:**
- Logo = Cormorant Garamond 400 / 18px name only — no icon, no monogram
- Mobile stack with 0px outer margin for portfolio images (full-bleed feel)

---

### Reference 5 — Antoine Wodniack Portfolio (CSSDA Nominee 2024)

**URL:** https://www.cssdesignawards.com/woty2024/sites/antoine-wodniack-portfolio  
**Award:** CSS Design Awards 2024 WOTY Nominee  
**What makes it exceptional:**
- Hamburger icon is a 3-line mark that morphs into an X via staggered translate + rotate — takes 300 ms, feels architectural
- Mobile menu occupies full viewport with large nav links (clamp 32px–56px) — no half-measures
- CTA button in nav uses a border that "fills" on hover rather than a background toggle

**Elements to adapt:**
- Animated hamburger → X morphism (exact timing specified in Phase 4)
- Full-viewport mobile menu with large type
- CTA button: border-fill technique using clip-path or pseudo-element

---

### Design Direction Brief

Ana López's site embodies the visual language of the fine-art photography book: sparse, intentional, and entirely subordinate to the image. The palette is warm-neutral rather than cold-minimal — the off-white and charcoal carry paper-and-ink references, while the gold accent reads as a discrete editorial mark rather than a promotional highlight. Navigation is so unobtrusive it disappears; the portfolio grid is the page. Motion is slow and revelatory — everything feels like a page turning, not a UI clicking. The only moments of assertiveness are the CTA button (a bordered gold element that fills on hover) and the card overlay (a bottom-to-top dark wipe that surfaces project metadata). Spacing is generous to the point of feeling like silence between photographs.

---

## PHASE 2 — Component Inventory

| # | Component | Category | Status |
|---|-----------|----------|--------|
| 1 | Button — CTA Primary ("Hablemos") | Primitive | **Required** |
| 2 | Button — Ghost (secondary actions) | Primitive | Optional |
| 3 | Nav Link — desktop | Primitive | **Required** |
| 4 | Nav Link — active/current state | Primitive | **Required** |
| 5 | Hamburger Icon (3-line → X) | Primitive | **Required** |
| 6 | Logo Wordmark | Primitive | **Required** |
| 7 | Image (portfolio card) | Primitive | **Required** |
| 8 | Badge / Category Label | Primitive | **Required** (in card overlay) |
| 9 | Divider | Primitive | Optional |
| 10 | Spinner / Loader | Primitive | Conditional (image load) |
| 11 | Tooltip | Primitive | Optional |
| 12 | Navbar — Desktop | Composite | **Required** |
| 13 | Navbar — Mobile | Composite | **Required** |
| 14 | Mobile Menu Overlay (fullscreen) | Composite | **Required** |
| 15 | Portfolio Card | Composite | **Required** |
| 16 | Portfolio Grid | Section | **Required** |
| 17 | Hero Section | Section | **Required** |
| 18 | Contact / CTA Section | Section | **Required** |
| 19 | Footer — Minimal | Section | **Required** |
| 20 | 404 / Empty State | Section | Conditional |

---

## PHASE 3 — Per-State Visual Specification

### Brand Tokens (referenced throughout)

```
--color-bg:          #F8F6F3   /* off-white */
--color-fg:          #2C2C2C   /* charcoal */
--color-accent:      #C9A96E   /* dorado */
--color-accent-dim:  #B8935A   /* dorado darkened 10% — pressed state */
--color-overlay:     rgba(44, 44, 44, 0.82)   /* card overlay bg */
--color-overlay-light: rgba(248, 246, 243, 0.96)  /* mobile menu bg */

--font-display: 'Cormorant Garamond', Georgia, serif
--font-body:    'Inter', system-ui, sans-serif

--font-size-xs:   11px
--font-size-sm:   13px
--font-size-base: 15px
--font-size-md:   17px
--font-size-lg:   20px
--font-size-xl:   24px
--font-size-2xl:  32px
--font-size-3xl:  48px
--font-size-4xl:  64px

--letter-spacing-nav:    0.08em
--letter-spacing-label:  0.12em
--letter-spacing-cta:    0.06em
```

---

## Component — Navbar (Desktop)

**Layout:** `position: sticky; top: 0; z-index: 100`  
**Height:** 72px  
**Background:** `--color-bg` at `opacity: 1`; transitions to `backdrop-filter: blur(12px); background: rgba(248, 246, 243, 0.92)` when page scrolled > 40px (transition: 300ms ease)  
**Border-bottom:** none by default; `1px solid rgba(44,44,44,0.08)` appears when scrolled > 40px (transition: 300ms ease)  
**Max-width container:** 1320px, centered with `margin: 0 auto; padding: 0 48px`  
**Flex layout:** `display: flex; align-items: center; justify-content: space-between`

**Left zone:** Logo wordmark  
**Center zone:** 3 nav links (absolute centered via `position: absolute; left: 50%; transform: translateX(-50%)`)  
**Right zone:** CTA button "Hablemos"

---

### Sub-component — Logo Wordmark

#### State: Default
- Font family: `--font-display`
- Font size: 18px
- Font weight: 400
- Letter spacing: 0.04em
- Color: `--color-fg` (#2C2C2C)
- Text transform: none
- Text decoration: none
- Cursor: pointer (links to `/`)

#### State: Hover
- Transition: `color 200ms ease`
- Color: `--color-accent` (#C9A96E)

#### State: Focus
- Outline: `2px solid #C9A96E; outline-offset: 4px`

#### State: Active
- Color: `--color-accent-dim` (#B8935A)
- Transition: `color 80ms ease`

---

### Sub-component — Nav Link (Desktop)

#### State: Default
- Font family: `--font-body`
- Font size: 12px
- Font weight: 400
- Letter spacing: `--letter-spacing-nav` (0.08em)
- Text transform: uppercase
- Color: `--color-fg` (#2C2C2C) at `opacity: 0.7`
- Text decoration: none
- Position: relative (for underline pseudo-element)
- Padding: 4px 0
- Gap between links: 40px

#### Underline decoration (pseudo-element):
```css
::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 0;
  height: 1px;
  background: --color-accent; /* #C9A96E */
  transition: width 300ms cubic-bezier(0.0, 0.0, 0.2, 1.0);
}
```

#### State: Hover
- Transition: `opacity 200ms ease`
- Opacity: 1 (from 0.7)
- `::after` width: 100% (underline slides in left-to-right)
- Cursor: pointer

#### State: Focus
- Outline: `2px solid #C9A96E; outline-offset: 6px`
- Opacity: 1

#### State: Active (current page)
- Opacity: 1
- Font weight: 400 (unchanged — weight distinction is avoided; underline is the marker)
- `::after` width: 100% (permanently visible, static)
- `::after` background: `--color-accent`

#### State: Pressed (click moment)
- Opacity: 0.5
- Transition: `opacity 80ms ease`

---

### Sub-component — CTA Button "Hablemos"

#### State: Default
- Font family: `--font-body`
- Font size: 11px
- Font weight: 500
- Letter spacing: `--letter-spacing-cta` (0.06em)
- Text transform: uppercase
- Color: `--color-fg` (#2C2C2C)
- Background: transparent
- Border: `1px solid rgba(44, 44, 44, 0.35)`
- Border-radius: 0 (sharp — editorial language)
- Padding: 10px 24px
- Height: 36px
- Shadow: none
- Position: relative
- Overflow: hidden

#### Hover fill effect (pseudo-element):
```css
::before {
  content: '';
  position: absolute;
  inset: 0;
  background: --color-accent; /* #C9A96E */
  transform: scaleX(0);
  transform-origin: left center;
  transition: transform 350ms cubic-bezier(0.0, 0.0, 0.2, 1.0);
  z-index: 0;
}
span { position: relative; z-index: 1; } /* text above fill */
```

#### State: Hover
- Transition (border): `border-color 350ms ease`
- Border-color: `--color-accent` (#C9A96E)
- `::before` transform: `scaleX(1)` — fill sweeps left to right
- Color: `--color-bg` (#F8F6F3) — text turns off-white as fill covers it
- Cursor: pointer

#### State: Focus
- Outline: `2px solid #C9A96E; outline-offset: 4px`
- (Fill does NOT trigger on focus-only; only visual outline marker)

#### State: Active (pressed)
- `::before` background: `--color-accent-dim` (#B8935A)
- Transform: `scale(0.98)` on the button element
- Transition: `transform 80ms ease, background 80ms ease`

#### State: Disabled (if form is sending)
- Opacity: 0.45
- Cursor: not-allowed
- Pointer-events: none
- Border-color: `rgba(44, 44, 44, 0.20)`

---

## Component — Navbar (Mobile)

**Breakpoint trigger:** `< 768px`  
**Height:** 60px  
**Background:** `--color-bg` (#F8F6F3) — same scroll-aware behavior as desktop  
**Border-bottom:** same conditional as desktop  
**Padding:** `0 20px`  
**Flex layout:** `display: flex; align-items: center; justify-content: space-between`

**Left zone:** Logo wordmark (same spec as desktop but font-size: 16px)  
**Right zone:** Hamburger icon (nav links and CTA are hidden; revealed in mobile menu overlay)

---

### Sub-component — Hamburger Icon

**Container size:** 40px × 40px (touch target — minimum 44×44 achieved with padding: 2px)  
**Icon visible area:** 20px × 14px  
**3 bars specs:**
- Width: 20px
- Height: 1.5px (not 2px — thinner is more refined)
- Background: `--color-fg` (#2C2C2C)
- Border-radius: 0
- Gap between bars: 5px (making total visual height = 1.5 + 5 + 1.5 + 5 + 1.5 = 14.5px)

#### State: Default (menu closed)
- Bar 1: `translateY(0) rotate(0deg)`
- Bar 2: `translateY(0) scaleX(1) opacity(1)`
- Bar 3: `translateY(0) rotate(0deg)`

#### State: Hover (menu closed)
- All 3 bars: `scaleX` varies — bar 1: 0.7, bar 2: 1.0, bar 3: 0.85 (staggered width creates rhythm)
- Transition: `transform 200ms ease` staggered at 0ms / 40ms / 80ms
- This "breathing" micro-motion signals interactivity without aggression

#### State: Focus
- Container outline: `2px solid #C9A96E; outline-offset: 4px; border-radius: 2px`

#### State: Active / Menu Open (X morphism):
```
Bar 1: translateY(6.5px) rotate(45deg)   — duration: 300ms
Bar 2: scaleX(0) opacity(0)              — duration: 200ms, delay: 0ms
Bar 3: translateY(-6.5px) rotate(-45deg) — duration: 300ms
```
- Easing: `cubic-bezier(0.34, 1.56, 0.64, 1.0)` (spring — slight overshoot gives the bars a snapping-shut feel)
- All three transitions fire simultaneously

#### State: Closing (X → burger):
- Reverse of above
- Same duration / easing
- Color during open state: `--color-fg` (#2C2C2C) — no color change

---

### Sub-component — Mobile Menu Overlay

**Trigger:** hamburger click; menu open state  
**Position:** `fixed; inset: 0; z-index: 200`  
**Background:** `--color-overlay-light` (`rgba(248, 246, 243, 0.96)`) + `backdrop-filter: blur(8px)`  
**Padding-top:** 100px (below fixed navbar)  
**Padding-horizontal:** 32px  

**Entrance animation:**
- Clip-path from `inset(0 0 100% 0)` → `inset(0 0 0% 0)` — wipes down from top
- Duration: 400ms
- Easing: `cubic-bezier(0.0, 0.0, 0.2, 1.0)` (ease-out)
- Children stagger: each link fades up (`translateY: 16px → 0, opacity: 0 → 1`) with 60ms stagger delay after menu wipe completes (delay: 320ms + n×60ms)

**Exit animation:** reverse clip-path wipe upward, 300ms ease-in; children fade out simultaneously (no stagger on exit)

#### Mobile Nav Links (inside overlay)
- Font family: `--font-display`
- Font size: `clamp(36px, 8vw, 56px)`
- Font weight: 300
- Letter spacing: 0.02em
- Color: `--color-fg` (#2C2C2C)
- Text decoration: none
- Display: block
- Margin-bottom: 24px
- Line-height: 1.1

**Hover state (mobile link):**
- Color: `--color-accent` (#C9A96E)
- Transition: `color 200ms ease`

**Active/current page (mobile link):**
- Color: `--color-accent` (#C9A96E) — permanent

#### CTA in Mobile Menu
- Same button spec as desktop CTA
- Margin-top: 48px
- Width: fit-content (not full width)
- Display: inline-block

#### Close affordance
- Hamburger icon (now showing X) remains tappable in fixed navbar position
- No separate "close" button needed — X in navbar position is sufficient and familiar

---

## Component — Portfolio Card

### Structure
```
<article class="portfolio-card">
  <div class="card-media">          <!-- aspect ratio wrapper -->
    <img src="…" alt="…" />
    <div class="card-overlay">      <!-- hover overlay -->
      <span class="overlay-category">EDITORIAL</span>
      <h3 class="overlay-title">Título del proyecto</h3>
    </div>
  </div>
</article>
```

### Image Container (card-media)
- Aspect ratio: `4 / 5` (portrait — optimal for photography)
- Overflow: hidden
- Background: `rgba(44, 44, 44, 0.04)` — placeholder color while loading
- Width: 100% (fills grid column)

### Image element
- Width: 100%
- Height: 100%
- Object-fit: cover
- Object-position: center center
- Display: block

---

### State: Default
- `img` scale: 1.0
- Overlay: `clip-path: inset(100% 0 0 0)` — overlay completely hidden below card (starts clipped at bottom)
- Overlay background: `--color-overlay` (`rgba(44, 44, 44, 0.82)`)
- Shadow: none
- Border-radius: 0 (consistent with editorial language)

### State: Hover
**On `.card-media` enter:**

1. **Image zoom (subtle):**
   - `img` transform: `scale(1.04)`
   - Duration: 600ms
   - Easing: `cubic-bezier(0.0, 0.0, 0.2, 1.0)`

2. **Overlay reveal (clip-path wipe — bottom to top):**
   - clip-path: `inset(100% 0 0 0)` → `inset(0 0 0 0)`
   - Duration: 500ms
   - Easing: `cubic-bezier(0.0, 0.0, 0.2, 1.0)`

3. **Sibling dimming:**
   - All other `.portfolio-card` elements in the grid: `opacity: 0.35`
   - Transition: `opacity 250ms ease`
   - The hovered card remains at `opacity: 1`

4. **Cursor:** pointer

### State: Leave (mouse out)
- Overlay clip-path: `inset(0 0 0 0)` → `inset(100% 0 0 0)` (wipes back down)
- Duration: 350ms (slightly faster than enter)
- Easing: `cubic-bezier(0.4, 0.0, 1.0, 1.0)` (ease-in — recedes quickly)
- Image scale: returns to 1.0, same 350ms ease-out
- Sibling opacity: returns to 1.0, 350ms ease

### State: Focus (keyboard navigation)
- `card-media` outline: `2px solid #C9A96E; outline-offset: 4px`
- Overlay reveals (same as hover) — triggered by `:focus-within`

---

### Overlay Content

#### Category Label (.overlay-category)
- Font family: `--font-body`
- Font size: 10px
- Font weight: 500
- Letter spacing: 0.14em
- Text transform: uppercase
- Color: `--color-accent` (#C9A96E)
- Margin-bottom: 8px
- Display: block

#### Project Title (.overlay-title)
- Font family: `--font-display`
- Font size: `clamp(18px, 2.5vw, 24px)`
- Font weight: 300
- Letter spacing: 0.01em
- Color: `--color-bg` (#F8F6F3)
- Line-height: 1.2
- Margin: 0

#### Overlay positioning
- Position: absolute; bottom: 0; left: 0; right: 0
- Padding: 24px 20px
- Background: `linear-gradient(to top, rgba(44,44,44,0.92) 0%, rgba(44,44,44,0.60) 60%, transparent 100%)`
  — gradient rather than flat color gives image more breathing room at top of overlay
- The clip-path sits on the outer overlay div, not the gradient background

**Entrance animation for overlay text:**
- Category label: `translateY(8px) opacity(0)` → `translateY(0) opacity(1)`, delay: 200ms (after clip-path completes), duration: 250ms
- Project title: `translateY(8px) opacity(0)` → `translateY(0) opacity(1)`, delay: 260ms, duration: 250ms
- Easing: ease-out

---

### Mobile Card State (< 768px)

- Overlay is ALWAYS VISIBLE at bottom of card (not hover-dependent — touch has no hover)
- Clip-path: `inset(0 0 0 0)` — fully revealed permanently
- Overlay background: gradient (same as hover on desktop)
- No sibling dimming (no hover concept on touch)
- Image scale: fixed at 1.0 (no zoom on mobile)
- Aspect ratio: `3 / 4` on mobile (slightly less portrait to show more image)

---

## Portfolio Grid

### Desktop (≥ 1024px)
- Columns: 3
- Implementation: `display: grid; grid-template-columns: repeat(3, 1fr)`
- Column gap: 16px
- Row gap: 16px
- Max-width: 1320px (full bleed up to this, then centered)
- Outer padding: 0 48px
- First card can optionally span 2 columns (`grid-column: span 2`) — see featured card variant below

### Featured Card Variant (optional, recommended for hero card)
- First card in grid: `grid-column: span 2; aspect-ratio: 8/5` (wider, landscape)
- Remaining 2 cards in row 1: `grid-column: span 1; aspect-ratio: 4/5`
- This creates a 2+1 first row, then 1+1+1 for subsequent rows
- Allows visual rhythm without a masonry complexity penalty

### Tablet (768px – 1023px)
- Columns: 2
- Implementation: `grid-template-columns: repeat(2, 1fr)`
- Column gap: 12px
- Row gap: 12px
- Outer padding: 0 32px
- Featured card: `grid-column: span 2; aspect-ratio: 16/9`

### Mobile (< 768px)
- Columns: 1
- Implementation: `grid-template-columns: 1fr`
- Column gap: n/a
- Row gap: 4px (near-zero gap — images tile with minimum breathing room, newspaper-strip feel)
- Outer padding: 0 (full-bleed — no side margins on images)
- Max 4 cards shown before "Ver más" link (reduces initial load weight on mobile)

---

## PHASE 4 — Animation & Micro-interaction Specification

### Motion System

**Philosophy:** Slow and revelatory — every animation references the pace of a page turning in a photography book. Motion exists to reveal, not to entertain. Nothing accelerates; everything decelerates.

**Default easing:** `cubic-bezier(0.25, 0.1, 0.25, 1.0)`  
**Ease-out (elements entering):** `cubic-bezier(0.0, 0.0, 0.2, 1.0)`  
**Ease-in (elements leaving):** `cubic-bezier(0.4, 0.0, 1.0, 1.0)`  
**Spring (snap interactions):** `cubic-bezier(0.34, 1.56, 0.64, 1.0)`  

**Duration scale:**
```
--duration-instant:   80ms    — button press feedback
--duration-fast:      200ms   — link hover, opacity fades
--duration-normal:    300ms   — navbar, dropdown, hamburger
--duration-slow:      500ms   — card overlay reveal, modal
--duration-cinematic: 800ms   — hero entrance, page transitions
--duration-reveal:    600ms   — scroll-triggered section reveals
```

---

### Scroll Animation — Portfolio Grid Section

**Trigger:** IntersectionObserver threshold: 0.15 (card enters view when 15% visible)  
**Effect:** Fade up — `translateY(32px) opacity(0)` → `translateY(0) opacity(1)`  
**Duration:** 600ms  
**Stagger:** 100ms between each card (cards that are visible simultaneously stagger left-to-right)  
**Easing:** ease-out `cubic-bezier(0.0, 0.0, 0.2, 1.0)`  
**Note:** Only first-time entrance — once a card has appeared, no re-trigger on scroll back up

---

### Scroll Animation — Hero Section

**Trigger:** on page load (no intersection needed — always visible)  
**Effect:** Two elements animate independently:
1. Headline: `translateY(24px) opacity(0)` → `translateY(0) opacity(1)`, delay 200ms, duration 800ms
2. Hero image: `scale(1.04) opacity(0)` → `scale(1.0) opacity(1)`, delay 0ms, duration 1000ms  
**Easing:** ease-out

---

### Scroll Animation — Section Titles / Text Blocks

**Trigger:** IntersectionObserver threshold: 0.2  
**Effect:** Fade up: `translateY(20px) opacity(0)` → `translateY(0) opacity(1)`  
**Duration:** 500ms  
**Easing:** ease-out  

---

### Micro-interaction — Navigation Link Hover

**What it communicates:** "This is a destination worth considering" — the underline grow signals selection possibility without commitment.  
**Mechanic:** Pseudo-element `::after` `width: 0 → 100%`, 300ms ease-out  
**Why slow for a nav link:** Photography sites reward deliberate browsing; a fast-snap underline would feel tech-product-like, not editorial.

---

### Micro-interaction — CTA Button "Hablemos"

**What it communicates:** "Clicking this initiates a human conversation" — the warm gold fill spreading from left signals warmth and invitation, not urgency.  
**Mechanic:** `::before` pseudo-element scaleX fill from 0 to 1, 350ms ease-out  
**Color change on fill:** text transitions from charcoal to off-white simultaneously, ensuring legibility throughout the fill animation  
**Why the border changes color before fill completes:** At t=0 border is charcoal-20; at hover start border transitions to accent gold over 350ms — this means the border is "welcoming" the fill before it arrives.

---

### Micro-interaction — Portfolio Card Reveal

**What it communicates:** "Hover is a moment of curiosity; the overlay rewards it without interrupting." The wipe from below mimics a title card emerging from darkness, referencing film credits.  
**Why sibling dimming:** When one image is foregrounded all others recede — this is how a gallery visitor's focus works physically. It's not decorative, it directs.

---

### Micro-interaction — Hamburger → X Morph

**What it communicates:** "The menu is now open and the icon tells you how to close it."  
**Spring easing** (`cubic-bezier(0.34, 1.56, 0.64, 1.0)`) on bars 1 and 3 gives a tiny overshoot — the bars "snap" into the X like a latch closing. Middle bar vanishes before snap completes, so the user sees 2 bars crossing cleanly.  
**Timing breakdown:**
- 0ms: bar 2 starts `scaleX: 1 → 0`
- 0ms: bars 1 and 3 start rotate+translate
- 200ms: bar 2 fully gone (scaleX 0, opacity 0)
- 300ms: bars 1 and 3 reach final X position (with spring overshoot completing at ~320ms)

---

### prefers-reduced-motion

```css
@media (prefers-reduced-motion: reduce) {
  /* Scroll reveals: opacity fade only — no translateY */
  [data-animate] {
    transform: none !important;
    transition-property: opacity !important;
    transition-duration: 300ms !important;
  }

  /* Card hover: no image scale, no clip-path wipe — overlay appears instantly */
  .card-media img {
    transition: none !important;
    transform: none !important;
  }
  .card-overlay {
    clip-path: none !important;
    transition: opacity 200ms ease !important;
    opacity: 0;
  }
  .portfolio-card:hover .card-overlay,
  .portfolio-card:focus-within .card-overlay {
    opacity: 1;
  }

  /* Hamburger: instant state change — no morph animation */
  .hamburger-bar {
    transition: none !important;
  }

  /* Hero entrance: no scale, opacity-only fade */
  .hero-image {
    transform: none !important;
  }

  /* CTA button: fill appears instantly on hover — no sweep */
  .cta-button::before {
    transition: none !important;
  }

  /* Mobile menu: no clip-path wipe — appears instantly with opacity */
  .mobile-menu {
    clip-path: none !important;
    transition: opacity 200ms ease !important;
  }
}
```

---

## PHASE 5 — Spacing & Grid System

### Spacing Tokens

```css
:root {
  --space-1:   4px;
  --space-2:   8px;
  --space-3:   12px;
  --space-4:   16px;
  --space-5:   20px;
  --space-6:   24px;
  --space-8:   32px;
  --space-10:  40px;
  --space-12:  48px;
  --space-16:  64px;
  --space-20:  80px;
  --space-24:  96px;
  --space-32:  128px;
  --space-40:  160px;
  --space-48:  192px;
}
```

### Token Usage Map

| Purpose | Token | Value |
|---------|-------|-------|
| Navbar height (desktop) | custom | 72px |
| Navbar height (mobile) | custom | 60px |
| Navbar horizontal padding (desktop) | `--space-12` | 48px |
| Navbar horizontal padding (mobile) | `--space-5` | 20px |
| Gap between desktop nav links | `--space-10` | 40px |
| CTA button padding (horizontal) | `--space-6` | 24px |
| CTA button padding (vertical) | custom | 10px |
| Portfolio grid column gap (desktop) | `--space-4` | 16px |
| Portfolio grid row gap (desktop) | `--space-4` | 16px |
| Portfolio grid column gap (tablet) | `--space-3` | 12px |
| Portfolio grid row gap (mobile) | `--space-1` | 4px |
| Card overlay inner padding | `--space-6` `--space-5` | 24px top/bottom / 20px left/right |
| Section vertical padding (desktop) | `--space-32` | 128px |
| Section vertical padding (tablet) | `--space-20` | 80px |
| Section vertical padding (mobile) | `--space-12` | 48px |
| Gap between section title and content | `--space-12` | 48px |
| Footer vertical padding | `--space-16` | 64px |
| Mobile menu padding-top | `--space-24` + navbar height | 100px |
| Mobile menu link margin-bottom | `--space-6` | 24px |
| Mobile menu CTA margin-top | `--space-12` | 48px |

---

### Grid Specification

#### Desktop (≥ 1280px)
- Columns: 12
- Column width: auto (fluid)
- Gutter: 24px
- Max content width: 1320px
- Outer margin: auto (centered)
- Outer padding: 48px (inside max-width container)
- Portfolio grid uses: 3 equal columns within the 12-col grid (each spans 4 of 12)

#### Tablet (768px – 1279px)
- Columns: 8
- Gutter: 20px
- Outer margin: 32px
- Portfolio grid uses: 2 equal columns (each spans 4 of 8)

#### Mobile (< 768px)
- Columns: 4
- Gutter: 16px
- Outer margin: 20px (but portfolio images are FULL-BLEED: outer margin = 0 for `.portfolio-grid`)
- Portfolio grid uses: 1 column, full width

#### Breakpoints
```css
--bp-sm:  640px;
--bp-md:  768px;
--bp-lg:  1024px;
--bp-xl:  1280px;
--bp-2xl: 1536px;
```

---

## PHASE 6 — Shadow & Border Radius Scale

### Shadow Tokens

```css
:root {
  --shadow-xs:    0 1px 2px rgba(0,0,0,0.05);
  --shadow-sm:    0 1px 3px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06);
  --shadow-md:    0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
  --shadow-lg:    0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05);
  --shadow-xl:    0 20px 25px rgba(0,0,0,0.1), 0 10px 10px rgba(0,0,0,0.04);
  --shadow-2xl:   0 25px 50px rgba(0,0,0,0.25);
  --shadow-inner: inset 0 2px 4px rgba(0,0,0,0.06);
  --shadow-none:  none;
}
```

### Shadow Usage Map

| Element | Token | Rationale |
|---------|-------|-----------|
| Portfolio card — default | `--shadow-none` | Images need no elevation — they live on the page surface |
| Portfolio card — hover | `--shadow-none` | Hover effect is the overlay reveal, not a lift. Adding shadow would dilute the clip-path reveal. |
| Navbar — default | `--shadow-none` | Navbar merges with page background |
| Navbar — scrolled (> 40px) | `--shadow-sm` | Subtle separation when navbar floats above content |
| Mobile menu overlay | `--shadow-none` | Full-viewport overlay has no need for drop shadow |
| CTA button — default | `--shadow-none` | Border-only aesthetic is intentional; no shadow |
| CTA button — hover | `--shadow-none` | Fill effect replaces any need for shadow |
| Modal / Dialog (if added later) | `--shadow-2xl` | Deep shadow for modal overlay separation |
| Focused input | `--shadow-inner` | Subtle inset signal for active input |

---

### Border Radius Tokens

```css
:root {
  --radius-none: 0;
  --radius-xs:   2px;
  --radius-sm:   4px;
  --radius-md:   6px;
  --radius-lg:   8px;
  --radius-xl:   12px;
  --radius-2xl:  16px;
  --radius-3xl:  24px;
  --radius-full: 9999px;
}
```

### Radius Usage Map

| Element | Token | Value | Rationale |
|---------|-------|-------|-----------|
| CTA Button "Hablemos" | `--radius-none` | 0 | Sharp corners = editorial, architectural — not app-like |
| Portfolio Card | `--radius-none` | 0 | Photographs extend edge-to-edge like printed prints |
| Nav link underline | `--radius-none` | 0 | Underline is a ruled line, not a pill |
| Hamburger icon container (focus ring) | `--radius-xs` | 2px | Minimal rounding on focus outline only |
| Mobile menu overlay | `--radius-none` | 0 | Full-viewport panel has no rounded corners |
| Input fields (contact form) | `--radius-none` | 0 | Consistent editorial sharpness |
| Badge / Category label | `--radius-none` | 0 | Labels are typographic, not pill-shaped |
| Tooltip (if used) | `--radius-sm` | 4px | Small functional UI element — minimal rounding acceptable |
| Modal (if added) | `--radius-none` | 0 | Consistent with site language |
| Toast notification | `--radius-xs` | 2px | Minimal rounding to distinguish from content |

---

## Complete Specification Summary

### Design System Quick Reference

```
COLORS
  Background:  #F8F6F3
  Foreground:  #2C2C2C
  Accent:      #C9A96E
  Accent dark: #B8935A
  Overlay bg:  rgba(44,44,44,0.82)
  Mobile menu: rgba(248,246,243,0.96)

TYPOGRAPHY
  Display (headings): Cormorant Garamond, 300–400 weight
  Body (UI):          Inter, 400–500 weight
  Nav links:          Inter 400, 12px, uppercase, 0.08em tracking
  CTA button:         Inter 500, 11px, uppercase, 0.06em tracking
  Logo:               Cormorant Garamond 400, 18px (desktop), 16px (mobile)
  Card category:      Inter 500, 10px, uppercase, 0.14em tracking
  Card title:         Cormorant Garamond 300, clamp(18px, 2.5vw, 24px)
  Mobile menu links:  Cormorant Garamond 300, clamp(36px, 8vw, 56px)

SPACING
  Navbar (desktop): 72px height, 48px h-padding
  Navbar (mobile):  60px height, 20px h-padding
  Section gap:      128px desktop / 80px tablet / 48px mobile
  Grid gap:         16px desktop / 12px tablet / 4px mobile

GRID
  Desktop:  3 cols, 1320px max, 48px outer padding
  Tablet:   2 cols, 32px outer margin
  Mobile:   1 col, full-bleed (0px margin for images)

ANIMATION
  Reveal:   clip-path bottom-to-top, 500ms ease-out
  Zoom:     scale 1.0→1.04, 600ms ease-out
  Siblings: opacity 1.0→0.35, 250ms ease
  Nav UL:   width 0→100%, 300ms ease-out
  Button:   scaleX fill 0→1, 350ms ease-out
  Hamburger→X: spring cubic-bezier(0.34,1.56,0.64,1.0), 300ms

RADIUS: All 0 (--radius-none) — zero rounding site-wide
SHADOWS: None on content; --shadow-sm on scrolled navbar only
```

---

*Specification produced following ui-designer skill Phase 1–6 workflow.*  
*References: Raphaël Bourdin (Awwwards SOTD), Cyd Stumpel Portfolio 2025 (Awwwards SOTD), Framer Hover Reveal pattern, Lara Jade Photography (Siteinspire editorial), Antoine Wodniack (CSSDA 2024 Nominee).*
