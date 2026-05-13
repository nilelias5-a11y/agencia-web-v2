# Design System Governance Violations — La Nonna
## Audit Report · 2026-05-13

---

## Overview

This report documents four design system violations detected in the La Nonna frontend codebase, provides the corrected code for each, proposes governance rules to prevent recurrence, and explains the correct process for incorporating the new brand color introduced in violation 2.

---

## Violation 1 — Hardcoded brand color in `.cta-button`

### Description

The developer used the raw hex value `#8B0000` directly inside the component stylesheet instead of referencing the established CSS custom property.

**Severity:** Critical  
**Category:** Token bypass — color  
**Location:** CSS of `.cta-button` component

### Problematic code

```css
/* WRONG */
.cta-button {
  background-color: #8B0000;
  color: #ffffff;
}
```

### Why this is a violation

- Breaks single-source-of-truth for color. If the brand primary changes (e.g. a seasonal campaign, a rebrand), every hardcoded instance must be hunted down manually.
- Makes theming (dark mode, high-contrast accessibility mode) impossible at scale.
- The token `--color-brand-primary` already exists and encodes this exact value. Duplicating the raw value creates silent inconsistency risk.

### Correct code

```css
/* CORRECT */
.cta-button {
  background-color: var(--color-brand-primary);
  color: var(--color-text-on-brand);   /* never hardcode #fff either */
}
```

If the token file needs verifying:

```css
/* tokens/color.css (existing definition — do not change) */
:root {
  --color-brand-primary: #8B0000;
  --color-text-on-brand: #ffffff;
}
```

---

## Violation 2 — Undocumented color `#D4A000` for 'Novedad' badge

### Description

A new gold/amber color was introduced inline for a "Novedad" (New) badge without going through the token intake process. The color has no name, no semantic role, no contrast documentation, and no entry in the token system.

**Severity:** Critical  
**Category:** Token creation bypass — color  
**Location:** Badge component or inline style

### Problematic code

```css
/* WRONG */
.badge-novedad {
  background-color: #D4A000;
  color: #ffffff;
}
```

### Why this is a violation

- An unnamed color is an invisible dependency. The next developer has no way to know this color is intentionally part of the system.
- Contrast ratio of `#D4A000` on `#ffffff` text is approximately 2.5:1 — well below WCAG AA (4.5:1 for normal text). This is an accessibility failure.
- The color cannot be reused consistently across other "Novedad" touchpoints (email templates, print, social assets) without the raw hex being copied again.

### Correct code (after token is registered — see Section D)

```css
/* CORRECT — after the token has been formally added */
.badge-novedad {
  background-color: var(--color-badge-novedad-bg);
  color: var(--color-badge-novedad-text);
}
```

The proper process for adding this color is described in detail in Section D of this report.

---

## Violation 3 — Magic number `padding: 18px` in `.card-nonna`

### Description

The developer wrote a literal pixel value for padding instead of using the spacing token defined for component inner padding.

**Severity:** High  
**Category:** Token bypass — spacing  
**Location:** `.card-nonna` component stylesheet

### Problematic code

```css
/* WRONG */
.card-nonna {
  padding: 18px;
}
```

### Why this is a violation

- `18px` is a "magic number" — it exists in isolation with no semantic meaning.
- `var(--space-component-padding-y)` already encodes the correct vertical rhythm for component padding. Using the raw value defeats the token.
- If the design team decides to adjust spacing density globally (e.g. for a more compact mobile layout), the token can be updated in one place. The hardcoded `18px` will not follow.
- Padding is likely asymmetric (different on X and Y axes). Using the shorthand `padding: 18px` collapses both axes to the same value, which may not match the design intent.

### Correct code

```css
/* CORRECT */
.card-nonna {
  padding-block: var(--space-component-padding-y);
  padding-inline: var(--space-component-padding-x);
}

/* If the token truly covers all four sides uniformly: */
.card-nonna {
  padding: var(--space-component-padding-y) var(--space-component-padding-x);
}
```

If only a vertical token exists and horizontal padding is different, define the missing token rather than hardcoding:

```css
/* tokens/spacing.css — add if missing */
:root {
  --space-component-padding-y: 18px;   /* document the current value */
  --space-component-padding-x: 24px;   /* define the horizontal counterpart */
}
```

---

## Violation 4 — Inline `font-size: 15px` in hero paragraph

### Description

