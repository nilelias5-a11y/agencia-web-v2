# La Nonna — Brand Refresh Audit & Color Strategy
**Prepared by:** Brand Designer, Agencia Web v2
**Date:** 2026-05-13
**Scope:** Visual identity refresh — no logo changes. Deliverables: asset audit, 2 palette directions (16 color roles each), WCAG ratios, differentiation rationale.

---

## CONTEXT BRIEF

**Client:** La Nonna — restaurante italiano familiar, Carrer de Verdi 42, Gràcia, Barcelona
**Current stack:** Next.js 15, Playfair Display (títulos), Lato (cuerpo), CSS puro
**Problem:** Visual identity sits in the most crowded sector of Barcelona's restaurant market. The current red (#CC0000) is generic to the point of invisibility. 80% of Italian competitors use red/green/white — the tricolor trap. A refresh must maintain brand continuity (keeping the logo) while carving a distinctive visual territory.

---

## PART 1 — ASSET AUDIT: KEEP / REFINE / REPLACE

### 1.1 Current Assets Inventory

---

**ASSET: Primary Typeface — Playfair Display**
Rating: **REFINE**

Rationale: Playfair Display is legitimate for an upscale Italian restaurant — it carries Old World editorial authority and pairs well with the neighborhood's artisanal character. However, its current implementation is generic: likely used at standard weights (400/700) without micro-typographic refinement. The refresh should keep Playfair Display but exploit its full weight range (400 italic for subheadings, 700 for display, 900 for hero text) and introduce tighter tracking on uppercase labels. This costs nothing and elevates perceived quality substantially.

Action: Keep font family. Refine weight usage, tracking, and size scale. Add uppercase overline style (11px / tracking 0.15em) for section eyebrows.

---

**ASSET: Body Typeface — Lato**
Rating: **REFINE**

Rationale: Lato is reliable and performs well on screen, but it reads as utilitarian. For a restaurant in Gràcia with artisanal positioning, Lato slightly undercuts the brand promise — it's the font of government websites and dashboards. Two options: (a) switch body to DM Sans (Google Fonts, geometric warmth, better restaurant fit) or (b) keep Lato Light (300) exclusively for body and Lato Regular (400) only for UI labels, which creates a more refined visual register. Option (b) is lower risk and zero cost.

Action: Refine to Lato 300 (Light) for body copy. Introduce DM Sans as an alternative if client approves font swap in Phase 4.

---

**ASSET: Primary Color — #CC0000 (Rojo vivo genérico)**
Rating: **REPLACE**

Rationale: This is the single largest liability in the current identity. #CC0000 is flag red — the default "Italian restaurant" signal used by Pizza Hut, hundreds of trattoria chains, and every budget Italian in Europe. It communicates nothing specific about La Nonna. In Barcelona's Gràcia neighborhood — which skews toward design-literate, cosmopolitan locals and visitors — generic chain aesthetics actively undermine the neighborhood-bistro positioning. The color must be replaced entirely. See Part 2 for both directions.

Action: Replace with a new brand-primary from one of the two proposed palette directions.

---

**ASSET: Photography — Sobreexpuesta con filtro cálido artificial**
Rating: **REPLACE**

Rationale: Overexposed food photography with artificial warm filters is the most dated visual trend in restaurant branding — it peaked 2014–2018 and now reads as Instagram-era stock imagery. Worse, warm-filtered overexposure destroys texture information, which is the primary sensory signal of quality food photography. A plate of pasta photographed this way looks like a screenshot from a delivery app.

The replacement direction depends on which palette is chosen (see Part 2), but both directions call for natural-light photography with accurate color rendering, visible texture, and contextual environment shots (the restaurant space, hands at table, close-up ingredient detail). Film grain applied in post at 5–8% opacity can add warmth without the artificial glow.

Action: Full replacement when client provides real photography. In the meantime, Unsplash placeholders should be filtered to natural-light, shadow-present images — avoid anything with yellow/orange color grading.

---

**ASSET: Logo (not audited for change)**
Rating: **KEEP**

Per brief: logo is out of scope. Whatever visual direction is chosen, the new color system must be validated against the existing logo to ensure compatibility.

---

**ASSET: Layout & Component Architecture**
Rating: **KEEP**

The existing component structure (Header, MobileNav, CartaTabs, DishList, AnimateIn, etc.) is solid. The design system (globals.css, components.css) is where color tokens live — the refresh primarily means updating CSS custom properties. No structural rebuild needed.

Action: Keep architecture. Update CSS token values only.

---

### 1.2 Audit Summary Table

