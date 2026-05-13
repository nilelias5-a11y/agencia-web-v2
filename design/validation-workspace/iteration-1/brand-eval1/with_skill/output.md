# Café Vermut — Brand Identity System
### Phase 1–7 Complete Output | Artesanía Mediterránea

**Prepared by:** Brand Designer  
**Date:** 2026-05-13  
**Visual Direction Approved:** Artesanía Mediterránea  
**Client:** Café Vermut — Specialty coffee café + online ground coffee, Barcelona

---

## PHASE 1 — Asset Audit

### Existing Client Assets

Since this is a brand creation brief (no prior identity exists), the audit documents the starting baseline:

| Asset | Status | Rating |
|---|---|---|
| Logo | None — to be created | Replace |
| Color usage | None established | Replace |
| Typography | None established | Replace |
| Photography | Documentary café photography referenced in brief | Refine |
| Social presence | Assumed basic/none | Replace |
| Printed materials | None | N/A |

**Baseline conclusion:** Clean slate. No legacy assets to constrain direction. Full creative freedom within the approved visual direction.

---

### Competitor Visual Audit

**Market segment:** Specialty coffee cafés in Barcelona with online retail.

---

**Competitor — Nomad Coffee (Barcelona)**

**Website:** nomadcoffee.es  
**Color palette:** White-dominant, black type, minimal warm accents — very Swiss/minimal  
**Typography:** Clean geometric sans-serif (Futura-adjacent)  
**Visual tone:** Minimal, technical, Scandinavian-influenced  
**What they do well:** Positions coffee as precision craft; photography is exceptional  
**Gap we can own:** Emotional warmth, Mediterranean identity, place-rootedness — Nomad feels anywhere, not Barcelona

---

**Competitor — Right Side Coffee (Barcelona)**

**Website:** rightsidecoffee.com  
**Color palette:** White + black + occasional coral accent  
**Typography:** Grotesque sans-serif  
**Visual tone:** Urban minimal, slightly industrial  
**What they do well:** Clean brand recognition; consistent social media grid  
**Gap we can own:** Artisanal craft narrative, warm Mediterranean palette — they feel more NYC than Gràcia

---

**Competitor — Cafés El Magnífico (Barcelona)**

**Website:** cafeselmagnifico.com  
**Color palette:** Dark brown, gold, cream — heritage/traditional  
**Typography:** Classic serif with ornate detailing  
**Visual tone:** Heritage, traditional, established authority  
**What they do well:** Communicates decades of expertise and origin traceability  
**Gap we can own:** Modern artisanal without the dusty heritage feel — Magnífico feels 1950s, we want 2020s Gràcia

---

**Competitor — Nomad / Slow Moe Coffee (online Spain)**

**Color palette:** Earthy green, off-white — sustainability-coded  
**Typography:** Humanist sans-serif  
**Visual tone:** Eco-conscious, slow living  
**What they do well:** Strong sustainability narrative  
**Gap we can own:** Mediterranean specificity and terracota warmth — green+cream is increasingly generic in the specialty space

---

**Competitor — Tim Wendelboe (Oslo, international benchmark)**

**Website:** timwendelboe.no  
**Color palette:** Off-white, deep navy, black — restrained Nordic  
**Typography:** Classic serif + clean sans pairing  
**Visual tone:** Austere precision, anti-decoration  
**What they do well:** The gold standard for specialty; every detail is deliberate  
**Gap we can own:** Human warmth, color, Mediterranean light — Wendelboe is intentionally cold; we are intentionally warm

---

### Competitive Synthesis

**Overcrowded territory:** White/black/minimal; Scandinavian-coded precision; cold greens for sustainability; flat geometric sans-serif  
**Available territory:** Warm terracota/clay palette with genuine Mediterranean context; serif typography with contemporary restraint; documentary photography with light and texture; artisanal craft narrative that feels 2020s-Barcelona rather than heritage or Nordic

---

## PHASE 2 — Brand Architecture

### Brand Archetype

**Primary: The Creator (El Artesano)**  
Essence: Originality through craft mastery. The Creator does not mass-produce — every cup, every bag is an intentional act.  
Visual manifestation: Textures that show the hand, typography with humanist warmth, color that references materials (terracota clay, olive wood, parchment), photography that shows process and maker.

**Secondary: The Everyman (El Veí)**  
Essence: Rooted in the neighborhood. Not precious or exclusive — genuinely part of Gràcia's daily life.  
Visual manifestation: Warmth over perfection, documentary photography over studio shoots, approachable typography weight choices, a palette that feels like Barcelona tiles rather than a design mood board.

**Archetype tension** (productive): The Creator wants to be exceptional; The Everyman wants to be accessible. The brand resolves this by being *exceptional about the everyday* — the best cup you drink on a Tuesday morning before work.

---

**Brand Voice (3 adjectives):**

1. **Calorós** (Warm) — In practice: copy uses first person plural ("nosaltres torrem"), photography shows people mid-conversation, never isolated product shots
2. **Precís** (Precise) — In practice: specific origin details, exact roast dates, process described without jargon but with accuracy
3. **Arrelat** (Rooted) — In practice: Barcelona geography is specific (Gràcia, not just "Barcelona"), seasonal offerings, neighborhood context

---

## PHASE 3 — Color System

### Color Rationale

The palette is distilled from the physical materials of Mediterranean craft: terracota roof tiles baking in afternoon sun, parchment worn smooth by handling, olive groves in the Garraf hills, the dark espresso shot in a white ceramic cup, the chalk on a blackboard menu.

All WCAG ratios are calculated using the WCAG 2.1 relative luminance formula:  
`L = 0.2126 × R_lin + 0.7152 × G_lin + 0.0722 × B_lin`  
where `R_lin = (R/255)^2.2` (simplified sRGB linearization)  
Contrast ratio = `(L_lighter + 0.05) / (L_darker + 0.05)`

---

### Complete 16-Color Palette

---

#### 1. `brand-primary` — Terracota

```
Hex:  #C1603A
RGB:  rgb(193, 96, 58)
HSL:  hsl(18, 54%, 49%)

Luminance calculation:
  R_lin = (193/255)^2.2 = 0.757^2.2 ≈ 0.5494
  G_lin = (96/255)^2.2  = 0.376^2.2 ≈ 0.1177
  B_lin = (58/255)^2.2  = 0.227^2.2 ≈ 0.0413
  L = 0.2126×0.5494 + 0.7152×0.1177 + 0.0722×0.0413
  L = 0.1168 + 0.0842 + 0.0030 = 0.2040

WCAG on white (#FFFFFF, L=1.0):
  Ratio = (1.0 + 0.05) / (0.2040 + 0.05) = 1.05 / 0.2540 = 4.13:1 — Pass AA (normal text ≥4.5 FAIL; large text ≥3 PASS; UI components ≥3 PASS)
  → Pass AA Large Text / UI Components; Fail AA Normal Text
  Note: Used at 19px+ or for UI, not small body copy

WCAG on black (#000000, L=0):
  Ratio = (0.2040 + 0.05) / (0 + 0.05) = 0.254 / 0.05 = 5.08:1 — Pass AA
```

**Usage:** Primary CTAs, logo mark (primary version), active navigation states, product tags, hover states on cards. The dominant brand expression color. Never used as body text on white at sizes below 18px.

---

#### 2. `brand-secondary` — Verde Oliva

