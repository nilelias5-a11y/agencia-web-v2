# Café Vermut — UI Component Specifications
**Brand tokens applied:** brand-primary #A0522D · brand-secondary #556B2F · bg-base #FAF7F0  
**Typography:** Playfair Display (headings, 400/700) + Lato (body, 400/600)  
**Aesthetic:** artisanal Mediterranean

---

## 1. PRIMARY BUTTON — All States

### Base anatomy

```
font-family: 'Lato', sans-serif
font-size: 0.9375rem       /* 15px */
font-weight: 600
letter-spacing: 0.06em
text-transform: uppercase
line-height: 1             /* single-line label */

padding-block: 0.75rem     /* 12px top + bottom */
padding-inline: 1.75rem    /* 28px left + right */
height: 2.75rem            /* 44px — WCAG tap target */
min-width: 9rem            /* 144px */
border-radius: 2px         /* nearly square, artisanal edge */
border: 1.5px solid transparent

display: inline-flex
align-items: center
justify-content: center
gap: 0.5rem                /* 8px — icon + label spacing */
cursor: pointer
user-select: none
text-decoration: none
```

### State: DEFAULT

```css
background-color: #A0522D               /* brand-primary */
border-color: #A0522D
color: #FAF7F0                          /* bg-base — warm cream on terracotta */
box-shadow: 0 1px 3px rgba(0,0,0,0.12),
            0 1px 2px rgba(160,82,45,0.18)

transition:
  background-color 200ms cubic-bezier(0.4, 0, 0.2, 1),
  border-color     200ms cubic-bezier(0.4, 0, 0.2, 1),
  box-shadow       200ms cubic-bezier(0.4, 0, 0.2, 1),
  transform        150ms cubic-bezier(0.34, 1.56, 0.64, 1)
```

### State: HOVER

```css
background-color: #8B4513               /* +10% darken — saddlebrown */
border-color: #8B4513
color: #FAF7F0
box-shadow: 0 4px 12px rgba(160,82,45,0.30),
            0 2px 4px rgba(0,0,0,0.10)
transform: translateY(-1px)

/* Subtle texture overlay via pseudo-element */
/* ::after { opacity: 0.06; background: #FAF7F0; border-radius: inherit; } */
```

Transition out (pointer leaving): same 200ms, transform returns in 150ms.

### State: FOCUS (keyboard / :focus-visible)

```css
background-color: #A0522D              /* unchanged from default */
border-color: #A0522D
color: #FAF7F0
box-shadow: 0 1px 3px rgba(0,0,0,0.12),
            0 1px 2px rgba(160,82,45,0.18)
outline: 3px solid #556B2F             /* brand-secondary — olive ring */
outline-offset: 3px
/* DO NOT suppress outline — accessibility critical */
```

No transform on focus. Ring color #556B2F contrasts well against bg-base.

### State: ACTIVE (mousedown / :active)

```css
background-color: #7A3B10               /* -15% darken from primary */
border-color: #7A3B10
color: #FAF7F0
box-shadow: 0 1px 2px rgba(0,0,0,0.10),
            inset 0 1px 3px rgba(0,0,0,0.15)
transform: translateY(0) scale(0.984)
transition-duration: 80ms              /* snappy press feedback */
```

### State: DISABLED

```css
background-color: #C8A98A              /* tinted/muted terracotta */
border-color: #C8A98A
color: rgba(250,247,240,0.55)          /* bg-base at 55% opacity */
box-shadow: none
transform: none
cursor: not-allowed
pointer-events: none
opacity: 1                             /* color values do the work — no opacity hack */
```

WCAG contrast ratio (label on disabled bg): ≈ 3.1:1 — acceptable for disabled (non-operable) state.

### State: LOADING

```css
/* Button dimensions lock — no layout shift */
background-color: #A0522D
border-color: #A0522D
color: transparent                     /* hide text but preserve width */
box-shadow: 0 1px 3px rgba(0,0,0,0.12)
cursor: wait
pointer-events: none
position: relative
```

Spinner (pseudo-element overlay, centered):