A font size was set using an inline style attribute directly on a paragraph element in the hero section, bypassing both the token system and the stylesheet architecture.

**Severity:** High  
**Category:** Token bypass — typography; inline style anti-pattern  
**Location:** Hero component, paragraph element (inline `style` attribute)

### Problematic code

```html
<!-- WRONG -->
<p style="font-size: 15px;">
  Bienvenidos a La Nonna, auténtica cocina italiana en el corazón de Gràcia.
</p>
```

### Why this is a violation

- Inline styles have the highest CSS specificity outside of `!important`, making them extremely difficult to override in responsive contexts or when the design system needs to update typography at scale.
- `15px` is not a standard step on most type scales. It falls between `--text-sm` (14px) and `--text-base` (16px), suggesting either a miscalculation or an attempt to achieve an intermediate size that the type scale intentionally does not support.
- Inline styles cannot be targeted by media queries. A responsive type scale (e.g. fluid type with `clamp()`) is impossible when the size is locked inline.
- Inline styles are invisible to design-system linters and style auditing tools.

### Correct code

```css
/* tokens/typography.css — existing definitions */
:root {
  --text-sm:   0.875rem;   /* 14px */
  --text-base: 1rem;       /* 16px */
  --text-lg:   1.125rem;   /* 18px */

  /* Semantic alias for hero body copy */
  --text-hero-body: var(--text-base);
}
```

```css
/* hero.css */
.hero__body {
  font-size: var(--text-hero-body);
  line-height: var(--leading-relaxed);
}
```

```html
<!-- CORRECT -->
<p class="hero__body">
  Bienvenidos a La Nonna, auténtica cocina italiana en el corazón de Gràcia.
</p>
```

If 15px is genuinely required by the design (not a mistake), the correct path is:

1. Open a design review with the designer to confirm this is intentional.
2. Add `--text-hero-body: 0.9375rem` (15px) to the token file with a comment explaining the exception.
3. Never encode it inline.

---

## Section C — Governance Rules to Prevent These Violations

### Rule 1: No raw color values in component stylesheets (linting)

Configure Stylelint with `stylelint-declaration-strict-value` to enforce that `color`, `background-color`, `border-color`, and `fill` properties may only accept CSS custom properties or `transparent`/`currentColor`.

```json
// .stylelintrc.json
{
  "plugins": ["stylelint-declaration-strict-value"],
  "rules": {
    "scale-unlimited/declaration-strict-value": [
      ["/color/", "fill", "stroke"],
      {
        "ignoreValues": ["transparent", "currentColor", "inherit", "none"],
        "message": "Use a CSS token (var(--color-*)) instead of a raw color value."
      }
    ]
  }
}
```

This rule causes CI to fail on any PR that introduces a raw hex, `rgb()`, or `hsl()` value for color properties.

### Rule 2: No raw spacing values for component padding/margin (linting)

Extend the same Stylelint rule to spacing properties:

```json
{
  "rules": {
    "scale-unlimited/declaration-strict-value": [
      ["padding", "padding-block", "padding-inline", "margin", "gap"],
      {
        "ignoreValues": ["0", "auto", "inherit"],
        "message": "Use a spacing token (var(--space-*)) instead of a raw pixel value."
      }
    ]
  }
}
```

### Rule 3: No inline styles (linting + PR review)

```json
// .eslintrc.json (React/JSX projects)
{
  "rules": {
    "react/forbid-component-props": [
      "error",
      { "forbid": ["style"] }
    ]
  }
}
```

For non-React HTML templates, add an HTMLHint rule or a custom Stylelint rule targeting inline styles.

### Rule 4: Token intake process for new values

Any new color, spacing step, or typographic size must be proposed via a Token Request before it appears in a component. The process:

1. **Design proposes** the new value in the design file (Figma/Canva) with a semantic name.
2. **Design systems team reviews** contrast, naming convention, and necessity.
3. **Token is added** to the relevant token file (`tokens/color.css`, `tokens/spacing.css`, etc.) in a dedicated PR — not bundled with a feature PR.
4. **Token PR is merged first.** Only after that may the feature PR reference `var(--new-token)`.

### Rule 5: PR checklist

Add a mandatory PR template checklist:

```markdown
## Design System Checklist
- [ ] No raw hex / rgb / hsl color values in CSS
- [ ] No raw pixel values for padding, margin, gap, or font-size
- [ ] No inline `style` attributes on HTML/JSX elements
- [ ] Any new visual property references an existing token
- [ ] If a new token was needed, it was added via the token intake process first
```

