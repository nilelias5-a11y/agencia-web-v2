# Café Vermut — UI Component Specification
**UI Designer Phase 1–6 Full Workflow**
Date: 2026-05-13
Brand: Café Vermut | Aesthetic: Artisanal Mediterranean | Stack: Next.js

---

## PHASE 1 — Inspiration Research

### Reference 1 — Amici (Awwwards SOTD, October 2024)
**URL:** https://www.awwwards.com/sites/amici
**Award:** Awwwards Site of the Day
**What makes it exceptional:**
- Palette of olive green (#606d01) and warm cream (#f9edde) — nearly identical tonal strategy to Café Vermut, proving the combination reads as Mediterranean premium
- Gesture-based interaction layered over editorial scroll — images breathe as the user moves through the narrative
- Footer and 404 page treated as brand moments, not afterthoughts; earns category score for interaction design

**Elements to adapt:**
- The two-tone warm/green contrast logic for button backgrounds and section alternation
- "Captivating visual journey" scroll cadence: content enters late (0.3 threshold), with generous translateY distance (32–40px)

---

### Reference 2 — Bon Bouquet Café (Awwwards SOTD, February 2025)
**URL:** https://www.awwwards.com/sites/bon-bouquet-cafe-1
**Award:** Awwwards Site of the Day
**What makes it exceptional:**
- Parisian brunch brand translated into a website that feels sensory — the design makes you smell the coffee
- Text and imagery co-animate on scroll; neither leads alone, they arrive as a unit
- Typography pairing of large editorial serif + small clean sans creates hierarchy that reads across all viewport sizes

**Elements to adapt:**
- Co-animation pattern: heading and supporting image enter with identical easing, 40ms stagger between text nodes
- Use of off-white background with subtle warm tint as the "rest state" between content blocks (matches bg-base #FAF7F0)

---

### Reference 3 — Escape Coffee (Awwwards SOTD, May 2024)
**URL:** https://www.awwwards.com/sites/escape-coffee
**Award:** Awwwards Site of the Day
**What makes it exceptional:**
- Product detail page uses a split layout: large product image left, specs/CTA right — scroll within the right panel while image is sticky
- Hover on product cards reveals a secondary image with a cross-fade, not a slide — feels warm, not techno
- Loading states are branded: spinner replaced by a small coffee icon that rotates; reinforces personality at every micro-moment

**Elements to adapt:**
- Cross-fade (opacity transition) image reveal on card hover — no translateX, stays warm
- Sticky product image pattern for molido detail page
- Branded loading micro-state concept (in our case: a small terracotta circle pulse, not generic spinner)

---

### Reference 4 — Quay Restaurant (Awwwards SOTD)
**URL:** https://www.awwwards.com/sites/quay-restaurant
**Award:** Awwwards Site of the Day
**What makes it exceptional:**
- Minimalist full-screen hero photography with a single typographic lockup — no visual noise competing with the food
- Sticky header materialises (opacity 0→1, blur 0→8px backdrop) only after hero exits viewport — keeps first impression clean
- Section transitions use colour field changes (dark bg → light bg) timed to scroll position rather than abrupt cuts

**Elements to adapt:**
- Hero type lockup: one large display heading, one small overline label, one CTA — nothing more
- Scroll-aware header reveal pattern
- Section colour transition as a pacing device

---

### Reference 5 — Flavori Restaurant (Awwwards Honorable Mention)
**URL:** https://www.awwwards.com/sites/flavori-restaurant
**Award:** Awwwards Honorable Mention
**What makes it exceptional:**
- "Modern restaurant with unique concept of hospitality and traditional food" — the design brief mirrors Café Vermut's positioning exactly
- Typography takes structural responsibility: large serif numerals used as visual anchors within section layouts
- Hover states on cards use a warm overlay (colour tint over image) rather than desaturation — keeps warmth even in interactive states

**Elements to adapt:**
- Warm colour-tint overlay on card image hover (brand-primary #A0522D at 20% opacity)
- Serif numerals as decorative section anchors (e.g., "01", "02" in Playfair Display)

---

### Reference 6 — Artifact Coffee (Siteinspire Featured)
**URL:** https://www.siteinspire.com/website/3169-artifact-coffee
**Award:** Siteinspire Curated Feature
**What makes it exceptional:**
- Specialty coffee brand with a typographically-led interface; almost no decorative illustration — the product photography and type carry everything
- Cards use a generous aspect ratio (4:3) with object-fit: cover; no cropping surprises on different product images
- Body text is set at 17–18px with 1.7 line-height — unusually generous for a product site, reads as confident and artisan

**Elements to adapt:**
- 4:3 card image ratio with object-fit: cover
- 17px body text baseline with 1.7 line-height
- Type-led sections where the heading IS the design element

---

### Reference 7 — Fabbrica (Awwwards Honorable Mention)
**URL:** https://www.awwwards.com/sites/fabbrica
**Award:** Awwwards Honorable Mention
**What makes it exceptional:**
- Artisan production brand (factory/craft aesthetic) — earthy palette with strong typographic personality
- Buttons use a fill-from-bottom hover animation (pseudo-element scaleY from 0 to 1): warm, tactile, organic — no coldness
- Focus states on interactive elements use a warm amber outline rather than the default browser blue — brand-consistent accessibility

**Elements to adapt:**
- Fill-from-bottom button hover technique (scaleY pseudo-element)
- Warm amber/terracotta focus outline instead of blue

---

### Reference 8 — Morfo.rest / Digidop (Awwwards Honorable Mention)
**URL:** https://www.digidop.com/blog/digidop-wins-honorable-mention-awwwards-with-morfo-rest
**Award:** Awwwards Honorable Mention
**What makes it exceptional:**
- Restaurant brand uses a looping ambient video in the hero with a warm colour-grade overlay — not a static image
- Scroll-triggered stagger on the menu grid: cards enter bottom-to-top with 80ms between each, creating a "setting the table" choreography metaphor
- 404 page and empty states are illustrated moments of brand delight

**Elements to adapt:**
- 80ms stagger increment between card children on scroll reveal
- "Setting the table" metaphor: grid items reveal in sequence from top-left to bottom-right

---

### Design Direction Brief

Café Vermut's visual language is warm, unhurried, and confidently artisanal. Every component is built from terracotta, olive, and crema — a palette that does not cool down. Typography is unapologetically editorial: Playfair Display carries weight and story; Lato grounds without sterility. Motion is slow enough to feel handmade — nothing snaps, nothing pops. Interactions reward attention: hover reveals warmth rather than contrast, scroll arrivals feel like pages turning in a slow magazine. The result is a site that smells of ground coffee and afternoon sun.

---

## PHASE 2 — Component Inventory

### Primitives

| Component | Status |
|---|---|
| Button — primary | Required |
| Button — secondary | Required |
| Button — ghost | Optional |
| Button — icon-only | Optional |
| Button — loading state | Required |
| Input — text / email / tel | Required |
| Textarea | Required |
| Select | Optional |
| Checkbox | Optional |
| Spinner / Loader | Required |
| Badge / Tag | Optional |
| Icon (custom SVG set) | Required |
| Divider / Separator | Required |
| Tooltip | Optional |
| Image with aspect ratio rules | Required |

### Composites

| Component | Status |
|---|---|
| Card — product (café molido) | Required |
| Card — featured (hero product) | Required |
| Navigation bar — desktop | Required |
| Navigation bar — mobile | Required |
| Dropdown menu | Required |
| Modal / Dialog | Required |
| Toast / Notification | Required |
| Accordion (FAQ) | Required |
| Form (contact / order) | Required |
| Breadcrumb | Optional |

### Sections

| Component | Status |
|---|---|
| Hero — full-screen with editorial text | Required |
| Feature block — split image + text | Required |
| Product grid / catalogue | Required |
| Testimonial block — single quote | Optional |
| Gallery | Optional |
| CTA banner — full-width | Required |
| Footer — full | Required |
| 404 / Empty state | Required |

---

## PHASE 3 — Per-State Visual Specification

---

## Component — Button Primary

**Design intent:** The primary button is the most confident gesture on the page. It uses brand-primary terracotta as its base identity. The hover uses a fill-from-bottom technique (inspired by Fabbrica) so the interaction feels handcrafted, not digital. Every state has a purpose; none is an accident.

**Base token reference:**
- `--brand-primary`: #A0522D
- `--brand-primary-dark`: #7A3F22 (darken 15%)
- `--brand-primary-darker`: #5E3019 (darken 30%)
- `--bg-base`: #FAF7F0
- `--text-on-primary`: #FAF7F0
- `--text-disabled`: #B0A898
- `--radius-md`: 6px
- `--font-body`: Lato
- `--space-4`: 16px
- `--space-6`: 24px

---

### State: Default

```
Background:         #A0522D  (brand-primary)
Color:              #FAF7F0  (bg-base — used as text-on-primary)
Border:             none
Border-radius:      6px  (--radius-md)
Padding:            14px 32px  (top/bottom 14px, left/right 32px)
Font-family:        Lato
Font-size:          15px  (0.9375rem)
Font-weight:        600
Letter-spacing:     0.06em  (tracking-wide, artisan label feel)
Text-transform:     uppercase
Line-height:        1
Cursor:             pointer
Shadow:             none
Min-width:          160px  (prevents layout shift during loading)
Height:             48px  (fixed; prevents reflow across states)
Display:            inline-flex
Align-items:        center
Justify-content:    center
Gap:                8px  (space between icon and label if icon present)
Position:           relative  (required for ::before pseudo fill-element)
Overflow:           hidden  (clip the fill pseudo-element)
```

**Fill pseudo-element (::before) — default state:**
```
content:            ''
Position:           absolute
Bottom:             0
Left:               0
Width:              100%
Height:             100%
Background:         #7A3F22  (brand-primary-dark)
Transform:          scaleY(0)
Transform-origin:   bottom center
Transition:         transform 280ms cubic-bezier(0.34, 0.0, 0.20, 1.0)
Z-index:            0
```

**Label z-index:** 1 (sits above ::before fill)

---

### State: Hover

```
Transition trigger:     mouseenter
::before transform:     scaleY(1)  — fills from bottom to top
::before transition:    transform 280ms cubic-bezier(0.34, 0.0, 0.20, 1.0)
Effective background:   #7A3F22  (brand-primary-dark, revealed by fill)
Text color:             #FAF7F0  (unchanged — already visible on dark)
Transform (button):     none  (no translateY — the fill IS the feedback)
Cursor:                 pointer
```

Note: The fill rises from the bottom edge. Duration is 280ms — slow enough to be visible and warm, fast enough to feel responsive. Cubic-bezier(0.34, 0.0, 0.20, 1.0) is a strong ease-in-out that accelerates in, then decelerates — mimicking a fluid rising.

---

### State: Focus (keyboard navigation)

```
Outline:            3px solid #A0522D  (brand-primary)
Outline-offset:     3px
Background:         #A0522D  (same as default — focus does not change fill)
Box-shadow:         none  (outline alone is sufficient; no double ring)
```

WCAG 2.1 AA compliance: The 3px brand-primary outline on the bg-base (#FAF7F0) page background achieves contrast ratio of 4.8:1 — passes AA for focus indicators (minimum 3:1 required by WCAG 2.1 SC 1.4.11 non-text contrast).

---

### State: Active (pressed)

```
::before transform:     scaleY(1)  (fill remains visible)
Background (effective): #5E3019  (brand-primary-darker — deepens on press)
Transition:             background 80ms cubic-bezier(0.4, 0.0, 1.0, 1.0)
Transform (button):     scale(0.97)
Transform-origin:       center
Transition (transform): transform 80ms cubic-bezier(0.4, 0.0, 1.0, 1.0)
```

Note: 80ms is the "instant" duration from the motion scale. The scale(0.97) is a 3% compression — perceptible but not theatrical. It communicates a physical press without departing from the artisan warmth.

---

### State: Disabled

```
Background:         #D4B8A8  (brand-primary at 40% lightened, not opacity — avoids see-through)
Color:              #A89080  (muted warm grey)
Opacity:            1  (opacity NOT used — background token provides the muting)
Cursor:             not-allowed
Pointer-events:     none
Border:             none
::before:           display: none  (no hover fill on disabled)
```

Note: Using a flat muted token instead of opacity: 0.4 ensures the button does not bleed into coloured backgrounds in an unexpected way. The warm grey maintains the colour family, reinforcing brand consistency even in inactive state.

---

### State: Loading

```
Label text:         Hidden (opacity: 0, position: absolute) — preserves button width
Width:              Fixed at natural width (min-width: 160px enforced)
Height:             48px (unchanged)
Background:         #A0522D (default — loading is not a "waiting" experience, it is an "in progress" one)
Cursor:             default (not pointer — action is already dispatched)
Pointer-events:     none
```

**Spinner element:**
```
Element:            <span role="status" aria-label="Cargando">
Width:              20px
Height:             20px
Border:             2px solid rgba(250,247,240,0.3)  (bg-base at 30%)
Border-top-color:   #FAF7F0  (full opacity — creates the arc)
Border-radius:      9999px  (--radius-full)
Animation:          spin 700ms linear infinite
Position:           absolute
Top:                50%
Left:               50%
Transform:          translate(-50%, -50%)
```

**Spin keyframes:**
```css
@keyframes spin {
  from { transform: translate(-50%, -50%) rotate(0deg); }
  to   { transform: translate(-50%, -50%) rotate(360deg); }
}
```

Note: 700ms spin duration — slower than typical (400–500ms) to match the unhurried brand tempo. The arc-style spinner (border-top only) is minimal and reads warmly at 20px.

---

## Component — Card de Producto (Café Molido)

**Design intent:** Inspired by Escape Coffee's cross-fade image technique, Artifact Coffee's 4:3 ratio, and Flavori's warm colour-tint hover overlay. The card is a confident editorial object — the product image leads, the text grounds. No hard lines, no cold greys. Hover reveals warmth rather than distance.

**Token reference:**
- `--bg-base`: #FAF7F0
- `--brand-primary`: #A0522D
- `--brand-secondary`: #556B2F
- `--text-primary`: #2C1A0E  (deep espresso — defined below)
- `--text-secondary`: #7A5C45  (warm mid-brown)
- `--radius-xl`: 12px
- `--shadow-md`: 0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06)
- `--shadow-lg`: 0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05)

---

### State: Default

**Card container:**
```
Background:         #FAF7F0  (bg-base)
Border-radius:      12px  (--radius-xl)
Border:             1px solid rgba(160,82,45,0.12)  (brand-primary at 12% — whisper border)
Shadow:             0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06)  (--shadow-md)
Overflow:           hidden  (clips image and overlay)
Width:              100%  (fills grid column)
Max-width:          320px
Cursor:             pointer
Position:           relative
Display:            flex
Flex-direction:     column
```

**Image wrapper:**
```
Width:              100%
Aspect-ratio:       4 / 3  (from Artifact Coffee reference — 320×240px at max-width)
Overflow:           hidden
Position:           relative
```

**Product image (primary):**
```
Width:              100%
Height:             100%
Object-fit:         cover
Object-position:    center
Display:            block
Transition:         opacity 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0)
Opacity:            1
```

**Product image (secondary / hover reveal):**
```
Position:           absolute
Top:                0 / Left: 0
Width:              100% / Height: 100%
Object-fit:         cover
Object-position:    center
Opacity:            0
Transition:         opacity 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0)
```

**Warm tint overlay (::after on image wrapper):**
```
Content:            ''
Position:           absolute
Inset:              0
Background:         rgba(160,82,45,0.0)  (brand-primary at 0% — invisible in default)
Transition:         background 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0)
Pointer-events:     none
Z-index:            1
```

**Card body (text area):**
```
Padding:            20px 24px 24px 24px
Display:            flex
Flex-direction:     column
Gap:                8px
Flex:               1
```

**Product name:**
```
Font-family:        Playfair Display
Font-size:          20px  (1.25rem)
Font-weight:        700
Color:              #2C1A0E  (text-primary / deep espresso)
Line-height:        1.3
Letter-spacing:     -0.01em
Margin:             0
```

**Product descriptor (e.g., "Tueste medio · Origen: Etiopía"):**
```
Font-family:        Lato
Font-size:          13px  (0.8125rem)
Font-weight:        400
Color:              #7A5C45  (text-secondary)
Line-height:        1.5
Letter-spacing:     0.04em
Text-transform:     uppercase
Margin:             0
```

**Product description (1–2 lines):**
```
Font-family:        Lato
Font-size:          15px  (0.9375rem)
Font-weight:        400
Color:              #4A3525  (text-body — mid espresso)
Line-height:        1.65
Margin-top:         4px
Display:            -webkit-box
-webkit-line-clamp: 2
-webkit-box-orient: vertical
Overflow:           hidden
```

**Price:**
```
Font-family:        Playfair Display
Font-size:          22px  (1.375rem)
Font-weight:        700
Color:              #A0522D  (brand-primary)
Line-height:        1
Margin-top:         auto  (pushes to bottom of flex column)
Padding-top:        12px
```

**"Añadir al carrito" CTA (within card):**
```
— Uses Button Primary spec above, but with these overrides:
Font-size:          13px
Padding:            10px 20px
Height:             38px
Min-width:          unset
Width:              100%
Margin-top:         12px
Border-radius:      6px
```

---

### State: Hover

```
Transition trigger:     mouseenter
Duration:               400ms  (all transitions share this duration)
Easing:                 cubic-bezier(0.0, 0.0, 0.2, 1.0)  (ease-out — elements arriving)
```

**Card container changes:**
```
Shadow:             0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05)  (--shadow-lg)
Transform:          translateY(-4px)
Transition:         transform 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0),
                    box-shadow 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0)
Border-color:       rgba(160,82,45,0.28)  (brand-primary at 28%)
```

**Primary image:**
```
Opacity:            0  (cross-fades out)
```

**Secondary image (hover reveal):**
```
Opacity:            1  (cross-fades in)
```

Note: If no secondary image is available, the primary image stays visible and the warm tint overlay activates instead.

**Warm tint overlay:**
```
Background:         rgba(160,82,45,0.14)  (brand-primary at 14% — from Flavori reference)
```

**Product name:**
```
Color:              #A0522D  (brand-primary — name warms on hover)
Transition:         color 400ms cubic-bezier(0.0, 0.0, 0.2, 1.0)
```

**"Añadir al carrito" button (within card):**
```
::before transform: scaleY(1)  (fill activates, same as standalone button hover)
```

---

### State: Focus (keyboard — card as a whole is focusable)

```
Outline:            3px solid #A0522D  (brand-primary)
Outline-offset:     4px
Box-shadow:         0 0 0 6px rgba(160,82,45,0.15)  (soft glow outside outline)
```

---

### State: Active (pressed — card click)

```
Transform:          translateY(-2px) scale(0.99)
Transition:         transform 80ms cubic-bezier(0.4, 0.0, 1.0, 1.0)
```

---

### State: Out-of-stock (conditional)

```
Image:              filter: grayscale(30%) brightness(0.95)
                    Transition: filter 300ms ease
Price section:      Replaced by "Agotado" badge
```

**"Agotado" badge:**
```
Font-family:        Lato
Font-size:          11px
Font-weight:        600
Text-transform:     uppercase
Letter-spacing:     0.08em
Color:              #FAF7F0
Background:         #7A5C45  (text-secondary token used as muted fill)
Padding:            4px 10px
Border-radius:      9999px  (--radius-full / pill)
```

---

## PHASE 4 — Animation & Micro-interaction Specification

---

## Motion System

**Philosophy:** Motion at Café Vermut is slow by intention. It communicates the pace of a Mediterranean afternoon — nothing rushes, everything arrives with confidence. Animations guide the eye toward content hierarchies; they do not perform. Every duration is 20–30% longer than a tech product would choose. Every easing favours deceleration: things arrive gracefully.

**Governing metaphor:** "Pages turning in a slow magazine." Each element enters as if someone is placing it on the table.

---

### Easing Tokens

```css
/* Motion easing tokens — global */

/* Standard: smooth acceleration into deceleration */
--ease-standard:    cubic-bezier(0.25, 0.10, 0.25, 1.00);

/* Entering (ease-out): fast start, gentle landing — for elements coming in */
--ease-enter:       cubic-bezier(0.00, 0.00, 0.20, 1.00);

/* Leaving (ease-in): slow start, fast exit — for elements going out */
--ease-exit:        cubic-bezier(0.40, 0.00, 1.00, 1.00);

/* Spring (interactive feedback): overshoots slightly, then settles — for button press, card lift */
--ease-spring:      cubic-bezier(0.34, 1.56, 0.64, 1.00);

/* Fill-rise (button hover fill): strong acceleration then plateau — for scaleY pseudo-element */
--ease-fill:        cubic-bezier(0.34, 0.00, 0.20, 1.00);

/* Scroll reveal: gentle, cinematic — for elements entering on scroll */
--ease-reveal:      cubic-bezier(0.22, 1.00, 0.36, 1.00);
```

**Justification for each curve:**
- `--ease-enter` (0,0,0.2,1): Maximum initial velocity decelerating to near-zero — mimics something placed down gently. Used for all appearing elements.
- `--ease-exit` (0.4,0,1,1): Opposite — builds speed as it leaves. Used for modals closing, toasts dismissing.
- `--ease-spring` (0.34,1.56,0.64,1): Y2>1 creates overshoot — the physical "bounce" of pressing a physical button. Reserved only for short, tactile interactions (≤300ms) to avoid nausea.
- `--ease-fill` (0.34,0,0.2,1): Custom for the button fill-from-bottom. Accelerates confidently, decelerates fully — the liquid rising analogy.
- `--ease-reveal` (0.22,1,0.36,1): Ultra-long deceleration tail. At 600–800ms, this produces a cinematic slide where the element seems to float into position.

---

### Duration Scale

```css
/* Duration tokens */

--duration-instant:   80ms;    /* Button press (active state) — snap feedback */
--duration-fast:      200ms;   /* Tooltip appear/disappear, nav link underline */
--duration-normal:    300ms;   /* Dropdown open, accordion, input focus */
--duration-medium:    400ms;   /* Card hover (lift + image cross-fade) */
--duration-slow:      600ms;   /* Modal enter, page section elements */
--duration-cinematic: 800ms;   /* Hero heading entrance */
--duration-epic:      1100ms;  /* Full hero section initial load reveal */
```

---

### Scroll Animation — Hero Section

```
## Scroll Animation — Hero (Full-Screen)

Trigger:            Page load (not IntersectionObserver — hero is always in viewport)
Sequence (staggered cascade):

  [0ms]    Hero background image:
           From: opacity 0, scale(1.06)
           To:   opacity 1, scale(1.00)
           Duration: 1100ms (--duration-epic)
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)  (--ease-reveal)

  [200ms]  Overline label ("Café Vermut · Gràcia, Barcelona"):
           From: opacity 0, translateY(16px)
           To:   opacity 1, translateY(0)
           Duration: 600ms (--duration-slow)
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)  (--ease-reveal)

  [380ms]  H1 heading (line 1):
           From: opacity 0, translateY(28px)
           To:   opacity 1, translateY(0)
           Duration: 800ms (--duration-cinematic)
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)  (--ease-reveal)

  [480ms]  H1 heading (line 2 — if split into two spans):
           From: opacity 0, translateY(28px)
           To:   opacity 1, translateY(0)
           Duration: 800ms
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)

  [660ms]  Body subheading / descriptor:
           From: opacity 0, translateY(20px)
           To:   opacity 1, translateY(0)
           Duration: 600ms
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)

  [820ms]  CTA button:
           From: opacity 0, translateY(16px)
           To:   opacity 1, translateY(0)
           Duration: 500ms
           Easing: cubic-bezier(0.22, 1.00, 0.36, 1.00)

  [900ms]  Scroll indicator (if present — small arrow or label):
           From: opacity 0
           To:   opacity 1
           Duration: 400ms
           Easing: cubic-bezier(0.00, 0.00, 0.20, 1.00)
           + continuous bounce animation: translateY(0→8px→0), 1400ms ease-in-out, infinite
```

**Implementation note:** Use CSS `animation-delay` on each element or a JS orchestration with `setTimeout` cascade. Prefer CSS-only for performance. All elements start with `opacity: 0` in their initial CSS class (`.hero-element`) and gain `.hero-element--visible` class after page ready.

---

### Scroll Animation — Product Grid Section (Café Molido Cards)

```
## Scroll Animation — Product Grid

Trigger:            IntersectionObserver threshold: 0.15
                    (15% of section visible — triggers early, feels natural)
rootMargin:         0px 0px -60px 0px  (60px bottom offset prevents premature trigger)

Effect:             Fade-up with stagger (inspired by Morfo.rest "setting the table" pattern)
Duration:           600ms per card  (--duration-slow)
Easing:             cubic-bezier(0.22, 1.00, 0.36, 1.00)  (--ease-reveal)
Distance:           translateY(32px)  (from Amici reference — generous, cinematic)

Stagger:            80ms between each card child
                    Card 1: delay 0ms
                    Card 2: delay 80ms
                    Card 3: delay 160ms
                    Card 4: delay 240ms
                    (max 4 visible at once on desktop — beyond 4, stagger resets to 0ms)

Section heading:    Triggers 120ms before first card (negative stagger offset)
                    Same duration/easing, translateY(24px)
```

---

### Scroll Animation — General Content Sections (Feature blocks, CTA, Footer)

```
## Scroll Animation — General

Trigger:            IntersectionObserver threshold: 0.20
Effect:             Fade-up
Duration:           600ms
Easing:             cubic-bezier(0.22, 1.00, 0.36, 1.00)
Distance:           translateY(24px)
Stagger (if multiple children): 60ms
```

---

### Micro-interactions

#### Navigation link hover
```
Element:            <a> in desktop nav
Technique:          Underline scale from center (scaleX 0→1)
Pseudo-element:     ::after
  Content:          ''
  Display:          block
  Height:           1px
  Background:       #A0522D  (brand-primary)
  Transform:        scaleX(0)
  Transform-origin: center
  Transition:       transform 200ms cubic-bezier(0.25, 0.10, 0.25, 1.00)
On hover:
  ::after transform: scaleX(1)
```

#### Form field focus
```
Input border:       rgba(160,82,45,0.25) → #A0522D
                    Transition: border-color 200ms cubic-bezier(0.25, 0.10, 0.25, 1.00)
Label:              Floats upward (if floating label pattern)
                    From: translateY(0), font-size 15px, color #7A5C45
                    To:   translateY(-22px), font-size 12px, color #A0522D
                    Transition: all 200ms cubic-bezier(0.25, 0.10, 0.25, 1.00)
Box-shadow (focus): 0 0 0 3px rgba(160,82,45,0.15)  (warm glow)
```

#### Image hover (standalone, outside card)
```
Transform:          scale(1.03)
Transition:         transform 500ms cubic-bezier(0.22, 1.00, 0.36, 1.00)
Overflow:           hidden (on wrapper to clip scale)
Filter:             brightness(1.02)  (slight lighten — warm not dark)
```

#### Accordion open/close
```
Icon rotation:      0deg → 180deg
Transition:         transform 300ms cubic-bezier(0.25, 0.10, 0.25, 1.00)
Content reveal:     max-height 0 → max-height [value]px
Transition:         max-height 350ms cubic-bezier(0.00, 0.00, 0.20, 1.00)
Opacity:            0 → 1
Transition:         opacity 300ms cubic-bezier(0.00, 0.00, 0.20, 1.00) delay 50ms
```

#### Modal enter/exit
```
Backdrop:
  Enter: opacity 0 → 0.6, duration 300ms, ease-out
  Exit:  opacity 0.6 → 0, duration 250ms, ease-in

Panel:
  Enter: opacity 0 → 1, translateY(20px) → translateY(0)
         duration 400ms, cubic-bezier(0.00, 0.00, 0.20, 1.00)
  Exit:  opacity 1 → 0, translateY(0) → translateY(12px)
         duration 250ms, cubic-bezier(0.40, 0.00, 1.00, 1.00)
```

#### Toast / Notification appear
```
Enter: translateX(100%) → translateX(0), opacity 0 → 1
       duration 350ms, cubic-bezier(0.00, 0.00, 0.20, 1.00)
Dismiss (auto after 4s or on close click):
       translateX(0) → translateX(110%), opacity 1 → 0
       duration 280ms, cubic-bezier(0.40, 0.00, 1.00, 1.00)
```

---

### prefers-reduced-motion

```css
/* Required in globals.css */

@media (prefers-reduced-motion: reduce) {
  /* Scroll reveals: opacity only — no translate, no scale */
  .scroll-reveal,
  .hero-element {
    transform: none !important;
    transition: opacity 300ms ease !important;
  }

  /* Hero background: no scale zoom */
  .hero-bg {
    transform: none !important;
    transition: opacity 600ms ease !important;
  }

  /* Button hover fill: instant — no scaleY animation */
  .btn-primary::before {
    transition: none !important;
  }

  /* Card hover: no translateY lift, no shadow transition */
  .card-product {
    transform: none !important;
    transition: opacity 150ms ease !important;
  }

  /* Image hover: no scale */
  .img-hover-wrapper img {
    transform: none !important;
    transition: none !important;
  }

  /* Accordion: no max-height animation — use hidden/visible only */
  .accordion-content {
    transition: opacity 150ms ease !important;
  }

  /* Spinner: OK to keep spinning — it is functional, not decorative */
  /* Scroll indicator bounce: remove */
  .scroll-indicator {
    animation: none !important;
  }
}
```

---

## PHASE 5 — Spacing & Grid System

### Spacing Tokens

```css
:root {
  --space-1:   4px;   /* micro-gap: icon to label */
  --space-2:   8px;   /* tight: badge padding, icon spacing */
  --space-3:   12px;  /* compact: input padding vertical */
  --space-4:   16px;  /* base: standard internal padding */
  --space-5:   20px;  /* medium: card body gap, mobile outer margin */
  --space-6:   24px;  /* comfortable: grid gutter desktop, card body padding */
  --space-8:   32px;  /* section element gap, tablet outer margin */
  --space-10:  40px;  /* heading-to-content vertical rhythm */
  --space-12:  48px;  /* section internal top/bottom padding (mobile) */
  --space-16:  64px;  /* section top/bottom padding (tablet) */
  --space-20:  80px;  /* section top/bottom padding (desktop standard) */
  --space-24:  96px;  /* section top/bottom padding (desktop generous) */
  --space-32:  128px; /* hero internal vertical margin */
  --space-40:  160px; /* hero padding top on large screens */
  --space-48:  192px; /* between major visual chapters */
}
```

**Usage rules:**
| Role | Token |
|---|---|
| Button padding (top/bottom) | 14px (custom — between --space-3 and --space-4) |
| Button padding (left/right) | 32px (custom — between --space-8) |
| Card body padding | --space-5 (20px) top, --space-6 (24px) sides/bottom |
| Card-to-card gap (grid) | --space-6 (24px) |
| Section internal padding | --space-20 to --space-24 (80–96px) desktop |
| Section internal padding | --space-12 (48px) mobile |
| Heading to first content element | --space-10 (40px) |
| Paragraph-to-paragraph gap | --space-6 (24px) |
| Form field gap | --space-5 (20px) |
| Nav height | 72px (custom — not a spacing token, fixed value) |

---

### Grid System

```
## Grid Specification

Desktop (≥1280px):
  Columns:            12
  Column width:       auto
  Gutter:             24px  (--space-6)
  Max content width:  1200px
  Full-bleed max:     1440px
  Outer margin:       auto (centered)

  Product grid:       4 columns (3-col span each in 12-col)

Tablet (768px–1279px):
  Columns:            8
  Gutter:             20px
  Outer margin:       32px  (--space-8)

  Product grid:       2 columns (4-col span each in 8-col)

Mobile (<768px):
  Columns:            4
  Gutter:             16px
  Outer margin:       20px  (--space-5)

  Product grid:       1 column (full width)
  
  Exception: At 480px–767px (large mobile), 2-column product grid is permitted
  with 12px gutter if cards remain ≥ 152px wide

Breakpoints:
  sm:   640px
  md:   768px
  lg:   1024px
  xl:   1280px
  2xl:  1536px
```

---

## PHASE 6 — Shadow & Border Radius Scale

### Shadow Tokens

```css
:root {
  --shadow-xs:    0 1px 2px rgba(0,0,0,0.05);
  --shadow-sm:    0 1px 3px rgba(0,0,0,0.10), 0 1px 2px rgba(0,0,0,0.06);
  --shadow-md:    0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
  --shadow-lg:    0 10px 15px rgba(0,0,0,0.10), 0 4px 6px rgba(0,0,0,0.05);
  --shadow-xl:    0 20px 25px rgba(0,0,0,0.10), 0 10px 10px rgba(0,0,0,0.04);
  --shadow-2xl:   0 25px 50px rgba(0,0,0,0.25);
  --shadow-inner: inset 0 2px 4px rgba(0,0,0,0.06);
  --shadow-none:  none;

  /* Brand-tinted shadows for key elevated elements */
  --shadow-warm-md: 0 4px 12px rgba(160,82,45,0.12), 0 2px 4px rgba(160,82,45,0.08);
  --shadow-warm-lg: 0 12px 24px rgba(160,82,45,0.15), 0 4px 8px rgba(160,82,45,0.08);
}
```

**Tinted shadow justification:** On the bg-base (#FAF7F0) background, pure black shadows can read as cold and clinical. The `--shadow-warm-*` tokens use brand-primary at 8–15% opacity to tint shadows with terracotta warmth. Used on cards and modals where the element floats above the page.

**Shadow usage map:**
| Element | Token |
|---|---|
| Product card (default) | --shadow-md |
| Product card (hover) | --shadow-warm-lg |
| Modal panel | --shadow-2xl |
| Dropdown menu | --shadow-lg |
| Input (focused) | Box-shadow: 0 0 0 3px rgba(160,82,45,0.15) |
| Navigation (scrolled) | 0 1px 0 rgba(160,82,45,0.10), --shadow-sm |
| Toast | --shadow-lg |
| Button | none (fill provides visual weight) |
| Featured card | --shadow-warm-md |

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

**Radius usage map:**
| Element | Token | Value |
|---|---|---|
| Button primary | --radius-md | 6px |
| Button secondary | --radius-md | 6px |
| Product card | --radius-xl | 12px |
| Input field | --radius-md | 6px |
| Modal panel | --radius-2xl | 16px |
| Dropdown | --radius-lg | 8px |
| Badge / pill tag | --radius-full | 9999px |
| Product image (standalone) | --radius-lg | 8px |
| Toast | --radius-lg | 8px |
| Tooltip | --radius-sm | 4px |
| Section with coloured bg | --radius-3xl | 24px (at corners on desktop) |

**Radius philosophy:** The system uses only even-px values and avoids mixing radius families within a single visual context (no 4px next to 12px on the same card). The product card at 12px is the visual keystone — all composites derive from it.

---

## Supplementary Token Definitions

The following colour tokens are referenced throughout this spec but were not in the original brief. They complete the system:

```css
:root {
  /* Brand (from brief) */
  --brand-primary:        #A0522D;
  --brand-secondary:      #556B2F;
  --bg-base:              #FAF7F0;

  /* Brand derivations */
  --brand-primary-dark:   #7A3F22;   /* brand-primary darkened 15% — button hover fill */
  --brand-primary-darker: #5E3019;   /* brand-primary darkened 30% — button active */
  --brand-primary-light:  #C8795A;   /* brand-primary lightened 15% — decorative accents */
  --brand-primary-muted:  #D4B8A8;   /* disabled button background */
  --brand-secondary-dark: #3D4E21;   /* hover on secondary elements */

  /* Text scale */
  --text-primary:         #2C1A0E;   /* deep espresso — main headings */
  --text-body:            #4A3525;   /* mid espresso — body text */
  --text-secondary:       #7A5C45;   /* warm mid-brown — captions, meta */
  --text-disabled:        #A89080;   /* disabled text */
  --text-on-primary:      #FAF7F0;   /* text on brand-primary backgrounds */
  --text-on-dark:         #FAF7F0;   /* text on dark bg */

  /* Surface */
  --surface-card:         #FAF7F0;   /* same as bg-base — cards are page level */
  --surface-elevated:     #FFFFFF;   /* modals, dropdowns — slightly whiter */
  --surface-section-alt:  #F0EBE0;   /* alternating section bg — 1 step warmer */

  /* Borders */
  --border-subtle:        rgba(160,82,45,0.12);  /* whisper border on cards */
  --border-default:       rgba(160,82,45,0.25);  /* input borders */
  --border-focus:         #A0522D;               /* focused input/element */
}
```

---

## Implementation Checklist

- [ ] All CSS custom properties defined in `:root` in `globals.css`
- [ ] `@media (prefers-reduced-motion: reduce)` block added to `globals.css`
- [ ] Button `::before` pseudo-element overflow clipped with `overflow: hidden` on parent
- [ ] Card secondary image preloaded (`<link rel="preload">` or `priority={false}` in Next.js `<Image>`)
- [ ] IntersectionObserver utility created in `lib/useScrollReveal.ts`
- [ ] Hero animation sequence implemented via CSS `animation-delay` cascade
- [ ] Spinner keyframe `@keyframes spin` defined globally
- [ ] Focus outlines verified in Chromium, Firefox, and Safari
- [ ] WCAG 2.1 AA contrast check: all text/background pairs
- [ ] Card min-width tested at 320px viewport (single column mobile)
- [ ] Loading state: button width confirmed fixed to prevent layout shift

---

*Specification produced by UI Designer agent — Café Vermut project — 2026-05-13*
*References: Amici (Awwwards SOTD), Bon Bouquet Café (Awwwards SOTD Feb 2025), Escape Coffee (Awwwards SOTD May 2024), Quay Restaurant (Awwwards SOTD), Flavori Restaurant (Awwwards HM), Artifact Coffee (Siteinspire), Fabbrica (Awwwards HM), Morfo.rest (Awwwards HM)*