```css
/* ::before */
content: ''
position: absolute
inset: 0
display: flex
align-items: center
justify-content: center

/* Spinner itself — ::after */
content: ''
position: absolute
top: 50%; left: 50%
width: 1rem            /* 16px */
height: 1rem
margin-top: -0.5rem
margin-left: -0.5rem
border: 2px solid rgba(250,247,240,0.30)
border-top-color: #FAF7F0
border-radius: 50%
animation: btn-spin 700ms linear infinite
```

```css
@keyframes btn-spin {
  to { transform: rotate(360deg); }
}
```

### Summary table — visual deltas per state

| State    | Background | Border  | Text opacity | Shadow depth | Transform          |
|----------|------------|---------|-------------|--------------|-------------------|
| Default  | #A0522D    | #A0522D | 100%        | 3px          | none               |
| Hover    | #8B4513    | #8B4513 | 100%        | 12px         | translateY(-1px)   |
| Focus    | #A0522D    | #A0522D | 100%        | 3px          | none + 3px ring    |
| Active   | #7A3B10    | #7A3B10 | 100%        | inset        | scale(0.984)       |
| Disabled | #C8A98A    | #C8A98A | 55%         | none         | none               |
| Loading  | #A0522D    | #A0522D | 0% (text)   | 3px          | spinner rotating   |

---

## 2. PRODUCT CARD — Café Molido

### Card container

```css
/* Structural */
display: flex
flex-direction: column
width: 100%
max-width: 18rem             /* 288px on grid */
border-radius: 4px
overflow: hidden
background-color: #FAF7F0    /* bg-base */
border: 1px solid rgba(160,82,45,0.12)   /* terracotta ghost border */

/* Elevation — resting */
box-shadow: 0 2px 6px rgba(0,0,0,0.07),
            0 1px 2px rgba(0,0,0,0.05)

/* Motion setup */
transform: translateY(0)
transition:
  box-shadow 260ms cubic-bezier(0.4, 0, 0.2, 1),
  transform  260ms cubic-bezier(0.34, 1.56, 0.64, 1),
  border-color 200ms cubic-bezier(0.4, 0, 0.2, 1)

cursor: pointer
```

### Image region

```
aspect-ratio: 4 / 3          /* 288 × 216px at max-width */
overflow: hidden
background-color: #EDE8DC    /* slightly darker cream for skeleton/loading */
position: relative
```

Image element:

```css
width: 100%
height: 100%
object-fit: cover
object-position: center
transform: scale(1)
transition: transform 400ms cubic-bezier(0.25, 0.46, 0.45, 0.94)
```

Badge (e.g. "Nuevo" / "Edición limitada"):

```css
position: absolute
top: 0.75rem          /* 12px */
left: 0.75rem
padding: 0.1875rem 0.5rem   /* 3px 8px */
background-color: #556B2F   /* brand-secondary */
color: #FAF7F0
font-family: 'Lato', sans-serif
font-size: 0.6875rem        /* 11px */
font-weight: 600
letter-spacing: 0.08em
text-transform: uppercase
border-radius: 2px
```

### Body region

```
padding: 1rem 1.125rem 0.875rem   /* 16px 18px 14px */
display: flex
flex-direction: column
gap: 0.25rem                       /* 4px between micro-elements */
flex: 1
```

Product name — Playfair Display 400, 1rem/1.3 (16px / ~21px line-height):

```css
font-family: 'Playfair Display', serif
font-size: 1rem
font-weight: 400
line-height: 1.3
color: #2C1A0E              /* deep espresso — not pure black */
letter-spacing: 0.01em
margin: 0
```

Subtitle / origin descriptor — Lato 400, 0.8125rem/1.5 (13px):

```css
font-family: 'Lato', sans-serif
font-size: 0.8125rem
font-weight: 400
line-height: 1.5
color: #7A6651              /* warm mid-brown */
letter-spacing: 0
margin: 0
```

Grind/roast tags — inline flex row, gap 0.375rem:

```css
/* each tag */
display: inline-block
padding: 0.125rem 0.4375rem   /* 2px 7px */
border: 1px solid rgba(85,107,47,0.35)   /* brand-secondary ghost */
border-radius: 999px
font-family: 'Lato', sans-serif
font-size: 0.6875rem          /* 11px */
font-weight: 600
color: #556B2F                /* brand-secondary */
letter-spacing: 0.04em
text-transform: capitalize
background: transparent
```

### Footer region (price + CTA)