### Rule 6: Automated token coverage report in CI

Add a script that diffs the token file against the compiled CSS to report any value that appears in a component stylesheet but not in any token file. Run this as a non-blocking warning in CI initially, then promote to a blocking check after the team has resolved all existing violations.

### Rule 7: Design system office hours

Hold a 30-minute weekly slot where developers can bring edge cases ("I need a color that isn't in the system") before implementing their own solution. This prevents violations born from urgency rather than negligence.

---

## Section D — How to Correctly Incorporate `#D4A000` into the Token System

### Step 1: Verify the color with the designer

Before the color enters the system, confirm with the designer:
- Is `#D4A000` the final, approved value or a placeholder?
- What is the intended use: badges only, or also backgrounds, borders, icons?
- Is there a dark-mode variant?

### Step 2: Accessibility audit

Run a contrast check before committing the token:

| Combination | Ratio | WCAG AA (normal text) | WCAG AA (large text / UI) |
|---|---|---|---|
| `#D4A000` on `#ffffff` | ~2.5:1 | FAIL | FAIL |
| `#D4A000` on `#1a1a1a` | ~7.2:1 | PASS | PASS |
| `#ffffff` on `#D4A000` | ~2.5:1 | FAIL | FAIL |
| `#000000` on `#D4A000` | ~8.0:1 | PASS | PASS |

**Conclusion:** `#D4A000` cannot be used as a background with white text. Either:
- Use dark text (`#1a1a1a` or `#000000`) on the gold background, or
- Darken the color to approximately `#A07800`, which achieves AA compliance with white text (~4.6:1).

### Step 3: Assign a semantic name

Avoid naming tokens after their visual appearance (`--color-gold`). Name them after their role:

```
--color-badge-novedad-bg:   #D4A000;   /* or adjusted accessible value */
--color-badge-novedad-text: #1a1a1a;   /* confirmed accessible pairing */
```

If the color will also be used outside badges (e.g. promotional banners, icons), split into a primitive and a semantic alias:

```css
/* Primitive — describes the hue */
--color-gold-500: #D4A000;

/* Semantic aliases — describe the intent */
--color-badge-promo-bg:   var(--color-gold-500);
--color-promo-accent:     var(--color-gold-500);
```

### Step 4: Add to the token file in a dedicated PR

```css
/* tokens/color.css — add under the brand palette section */

/* --- Promotional / Badge palette --- */
--color-gold-500:          #D4A000;          /* Amber gold — promo accent */
--color-badge-novedad-bg:  var(--color-gold-500);
--color-badge-novedad-text: #1a1a1a;         /* Dark text for AA compliance */
```

Commit message format:
```
feat(tokens): add gold-500 and badge-novedad color tokens

Introduces the amber accent color approved in design review [link].
Accessibility: #D4A000 / #1a1a1a passes WCAG AA (7.2:1).
```

### Step 5: Update the Figma/Canva design file

The token must be reflected as a named color style in the design tool so designers also reference `color-badge-novedad-bg` by name, not by hex value. This closes the loop between design and code.

### Step 6: Use the token in the component

Only after the token PR is merged:

```css
/* badge.css */
.badge-novedad {
  background-color: var(--color-badge-novedad-bg);
  color: var(--color-badge-novedad-text);
  font-size: var(--text-xs);
  font-weight: var(--font-weight-semibold);
  padding: var(--space-badge-padding-y) var(--space-badge-padding-x);
  border-radius: var(--radius-badge);
  text-transform: uppercase;
  letter-spacing: var(--tracking-wide);
}
```

```html
<span class="badge-novedad">Novedad</span>
```

---

## Summary Table

| # | Location | Violation | Severity | Fix |
|---|---|---|---|---|
| 1 | `.cta-button` CSS | `#8B0000` hardcoded instead of `var(--color-brand-primary)` | Critical | Replace with token reference |
| 2 | `.badge-novedad` | `#D4A000` undocumented, no token, fails WCAG AA with white text | Critical | Run token intake process; fix contrast |
| 3 | `.card-nonna` CSS | `padding: 18px` magic number instead of `var(--space-component-padding-y)` | High | Replace with spacing tokens |
| 4 | Hero paragraph | `style="font-size: 15px"` inline style bypassing token and stylesheet | High | Move to class + typography token |

---

*Report generated by Design Systems Expert review · La Nonna · 2026-05-13*
