# Hero Section Spec — Café Vermut

**Client:** Café Vermut — Specialty Coffee, Barcelona  
**Visual Direction:** "Japan meets Barcelona" — minimal, premium, almost empty  
**Brand Colors:** Near-white #F8F7F4 | Pure black #0A0A0A | Gold accent #C9A84C  
**Primary CTA:** "Explorar cafés"  
**Stack:** Next.js 16 + Tailwind + Framer Motion

---

## Comparison Engine — Two Variants

The director requires 2 distinct directions. Each variant is named, fully specified, and argued on its own merits. The recommendation follows both specs.

---

## Variant A — "El Silencio"

**Concept:** The coffee speaks alone. Near-empty canvas. Maximum confidence through restraint — inspired by Japanese white-space philosophy (ma) combined with the unhurried pace of a Barcelona vermut hour.

---

**Layout pattern:** Pattern 3 — Typographic Hero (oversized type as visual element)

**Background:**  
Solid #F8F7F4 (near-white). No image, no video. A single, very faint circular watermark graphic — a stylized coffee ring stain at 4% opacity centered on the right half of the canvas. Adds texture without noise.

**Headline:**  
"Café del origen."  
Font: Shippori Mincho (Google Fonts — Japanese-style serif with Latin support, refined stroke contrast)  
Size: 96px desktop / 56px mobile  
Weight: 500 (medium — avoids heaviness)  
Color: #0A0A0A  
Letter-spacing: -0.02em  
Line-height: 1.0  
Positioned: Left-aligned, starting at 12% from left edge, vertically centered in viewport

**Subheadline:**  
"Etiopía. Colombia. Guatemala. Enviado desde Barcelona."  
Font: Inter (or DM Sans) — clean grotesque for contrast with the serif headline  
Size: 16px desktop / 14px mobile  
Weight: 400  
Color: #0A0A0A at 55% opacity  
Letter-spacing: 0.08em (wide tracking — editorial tone)  
Position: 20px below headline baseline

**Primary CTA:**  
"Explorar cafés" → /tienda  
Style: Text link with a thin underline in gold #C9A84C; no filled button  
Font: Inter, 14px, weight 500, letter-spacing 0.12em, uppercase  
Color: #0A0A0A  
Underline: 1px solid #C9A84C, offset 4px  
Hover: underline slides right-to-left (Framer Motion layoutId transition), text shifts to #C9A84C  
Position: 32px below subheadline

**Secondary CTA (none):** The single-CTA approach reinforces the focused, editorial character. Adding a second CTA would dilute.

**Gold accent detail:**  
A 1px horizontal rule in #C9A84C, 48px wide, placed 24px above the headline. Acts as a breath mark — signals premium without decoration.

**Animation:**  
Entry sequence on page load (total duration: 1.2s, then idle):  
1. Gold rule fades in and extends left-to-right: delay 0ms, duration 500ms, ease out-expo  
2. Headline reveals word by word (3 words), staggered 80ms each, translateY(16px) → 0, opacity 0 → 1: starts at 200ms  
3. Subheadline fades in: delay 600ms, duration 400ms, opacity 0 → 1, translateY(8px) → 0  
4. CTA appears: delay 900ms, duration 300ms, opacity 0 → 1  
All animations use Framer Motion `variants` with `staggerChildren`. GPU-accelerated via transform only. `prefers-reduced-motion`: all animations disabled, elements render at full opacity immediately.

**Mobile adaptation (375px):**  
- Headline: 56px, full-width left-aligned, 24px horizontal padding  
- Subheadline: 13px, line-height 1.6  
- Gold rule: 32px wide  
- Coffee ring watermark: hidden (prevents visual noise on small screen)  
- CTA tap target: minimum 44px height via padding  
- Layout remains left-aligned — no centering; preserves the editorial voice