```
padding: 0 1.125rem 1rem    /* 0 18px 16px */
display: flex
align-items: center
justify-content: space-between
margin-top: auto
```

Price:

```css
font-family: 'Playfair Display', serif
font-size: 1.1875rem        /* 19px */
font-weight: 700
color: #A0522D              /* brand-primary */
letter-spacing: -0.01em
line-height: 1
```

Price unit (e.g. "/ 250g"):

```css
font-family: 'Lato', sans-serif
font-size: 0.75rem          /* 12px */
font-weight: 400
color: #7A6651
margin-left: 0.125rem
```

Add-to-cart icon button:

```css
width: 2.25rem              /* 36px */
height: 2.25rem
border-radius: 50%
background-color: #A0522D
color: #FAF7F0
border: none
display: flex
align-items: center
justify-content: center
cursor: pointer
transition:
  background-color 180ms cubic-bezier(0.4, 0, 0.2, 1),
  transform        150ms cubic-bezier(0.34, 1.56, 0.64, 1)

/* icon inside: 1rem (16px) SVG/icon-font */
```

### State: CARD DEFAULT

As described above — box-shadow at 6px spread, border rgba(160,82,45,0.12), image scale(1).

### State: CARD HOVER

```css
/* Card container */
box-shadow: 0 8px 24px rgba(0,0,0,0.11),
            0 3px 8px rgba(160,82,45,0.12)
transform: translateY(-3px)
border-color: rgba(160,82,45,0.28)     /* border becomes more visible */

/* Image inside */
img {
  transform: scale(1.04)               /* gentle Ken Burns pan */
  transition-duration: 400ms
}

/* Add-to-cart button inside */
.add-btn {
  background-color: #8B4513
  transform: scale(1.08)
}
```

### State: CARD FOCUS (keyboard navigation / :focus-within)

```css
/* Card container — when any focusable child receives :focus-visible */
outline: 3px solid #556B2F
outline-offset: 3px
box-shadow: 0 2px 6px rgba(0,0,0,0.07),
            0 1px 2px rgba(0,0,0,0.05)
transform: none             /* no lift on keyboard focus — predictable */
```

### State: CARD OUT-OF-STOCK (modifier variant)

```css
/* Image region — greyscale overlay */
.image-region::after {
  content: ''
  position: absolute; inset: 0
  background: rgba(250,247,240,0.55)
}

/* Badge overridden to "Agotado" */
badge.background-color: #9E9E9E

/* Add-to-cart button */
.add-btn {
  background-color: #C8A98A
  cursor: not-allowed
  pointer-events: none
}

/* Price */
color: #9E9E9E
```

---

## 3. MOTION SYSTEM

### 3.1 Global Philosophy

**Principle:** Motion communicates meaning, not decoration. Every transition either:
- (a) Confirms a user action (button feedback, add-to-cart confirmation)
- (b) Reveals spatial hierarchy (page entries, drawer slide-ins)
- (c) Guides attention without demanding it (scroll reveals, parallax at ≤4px delta)

**Brand personality in motion:** Artisanal Mediterranean — unhurried but responsive. Motion should feel like a slow pour, not a snap. Curves bias toward ease-out (things decelerate into place, like setting a ceramic cup down). Spring curves (cubic-bezier with overshoot) used only for micro-interactions (button press release, icon bounce) — never for page-level transitions.

**Reduced motion:** All durations collapse to 0ms and transforms are eliminated when `prefers-reduced-motion: reduce` is active. Opacity fades (≤150ms) are preserved as the only motion channel.

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

---

### 3.2 Duration Scale

| Token name          | Value  | Use case                                            |
|---------------------|--------|-----------------------------------------------------|
| `duration-instant`  | 80ms   | Active/pressed state feedback (button scale)        |
| `duration-micro`    | 150ms  | Icon transitions, checkbox toggle, color swap       |
| `duration-fast`     | 200ms  | Hover color/shadow on interactive elements          |
| `duration-base`     | 260ms  | Card lift, tooltip appear, dropdown open            |
| `duration-moderate` | 350ms  | Drawer/modal enter, image zoom on hover             |
| `duration-slow`     | 450ms  | Page section fade-in (scroll reveal)                |
| `duration-deliberate`| 600ms | Hero entrance animation, full-page transitions      |
| `duration-cinematic`| 900ms | Parallax/ambient background motion                  |