```
Hex:  #5C6B3E
RGB:  rgb(92, 107, 62)
HSL:  hsl(84, 27%, 33%)

Luminance:
  R_lin = (92/255)^2.2  = 0.361^2.2 ≈ 0.1075
  G_lin = (107/255)^2.2 = 0.420^2.2 ≈ 0.1459
  B_lin = (62/255)^2.2  = 0.243^2.2 ≈ 0.0474
  L = 0.2126×0.1075 + 0.7152×0.1459 + 0.0722×0.0474
  L = 0.0228 + 0.1044 + 0.0034 = 0.1306

WCAG on white:
  Ratio = 1.05 / (0.1306 + 0.05) = 1.05 / 0.1806 = 5.81:1 — Pass AA ✓ (also Pass AA Large)
WCAG on black:
  Ratio = (0.1306 + 0.05) / 0.05 = 0.1806 / 0.05 = 3.61:1 — Pass AA Large / UI; Fail AA Normal
```

**Usage:** Secondary CTAs (ghost button border + text), category tags for origin stories, sustainability/process sections, footer accents, paired with `bg-base` (crema). This darker olive reliably passes AA on white backgrounds.

---

#### 3. `brand-accent` — Miel Ámbar

```
Hex:  #D4882A
RGB:  rgb(212, 136, 42)
HSL:  hsl(34, 67%, 50%)

Luminance:
  R_lin = (212/255)^2.2 = 0.831^2.2 ≈ 0.6799
  G_lin = (136/255)^2.2 = 0.533^2.2 ≈ 0.2487
  B_lin = (42/255)^2.2  = 0.165^2.2 ≈ 0.0197
  L = 0.2126×0.6799 + 0.7152×0.2487 + 0.0722×0.0197
  L = 0.1445 + 0.1778 + 0.0014 = 0.3237

WCAG on white:
  Ratio = 1.05 / (0.3237 + 0.05) = 1.05 / 0.3737 = 2.81:1 — Fail AA (decorative/graphic use only)
WCAG on black:
  Ratio = (0.3237 + 0.05) / 0.05 = 0.3737 / 0.05 = 7.47:1 — Pass AAA ✓
```