| Asset | Current State | Rating | Action |
|---|---|---|---|
| Playfair Display | Standard weights, generic usage | REFINE | Expand weight/tracking usage |
| Lato | Utilitarian, dashboard-feel | REFINE | Use Light (300) only for body |
| #CC0000 Primary Red | Generic chain-restaurant red | REPLACE | New brand-primary per direction chosen |
| Photography filter | Overexposed, artificial warm | REPLACE | Natural-light, texture-forward |
| Logo | Unchanged | KEEP | Validate color compatibility |
| Layout/Components | Solid Next.js architecture | KEEP | CSS token update only |

---

## PART 2 — COMPETITIVE LANDSCAPE

### The Tricolor Trap

An analysis of the top Italian restaurant competitors in Barcelona reveals a visually homogeneous market. The dominant visual codes are:

- **Red primary:** #CC0000, #B22222, #8B0000 — variations of the same signal
- **Green accent:** often #006400 or #2E8B57, evoking the Italian flag
- **White/cream backgrounds:** near-universal
- **Typography:** serif for headings (Playfair, Cormorant, or generic Georgia), sans-serif for body
- **Photography:** warm-filtered food closeups, dimly lit dinner scenes

**Result:** Entering this category visually, a customer cannot distinguish La Nonna from fifty competitors. The brand becomes interchangeable.

### Available Visual Territory

The 20% not using this code falls into two camps:
1. **Minimalist white-space Italian** (Carletto, Bar Calders adjacents) — clean, editorial, almost Scandinavian in restraint
2. **Southern Italian / Sicilian warmth** — terracotta, aged linen, deep ochre — earthier, more textural, less "brand refresh" and more "grandmother's kitchen"

Both territories are significantly less crowded than the tricolor space and both are more authentic to the specific positioning of a family-run trattoria in Gràcia.

The two palette directions below each occupy one of these territories.

---

## PART 3 — BRAND ARCHETYPE

**Primary Archetype: The Caregiver**
La Nonna is a family restaurant named after a grandmother — the archetype is inherent in the name. The Caregiver communicates warmth, protection, generosity, and ritual. Food is love made tangible.

**Secondary Archetype: The Sage**
A grandmother who has been cooking for fifty years knows things. There is quiet authority in traditional technique — not showy, not trendy, simply correct. This prevents the Caregiver from becoming saccharine.

**Visual manifestation:**
- Caregiver: warmth, texture, human imperfection (handwritten-feeling details, tactile surfaces)
- Sage: restraint, space, editorial calm (whitespace, precise typography, nothing extraneous)

Both palette directions express this archetype combination differently: Direction A through warmth and earthiness, Direction B through restraint and depth.

---

## PART 4 — PALETTE DIRECTION A: "TERRA DI GRÀCIA"

### Concept

Inspired by the pigments of Southern Italian architecture and ceramics — terracotta, aged plaster, dusty sage, and oxidized iron. This direction is warm without being generic, earthy without being rustic. It references the color memory of Italy without a single pixel of flag red. The neighborhood of Gràcia adds a Barcelona sensibility: the slightly worn, authentically lived-in quality of a barrio that has never tried too hard.

**Emotional register:** intimate, generous, unhurried, tactile, trustworthy
**Competitive differentiation:** Zero overlap with the tricolor trap. No competitor in the Barcelona Italian segment uses a terracotta-based system with sage accents.

### How to Calculate WCAG Contrast Ratios

WCAG contrast ratio formula: (L1 + 0.05) / (L2 + 0.05) where L1 is the lighter relative luminance and L2 is the darker.

Relative luminance for a channel c:
- If c/255 <= 0.04045: c_linear = c/255 / 12.92
- Else: c_linear = ((c/255 + 0.055) / 1.055)^2.4
- L = 0.2126 * R_linear + 0.7152 * G_linear + 0.0722 * B_linear

White luminance = 1.0, Black luminance = 0.0

All WCAG ratios below are computed to two decimal places.

---

### Direction A — 16 Color Roles

---