Rule: durations above `duration-slow` (450ms) are used only for elements outside the primary interaction layer (backgrounds, decorative illustration, page-load orchestration).

---

### 3.3 Easing Tokens — Exact cubic-bezier values

#### Standard easings

```
ease-standard:   cubic-bezier(0.4, 0.0, 0.2, 1)
  — Material Design "standard curve"; symmetric acceleration/deceleration.
  — Use: most UI transitions where both entry and exit matter equally.
  — Applied to: color, shadow, border transitions on buttons and cards.

ease-decelerate: cubic-bezier(0.0, 0.0, 0.2, 1)
  — Fast entry, gentle settle. Elements coming INTO view.
  — Applied to: dropdown panels opening, toast appearing, modal entering from below.

ease-accelerate: cubic-bezier(0.4, 0.0, 1.0, 1)
  — Slow start, fast exit. Elements going OUT of view.
  — Applied to: modal closing, tooltip disappearing, drawer exiting.

ease-linear:     cubic-bezier(0.0, 0.0, 1.0, 1.0)
  — Constant rate. Use sparingly.
  — Applied to: spinner rotation, progress bars, looping ambient animations.
```

#### Expressive/spring easings

```
ease-spring-soft:    cubic-bezier(0.34, 1.56, 0.64, 1)
  — Slight overshoot (~5.6% above target), gentle bounce.
  — Applied to: button hover lift (translateY), card hover lift (translateY),
    add-to-cart icon scale, product image scale-in on mount.

ease-spring-bouncy:  cubic-bezier(0.68, -0.55, 0.265, 1.55)
  — Stronger overshoot (both ends). Use only for playful micro-moments.
  — Applied to: cart counter badge increment (+1 scale pop), success checkmark appear.
  — NOT for layout-shifting elements.

ease-snap:           cubic-bezier(0.23, 1, 0.32, 1)
  — Quick out, almost instantaneous deceleration. "Snappy organic."
  — Applied to: navigation link underline expand, tag pill border-color on hover.
```

#### Scroll / IntersectionObserver easings

```
ease-reveal-enter:   cubic-bezier(0.16, 1, 0.3, 1)
  — Aggressive ease-out. Content rushes up and snaps into reading position.
  — Applied to: all scroll-triggered opacity + translateY reveals.

ease-reveal-stagger: cubic-bezier(0.22, 1, 0.36, 1)
  — Slightly softer version for staggered children.
  — Applied to: grid cards appearing one after another with 80ms delay offset.
```

---

### 3.4 Motion Recipes — Component pairings

```
Button default → hover:
  duration: duration-fast (200ms)
  easing: ease-standard
  properties: background-color, border-color, box-shadow

Button default → hover (translateY):
  duration: 150ms
  easing: ease-spring-soft
  property: transform

Button press (active):
  duration: duration-instant (80ms)
  easing: ease-standard
  properties: background-color, transform (scale 0.984), box-shadow

Card default → hover (lift):
  duration: duration-base (260ms)
  easing: ease-spring-soft
  properties: transform (translateY -3px), box-shadow

Card image zoom:
  duration: duration-moderate (350ms)
  easing: cubic-bezier(0.25, 0.46, 0.45, 0.94)   /* gentle ease-out, no spring */
  property: transform (scale 1.04)
```

---

### 3.5 Hero Section — Scroll Animation Spec

#### Context

The hero consists of:
- **Background:** full-bleed photograph or texture (Mediterranean café scene)
- **Tagline:** Playfair Display 700, large — "El café que cuenta una historia"
- **Subline:** Lato 400, medium — origin descriptor
- **CTA button:** primary button variant
- **Decorative element:** illustrated sprig / ceramic illustration (optional)

#### Page Load — Orchestrated entrance (no scroll trigger, fires on DOMContentLoaded)

All elements start in their initial (hidden) state and animate in sequentially:

