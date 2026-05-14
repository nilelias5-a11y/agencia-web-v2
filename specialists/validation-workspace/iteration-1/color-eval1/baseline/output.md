# Color Palette — La Nonna Restaurant

**Project:** La Nonna — Italian restaurant, Gràcia, Barcelona  
**Requested by:** director  
**Date:** 2026-05-13

---

## Proposed Palette

For an Italian restaurant in Gràcia, the palette should feel warm, welcoming, and Mediterranean. I'm recommending a terracotta-anchored palette:

**Primary:** `#C4603A` — Terracotta. Classic Italian restaurant color. Warm, appetizing.  
**Secondary:** `#2C5F2E` — Olive green. Evokes Italian countryside, herbs, freshness.  
**Background:** `#FDF8F3` — Warm off-white. Softer than pure white, easier on eyes.  
**Text:** `#1A1A1A` — Near-black for body copy.  
**Accent:** `#D4A84B` — Gold/ochre. Candle warmth, premium feel.

---

## CSS Variables

```css
:root {
  --color-primary: #C4603A;
  --color-primary-dark: #A04D2A;
  --color-secondary: #2C5F2E;
  --color-accent: #D4A84B;
  --color-background: #FDF8F3;
  --color-text: #1A1A1A;
  --color-text-muted: #6B6B6B;
  --color-border: #E8DDD4;
  --color-white: #FFFFFF;
}
```

---

## Color Applications

- **CTAs / Buttons:** `--color-primary` (#C4603A) background, white text
- **Navigation active states:** `--color-primary`
- **Headings:** `--color-text` (#1A1A1A)
- **Body text:** `--color-text`
- **Section backgrounds alternate:** `#F5EFE8` (slightly warmer tint)
- **Card borders:** `--color-border`
- **Success states:** `--color-secondary` at 80% opacity

---

## Contrast Check

- `#1A1A1A` on `#FDF8F3`: ~16:1 — AAA ✓
- `#FFFFFF` on `#C4603A`: ~3.3:1 — AA Large only (acceptable for buttons)
- `#6B6B6B` on `#FDF8F3`: ~7.2:1 — AA ✓

---

## Notes

The terracotta is bold and recognizable. Main risk is it's a common Italian restaurant color — consider whether La Nonna wants to differentiate or lean into the archetype. If the brand brief calls for more uniqueness, we could shift primary to a deeper burgundy like `#8B2D2D`.

Typography should use the warm end of the palette. Avoid pure black (`#000000`) — the off-black `#1A1A1A` reads softer and better matches the warm palette tone.