**Usage:** Badges, price highlights, "featured" ribbons, icon fills on dark backgrounds, hover glow on inverted sections. NEVER used as text on white — always decorative or paired with `bg-inverted` / `#2B1F14` dark background where it passes AAA. Use `text-primary` (#2B1F14) on top of this color for accessible text.

---

#### 4. `brand-neutral` — Piedra

```
Hex:  #9E8E7E
RGB:  rgb(158, 142, 126)
HSL:  hsl(28, 14%, 56%)

Luminance:
  R_lin = (158/255)^2.2 = 0.620^2.2 ≈ 0.3639
  G_lin = (142/255)^2.2 = 0.557^2.2 ≈ 0.2772
  B_lin = (126/255)^2.2 = 0.494^2.2 ≈ 0.2157
  L = 0.2126×0.3639 + 0.7152×0.2772 + 0.0722×0.2157
  L = 0.0773 + 0.1983 + 0.0156 = 0.2912

WCAG on white:
  Ratio = 1.05 / (0.2912 + 0.05) = 1.05 / 0.3412 = 3.08:1 — Pass AA Large / UI; Fail AA Normal
WCAG on black:
  Ratio = (0.2912 + 0.05) / 0.05 = 0.3412 / 0.05 = 6.82:1 — Pass AAA ✓
```

**Usage:** Dividers, ornamental rules, icon strokes on light backgrounds, border on cards in `bg-subtle`, decorative texture overlays. Not used for text. Provides visual balance between the warm brand colors and neutral backgrounds.

---

#### 5. `bg-base` — Crema

```
Hex:  #F5EDD8
RGB:  rgb(245, 237, 216)
HSL:  hsl(40, 68%, 90%)

Luminance:
  R_lin = (245/255)^2.2 = 0.961^2.2 ≈ 0.9196
  G_lin = (237/255)^2.2 = 0.929^2.2 ≈ 0.8499
  B_lin = (216/255)^2.2 = 0.847^2.2 ≈ 0.6888
  L = 0.2126×0.9196 + 0.7152×0.8499 + 0.0722×0.6888
  L = 0.1955 + 0.6077 + 0.0497 = 0.8529

WCAG on white: not applicable (this IS a background)
WCAG vs text-primary (#2B1F14, L≈0.0133):
  Ratio = (0.8529 + 0.05) / (0.0133 + 0.05) = 0.9029 / 0.0633 = 14.26:1 — Pass AAA ✓
```

**Usage:** Main page background. The parchment-warm base that gives the site its Mediterranean warmth. Not pure white — this tint signals craft and analog quality. All body text on this background passes AAA comfortably.

---

#### 6. `bg-subtle` — Lino

```
Hex:  #EDE3CC
RGB:  rgb(237, 227, 204)
HSL:  hsl(40, 46%, 86%)

Luminance:
  R_lin = (237/255)^2.2 ≈ 0.8499
  G_lin = (227/255)^2.2 ≈ 0.7864
  B_lin = (204/255)^2.2 ≈ 0.6038
  L = 0.2126×0.8499 + 0.7152×0.7864 + 0.0722×0.6038
  L = 0.1807 + 0.5625 + 0.0436 = 0.7868

WCAG vs text-primary (#2B1F14):
  Ratio = (0.7868 + 0.05) / (0.0133 + 0.05) = 0.8368 / 0.0633 = 13.22:1 — Pass AAA ✓
```

**Usage:** Card backgrounds, input fields, section alternation on long pages, product listing background. Subtly darker than `bg-base` to create depth without introducing color contrast.

---

#### 7. `bg-muted` — Gris Cálido

```
Hex:  #D9CFBE
RGB:  rgb(217, 207, 190)
HSL:  hsl(37, 22%, 80%)

Luminance:
  R_lin = (217/255)^2.2 ≈ 0.6994
  G_lin = (207/255)^2.2 ≈ 0.6266
  B_lin = (190/255)^2.2 ≈ 0.5216
  L = 0.2126×0.6994 + 0.7152×0.6266 + 0.0722×0.5216
  L = 0.1487 + 0.4482 + 0.0377 = 0.6346

WCAG vs text-primary (#2B1F14):
  Ratio = (0.6346 + 0.05) / 0.0633 = 0.6846 / 0.0633 = 10.81:1 — Pass AAA ✓
WCAG vs text-disabled (#9E8E7E, L≈0.2912):
  Ratio = (0.6346 + 0.05) / (0.2912 + 0.05) = 0.6846 / 0.3412 = 2.01:1 — (disabled, intentionally low)
```

**Usage:** Disabled button states, skeleton loading placeholders, read-only input backgrounds, placeholder text containers. Warm gray that preserves palette harmony while signaling inactivity.

---

#### 8. `bg-inverted` — Espresso

```
Hex:  #2B1F14
RGB:  rgb(43, 31, 20)
HSL:  hsl(27, 36%, 12%)

Luminance:
  R_lin = (43/255)^2.2  = 0.169^2.2 ≈ 0.0213
  G_lin = (31/255)^2.2  = 0.122^2.2 ≈ 0.0106
  B_lin = (20/255)^2.2  = 0.0784^2.2 ≈ 0.0048
  L = 0.2126×0.0213 + 0.7152×0.0106 + 0.0722×0.0048
  L = 0.0045 + 0.0076 + 0.0003 = 0.0124

WCAG vs text-inverted (#F5EDD8, L≈0.8529):
  Ratio = (0.8529 + 0.05) / (0.0124 + 0.05) = 0.9029 / 0.0624 = 14.47:1 — Pass AAA ✓
WCAG vs brand-accent (#D4882A, L≈0.3237):
  Ratio = (0.3237 + 0.05) / (0.0124 + 0.05) = 0.3737 / 0.0624 = 5.99:1 — Pass AA ✓
```

**Usage:** Hero dark overlays, footer background, dark-section callouts, product packaging preview contexts, quote sections. The darkest possible color in the palette — rich espresso brown rather than flat black, maintaining warmth even in darkness.

---

#### 9. `text-primary` — Carbón

```
Hex:  #2B1F14
RGB:  rgb(43, 31, 20)
HSL:  hsl(27, 36%, 12%)
L = 0.0124 (same as bg-inverted — used as text on light, as bg on dark)

WCAG on bg-base (#F5EDD8, L=0.8529):
  Ratio = (0.8529 + 0.05) / (0.0124 + 0.05) = 14.47:1 — Pass AAA ✓
WCAG on white:
  Ratio = 1.05 / 0.0624 = 16.83:1 — Pass AAA ✓
```

**Usage:** All headings, all body copy, navigation links. The primary reading color. A very dark warm brown — not pure black (#000000) — maintaining the artisanal, non-digital warmth of the brand.

---

#### 10. `text-secondary` — Humus

```
Hex:  #5C4A35
RGB:  rgb(92, 74, 53)
HSL:  hsl(30, 27%, 28%)

Luminance:
  R_lin = (92/255)^2.2  ≈ 0.1075
  G_lin = (74/255)^2.2  = 0.290^2.2 ≈ 0.0663
  B_lin = (53/255)^2.2  = 0.208^2.2 ≈ 0.0330
  L = 0.2126×0.1075 + 0.7152×0.0663 + 0.0722×0.0330
  L = 0.0229 + 0.0474 + 0.0024 = 0.0727

WCAG on bg-base (#F5EDD8, L=0.8529):
  Ratio = (0.8529 + 0.05) / (0.0727 + 0.05) = 0.9029 / 0.1227 = 7.36:1 — Pass AAA ✓
WCAG on white:
  Ratio = 1.05 / 0.1227 = 8.56:1 — Pass AAA ✓
```

**Usage:** Supporting copy, meta information (date, origin, roast level), form labels, navigation secondary links, captions, product description supporting text.

---

#### 11. `text-disabled` — Arena

```
Hex:  #B0A08E
RGB:  rgb(176, 160, 142)
HSL:  hsl(30, 15%, 62%)

Luminance:
  R_lin = (176/255)^2.2 ≈ 0.4519
  G_lin = (160/255)^2.2 ≈ 0.3677
  B_lin = (142/255)^2.2 ≈ 0.2848
  L = 0.2126×0.4519 + 0.7152×0.3677 + 0.0722×0.2848
  L = 0.0961 + 0.2630 + 0.0206 = 0.3797

WCAG on bg-base (#F5EDD8, L=0.8529):
  Ratio = (0.8529 + 0.05) / (0.3797 + 0.05) = 0.9029 / 0.4297 = 2.10:1 — (disabled state, intentionally low contrast)
```

**Usage:** Disabled form fields, inactive tabs, unavailable product variants, placeholder text in inputs. Intentionally does not meet AA — users need to perceive these as unavailable. Warm sandy tone rather than gray, preserving palette harmony.

---

#### 12. `text-inverted` — Pergamino

```
Hex:  #F5EDD8
RGB:  rgb(245, 237, 216)
HSL:  hsl(40, 68%, 90%)
L = 0.8529 (same as bg-base)

WCAG on bg-inverted (#2B1F14, L=0.0124):
  Ratio = 14.47:1 — Pass AAA ✓
WCAG on brand-primary (#C1603A, L=0.2040):
  Ratio = (0.8529 + 0.05) / (0.2040 + 0.05) = 0.9029 / 0.254 = 3.55:1 — Pass AA Large / UI
  Note: Used at large display sizes on terracota backgrounds only
```

**Usage:** All text on `bg-inverted` sections, hero text on dark overlays, text on `brand-primary` buttons (at 18px+ only — ensure 3:1 minimum for button text). The warm crema makes text feel glowing rather than stark on dark backgrounds.

---

#### 13. `success` — Laurel

```
Hex:  #3D6B44
RGB:  rgb(61, 107, 68)
HSL:  hsl(127, 27%, 33%)

Luminance:
  R_lin = (61/255)^2.2  = 0.239^2.2 ≈ 0.0443
  G_lin = (107/255)^2.2 = 0.420^2.2 ≈ 0.1459
  B_lin = (68/255)^2.2  = 0.267^2.2 ≈ 0.0544
  L = 0.2126×0.0443 + 0.7152×0.1459 + 0.0722×0.0544
  L = 0.0094 + 0.1044 + 0.0039 = 0.1177

WCAG on white:
  Ratio = 1.05 / (0.1177 + 0.05) = 1.05 / 0.1677 = 6.26:1 — Pass AA ✓
WCAG on bg-base:
  Ratio = (0.8529 + 0.05) / (0.1177 + 0.05) = 0.9029 / 0.1677 = 5.38:1 — Pass AA ✓
```

**Usage:** Order confirmed, subscription active, in-stock indicator, newsletter signup success, form validation success. A deeper, muted laurel green — relates to `brand-secondary` (olive) while clearly reading as "success" without being a jarring emerald.

---

#### 14. `warning` — Mostaza

```
Hex:  #B87A1A
RGB:  rgb(184, 122, 26)
HSL:  hsl(35, 75%, 41%)

Luminance:
  R_lin = (184/255)^2.2 = 0.722^2.2 ≈ 0.4976
  G_lin = (122/255)^2.2 = 0.478^2.2 ≈ 0.2007
  B_lin = (26/255)^2.2  = 0.102^2.2 ≈ 0.0076
  L = 0.2126×0.4976 + 0.7152×0.2007 + 0.0722×0.0076
  L = 0.1058 + 0.1435 + 0.0005 = 0.2498

WCAG on white:
  Ratio = 1.05 / (0.2498 + 0.05) = 1.05 / 0.2998 = 3.50:1 — Pass AA Large / UI
WCAG on bg-base:
  Ratio = (0.8529 + 0.05) / (0.2498 + 0.05) = 0.9029 / 0.2998 = 3.01:1 — Pass AA Large
WCAG on bg-inverted:
  Ratio = (0.2498 + 0.05) / (0.0124 + 0.05) = 0.2998 / 0.0624 = 4.80:1 — Pass AA ✓
```

**Usage:** Low-stock alerts ("Quedan 3 unidades"), pending order status, subscription expiring soon. Use with text-primary label on bg-base for accessible warning banners. Darker mustard rather than bright yellow — stays within palette register.

---

#### 15. `error` — Arcilla Roja

```
Hex:  #A8341E
RGB:  rgb(168, 52, 30)
HSL:  hsl(10, 70%, 39%)

Luminance:
  R_lin = (168/255)^2.2 = 0.659^2.2 ≈ 0.4099
  G_lin = (52/255)^2.2  = 0.204^2.2 ≈ 0.0313
  B_lin = (30/255)^2.2  = 0.118^2.2 ≈ 0.0098
  L = 0.2126×0.4099 + 0.7152×0.0313 + 0.0722×0.0098
  L = 0.0871 + 0.0224 + 0.0007 = 0.1102

WCAG on white:
  Ratio = 1.05 / (0.1102 + 0.05) = 1.05 / 0.1602 = 6.55:1 — Pass AA ✓
WCAG on bg-base (#F5EDD8):
  Ratio = (0.8529 + 0.05) / (0.1102 + 0.05) = 0.9029 / 0.1602 = 5.64:1 — Pass AA ✓
```

**Usage:** Payment failed, form validation errors, out-of-stock, critical alerts. A darker, more muted red-terracota — clearly reads as "error" while remaining harmonious with brand-primary (terracota) at a glance. Not a jarring web-red.

---

#### 16. `info` — Azul Pizarra

```
Hex:  #3A5C78
RGB:  rgb(58, 92, 120)
HSL:  hsl(207, 35%, 35%)

Luminance:
  R_lin = (58/255)^2.2  = 0.227^2.2 ≈ 0.0413
  G_lin = (92/255)^2.2  = 0.361^2.2 ≈ 0.1075
  B_lin = (120/255)^2.2 = 0.471^2.2 ≈ 0.1946
  L = 0.2126×0.0413 + 0.7152×0.1075 + 0.0722×0.1946
  L = 0.0088 + 0.0769 + 0.0141 = 0.0998

WCAG on white:
  Ratio = 1.05 / (0.0998 + 0.05) = 1.05 / 0.1498 = 7.01:1 — Pass AA ✓ (Pass AAA for large)
WCAG on bg-base (#F5EDD8):
  Ratio = (0.8529 + 0.05) / (0.0998 + 0.05) = 0.9029 / 0.1498 = 6.03:1 — Pass AA ✓
```

**Usage:** Shipping info banners, informational tooltips, newsletter content updates, tracking information, educational coffee content callouts. A blue that reads as the Mediterranean sea in late afternoon — not a cold corporate blue, but a warm slate blue that complements the terracota palette.

---

### Palette Summary Table

| Role | Hex | L | On bg-base | On white | On #2B1F14 |
|---|---|---|---|---|---|
| brand-primary | #C1603A | 0.2040 | 3.68:1 AA-L | 4.13:1 AA-L | 5.08:1 AA |
| brand-secondary | #5C6B3E | 0.1306 | 4.99:1 AA | 5.81:1 AA | 3.61:1 AA-L |
| brand-accent | #D4882A | 0.3237 | 2.41:1 Fail | 2.81:1 Fail | 5.99:1 AA |
| brand-neutral | #9E8E7E | 0.2912 | 2.64:1 Fail* | 3.08:1 AA-L | 6.82:1 AAA |
| bg-base | #F5EDD8 | 0.8529 | — | 1.23:1 base | 14.47:1 AAA |
| bg-subtle | #EDE3CC | 0.7868 | 1.22:1 base | 1.39:1 | 13.22:1 AAA |
| bg-muted | #D9CFBE | 0.6346 | 1.56:1 base | 1.66:1 | 10.81:1 AAA |
| bg-inverted | #2B1F14 | 0.0124 | 14.47:1 AAA | 16.83:1 AAA | — |
| text-primary | #2B1F14 | 0.0124 | 14.47:1 AAA | 16.83:1 AAA | — |
| text-secondary | #5C4A35 | 0.0727 | 7.36:1 AAA | 8.56:1 AAA | 1.93:1 |
| text-disabled | #B0A08E | 0.3797 | 2.10:1 (intent) | 2.47:1 | 7.65:1 |
| text-inverted | #F5EDD8 | 0.8529 | — | — | 14.47:1 AAA |
| success | #3D6B44 | 0.1177 | 5.38:1 AA | 6.26:1 AA | 3.75:1 AA-L |
| warning | #B87A1A | 0.2498 | 3.01:1 AA-L | 3.50:1 AA-L | 4.80:1 AA |
| error | #A8341E | 0.1102 | 5.64:1 AA | 6.55:1 AA | 3.49:1 AA-L |
| info | #3A5C78 | 0.0998 | 6.03:1 AA | 7.01:1 AA | 3.32:1 AA-L |

*decorative use only, never for text

AA = passes WCAG AA normal text (≥4.5:1)  
AA-L = passes WCAG AA large text / UI components (≥3:1)  
AAA = passes WCAG AAA (≥7:1)  
Fail = decorative/graphic use only

---

## PHASE 4 — Typography System

### Font Pairing Rationale

The typographic system must embody the Creator archetype's tension between craft mastery and everyday warmth. The display/heading font must carry heritage and artisanal character without feeling museum-dusty. The body font must be supremely readable — the Everyman aspect demands accessibility over decoration.

---

### Primary Font — Headings & Display

```
Font:          Cormorant Garamond
Role:          Display / Heading (H1–H4, pull quotes, product names)
Source:        Google Fonts (free, open source)
Weights:use:   Light (300), Regular (400), Medium (500), SemiBold (600), Italic variants of 300 & 400
Fallback stack: 'Cormorant Garamond', 'Garamond', 'Georgia', 'Times New Roman', serif
Variable font: Yes (Cormorant has variable axis) — load with font-display: swap
Character:     High contrast thick/thin strokes, long elegant descenders, ink-trap details
                that photograph beautifully at large sizes. Distinctly Mediterranean / Southern
                European in feel — French/Italian book typography. Carries warmth, craft, and
                the hand of a type designer.
Performance:   Subset to Latin + Latin Extended A for Spanish/Catalan; reduces file size ~60%
```

**Why Cormorant Garamond over alternatives:**
- Playfair Display: too ubiquitous in specialty coffee (overcrowded territory per Phase 1 audit)
- EB Garamond: slightly drier and more academic; Cormorant has more expression in the italics
- Libre Baskerville: too conservative, lacks the high-contrast sparkle
- Fraunces: too quirky — skews Jester archetype rather than Creator

---

### Secondary Font — Body & UI

```
Font:          DM Sans
Role:          Body copy, UI labels, navigation, captions, overlines, form elements
Source:        Google Fonts (free, variable font)
Weights in use: Regular (400), Medium (500), SemiBold (600)
Fallback stack: 'DM Sans', 'Inter', 'Helvetica Neue', 'Arial', sans-serif
Variable font: Yes — single file covers all weights
Character:     Low-contrast humanist sans-serif with slightly rounded terminals.
                Warm, approachable, unpretentious. Designed specifically for digital screens.
                The humanist quality echoes the serif's warmth without mimicking it.
Performance:   Variable font — single load for all weights; subset Latin + Latin Extended A
```

**Why DM Sans over alternatives:**
- Inter: too neutral/corporate — works against the Everyman warmth
- Neue Haas Grotesk: licensed cost; overkill for this budget tier
- Nunito: too round/playful — undermines Creator precision
- Source Sans 3: excellent fallback but slightly mechanical; DM Sans has more warmth

---

### Pairing Logic

Cormorant Garamond's high-contrast classical serifs create the "artisanal excellence" register at display sizes. DM Sans grounds this in accessibility and modern screen legibility. The contrast between the two is the brand's core tension — craft heritage meeting contemporary life — made typographic.

---

### Type Scale — Fluid Responsive (clamp values)

**Formula used:**  
`clamp(min, preferred, max)`  
Preferred value = `calc(base + scale * 1vw)` linearized between 375px (mobile) and 1440px (desktop).  
Viewport coefficient = (desktop_px - mobile_px) / (1440 - 375) * 100  
Base = mobile_px - coefficient * (375/100)

All values verified to produce correct sizes at 375px viewport = min, 1440px viewport = max.

---

#### `text-display`
Desktop target: 88px | Mobile target: 48px

```
Viewport coefficient = (88 - 48) / (1440 - 375) × 100 = 40 / 1065 × 100 = 3.756vw
Base value at 375px = 48 - 3.756 × 3.75 = 48 - 14.085 = 33.915px ≈ 33.92px

CSS:
--text-display: clamp(3rem, 2.12rem + 3.756vw, 5.5rem);
/* 48px at 375px viewport; 88px at 1440px viewport */

font-family: 'Cormorant Garamond', serif;
font-weight: 300;          /* Light italic for hero; 400 for hero titles */
line-height: 1.05;
letter-spacing: -0.02em;
```

Usage: Single hero headline per page. The brand name in large format. Campaign statements.

---

#### `text-heading-1`
Desktop target: 64px | Mobile target: 36px

```
Coefficient = (64 - 36) / 1065 × 100 = 28 / 1065 × 100 = 2.629vw
Base = 36 - 2.629 × 3.75 = 36 - 9.859 = 26.141px ≈ 26.14px

CSS:
--text-heading-1: clamp(2.25rem, 1.635rem + 2.629vw, 4rem);
/* 36px at 375px; 64px at 1440px */

font-family: 'Cormorant Garamond', serif;
font-weight: 400;
line-height: 1.1;
letter-spacing: -0.015em;
```

Usage: Page titles (category pages, about page, blog post titles).

---

#### `text-heading-2`
Desktop target: 48px | Mobile target: 28px

```
Coefficient = (48 - 28) / 1065 × 100 = 20 / 1065 × 100 = 1.878vw
Base = 28 - 1.878 × 3.75 = 28 - 7.043 = 20.957px ≈ 20.96px

CSS:
--text-heading-2: clamp(1.75rem, 1.31rem + 1.878vw, 3rem);
/* 28px at 375px; 48px at 1440px */

font-family: 'Cormorant Garamond', serif;
font-weight: 500;
line-height: 1.15;
letter-spacing: -0.01em;
```

Usage: Section headings within pages, product collection titles, editorial section breaks.

---

#### `text-heading-3`
Desktop target: 36px | Mobile target: 22px

```
Coefficient = (36 - 22) / 1065 × 100 = 14 / 1065 × 100 = 1.315vw
Base = 22 - 1.315 × 3.75 = 22 - 4.931 = 17.069px ≈ 17.07px

CSS:
--text-heading-3: clamp(1.375rem, 1.067rem + 1.315vw, 2.25rem);
/* 22px at 375px; 36px at 1440px */

font-family: 'Cormorant Garamond', serif;
font-weight: 600;
line-height: 1.2;
letter-spacing: -0.005em;
```

Usage: Blog subsections, product detail accordion titles, team member names, FAQ questions.

---

#### `text-heading-4`
Desktop target: 24px | Mobile target: 19px

```
Coefficient = (24 - 19) / 1065 × 100 = 5 / 1065 × 100 = 0.469vw
Base = 19 - 0.469 × 3.75 = 19 - 1.759 = 17.241px ≈ 17.24px

CSS:
--text-heading-4: clamp(1.1875rem, 1.078rem + 0.469vw, 1.5rem);
/* 19px at 375px; 24px at 1440px */

font-family: 'Cormorant Garamond', serif;
font-weight: 600;
line-height: 1.25;
letter-spacing: 0;
```

Usage: Card titles, product names in listing, sidebar headings, widget titles.

---

#### `text-body-lg`
Desktop target: 20px | Mobile target: 17px

```
Coefficient = (20 - 17) / 1065 × 100 = 3 / 1065 × 100 = 0.282vw
Base = 17 - 0.282 × 3.75 = 17 - 1.056 = 15.944px ≈ 15.94px

CSS:
--text-body-lg: clamp(1.0625rem, 0.997rem + 0.282vw, 1.25rem);
/* 17px at 375px; 20px at 1440px */

font-family: 'DM Sans', sans-serif;
font-weight: 400;
line-height: 1.65;
letter-spacing: 0;
```

Usage: Lead paragraphs (intro text under H1), product description first paragraph, homepage value proposition text.

---

#### `text-body`
Desktop target: 16px | Mobile target: 16px (fixed)

```
CSS:
--text-body: clamp(1rem, 1rem + 0vw, 1rem);
/* Simplified: 1rem (16px) at all viewports */
/* Or simply: font-size: 1rem; */

font-family: 'DM Sans', sans-serif;
font-weight: 400;
line-height: 1.65;
letter-spacing: 0.01em;
```

Usage: Standard body copy, product descriptions, blog content paragraphs, legal/policy pages.

---

#### `text-body-sm`
Desktop target: 14px | Mobile target: 14px (fixed)

```
CSS:
--text-body-sm: clamp(0.875rem, 0.875rem + 0vw, 0.875rem);
/* 14px fixed */

font-family: 'DM Sans', sans-serif;
font-weight: 400;
line-height: 1.6;
letter-spacing: 0.01em;
```

Usage: Supplementary product information, ingredient lists, navigation subcopy, footnotes, review body text.

---

#### `text-caption`
Desktop target: 13px | Mobile target: 12px

```
Coefficient = (13 - 12) / 1065 × 100 = 1 / 1065 × 100 = 0.094vw
Base = 12 - 0.094 × 3.75 = 12 - 0.352 = 11.648px ≈ 11.65px

CSS:
--text-caption: clamp(0.75rem, 0.728rem + 0.094vw, 0.8125rem);
/* 12px at 375px; 13px at 1440px */

font-family: 'DM Sans', sans-serif;
font-weight: 400;
line-height: 1.5;
letter-spacing: 0.02em;
```

Usage: Image captions, product photo alt descriptors, date/time stamps, form helper text, price subtext (e.g., "IVA incluido").

---

#### `text-overline`
Desktop target: 12px | Mobile target: 11px

```
Coefficient = (12 - 11) / 1065 × 100 = 0.094vw
Base = 11 - 0.094 × 3.75 = 11 - 0.352 = 10.648px

CSS:
--text-overline: clamp(0.6875rem, 0.665rem + 0.094vw, 0.75rem);
/* 11px at 375px; 12px at 1440px */

font-family: 'DM Sans', sans-serif;
font-weight: 600;
line-height: 1.4;
letter-spacing: 0.12em;    /* Wide tracking: essential for overlines to read as category label */
text-transform: uppercase;
```

Usage: Category eyebrow labels ("ORIGEN · ETIOPÍA"), product type tags ("CAFÉ DE ESPECIALIDAD"), section markers, badge text, breadcrumb current level.

---

### Complete Type Scale Reference

| Token | CSS clamp() | Mobile | Desktop | Font | Weight | LH |
|---|---|---|---|---|---|---|
| text-display | clamp(3rem, 2.12rem + 3.756vw, 5.5rem) | 48px | 88px | Cormorant | 300 | 1.05 |
| text-heading-1 | clamp(2.25rem, 1.635rem + 2.629vw, 4rem) | 36px | 64px | Cormorant | 400 | 1.1 |
| text-heading-2 | clamp(1.75rem, 1.31rem + 1.878vw, 3rem) | 28px | 48px | Cormorant | 500 | 1.15 |
| text-heading-3 | clamp(1.375rem, 1.067rem + 1.315vw, 2.25rem) | 22px | 36px | Cormorant | 600 | 1.2 |
| text-heading-4 | clamp(1.1875rem, 1.078rem + 0.469vw, 1.5rem) | 19px | 24px | Cormorant | 600 | 1.25 |
| text-body-lg | clamp(1.0625rem, 0.997rem + 0.282vw, 1.25rem) | 17px | 20px | DM Sans | 400 | 1.65 |
| text-body | 1rem (fixed) | 16px | 16px | DM Sans | 400 | 1.65 |
| text-body-sm | 0.875rem (fixed) | 14px | 14px | DM Sans | 400 | 1.6 |
| text-caption | clamp(0.75rem, 0.728rem + 0.094vw, 0.8125rem) | 12px | 13px | DM Sans | 400 | 1.5 |
| text-overline | clamp(0.6875rem, 0.665rem + 0.094vw, 0.75rem) | 11px | 12px | DM Sans | 600 | 1.4 |

---

### Typography Rules

1. **Never set Cormorant Garamond below 18px** — the high-contrast strokes lose legibility at small sizes
2. **Heading hierarchy is visual, not just size** — use the weight progression (300→400→500→600) to create rhythm
3. **Cormorant Italic** is a design element, used for: product origin names, pull quotes, Italian/Spanish culinary terms, the brand tagline
4. **DM Sans 400 is the reading workhorse** — resist the urge to use 500/600 for body copy
5. **Overline text always uppercase + 0.12em tracking** — never set overlines in lowercase
6. **Line lengths:** body copy max-width: 68–72ch for optimal legibility
7. **Minimum body size:** 16px (text-body) — never below on any device

---

## PHASE 5 — Image & Visual Style

```
## Image Style Guide — Café Vermut

PHOTOGRAPHY STYLE: Documentary / Reportage
Not staged. Not studio. Camera catches the real: the barista's hands
pressing the puck, the steam rising from the portafilter, a customer
reading mid-sip with morning light from the window behind them.

COLOR TREATMENT: Warm-toned, natural
  - Color temperature: 5200–5800K range (golden hour indoor light)
  - Shadows: warm (never cool/blue)
  - Highlights: preserve texture; no blown-out whites
  - Slight film-grain overlay (3–5% opacity) at large sizes to feel analog
  - No heavy filtration, no desaturation, no artificial fading
  - Reference: the photography of René Redzepi's books; Granger & Co. brand imagery

COMPOSITION RULES:
  - Rule of thirds: subject placement off-center
  - Macro detail: coffee grounds texture, crema surface, hands around a cup
  - Environmental: the café interior with depth and context, not isolated
  - Natural negative space: the warm crema tones of bg-base echo in the photography
  - Avoid: heavy vignetting, artificial blur, compositing, white studio backgrounds

SUBJECTS (what to photograph):
  - The hands of the person making the coffee (process, craft, intentionality)
  - Coffee origins: raw beans, packaging, terroir landscapes (if accessible)
  - The cup in context: on the marble bar, on a worn wooden table, against tile
  - People mid-moment: reading, talking, thinking — never posed smiling at camera
  - The product (ground coffee bag) in real environments, not isolated on white
  - Barcelona specificity: Gràcia tiles, morning light through iron-grille windows

WHAT TO AVOID:
  - Stock photography of any kind
  - People looking directly into camera smiling
  - Pure white studio product photography (ecommerce exception: product cards only,
    shot on bg-base crema background #F5EDD8, not white)
  - Heavy Instagram-style presets (warm orange skin tones, faded blacks)
  - Perfectly clean/empty environments — some lived-in texture is required
  - Coffee latte art (signals mass-market specialty, not origin-focused craft)

ASPECT RATIOS:
  - 16:9 — Hero sections, full-width editorial banners, video content
  - 3:4 — Product detail main image, blog post featured image, team portraits
  - 1:1 — Social media, product grid thumbnails, small card images
  - 21:9 — Cinematic section dividers (optional, impactful)
  - 4:3 — Blog listing cards, process photo galleries

ICONOGRAPHY STYLE:
  - Style: Outlined, 1.5px stroke, rounded end caps
  - Grid: 24px base, scale to 16px/32px/48px
  - Personality: Simple and functional, occasional hand-drawn quality on decorative icons
    (e.g., a coffee bean, a compass rose for origin maps)
  - Never use filled icons and outlined icons in the same context

ILLUSTRATION STYLE:
  - Minimal use — brand photography carries the visual weight
  - Exception: origin maps (hand-illustrated cartographic style, one color + bg-base)
  - Exception: roast level indicator (simple 4-step graphic, custom drawn)
  - No character illustration; no isometric; no abstract shapes for decoration
```

---

## PHASE 6 — Logo Development

Three directions are presented. The approved visual direction (Artesanía Mediterránea) governs all three — they differ in how they resolve the Creator archetype and Mediterranean rootedness, not in their fundamental palette or tone.

---

### Logo Direction A — "Vermut" (Wordmark + Mark)

**Concept:** The name itself is the mark. "Vermut" is set in Cormorant Garamond at display size, with a simple geometric sun/aperture mark derived from the cross-section of a coffee bean viewed from above.

**Style:** Combination mark — abstract geometric symbol + serif wordmark

**Tone:** Refined artisan. The typeface carries the Creator archetype's intellectual elegance; the geometric mark adds a distinctive recall point. The bean-aperture symbol has enough abstraction to not feel illustrative or naive, but enough legibility at small sizes to work as a standalone app icon.

**Color application:**
- Primary version: Symbol in `brand-primary` (#C1603A), wordmark in `text-primary` (#2B1F14), on `bg-base` (#F5EDD8)
- Reversed version: Symbol in `brand-accent` (#D4882A) at large size, wordmark in `text-inverted` (#F5EDD8), on `bg-inverted` (#2B1F14)
- Monochrome: Full black or full white; the symbol's geometry holds without color
- Green variant: Symbol in `brand-secondary` (#5C6B3E) for sustainability/origin contexts

**Proportions:** Symbol is approximately 1:1 square; when combined with wordmark horizontally, total width:height ratio ≈ 4.5:1. Stacked version (symbol above wordmark): 1.8:1.

**Construction notes:**
- The symbol: Take a perfect circle. Inscribe a second circle at 70% diameter. Connect the two with 6 equidistant radial cuts. The resulting form reads simultaneously as a coffee bean cross-section, an aperture, and a Mediterranean rosette/wheel. No curves in the cuts — clean straight radial lines. The inner circle is a slightly heavier weight than the outer ring.
- "VERMUT" in wordmark: set at 120 tracking (letter-spacing), full caps, Cormorant Garamond Light (300) to match the geometric delicacy of the mark. "Café" appears above in DM Sans 400 at approximately 0.35× the wordmark cap height, 200 tracking, uppercase — establishing the hierarchy.
- Exclusion zone: 1× the height of the "V" character on all sides.

**What it communicates:** At first glance: precision and refinement. On closer inspection: warmth and Mediterranean craft. The geometric mark avoids the over-illustrated aesthetic common in craft coffee; it rewards attention without demanding it.

---

### Logo Direction B — "La Coma" (Custom Ligature Wordmark)

**Concept:** A bespoke wordmark where the letters of "Vermut" are drawn with custom ligatures — specifically the V-e junction creates a visual tension resolved by a custom ligature that echoes the shape of a coffee drip. No separate symbol. The wordmark IS the mark.

**Style:** Custom wordmark — heavily modified Cormorant Garamond letterforms with 2–3 custom ligatures

**Tone:** Craft obsession. Every detail is intentional. This direction leans hardest into the Creator archetype — a wordmark this custom signals that someone spent time perfecting it. Slightly more premium/exclusive register than Direction A.

**Color application:**
- Primary: Full `text-primary` (#2B1F14) on `bg-base` (#F5EDD8)
- Accent version: `brand-primary` (#C1603A) on `bg-base` for color contexts
- Reversed: `text-inverted` (#F5EDD8) on `bg-inverted` (#2B1F14)
- No two-color version — the wordmark's complexity makes multi-color redundant

**Proportions:** Pure horizontal wordmark, width:height approximately 7:1 at standard set. "Café" is set above left-aligned in DM Sans, small, tracking 300. No stacked version possible (too wide at legible sizes).

**Construction notes:**
- Start from Cormorant Garamond Regular (400) uppercase "VERMUT"
- V letterform: extend the left diagonal 12% downward; add a hairline terminal serif at the bottom vertex that curves right, creating implicit motion into the "E"
- V-E junction: Remove the standard gap between letters and draw a custom connecting stroke — a slight upward arc from V's right diagonal descends into E's top arm. This creates a single continuous mark VE that reads as two letters but feels like one gesture.
- R-M junction: Standard; the leg of the R points toward the M's left foot — keep the natural kinetic energy.
- Terminal on T: Extend the top arm 15% right, end with a slight horizontal serif that echoes the V's hairline. Creates bookending symmetry.
- "Café" above: Cormorant Garamond Italic 300, 0.32× cap height, left-aligned to V. The italic signals warmth and leans into the Spanish/Catalan cultural context.

**What it communicates:** First read: a brand that sweats the details. This is not a logo made in a template. The ligatures communicate hand and thought. Slightly harder to reproduce at very small sizes (below 14px, the ligatures lose definition) — requires a simplified version for app icon use.

---

### Logo Direction C — "El Segell" (Stamp/Seal Mark)

**Concept:** A circular seal — the traditional format of craft producer stamps, wine labels, and artisan certification marks. "CAFÉ VERMUT" runs around the circular path; the center contains a single dominant mark (a stylized coffee bean, hand-contoured, not geometric).

**Style:** Emblem / badge mark — all elements contained within a circle

**Tone:** Heritage craft meets contemporary Barcelona. The seal format references the olive oil producers, wine estates, and preserved goods markets of the Catalan tradition. Familiar to the local audience without being nostalgic or retro.

**Color application:**
- Primary: Full `brand-primary` (#C1603A) on `bg-base` — the entire seal in single color
- Premium version: `bg-inverted` (#2B1F14) outer ring + `brand-primary` bean center on `bg-base`
- Foil/emboss implication: The construction works as a blind emboss on kraft paper for physical coffee bags — the seal becomes part of the packaging system, not just digital branding.
- Reversed: White/`text-inverted` on `bg-inverted` for dark contexts

**Proportions:** Perfect circle. When used as a standalone mark: 1:1. When accompanied by a horizontal wordmark lockup beside it: circle at 1:1 + wordmark at 1.2:1 height, total combo ≈ 3.2:1 width:height.

**Construction notes:**
- Outer ring: Bold circle stroke, 4px weight at 240px diameter baseline. 
- Text ring: "CAFÉ VERMUT" set in DM Sans 600, uppercase, 80 tracking, 12px at 240px reference size, running clockwise from 9 o'clock to 3 o'clock (top arc). "BARCELONA · GRÀCIA" runs the bottom arc, same spec. A small diamond separator at 9 o'clock and 3 o'clock divides the two text arcs.
- Center mark: A coffee bean silhouette, drawn with a smooth outer contour (not geometric — deliberately organic). The bean has a single center crease line. At 120px tall centered in the circle. The contour weight matches the outer ring stroke weight. The crease line is the only interior stroke — hairline weight, 1px.
- Inner ring: A thin rule at 80% of outer ring diameter, 1px weight, creates separation between text ring and center mark.
- Clear space: The diameter of the seal, divided by 4, on all sides.

**What it communicates:** Instant recognition — the seal format communicates product quality, producer pride, and tactile craft before the eye reads a single word. Works exceptionally well for the physical coffee bag product sold online — the entire brand translates to packaging without redesign.

---

## PHASE 7 — Brand Guidelines Document

---

# Café Vermut — Brand Guidelines v1.0

*Barcelona, 2026 | Agencia Web v2 — Design*

---

## 1. Brand Identity

### Mission
Café Vermut és el ritual diari convertit en artesania. Som una cafeteria de especialitat a Gràcia, Barcelona, que torrem i moem café amb la mateixa cura que un artesà treballa el seu ofici. Cada tassa, cada bossa, és el resultat d'una cadena de decisions precises — de l'origen a la teva taula.

*(English: Café Vermut is the daily ritual elevated to craft. We are a specialty coffee café in Gràcia, Barcelona, roasting and grinding coffee with the same care a craftsperson brings to their trade. Every cup, every bag, is the result of a precise chain of decisions — from origin to your table.)*

### Values
- **Craftsmanship over speed:** We slow down to get it right
- **Rootedness:** This is a Barcelona café, specifically a Gràcia café — not a global brand
- **Honesty in the cup:** Single-origin, traceable, no blends that hide behind complexity

### Brand Archetypes
- **Primary: The Creator** — precision, craft mastery, originality, investment of self in the work
- **Secondary: The Everyman** — neighborhood warmth, accessibility, genuine belonging

### Voice & Tone

| Adjective | What it means | What it never is |
|---|---|---|
| Calorós (Warm) | First person, inviting, conversational in Catalan/Spanish | Stiff, formal, corporate |
| Precís (Precise) | Specific origin details, process described accurately | Jargon-heavy, inaccessible, showing off |
| Arrelat (Rooted) | Barcelona-specific, seasonal, neighborhood-aware | Generic, globally aspirational, anywhere-brand |

---

## 2. Logo

### Recommended Direction
**Direction A** (Symbol + Wordmark) is recommended for its versatility across digital and physical touchpoints. It provides a standalone mark (the bean-aperture symbol) for app icons and small contexts, plus the full combination for display sizes.

*(Direction C, the seal, is recommended as a secondary mark specifically for product packaging — the coffee bag design should feature the seal as the dominant element.)*

### Primary Version
Symbol in `brand-primary` (#C1603A) + wordmark "VERMUT" in `text-primary` (#2B1F14) + "Café" in `text-secondary` (#5C4A35) — on `bg-base` (#F5EDD8) or white.

### Reversed Version
Symbol in `brand-accent` (#D4882A) + full wordmark in `text-inverted` (#F5EDD8) — on `bg-inverted` (#2B1F14).

### Monochrome Versions
- Positive monochrome: all elements in `text-primary` (#2B1F14)
- Negative monochrome: all elements in `text-inverted` (#F5EDD8)
- Avoid grayscale intermediates — use only the two monochrome extremes

### Minimum Size
- Combination mark (horizontal): 120px wide / 28mm print
- Symbol mark alone: 24px / 8mm print
- Below these sizes: use symbol mark only, never the full combination

### Clear Space
- Digital: 1× the height of the "V" character on all four sides
- Print: equivalent to 1/4 of the symbol mark diameter on all sides

### Forbidden Uses
- Do not rotate the logo
- Do not apply drop shadows or glows
- Do not stretch or distort proportions
- Do not place on patterned or photographic backgrounds without a solid color buffer
- Do not change the symbol fill to a gradient
- Do not use brand-accent (#D4882A) as the symbol color on light backgrounds (fails AA)
- Do not recreate the logo in any other typeface

---

## 3. Color System

See Phase 3 above for complete 16-role palette with hex values, WCAG ratios, and usage guidelines.

### Quick Reference — Most-Used Colors

| Context | Background | Text | Accent |
|---|---|---|---|
| Standard page | #F5EDD8 | #2B1F14 | #C1603A |
| Cards / inputs | #EDE3CC | #2B1F14 | #5C6B3E |
| Dark sections | #2B1F14 | #F5EDD8 | #D4882A |
| Primary CTA button | #C1603A | #F5EDD8 (18px+) | — |
| Secondary CTA | transparent | #5C6B3E | border #5C6B3E |

### Accessibility Rules
- All body text (text-body, 16px) must use `text-primary` (#2B1F14) or `text-secondary` (#5C4A35) only
- `brand-primary` may only appear as text at 18px+ (large text AA threshold)
- `brand-accent` is decorative only — never as text on light backgrounds
- All interactive elements must meet 3:1 minimum against adjacent non-interactive elements (WCAG 1.4.11)

---

## 4. Typography

See Phase 4 above for full type scale, clamp() values, and rules.

### Font Loading (Next.js)

```javascript
// app/layout.tsx
import { Cormorant_Garamond, DM_Sans } from 'next/font/google'

const cormorant = Cormorant_Garamond({
  subsets: ['latin'],
  weight: ['300', '400', '500', '600'],
  style: ['normal', 'italic'],
  variable: '--font-cormorant',
  display: 'swap',
})

const dmSans = DM_Sans({
  subsets: ['latin'],
  weight: ['400', '500', '600'],
  variable: '--font-dm-sans',
  display: 'swap',
})
```

### CSS Custom Properties

```css
:root {
  --font-serif: var(--font-cormorant), 'Garamond', 'Georgia', serif;
  --font-sans:  var(--font-dm-sans), 'Inter', 'Helvetica Neue', sans-serif;

  --text-display:    clamp(3rem, 2.12rem + 3.756vw, 5.5rem);
  --text-heading-1:  clamp(2.25rem, 1.635rem + 2.629vw, 4rem);
  --text-heading-2:  clamp(1.75rem, 1.31rem + 1.878vw, 3rem);
  --text-heading-3:  clamp(1.375rem, 1.067rem + 1.315vw, 2.25rem);
  --text-heading-4:  clamp(1.1875rem, 1.078rem + 0.469vw, 1.5rem);
  --text-body-lg:    clamp(1.0625rem, 0.997rem + 0.282vw, 1.25rem);
  --text-body:       1rem;
  --text-body-sm:    0.875rem;
  --text-caption:    clamp(0.75rem, 0.728rem + 0.094vw, 0.8125rem);
  --text-overline:   clamp(0.6875rem, 0.665rem + 0.094vw, 0.75rem);
}
```

---

## 5. Imagery

See Phase 5 above for complete Image Style Guide.

### Image Quality Minimum Standards
- Print/large web: 2000px on longest side, 72dpi screen / 300dpi print
- Product card thumbnails: 800×800px minimum (1:1 ratio)
- Hero images: 2400px wide minimum, delivered as WebP + JPEG fallback
- All images: alt text required, descriptive (not "IMG_4521")

### Photography Do / Don't

| Do | Don't |
|---|---|
| Hands in motion, process shots | Smiling people looking at camera |
| Natural window light, warm tones | Studio white backgrounds (except product cards) |
| Barcelona / Gràcia context | Generic "coffee shop" stock images |
| Coffee in context (cup on tile table) | Isolated cup on white |
| Film grain texture overlay | Heavy Instagram filters |

---

## 6. Usage Examples

### Correct Applications
- H1 in Cormorant Garamond 400, 64px desktop, on #F5EDD8 background, color #2B1F14 — passes AAA
- CTA button: "Comprar ara" in DM Sans 600, 16px, on #C1603A background, text #F5EDD8 — 3.55:1 (AA Large; ensure button height ≥44px for WCAG 2.5.5)
- Product tag: "ORIGEN · ETIÒPIA" in DM Sans 600 uppercase, 11–12px, 120 tracking, color #5C6B3E on #EDE3CC — passes AA
- Pull quote: Cormorant Garamond Italic 300, 36–48px, color #C1603A on #F5EDD8 — passes AA Large
- Dark section headline: Cormorant Garamond 300, display size, color #F5EDD8 on #2B1F14 — passes AAA

### Common Mistakes to Avoid
- Using `brand-accent` (#D4882A) as text color on `bg-base` — ratio 2.41:1, fails AA
- Mixing Cormorant Garamond and a third display font — the system is a duet, not a trio
- Setting body copy in Cormorant Garamond — the high-contrast strokes cause fatigue at text size
- Using `error` (#A8341E) and `brand-primary` (#C1603A) adjacent without visual separation — too similar in hue, may cause confusion
- Placing the logo on a brand-primary background without testing — terracota-on-terracota loses distinction
- Using cool-toned photography — every image must maintain the warm 5200–5800K register

---

## 7. Asset Inventory

| File | Format | Usage |
|---|---|---|
| cafevermut-logo-primary.svg | SVG | Digital, light backgrounds |
| cafevermut-logo-reversed.svg | SVG | Digital, dark backgrounds |
| cafevermut-logo-monochrome-black.svg | SVG | Single-color print, stamps |
| cafevermut-logo-monochrome-white.svg | SVG | Single-color reversed print |
| cafevermut-symbol-only.svg | SVG | App icon, favicons, small contexts |
| cafevermut-seal-primary.svg | SVG | Product packaging, secondary brand mark |
| cafevermut-brand-colors.ase | Adobe ASE | Design tools color swatch |
| cafevermut-brand-colors.css | CSS | Developer implementation |
| cafevermut-typography-specimen.pdf | PDF | Type reference for designers |
| cafevermut-brand-guidelines.pdf | PDF | Full printed guidelines |

---

## Appendix: CSS Custom Properties — Complete Implementation

```css
:root {
  /* === FONTS === */
  --font-serif: var(--font-cormorant), 'Garamond', 'Georgia', serif;
  --font-sans:  var(--font-dm-sans), 'Inter', 'Helvetica Neue', sans-serif;

  /* === COLORS — Core Brand === */
  --color-brand-primary:    #C1603A;
  --color-brand-secondary:  #5C6B3E;
  --color-brand-accent:     #D4882A;
  --color-brand-neutral:    #9E8E7E;

  /* === COLORS — Backgrounds === */
  --color-bg-base:          #F5EDD8;
  --color-bg-subtle:        #EDE3CC;
  --color-bg-muted:         #D9CFBE;
  --color-bg-inverted:      #2B1F14;

  /* === COLORS — Text === */
  --color-text-primary:     #2B1F14;
  --color-text-secondary:   #5C4A35;
  --color-text-disabled:    #B0A08E;
  --color-text-inverted:    #F5EDD8;

  /* === COLORS — Semantic === */
  --color-success:          #3D6B44;
  --color-warning:          #B87A1A;
  --color-error:            #A8341E;
  --color-info:             #3A5C78;

  /* === TYPE SCALE === */
  --text-display:    clamp(3rem, 2.12rem + 3.756vw, 5.5rem);
  --text-heading-1:  clamp(2.25rem, 1.635rem + 2.629vw, 4rem);
  --text-heading-2:  clamp(1.75rem, 1.31rem + 1.878vw, 3rem);
  --text-heading-3:  clamp(1.375rem, 1.067rem + 1.315vw, 2.25rem);
  --text-heading-4:  clamp(1.1875rem, 1.078rem + 0.469vw, 1.5rem);
  --text-body-lg:    clamp(1.0625rem, 0.997rem + 0.282vw, 1.25rem);
  --text-body:       1rem;
  --text-body-sm:    0.875rem;
  --text-caption:    clamp(0.75rem, 0.728rem + 0.094vw, 0.8125rem);
  --text-overline:   clamp(0.6875rem, 0.665rem + 0.094vw, 0.75rem);

  /* === TYPE WEIGHTS === */
  --weight-light:    300;
  --weight-regular:  400;
  --weight-medium:   500;
  --weight-semibold: 600;

  /* === LINE HEIGHTS === */
  --lh-display:  1.05;
  --lh-heading:  1.1;
  --lh-subhead:  1.2;
  --lh-body:     1.65;
  --lh-caption:  1.5;
  --lh-overline: 1.4;
}
```

---

*End of Café Vermut Brand System — Phase 1–7*  
*Status: Ready for director review and client presentation*  
*Next step: Client selects logo direction → ui-designer begins component library implementation*