```
Step 0 — Background image:
  initial:  opacity: 0, transform: scale(1.04)
  animate:  opacity: 1, transform: scale(1)
  duration: duration-cinematic (900ms)
  easing:   ease-decelerate — cubic-bezier(0.0, 0.0, 0.2, 1)
  delay:    0ms
  — Image "breathes in" from slightly large to resting size.

Step 1 — Tagline (Playfair Display h1):
  initial:  opacity: 0, transform: translateY(28px)
  animate:  opacity: 1, transform: translateY(0)
  duration: duration-deliberate (600ms)
  easing:   ease-reveal-enter — cubic-bezier(0.16, 1, 0.3, 1)
  delay:    180ms   /* after image has started settling */

Step 2 — Subline:
  initial:  opacity: 0, transform: translateY(20px)
  animate:  opacity: 1, transform: translateY(0)
  duration: duration-slow (450ms)
  easing:   ease-reveal-enter — cubic-bezier(0.16, 1, 0.3, 1)
  delay:    320ms

Step 3 — CTA button:
  initial:  opacity: 0, transform: translateY(16px) scale(0.96)
  animate:  opacity: 1, transform: translateY(0) scale(1)
  duration: duration-slow (450ms)
  easing:   ease-spring-soft — cubic-bezier(0.34, 1.56, 0.64, 1)
  delay:    440ms

Step 4 — Decorative illustration (if present):
  initial:  opacity: 0, transform: translateX(12px) rotate(-2deg)
  animate:  opacity: 1, transform: translateX(0) rotate(0deg)
  duration: duration-deliberate (600ms)
  easing:   ease-decelerate — cubic-bezier(0.0, 0.0, 0.2, 1)
  delay:    500ms
```

CSS implementation pattern:

```css
.hero-bg {
  animation: hero-bg-enter 900ms cubic-bezier(0.0, 0.0, 0.2, 1) 0ms both;
}
.hero-tagline {
  animation: fade-up 600ms cubic-bezier(0.16, 1, 0.3, 1) 180ms both;
}
.hero-subline {
  animation: fade-up 450ms cubic-bezier(0.16, 1, 0.3, 1) 320ms both;
}
.hero-cta {
  animation: fade-up-spring 450ms cubic-bezier(0.34, 1.56, 0.64, 1) 440ms both;
}
.hero-decoration {
  animation: fade-slide-in 600ms cubic-bezier(0.0, 0.0, 0.2, 1) 500ms both;
}

@keyframes hero-bg-enter {
  from { opacity: 0; transform: scale(1.04); }
  to   { opacity: 1; transform: scale(1); }
}

@keyframes fade-up {
  from { opacity: 0; transform: translateY(28px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes fade-up-spring {
  from { opacity: 0; transform: translateY(16px) scale(0.96); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

@keyframes fade-slide-in {
  from { opacity: 0; transform: translateX(12px) rotate(-2deg); }
  to   { opacity: 1; transform: translateX(0) rotate(0deg); }
}
```

#### Parallax scroll — ambient depth (hero visible while scrolling)

```
Background image parallax:
  trigger:   scroll position within hero viewport
  technique: CSS transform: translateY() driven by scroll offset
  formula:   translateY(scrollY * 0.30)
             — background travels at 30% of scroll speed (0.7x parallax factor)
  max delta: ±40px (capped — prevents content bleed at edges)
  easing:    linear (continuous tracking, no easing on scroll)
  implementation: requestAnimationFrame or CSS scroll-driven animation
                  (@keyframes + animation-timeline: scroll())

Decorative illustration micro-parallax:
  formula:   translateY(scrollY * -0.08) + slight rotate(scrollY * 0.005deg)
  max delta: ±10px vertical, ±1.5deg rotation
  — Moves against scroll direction = floating sensation
```

CSS scroll-driven implementation (modern browsers):

```css
.hero-bg {
  animation: hero-parallax linear both;
  animation-timeline: scroll(root block);
  animation-range: 0% 100%;
}

@keyframes hero-parallax {
  from { transform: scale(1) translateY(0); }
  to   { transform: scale(1) translateY(40px); }   /* 30% of ~133px hero height delta */
}

.hero-decoration {
  animation: deco-parallax linear both;
  animation-timeline: scroll(root block);
  animation-range: 0% 100%;
}

@keyframes deco-parallax {
  from { transform: translateX(0) translateY(0) rotate(0deg); }
  to   { transform: translateX(0) translateY(-10px) rotate(-1.5deg); }
}
```

Fallback (JS, for broader support):