**Accessibility notes:**  
- Headline on #F8F7F4: contrast ratio #0A0A0A / #F8F7F4 = 18.9:1 (AAA)  
- Subheadline at 55% opacity (#737373 effective on white): 4.6:1 (AA for large text, borderline AA for body — increase to 60% opacity if QA flags it)  
- Gold CTA underline is decorative; text itself is full contrast  
- Background watermark: role="presentation", aria-hidden="true"

**Why this works for Café Vermut:**  
Specialty coffee buyers are informed, aesthetically literate, and skeptical of hype. "El Silencio" signals confidence: a brand that does not need to shout. The Japanese serif mixed with a tight grotesque subheadline delivers the "Japan meets Barcelona" brief precisely at the typographic level — no mood board required. The total absence of product photography in the hero creates intrigue that pulls the visitor deeper.

### Reference inspirations
1. Kinto (kinto.co) — the near-empty white canvas with a single serif object; borrow the quiet authority
2. Nomad Coffee (nomadcoffee.es, Barcelona) — local specialty coffee brand; borrow the editorial headline tone and the resistance to lifestyle photography above the fold

---

## Variant B — "La Hora Dorada"

**Concept:** One commanding product image; the coffee takes physical space. A full-bleed left panel of a single cup, shot from above on a raw concrete surface, warm morning Barcelona light. Typography holds the right side. Split composition references Japanese asymmetry (left heavy / right sparse).

---

**Layout pattern:** Pattern 2 — Left-Aligned Split (image left / text right)

**Background:**  
Left panel (55% viewport width): Full-height product photograph. Art direction: overhead shot of a single espresso cup on raw Catalonian stone, golden morning light raking from upper-left, steam barely visible. Color palette of the photo must be warm near-blacks and amber — post-processed to align with #C9A84C gold tones. Image occupies left 55% with object-cover, no parallax on desktop.  
Right panel (45% viewport width): Solid #F8F7F4. All text lives here.

**Headline:**  
"Origen puro."  
Font: Shippori Mincho  
Size: 80px desktop / 48px mobile  
Weight: 600  
Color: #0A0A0A  
Letter-spacing: -0.01em  
Line-height: 1.0  
Position: Right panel, vertically centered, left-aligned within the panel, 48px left padding

**Subheadline:**  
"Café de especialidad de Etiopía, Colombia y Guatemala. Tostado en Barcelona."  
Font: DM Sans  
Size: 15px desktop / 14px mobile  
Weight: 400  
Color: #0A0A0A at 60% opacity  
Letter-spacing: 0.04em  
Line-height: 1.7  
Position: 24px below headline

**Primary CTA:**  
"Explorar cafés" → /tienda  
Style: Solid filled button  
Background: #0A0A0A  
Text: #F8F7F4, Inter 13px, weight 500, letter-spacing 0.10em, uppercase  
Size: min-width 180px, height 48px  
Border-radius: 0px (sharp corners — premium, not playful)  
Hover state: background transitions to #C9A84C (gold), text remains #0A0A0A, transition 250ms ease  
Position: 32px below subheadline

**Gold accent detail:**  
A thin vertical rule in #C9A84C, 1px wide, 64px tall, placed to the left of the headline at 32px margin. Visually anchors the text block and introduces the brand's accent color in a structural (not decorative) way.

**Animation:**  
1. Left image panel: enters from left with translateX(-40px) → 0, opacity 0 → 1: delay 0ms, duration 700ms, ease out-expo  
2. Gold vertical rule draws top-to-bottom (scaleY 0 → 1, transform-origin top): delay 400ms, duration 400ms  
3. Headline: translateX(20px) → 0, opacity 0 → 1: delay 500ms, duration 500ms  
4. Subheadline: opacity 0 → 1: delay 800ms, duration 350ms  
5. CTA: opacity 0 → 1, slight translateY(8px) → 0: delay 1000ms, duration 300ms  
Total sequence: 1.3s. GPU-accelerated via transform only. `prefers-reduced-motion`: all elements render at full opacity/position immediately with no motion.

**Mobile adaptation (375px):**  
- Layout switches from split (side-by-side) to stacked  
- Product image occupies top 45vh of screen, full width, object-cover object-position: center 30%  
- Text block sits below the image on #F8F7F4 background  
- Headline: 44px, left-aligned, 20px horizontal padding  
- Gold vertical rule: replaced with a 32px horizontal rule above headline (vertical rules lose meaning in stacked layout)  
- CTA: full-width on mobile (width: 100%, max-width: 320px, centered)  
- Image has a subtle bottom gradient fade (#F8F7F4 at 40% opacity) to blend image into text zone

**Accessibility notes:**  
- All text is on #F8F7F4 panel, not on image: contrast is fully predictable at 18.9:1 (AAA) — no overlay contrast risk  
- Product image: alt="Espresso sobre piedra, luz de mañana Barcelona"  
- CTA hover state (#C9A84C background with #0A0A0A text): contrast ratio 8.2:1 (AAA)  
- Vertical gold rule: aria-hidden="true"

**Why this works for Café Vermut:**  
Some visitors need tangible evidence before they trust a specialty coffee brand. "La Hora Dorada" gives them that — a single powerful image that smells like coffee before a word is read. The asymmetric split (55/45) is inherently Japanese in composition, heavier on the visual side, with the text breathing quietly to the right. The gold CTA hover converts the exploration prompt into a brand moment. This variant is more likely to convert a first-time visitor with no prior brand awareness, and better serves the primary goal of online sales.

### Reference inspirations
1. Onibus Coffee (Tokyo) — the disciplined use of a single product photograph to do all the emotional work; borrow the restraint in image selection (one cup, no lifestyle clutter)
2. Nomad Coffee (Barcelona) — the pairing of honest material surfaces (concrete, stone) with clean typography; borrow the material authenticity in photo art direction

---

## Director Recommendation

**Recommended: Variant B — "La Hora Dorada"**

**Rationale:**  
The primary goal is online coffee sales — not editorial prestige or agency portfolio awards. Variant B answers the 3-second rule more completely for a first-time visitor: it shows what the product is (coffee, specialty, premium) before any text is read, builds sensory desire through the image, and converts that desire directly through the CTA. Variant A demands more from the visitor — it requires them to already appreciate editorial minimalism to feel pulled in. For an established brand (Kinto, Blue Bottle at scale), Variant A would be correct. For Café Vermut launching online sales, Variant B earns trust faster and converts higher.

**However:** Variant A is the stronger artistic statement and the more memorable long-term brand anchor. If the director's priority is brand positioning and the store already has organic traffic from an existing customer base, Variant A is the bolder, more appropriate choice.

**Suggested director question:** "Who is the typical visitor arriving at this hero for the first time — someone who already knows Café Vermut, or a cold visitor discovering us through Instagram or search?" The answer determines the variant.

---

*Spec produced by hero-specialist. Deliver to frontend-developer for implementation. Pending: final photography art direction confirmation for Variant B; final typeface license check for Shippori Mincho (Google Fonts — free, no license issue).*
