# Café Vermut — Hero Variants

**Brand**: Café Vermut — specialty coffee, Barcelona
**Visual direction**: Japan meets Barcelona — minimal, premium, near-empty
**Colors**: #F8F7F4 (near-white) · #0A0A0A (pure black) · #C9A84C (gold)
**Primary CTA**: Explorar cafés

---

## Variant A — "The Still Object"

### Layout

Full-viewport, asymmetric two-column split on desktop (60 / 40).

Left column (60%): pure #F8F7F4 background. A single ceramic cup or cloth bag of single-origin coffee sits centered in the left zone, photographed overhead or at a precise 15° tilt. Extreme white space above and below. No border, no shadow — the object floats.

Right column (40%): #0A0A0A background. All text lives here, vertically centered, left-aligned, with generous line height. The column functions as a controlled dark panel — the contrast with the left creates immediate visual tension without any decorative element.

**Text hierarchy (right column):**
- Overline (small caps, #C9A84C, 11px tracked): BARCELONA · ESPECIALIDAD
- Headline (#F8F7F4, 52px, light weight): "Un solo origen.  
  Toda la historia."
- Body (#F8F7F4 at 60% opacity, 15px): "Café de especialidad de Etiopía, Colombia y Guatemala. Seleccionado, tostado y enviado a tu puerta."
- CTA button: gold border (#C9A84C), transparent fill, white label "Explorar cafés" — on hover, fills to solid #C9A84C, label turns #0A0A0A.

### Animation Concept

Staggered entrance, no motion for motion's sake:

1. Page load → the dark right column fades in at 0 opacity over 600ms.
2. The coffee object on the left begins at scale 0.96 and eases to 1.0 over 900ms (cubic-bezier ease-out). No bounce.
3. Text elements in the right column cascade in with a 14px upward translate + fade, each line offset by 120ms.
4. The gold overline draws in left-to-right via a clip-path mask over 400ms, last to appear.
5. On scroll-down: the coffee object has a subtle parallax offset (0.15 factor) — it rises slightly slower than the viewport, reinforcing the feeling of weight and stillness.

No looping animations. Once in, everything is static.

### Mobile Adaptation

Single column, full-width stack:

- The dark panel becomes a full-width header band (#0A0A0A) occupying the top 45% of the viewport with the text centered.
- The product image sits below, edge-to-edge at 100vw, cropped to a square. Background remains #F8F7F4.
- CTA is full-width, 48px tall, gold fill, black label — optimized for thumb reach.
- Font size scales down to 36px headline; overline drops to 10px.
- Animation sequence is identical but compressed: total entrance duration cut to 700ms to feel snappy on mobile.

---

## Variant B — "The Grid of Origins"

### Layout

Full-viewport, single-column centered layout. Background is #F8F7F4 across the entire viewport.

The hero is divided into two horizontal zones:

**Top zone (upper 55%):** Three product images — one per origin (Ethiopia, Colombia, Guatemala) — arranged in a horizontal triptych. Images are square, approximately 280px each on desktop, separated by 24px gaps. They are desaturated to near-monochrome (CSS filter: saturate(0.15) contrast(1.1)), with a thin 1px gold border (#C9A84C). The three panels feel like a curated museum grid.

**Bottom zone (lower 45%):** All text, centered:
- Origin labels float above each image as tiny gold caps (Ethiopia / Colombia / Guatemala), each aligned to its image column — these act as a visual key to the triptych above.
- Headline (#0A0A0A, 64px, ultra-light): "Tres orígenes.  
  Una obsesión."
- Subtext (#0A0A0A at 50% opacity, 15px): "Café de especialidad seleccionado en origen. Envío a toda España."
- CTA: solid black pill (#0A0A0A fill, #F8F7F4 label, 20px border-radius). On hover, background becomes #C9A84C, label turns #0A0A0A.

### Animation Concept

The triptych is the star:

1. The three image panels enter from the top, staggered — left panel first (delay 0ms), center (delay 100ms), right (delay 200ms). Each translates down from -20px to 0 with a 700ms ease-out and simultaneous opacity 0 → 1.
2. The desaturation lifts slightly on hover per image: each panel transitions from saturate(0.15) to saturate(0.40) over 400ms, and a very subtle scale(1.02) on the hovered panel. This invites exploration without being loud.
3. Headline and CTA enter after the images settle (delay 600ms total), with a simple fade-up.
4. Micro-interaction on CTA hover: the gold fill slides in from left via a pseudo-element, not an instant color switch — reinforcing the premium, considered feel.

No parallax scroll here — the grid is architecturally rigid and should remain fixed as a composition.

### Mobile Adaptation

The triptych cannot survive at mobile widths without becoming illegible. Adaptation:

- The three images collapse into a horizontal scroll strip (overflow-x: scroll, snap-type: x mandatory). Each image card is 72vw wide — the visible bleed of the next card at the right edge signals scrollability without a visible scroll indicator.
- Monochrome treatment and gold borders are preserved — the premium feel holds.
- Origin labels move below each card within the scroll strip.
- Headline scales to 40px; remains centered below the strip.
- CTA becomes full-width, same pill shape, 48px height.
- The scroll strip replaces the staggered entrance animation — instead, a single gentle fade-in of the strip as a unit (no per-card stagger) to keep perceived performance fast.

---

## Recommendation

**Variant A is recommended** for the initial launch.

Rationale:

1. **Sales focus first.** Café Vermut's primary goal is online coffee sales. Variant A puts a single, hero product object in direct visual focus. The object fills an entire 60% panel — the visitor's eye lands on the product immediately. Variant B distributes attention across three images simultaneously; that works well for a catalog page, but it can dilute purchase intent at the top of the funnel.

2. **The "Japan meets Barcelona" direction is more legible.** Japanese aesthetic principles (ma — negative space, kanso — simplicity) favor the one-object composition. Variant A's floating cup against pure white space is the more direct translation of that direction. Variant B is still minimal, but a triptych introduces structural complexity that edges away from that reference.

3. **Animation risk is lower.** Variant A's entrance is restrained and has no interactive states on the main visual element. Variant B's per-image hover desaturation shift, while subtle, adds interaction surface area that requires careful QA across devices and could feel inconsistent if tuned imprecisely.

4. **Mobile simplicity.** Variant A's mobile stack is clean and predictable — two full-width zones, thumb-friendly CTA. Variant B's horizontal scroll strip is a good solution but introduces an unconventional scroll axis at the very top of the page, which carries a small UX risk for first-time visitors.

**When to choose Variant B instead:** If the client wants to foreground the three origins as a core brand story (rather than driving immediately to the catalog), or if a second product shoot with three distinct, high-quality origin shots is available, Variant B becomes compelling for a future iteration or a secondary "About the coffees" section.