```javascript
// requestAnimationFrame scroll parallax
let ticking = false;
const heroBg = document.querySelector('.hero-bg');
const heroDeco = document.querySelector('.hero-decoration');

window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      const scrollY = window.scrollY;
      const bgDelta = Math.min(scrollY * 0.30, 40);          // cap at 40px
      const decoDelta = Math.max(scrollY * -0.08, -10);      // cap at -10px
      const decoRotate = Math.max(scrollY * -0.005, -1.5);   // cap at -1.5deg

      if (heroBg) heroBg.style.transform = `scale(1) translateY(${bgDelta}px)`;
      if (heroDeco) heroDeco.style.transform =
        `translateY(${decoDelta}px) rotate(${decoRotate}deg)`;

      ticking = false;
    });
    ticking = true;
  }
});
```

#### Scroll reveal — sections below the hero

IntersectionObserver configuration:

```javascript
const revealObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible');
        revealObserver.unobserve(entry.target); // fire once
      }
    });
  },
  {
    threshold: 0.12,       // 12% of element visible triggers reveal
    rootMargin: '0px 0px -48px 0px'  // 48px bottom offset — fires slightly before
  }
);

document.querySelectorAll('[data-reveal]').forEach(el => revealObserver.observe(el));
```

CSS for scroll-reveal elements:

```css
[data-reveal] {
  opacity: 0;
  transform: translateY(24px);
  transition:
    opacity  450ms cubic-bezier(0.16, 1, 0.3, 1),
    transform 450ms cubic-bezier(0.16, 1, 0.3, 1);
}

[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* Staggered children in a grid (e.g. product cards) */
[data-reveal-group] > * {
  opacity: 0;
  transform: translateY(20px);
  transition:
    opacity  400ms cubic-bezier(0.22, 1, 0.36, 1),
    transform 400ms cubic-bezier(0.22, 1, 0.36, 1);
}

[data-reveal-group].is-visible > *:nth-child(1) { transition-delay:   0ms; }
[data-reveal-group].is-visible > *:nth-child(2) { transition-delay:  80ms; }
[data-reveal-group].is-visible > *:nth-child(3) { transition-delay: 160ms; }
[data-reveal-group].is-visible > *:nth-child(4) { transition-delay: 240ms; }
[data-reveal-group].is-visible > *:nth-child(n+5) { transition-delay: 320ms; }

[data-reveal-group].is-visible > * {
  opacity: 1;
  transform: translateY(0);
}
```

---

## Appendix — Design Tokens Summary (CSS custom properties)

```css
:root {
  /* Brand colors */
  --color-brand-primary:     #A0522D;
  --color-brand-secondary:   #556B2F;
  --color-bg-base:           #FAF7F0;
  --color-text-primary:      #2C1A0E;
  --color-text-secondary:    #7A6651;
  --color-primary-dark-1:    #8B4513;   /* hover state */
  --color-primary-dark-2:    #7A3B10;   /* active state */
  --color-primary-muted:     #C8A98A;   /* disabled state */

  /* Typography */
  --font-heading: 'Playfair Display', Georgia, serif;
  --font-body:    'Lato', system-ui, sans-serif;

  /* Motion durations */
  --dur-instant:    80ms;
  --dur-micro:     150ms;
  --dur-fast:      200ms;
  --dur-base:      260ms;
  --dur-moderate:  350ms;
  --dur-slow:      450ms;
  --dur-deliberate:600ms;
  --dur-cinematic: 900ms;

  /* Motion easings */
  --ease-standard:        cubic-bezier(0.4,  0.0, 0.2, 1);
  --ease-decelerate:      cubic-bezier(0.0,  0.0, 0.2, 1);
  --ease-accelerate:      cubic-bezier(0.4,  0.0, 1.0, 1);
  --ease-linear:          cubic-bezier(0.0,  0.0, 1.0, 1);
  --ease-spring-soft:     cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-spring-bouncy:   cubic-bezier(0.68,-0.55, 0.265,1.55);
  --ease-snap:            cubic-bezier(0.23, 1,    0.32, 1);
  --ease-reveal-enter:    cubic-bezier(0.16, 1,    0.3,  1);
  --ease-reveal-stagger:  cubic-bezier(0.22, 1,    0.36, 1);
}
```

---

*Spec version: 1.0 — Café Vermut baseline*  
*Date: 2026-05-13*  
*Scope: Primary Button (6 states) + Product Card Café Molido (5 states) + Motion System*
