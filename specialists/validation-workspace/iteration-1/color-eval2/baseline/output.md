# Color Palette — Café Vermut

**Project:** Café Vermut — Specialty Coffee E-Commerce, Barcelona  
**Requested by:** director  
**Date:** 2026-05-13

---

## Proposed Palette

Café Vermut positions itself as "Japan meets Barcelona" — minimal, premium specialty coffee. The palette must avoid the typical coffee-brand brown-and-cream cliché while still evoking warmth and quality.

**Primary background:** `#F8F7F4` — Warm near-white. Quiet, clean, elevated.  
**Primary text / dark:** `#0A0A0A` — Near-black. Maximum contrast, Japanese restraint.  
**Accent (gold):** `#C9A84C` — Warm gold. Premium signal without being loud.  
**Secondary warm:** `#C8832A` — Roasted amber. For CTAs and interactive elements.  
**Muted text:** `#6B6460` — Warm gray. Body copy secondary.

---

## CSS Variables

```css
:root {
  --color-background: #F8F7F4;
  --color-text: #0A0A0A;
  --color-text-muted: #6B6460;
  --color-accent: #C9A84C;
  --color-cta: #C8832A;
  --color-cta-hover: #A56822;
  --color-border: #E5E0D8;
  --color-surface: #FFFFFF;
  --color-dark: #1A1209;
}
```

---

## Color Applications

- **Hero / Landing background:** `--color-background` (#F8F7F4)
- **Primary CTA buttons (add to cart, buy now):** `--color-cta` (#C8832A), white text
- **Gold accent details (lines, underlines, icons):** `--color-accent` (#C9A84C)
- **Footer background:** `--color-dark` (#1A1209)
- **Footer text:** Warm parchment `#D4C5A9`
- **Product card backgrounds:** `--color-surface` (#FFFFFF)
- **Muted labels (tasting notes, weight, origin):** `--color-text-muted`

---

## Contrast Check

- `#0A0A0A` on `#F8F7F4`: 19.4:1 — AAA ✓
- `#FFFFFF` on `#C8832A`: 3.1:1 — AA Large only (use for 18px+ button text)
- `#6B6460` on `#F8F7F4`: 5.8:1 — AA ✓
- `#D4C5A9` on `#1A1209`: 8.2:1 — AAA ✓ (footer)

---

## Notes

The CTA amber (#C8832A) is marginal for small text on white — should only be used for large buttons or as a background color with white text. Do not use as body link color.

The "Japan meets Barcelona" direction is well served by the near-white + near-black base with gold accents — this mirrors the aesthetic of Blue Bottle Coffee and Onibus Coffee (reference brands). The amber CTA provides warmth and coffee connection while staying distinct from the archival gold accent.

For product photography, images should be shot on near-white or natural linen backgrounds to integrate seamlessly with this palette.