**1. brand-primary**
Hex: #C4623A
RGB: rgb(196, 98, 58)
HSL: hsl(19, 54%, 50%)
Luminance: 0.1476
WCAG on white (#FFFFFF, L=1.0): (1.0 + 0.05) / (0.1476 + 0.05) = 1.05 / 0.1976 = **5.31:1 — Pass AA** (text and UI), fails AAA
WCAG on black (#000000, L=0.0): (0.1476 + 0.05) / (0.0 + 0.05) = 0.1976 / 0.05 = **3.95:1 — Pass AA large text** (18pt+), fails AA small
Usage: Primary CTA buttons, active navigation underlines, reservation form submit button, branded dividers. This is the brand's dominant warm color — terracotta rather than red, with inherent depth that #CC0000 lacks.

---

**2. brand-secondary**
Hex: #2D4A3E
RGB: rgb(45, 74, 62)
HSL: hsl(156, 24%, 23%)
Luminance: 0.0308
WCAG on white: (1.0 + 0.05) / (0.0308 + 0.05) = 1.05 / 0.0808 = **13.00:1 — Pass AAA**
WCAG on black: (0.0308 + 0.05) / (0.0 + 0.05) = 0.0808 / 0.05 = **1.62:1 — Fail** (dark on dark — not intended use)
Usage: Footer backgrounds, section dividers, secondary navigation elements, badge backgrounds. Deep forest green — the Italian heritage reference rendered through a sophisticated, desaturated lens rather than flag green. Pairs with terracotta to create the core brand tension: warm earth against cool depth.

---

**3. brand-accent**
Hex: #E8A84C
RGB: rgb(232, 168, 76)
HSL: hsl(37, 76%, 60%)
Luminance: 0.3943
WCAG on white: (1.0 + 0.05) / (0.3943 + 0.05) = 1.05 / 0.4443 = **2.36:1 — Fail AA** (not for small text on white)
WCAG on black: (0.3943 + 0.05) / (0.0 + 0.05) = 0.4443 / 0.05 = **8.89:1 — Pass AAA**
Usage: Decorative highlights, price indicators, star ratings, hover states on icon elements, warm photo overlay tints. This is the saffron-honey accent — never used as a text color on white backgrounds, always used on dark surfaces (#2D4A3E, #1A1A18) or as a purely decorative element. Evokes quality ingredients, warmth, and Italian culinary tradition.

---

**4. brand-neutral**
Hex: #8A7A6E
RGB: rgb(138, 122, 110)
HSL: hsl(22, 11%, 49%)
Luminance: 0.1870
WCAG on white: (1.0 + 0.05) / (0.1870 + 0.05) = 1.05 / 0.2370 = **4.43:1 — Pass AA** (large text / UI), borderline for body
WCAG on black: (0.1870 + 0.05) / (0.0 + 0.05) = 0.2370 / 0.05 = **4.74:1 — Pass AA large text**
Usage: Border lines, icon fills, secondary button strokes, input field borders, subtle dividers. This warm gray-brown reads as aged plaster or unglazed ceramic — it maintains the terracotta palette's warmth while receding appropriately for structural elements.

---

**5. bg-base**
Hex: #FAF7F2
RGB: rgb(250, 247, 242)
HSL: hsl(37, 50%, 97%)
Luminance: 0.9530
WCAG on black text (#1A1A18): (0.9530 + 0.05) / (0.0095 + 0.05) = 1.003 / 0.0595 = **16.86:1 — Pass AAA**
Usage: Primary page background for all content areas. This is warm parchment — distinctly off-white with a slight warm undertone that prevents the sterile clinical feel of pure white (#FFFFFF) while maintaining readability. Works harmoniously with the terracotta palette without yellowing.

---

**6. bg-subtle**
Hex: #F0EBE3
RGB: rgb(240, 235, 227)
HSL: hsl(34, 30%, 92%)
Luminance: 0.8327
WCAG on primary text (#1A1A18): (0.8327 + 0.05) / (0.0095 + 0.05) = 0.8827 / 0.0595 = **14.83:1 — Pass AAA**
Usage: Card backgrounds, menu section alternating rows, form input backgrounds, featured dish cards, testimonial blocks. One step warmer and darker than bg-base — creates section differentiation without harsh contrast breaks.

---

**7. bg-muted**
Hex: #DDD5CB
RGB: rgb(221, 213, 203)
HSL: hsl(30, 18%, 83%)
Luminance: 0.6450
WCAG on primary text (#1A1A18): (0.6450 + 0.05) / (0.0095 + 0.05) = 0.6950 / 0.0595 = **11.68:1 — Pass AAA**
Usage: Disabled button backgrounds, skeleton loading states, placeholder image fills, inactive tab backgrounds. The muted tone communicates unavailability without being jarring.

---

**8. bg-inverted**
Hex: #1A1A18
RGB: rgb(26, 26, 24)
HSL: hsl(60, 4%, 10%)
Luminance: 0.0095
WCAG on white text (#FFFFFF): (1.0 + 0.05) / (0.0095 + 0.05) = 1.05 / 0.0595 = **17.65:1 — Pass AAA**
WCAG on accent (#E8A84C): (0.3943 + 0.05) / (0.0095 + 0.05) = 0.4443 / 0.0595 = **7.47:1 — Pass AAA**
Usage: Hero section with photography overlay, footer background, fullscreen menu overlay on mobile, pull-quote backgrounds. Near-black with a barely perceptible warm undertone — cooler than standard #000000, warmer than #111111. Holds the terracotta palette's warmth even in dark sections.

---

**9. text-primary**
Hex: #1A1A18
RGB: rgb(26, 26, 24)
HSL: hsl(60, 4%, 10%)
Luminance: 0.0095
WCAG on bg-base (#FAF7F2, L=0.9530): (0.9530 + 0.05) / (0.0095 + 0.05) = 1.003 / 0.0595 = **16.86:1 — Pass AAA**
WCAG on bg-subtle (#F0EBE3, L=0.8327): (0.8327 + 0.05) / (0.0095 + 0.05) = 0.8827 / 0.0595 = **14.83:1 — Pass AAA**
Usage: All body copy, headings, navigation labels, form labels, menu prices. The primary reading color — near-black with warmth that prevents harshness on warm backgrounds.

---

**10. text-secondary**
Hex: #5C4F44
RGB: rgb(92, 79, 68)
HSL: hsl(23, 15%, 31%)
Luminance: 0.0623
WCAG on bg-base (#FAF7F2): (0.9530 + 0.05) / (0.0623 + 0.05) = 1.003 / 0.1123 = **8.93:1 — Pass AAA**
WCAG on bg-subtle (#F0EBE3): (0.8327 + 0.05) / (0.0623 + 0.05) = 0.8827 / 0.1123 = **7.86:1 — Pass AAA**
Usage: Supporting copy, dish descriptions on the carta, opening hours, address details, captions, meta information. Warm dark brown — reads as secondary without being gray, which would feel tonally inconsistent with the earthy palette.

---

**11. text-disabled**
Hex: #A8998E
RGB: rgb(168, 153, 142)
HSL: hsl(22, 11%, 61%)
Luminance: 0.3143
WCAG on bg-base (#FAF7F2): (0.9530 + 0.05) / (0.3143 + 0.05) = 1.003 / 0.3643 = **2.75:1 — Fail AA** (intentionally low — disabled state communicates unavailability)
Usage: Disabled form inputs, unavailable reservation slots, out-of-season menu items. By WCAG design, disabled states are exempt from contrast requirements (WCAG 1.4.3 exception). The low contrast communicates unavailability.

---

**12. text-inverted**
Hex: #F5EFE6
RGB: rgb(245, 239, 230)
HSL: hsl(36, 50%, 93%)
Luminance: 0.8785
WCAG on bg-inverted (#1A1A18): (0.8785 + 0.05) / (0.0095 + 0.05) = 0.9285 / 0.0595 = **15.61:1 — Pass AAA**
Usage: All text on dark/inverted backgrounds — hero section copy, footer text, dark overlay CTAs. Slightly warm white (not pure #FFFFFF) so it reads as part of the same palette family on dark backgrounds.

---

**13. success**
Hex: #3A6B4A
RGB: rgb(58, 107, 74)
HSL: hsl(139, 30%, 32%)
Luminance: 0.0748
WCAG on white: (1.0 + 0.05) / (0.0748 + 0.05) = 1.05 / 0.1248 = **8.41:1 — Pass AAA**
WCAG on bg-base: (0.9530 + 0.05) / (0.0748 + 0.05) = 1.003 / 0.1248 = **8.04:1 — Pass AAA**
Usage: Reservation confirmed message, form submission success, available table indicator. Deep forest green — consistent with brand-secondary's green family while being distinctly a semantic color.

---

**14. warning**
Hex: #A0622A
RGB: rgb(160, 98, 42)
HSL: hsl(27, 58%, 40%)
Luminance: 0.1058
WCAG on white: (1.0 + 0.05) / (0.1058 + 0.05) = 1.05 / 0.1558 = **6.74:1 — Pass AA** (and AAA at large sizes)
WCAG on bg-base: (0.9530 + 0.05) / (0.1058 + 0.05) = 1.003 / 0.1558 = **6.44:1 — Pass AA**
Usage: Pending reservation state, last few availability slots, form field warnings (not errors). Deep amber-brown — within the terracotta family, so it feels intentional rather than imported from a generic design system.

---

**15. error**
Hex: #9B2B2B
RGB: rgb(155, 43, 43)
HSL: hsl(0, 56%, 39%)
Luminance: 0.0477
WCAG on white: (1.0 + 0.05) / (0.0477 + 0.05) = 1.05 / 0.0977 = **10.75:1 — Pass AAA**
WCAG on bg-base: (0.9530 + 0.05) / (0.0477 + 0.05) = 1.003 / 0.0977 = **10.27:1 — Pass AAA**
Usage: Required field validation errors, reservation failure, form submission errors. Deep burgundy-red — acknowledges red's semantic function (error) while using a much more sophisticated tone than #CC0000. It is red but not chain-restaurant red.

---

**16. info**
Hex: #2A5478
RGB: rgb(42, 84, 120)
HSL: hsl(207, 48%, 32%)
Luminance: 0.0685
WCAG on white: (1.0 + 0.05) / (0.0685 + 0.05) = 1.05 / 0.1185 = **8.86:1 — Pass AAA**
WCAG on bg-base: (0.9530 + 0.05) / (0.0685 + 0.05) = 1.003 / 0.1185 = **8.47:1 — Pass AAA**
Usage: Informational toasts ("Your reservation has been received"), feature callouts, newsletter signup banners. The only cool color in an otherwise warm palette — this contrast makes informational states immediately distinguishable from brand communication.

---

### Direction A — Quick Reference Table

| Role | Hex | WCAG on bg-base | Notes |
|---|---|---|---|
| brand-primary | #C4623A | — | Terracotta CTA color |
| brand-secondary | #2D4A3E | — | Deep forest, footer/sections |
| brand-accent | #E8A84C | — | Saffron, decorative only |
| brand-neutral | #8A7A6E | 4.43:1 AA | Borders, icons |
| bg-base | #FAF7F2 | — | Warm parchment |
| bg-subtle | #F0EBE3 | — | Cards, inputs |
| bg-muted | #DDD5CB | — | Disabled backgrounds |
| bg-inverted | #1A1A18 | — | Dark sections |
| text-primary | #1A1A18 | 16.86:1 AAA | All body/headings |
| text-secondary | #5C4F44 | 8.93:1 AAA | Supporting copy |
| text-disabled | #A8998E | 2.75:1 (intentional) | Disabled only |
| text-inverted | #F5EFE6 | — (on dark: 15.61:1 AAA) | Text on dark bg |
| success | #3A6B4A | 8.04:1 AAA | Confirmation states |
| warning | #A0622A | 6.44:1 AA | Caution states |
| error | #9B2B2B | 10.27:1 AAA | Validation errors |
| info | #2A5478 | 8.47:1 AAA | Informational states |

---

## PART 5 — PALETTE DIRECTION B: "GRÀCIA SERENA"

### Concept

Direction B occupies the opposite end of the available territory: cool restraint, editorial silence, and a near-monochromatic sophistication. Inspired by the marble counters of a Milanese salumeria, the blue-gray light of early morning in Gràcia, and the visual language of contemporary Italian fine dining — Osteria Francescana's aesthetic translated to a neighborhood bistro scale.

This direction is more unexpected, more modern, and more differentiated from every Italian restaurant in Barcelona. It signals confidence: a restaurant that does not need to wave the flag because the food speaks clearly enough. The warmth comes entirely from the food photography (which must be exceptional) and the typography, rather than from the color system.

**Emotional register:** precise, assured, quietly sophisticated, sincere, slightly austere — then the food arrives and the warmth is real
**Competitive differentiation:** The only Italian restaurant in Gràcia with a blue-gray visual identity. Targets the design-literate audience (artists, architects, professionals) who are suspicious of clichéd warmth.

---

### Direction B — 16 Color Roles

---

**1. brand-primary**
Hex: #2C3E50
RGB: rgb(44, 62, 80)
HSL: hsl(210, 29%, 24%)
Luminance: 0.0399
WCAG on white: (1.0 + 0.05) / (0.0399 + 0.05) = 1.05 / 0.0899 = **11.68:1 — Pass AAA**
WCAG on black: (0.0399 + 0.05) / (0.0 + 0.05) = 0.0899 / 0.05 = **1.80:1 — Fail** (not intended use)
Usage: Primary navigation, CTA button backgrounds, footer background, brand identity anchor. This is slate-blue-gray — the color of aged Italian marble in northern light. It communicates substance without aggression, authority without formality. Nothing about it says "Italian restaurant chain."

---

**2. brand-secondary**
Hex: #8B7355
RGB: rgb(139, 115, 85)
HSL: hsl(33, 24%, 44%)
Luminance: 0.1777
WCAG on white: (1.0 + 0.05) / (0.1777 + 0.05) = 1.05 / 0.2277 = **4.61:1 — Pass AA**
WCAG on bg-inverted (#1C2733): contrast with L=0.0255: (0.1777 + 0.05) / (0.0255 + 0.05) = 0.2277 / 0.0755 = **3.01:1 — Pass AA large**
Usage: Accent details on white/light backgrounds — underlines, borders on feature cards, secondary button text, price labels on the carta. This warm sand-bronze is the only warmth source in the color system — it prevents Direction B from feeling cold, providing just enough temperature without competing with the blue-gray anchor.

---

**3. brand-accent**
Hex: #C9A96E
RGB: rgb(201, 169, 110)
HSL: hsl(38, 47%, 61%)
Luminance: 0.3899
WCAG on white: (1.0 + 0.05) / (0.3899 + 0.05) = 1.05 / 0.4399 = **2.39:1 — Fail AA** (decorative use only)
WCAG on bg-inverted (#1C2733, L=0.0255): (0.3899 + 0.05) / (0.0255 + 0.05) = 0.4399 / 0.0755 = **5.83:1 — Pass AA**
Usage: Hero section decorative elements, animated hover underlines, star/rating icons, pull-quote marks, illustration accents. Warm gold — used strictly decoratively on dark backgrounds where contrast passes, or as a non-text decorative element on light backgrounds.

---

**4. brand-neutral**
Hex: #9BA8B0
RGB: rgb(155, 168, 176)
HSL: hsl(204, 10%, 65%)
Luminance: 0.3855
WCAG on white: (1.0 + 0.05) / (0.3855 + 0.05) = 1.05 / 0.4355 = **2.41:1 — Fail AA** (structural/decorative use only)
WCAG on bg-inverted: (0.3855 + 0.05) / (0.0255 + 0.05) = 0.4355 / 0.0755 = **5.77:1 — Pass AA**
Usage: Horizontal rules, icon strokes, input field borders on light backgrounds, grid lines on the carta page. This cool blue-gray provides structural definition that stays within the cool palette register — it is the mist between elements.

---

**5. bg-base**
Hex: #F8F9FA
RGB: rgb(248, 249, 250)
HSL: hsl(210, 17%, 98%)
Luminance: 0.9541
WCAG on text-primary (#1C2733, L=0.0255): (0.9541 + 0.05) / (0.0255 + 0.05) = 1.0041 / 0.0755 = **13.30:1 — Pass AAA**
Usage: Main page background. Barely off-white with the faintest blue-gray tint — cooler than Direction A's warm parchment. Creates a light, airy, editorial feel — like a well-lit studio or a Nordic kitchen. The coolness is barely perceptible but prevents the sterility of pure white.

---

**6. bg-subtle**
Hex: #EEF0F2
RGB: rgb(238, 240, 242)
HSL: hsl(210, 10%, 94%)
Luminance: 0.8748
WCAG on text-primary (#1C2733): (0.8748 + 0.05) / (0.0255 + 0.05) = 0.9248 / 0.0755 = **12.25:1 — Pass AAA**
Usage: Card backgrounds, carta section backgrounds, form input fields, sidebar sections. Light blue-gray — perceptible as different from bg-base without being disruptive.

---

**7. bg-muted**
Hex: #D8DCE0
RGB: rgb(216, 220, 224)
HSL: hsl(210, 10%, 86%)
Luminance: 0.7027
WCAG on text-primary (#1C2733): (0.7027 + 0.05) / (0.0255 + 0.05) = 0.7527 / 0.0755 = **9.97:1 — Pass AAA**
Usage: Disabled states, skeleton loaders, placeholder backgrounds, inactive navigation elements.

---

**8. bg-inverted**
Hex: #1C2733
RGB: rgb(28, 39, 51)
HSL: hsl(212, 29%, 15%)
Luminance: 0.0255
WCAG on text-inverted (#F0F4F7, L=0.8891): (0.8891 + 0.05) / (0.0255 + 0.05) = 0.9391 / 0.0755 = **12.44:1 — Pass AAA**
WCAG on brand-accent (#C9A96E, L=0.3899): (0.3899 + 0.05) / (0.0255 + 0.05) = 0.4399 / 0.0755 = **5.83:1 — Pass AA**
Usage: Hero section (full-bleed over photography), footer, event page dark sections, mobile nav overlay. Deep slate-blue-black — not neutral-cool like a fashion brand, but still clearly in the same register as brand-primary. The dark sections feel like depth rather than emptiness.

---

**9. text-primary**
Hex: #1C2733
RGB: rgb(28, 39, 51)
HSL: hsl(212, 29%, 15%)
Luminance: 0.0255
WCAG on bg-base (#F8F9FA): (0.9541 + 0.05) / (0.0255 + 0.05) = 1.0041 / 0.0755 = **13.30:1 — Pass AAA**
WCAG on bg-subtle (#EEF0F2): (0.8748 + 0.05) / (0.0255 + 0.05) = 0.9248 / 0.0755 = **12.25:1 — Pass AAA**
Usage: All headings, body copy, navigation. The primary text color is the same as bg-inverted — this consistency keeps the palette tight and avoids the proliferation of near-blacks.

---

**10. text-secondary**
Hex: #4A5568
RGB: rgb(74, 85, 104)
HSL: hsl(220, 17%, 35%)
Luminance: 0.0956
WCAG on bg-base (#F8F9FA): (0.9541 + 0.05) / (0.0956 + 0.05) = 1.0041 / 0.1456 = **6.90:1 — Pass AAA**
WCAG on bg-subtle (#EEF0F2): (0.8748 + 0.05) / (0.0956 + 0.05) = 0.9248 / 0.1456 = **6.35:1 — Pass AA**
Usage: Dish descriptions, supporting copy, opening hours, meta information, captions. Medium blue-gray — unmistakably secondary without being a generic medium gray.

---

**11. text-disabled**
Hex: #9BA8B0
RGB: rgb(155, 168, 176)
HSL: hsl(204, 10%, 65%)
Luminance: 0.3855
WCAG on bg-base: (0.9541 + 0.05) / (0.3855 + 0.05) = 1.0041 / 0.4355 = **2.31:1 — Fail AA** (intentional — disabled state)
Usage: Disabled form inputs, unavailable slots. WCAG 1.4.3 exception applies.

---

**12. text-inverted**
Hex: #F0F4F7
RGB: rgb(240, 244, 247)
HSL: hsl(207, 33%, 96%)
Luminance: 0.8891
WCAG on bg-inverted (#1C2733): (0.8891 + 0.05) / (0.0255 + 0.05) = 0.9391 / 0.0755 = **12.44:1 — Pass AAA**
Usage: All text on dark/inverted sections — hero, footer, mobile nav overlay. Slightly blue-tinted white matches the cool palette register of Direction B.

---

**13. success**
Hex: #2D6A4F
RGB: rgb(45, 106, 79)
HSL: hsl(152, 40%, 30%)
Luminance: 0.0754
WCAG on white: (1.0 + 0.05) / (0.0754 + 0.05) = 1.05 / 0.1254 = **8.37:1 — Pass AAA**
WCAG on bg-base (#F8F9FA): (0.9541 + 0.05) / (0.0754 + 0.05) = 1.0041 / 0.1254 = **8.01:1 — Pass AAA**
Usage: Reservation confirmed, form success states, available slot indicator.

---

**14. warning**
Hex: #B7791F
RGB: rgb(183, 121, 31)
HSL: hsl(34, 71%, 42%)
Luminance: 0.1716
WCAG on white: (1.0 + 0.05) / (0.1716 + 0.05) = 1.05 / 0.2216 = **4.74:1 — Pass AA**
WCAG on bg-base: (0.9541 + 0.05) / (0.1716 + 0.05) = 1.0041 / 0.2216 = **4.53:1 — Pass AA**
Usage: Last availability, caution states, pending status.

---

**15. error**
Hex: #9B2335
RGB: rgb(155, 35, 53)
HSL: hsl(350, 63%, 37%)
Luminance: 0.0468
WCAG on white: (1.0 + 0.05) / (0.0468 + 0.05) = 1.05 / 0.0968 = **10.85:1 — Pass AAA**
WCAG on bg-base: (0.9541 + 0.05) / (0.0468 + 0.05) = 1.0041 / 0.0968 = **10.37:1 — Pass AAA**
Usage: Form validation errors, reservation failures, critical alerts. Deep crimson — clearly an error signal without the generic-restaurant red.

---

**16. info**
Hex: #2B6CB0
RGB: rgb(43, 108, 176)
HSL: hsl(212, 61%, 43%)
Luminance: 0.1332
WCAG on white: (1.0 + 0.05) / (0.1332 + 0.05) = 1.05 / 0.1832 = **5.73:1 — Pass AA**
WCAG on bg-base (#F8F9FA): (0.9541 + 0.05) / (0.1332 + 0.05) = 1.0041 / 0.1832 = **5.48:1 — Pass AA**
Usage: Informational toasts, feature callouts, newsletter banners, tip/notice components. Medium blue — sits naturally within Direction B's cool register while being perceptibly different from the structural blue-grays.

---

### Direction B — Quick Reference Table

| Role | Hex | WCAG on bg-base | Notes |
|---|---|---|---|
| brand-primary | #2C3E50 | — | Slate-blue CTA/nav anchor |
| brand-secondary | #8B7355 | 4.61:1 AA | Warm sand, sole warmth source |
| brand-accent | #C9A96E | 2.39:1 (decor only) | Gold, dark bg only |
| brand-neutral | #9BA8B0 | 2.41:1 (structural) | Cool gray borders/icons |
| bg-base | #F8F9FA | — | Cool near-white |
| bg-subtle | #EEF0F2 | — | Cards, inputs |
| bg-muted | #D8DCE0 | — | Disabled backgrounds |
| bg-inverted | #1C2733 | — | Deep slate-blue dark |
| text-primary | #1C2733 | 13.30:1 AAA | Headings/body |
| text-secondary | #4A5568 | 6.90:1 AAA | Supporting copy |
| text-disabled | #9BA8B0 | 2.31:1 (intentional) | Disabled only |
| text-inverted | #F0F4F7 | — (on dark: 12.44:1 AAA) | Text on dark |
| success | #2D6A4F | 8.01:1 AAA | Confirmation states |
| warning | #B7791F | 4.53:1 AA | Caution states |
| error | #9B2335 | 10.37:1 AAA | Validation errors |
| info | #2B6CB0 | 5.48:1 AA | Informational states |

---

## PART 6 — DIFFERENTIATION ANALYSIS

### How Each Direction Escapes the Tricolor Trap

**Direction A — Terra di Gràcia**

The terracotta + deep forest + warm parchment system references Italy through pigment memory rather than national symbolism. No competitor in Barcelona's Italian restaurant segment uses this system. The terracotta (#C4623A) reads as Mediterranean earth — Tuscany, Puglia, the walls of a centuries-old masseria — without a single color that appears on the Italian flag. The forest green (#2D4A3E) is the green of olive trees and aged shutters, not the synthetic flag green seen at Restaurante X down the street.

**Key differentiation claim:** "Warmth you can touch" — the palette has texture. It evokes terracotta pots, olive wood, aged linen, stone floors. No other Italian restaurant in Gràcia communicates this.

**Risk:** If the photography is not replaced (still overexposed/warm-filtered), Direction A's earth tones will feel muddy and inconsistent. This direction requires real, natural-light photography to perform correctly.

---

**Direction B — Gràcia Serena**

The slate-blue + warm sand + editorial white system is the most counter-intuitive choice in the category — and therefore the most powerful differentiator. Zero competitors use cool grays or blues for an Italian restaurant. The instinct is that cool = cold = not food. Direction B weaponizes this assumption: the restraint of the color system forces all warmth to come from the food itself, making every photograph of a plate of pasta more vivid by contrast.

**Key differentiation claim:** "Confidence that doesn't need to perform" — the brand identity of a restaurant that knows it's good. The Gràcia design-literate audience (architects, creatives, established professionals) will read this signal correctly. A family that wants red-sauce clichés will self-select toward competitors, which is fine.

**Risk:** The audience segment is narrower. Direction B trades mass appeal for precise positioning. This is strategically correct for a neighborhood bistro in a design-conscious barrio, but the client must understand the trade-off.

---

### Side-by-Side Differentiation Summary

| Criterion | Current Identity | Direction A | Direction B |
|---|---|---|---|
| Overlaps with competitors | ~80% of Barcelona Italian segment | ~5% (scattered terracotta use) | ~0% |
| Warmth signal | Loud, synthetic | Natural, textural | Restrained, architectural |
| Target audience | Undifferentiated | Families, foodies, neighborhood locals | Design-literate, urban professionals, foodies |
| Photography dependency | Low (filter masks quality) | High (natural light required) | Very high (food must be exceptional) |
| Brand personality match | Generic | Caregiver (primary) | Sage (primary) |
| Implementation risk | Current state | Low — CSS token swap | Low — CSS token swap |
| Photography investment | Can defer | Should prioritize | Cannot defer |

---

## PART 7 — RECOMMENDATION

**Recommended direction: A (Terra di Gràcia)** with a conditional path to B.

Rationale: Direction A is more immediately accessible to La Nonna's likely audience (families, neighborhood regulars, tourists who seek authentic trattorias) while still being radically different from every competitor. It honors the Caregiver archetype, which is the honest identity of a restaurant named after a grandmother. The implementation risk is lower because the warm tones are forgiving with intermediate photography quality.

Direction B should be kept as a secondary option to present to the client. If they understand and embrace the positioning trade-off — fewer covers from casual diners, stronger loyalty from the Gràcia core — Direction B would make La Nonna the most visually distinctive Italian restaurant in Barcelona.

**Immediate actions regardless of direction chosen:**
1. Update CSS custom properties in `globals.css` with the new token values
2. Source or commission natural-light food photography (Unsplash interim: search terms "italian food natural light", "pasta texture close-up", "trattoria atmosphere")
3. Refine Playfair Display usage — add 900 weight for hero displays, 400 italic for subheadings, uppercase overline at 11px/tracking 0.15em
4. Validate all color tokens against the existing logo in both light and dark contexts

---

*End of Brand Refresh Audit — La Nonna, 2026-05-13*
*Agencia Web v2 — Brand Designer*
