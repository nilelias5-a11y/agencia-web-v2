# Design System Governance Report — La Nonna
## Design System Manager | Violation Audit

**Project:** La Nonna Restaurante, Gràcia, Barcelona
**Reported by:** Design System Manager
**Severity:** CRITICAL — 4 violations detected in frontend-developer implementation
**Date:** 2026-05-13
**Token source:** `globals.css` v1.0 (generated 2026-05-11)
**Action required:** Flagged to Director per HARMONY PROTOCOL

---

## PROTOCOL NOTICE

Per the **HARMONY PROTOCOL**, the `globals.css` and token documentation are the contract that `frontend-developer` must implement faithfully. Any deviation from the token system is flagged immediately to the `director`. All 4 violations below require correction before the component ships.

---

## VIOLATION AUDIT — 4 Violations Found

---

### Violation #1 — Raw Hex in .cta-button Color

**Severity:** CRITICAL
**Location:** Component `.cta-button`, `color` property

#### Code Submitted (Wrong)
```css
.cta-button {
  color: #8B0000;
}
```

#### Rule Broken
**Governance Rule 1:** *"Never use raw hex values in component CSS — always use tokens."*

The token system explicitly defines `--color-brand-primary` as the semantic token for the primary brand color. Using `#8B0000` (dark red) directly:

1. Bypasses the 3-level token architecture entirely (jumps from component to raw value, skipping Level 2 semantic token)
2. Hardcodes a value that is **not even in the primitive palette** — `#8B0000` does not appear in the La Nonna primitive tokens. The closest primitive is `--color-terracota-800: #57271A` or `--color-terracota-900: #381812`. This is an undocumented, unregistered color.
3. Creates an invisible dependency: dark mode, rebranding, or any palette update will not cascade to this component
4. Breaks the WCAG audit trail — no contrast ratio is documented for this color against any background

#### Corrected Code
```css
.cta-button {
  /* CORRECT: reference the semantic token */
  color: var(--color-brand-primary);
}
```

If the intent was a *darker* variant (e.g. on hover), use the existing semantic token:
```css
.cta-button:hover,
.cta-button:focus-visible {
  color: var(--color-brand-primary-dark);
}
```

**Note to developer:** `--color-brand-primary` resolves to `var(--color-terracota-500)` = `#C4603A`. If the design spec actually requires a color as dark as `#8B0000`, open a director-approval request — it is not in the current token system and cannot be used unilaterally.

---

### Violation #2 — Unregistered Color #D4A000 in .badge-novedad

**Severity:** CRITICAL
**Location:** New `.badge-novedad` component (or modifier)

#### Code Submitted (Wrong)
```css
.badge-novedad {
  background-color: #D4A000;
  /* or */
  color: #D4A000;
}
```

#### Rule Broken
**Governance Rule — What Requires Director Approval:** *"Adding a new color that is not in the token system."*
**Governance Rule 1:** *"Never use raw hex values in component CSS — always use tokens."*
**Governance Rule 3:** *"Never create a one-off value — if it's needed twice, it becomes a token."*

`#D4A000` (a saturated golden amber) is not defined anywhere in the token system. The closest existing token is `--color-brand-accent: var(--color-oro-500)` = `#C9953A`, which is the design-system-approved gold accent. Using `#D4A000` directly:

1. Creates a visual color that has no semantic meaning in the system
2. Cannot be reliably themed for dark mode
3. Has no documented contrast ratio for accessibility compliance
4. Bypasses director approval required for new color introduction

#### Corrected Code (using the existing accent token)
```css
/* CORRECT: Use the existing gold/amber semantic token */
.badge-novedad {
  background-color: var(--color-brand-accent);   /* #C9953A — oro-500 */
  color: var(--color-bg-inverse);                /* #2C1A0E — dark text for contrast */
  border-radius: var(--badge-border-radius);
  padding: var(--badge-padding);
  font-size: var(--text-body-xs);
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}
```

**If `#D4A000` must be kept:** See Section D — Proper Token Incorporation Process.

---

### Violation #3 — Magic Number Padding in .card-nonna

**Severity:** HIGH
**Location:** Component `.card-nonna`, `padding` property

#### Code Submitted (Wrong)
```css
.card-nonna {
  padding: 18px;
}
```

#### Rule Broken
**Governance Rule 1:** *"Never use raw hex values in component CSS — always use tokens."* (extends to all raw values, not just colors)
**Governance Rule 3:** *"Never create a one-off value — if it's needed twice, it becomes a token."*

`18px` is not a step in the design system's base-4 spacing scale. The spacing primitive scale is:
- `--space-4: 1rem (16px)`
- `--space-5: 1.25rem (20px)`
- `--space-6: 1.5rem (24px)`

The value `18px` falls between `--space-4` and `--space-5` — it is an off-scale "magic number" that fragments the spacing rhythm of the layout.

The design system defines `--card-padding: var(--space-6)` as the component-level token for card internal padding. This is the contract `.card-nonna` must reference.

#### Corrected Code
```css
/* CORRECT: reference the component token, which resolves to --space-6 (24px) */
.card-nonna {
  padding: var(--card-padding);
}
```

If asymmetric padding is required by the design spec:
```css
.card-nonna {
  padding-block: var(--card-padding);          /* 24px vertical */
  padding-inline: var(--space-component-padding-x, var(--space-6));  /* horizontal */
}
```

If the component genuinely needs a different padding from the standard card, the correct path is:
1. Define a component-scoped override token in the component's scope block in `globals.css`
2. Get approval from DSM (and director if it deviates from the type/spacing scale)

```css
/* CORRECT pattern for a project-specific card variant: */
.card-nonna {
  --card-nonna-padding: var(--space-5);  /* 20px — closest valid scale step */
  padding: var(--card-nonna-padding);
}
```

---

### Violation #4 — Inline Font Size in Hero Paragraph

**Severity:** HIGH
**Location:** Hero component, inline style on `<p>` element

#### Code Submitted (Wrong)
```html
<p style="font-size: 15px">...</p>
```
or in CSS:
```css
.hero p {
  font-size: 15px;
}
```

#### Rule Broken
**Governance Rule — What Requires Director Approval:** *"Deviating from the type scale."*
**Governance Rule 1:** *"Never use raw values in component CSS — always use tokens."*

`15px` is not a step in the La Nonna type scale. The type scale is:
- `--text-body-sm: var(--size-sm)` = `0.875rem` = **14px**
- `--text-body-md: var(--size-md)` = `1rem` = **16px**

`15px` sits between two defined steps. Additionally, using `px` units for font-size is an accessibility anti-pattern — it overrides user browser font-size preferences. The design system uses `rem` throughout.

Inline styles (`style="..."`) are categorically forbidden in the design system — they carry the highest CSS specificity, cannot be overridden by component tokens, and are invisible to dark mode or theme cascades.

#### Corrected Code
```html
<!-- CORRECT: remove inline style, apply text utility class -->
<p class="text-body-lg">...</p>
```

```css
/* If a specific class is needed in the hero: */
.hero__lead {
  font-size: var(--text-body-lg);    /* 18px — intended for hero lead copy */
  line-height: 1.7;
  color: var(--color-text-secondary);
}
```

Or if 14px body-small is appropriate:
```css
.hero__caption {
  font-size: var(--text-body-sm);    /* 14px */
  color: var(--color-text-disabled);
}
```

**Rule:** If the design genuinely requires 15px (an off-scale value), this requires director approval and a new primitive token `--size-[15]` followed by a semantic assignment.

---

## Summary Table

| # | Location | Violation | Rule Broken | Severity |
|---|---|---|---|---|
| 1 | `.cta-button { color: #8B0000 }` | Raw hex, unregistered color | Rule 1: no raw values | CRITICAL |
| 2 | `.badge-novedad` uses `#D4A000` | Unregistered color, no token, no director approval | Rule 1 + Director approval | CRITICAL |
| 3 | `.card-nonna { padding: 18px }` | Off-scale magic number, bypasses `--card-padding` | Rule 1 + Rule 3 | HIGH |
| 4 | `<p style="font-size: 15px">` | Inline style, off-scale font size, px unit | Rule 1 + Type scale deviation | HIGH |

---

## Section C — Governance Rules That Would Have Prevented This

The following rules, had they been enforced at the start of Phase 3, would have caught all four violations at PR review time or earlier:

### Rule G-01: Mandatory Token Reference Linting (Pre-Commit)
**What it does:** A pre-commit or CI lint step that scans all CSS/TSX files for:
- Raw hex patterns (`#[0-9A-Fa-f]{3,6}`)
- Raw pixel values in spacing/font-size properties
- `style=` attributes on JSX elements

**Implementation:**
```json
// .stylelintrc.json
{
  "plugins": ["stylelint-no-hardcoded-colors"],
  "rules": {
    "plugin/no-hardcoded-colors": true,
    "declaration-property-value-disallowed-list": {
      "font-size": ["/^[0-9]+px$/"],
      "padding": ["/^[0-9]+px$/"],
      "margin": ["/^[0-9]+px$/"]
    }
  }
}
```

**Would have caught:** Violations #1, #2, #3, #4

---

### Rule G-02: New Color Registration Protocol
**What it does:** No color value — whether brand, feedback, or decorative — may appear in any component file unless it is:
1. Registered as a primitive token in the Level 1 block of `globals.css`
2. Assigned a semantic token in the Level 2 block
3. Approved by the Director if it expands the palette

**Trigger:** Any commit introducing a new CSS custom property of category `--color-*` requires a DSM review comment documenting:
- The primitive hex value
- The semantic token name
- The purpose and the component(s) using it
- Contrast ratio against surfaces it will appear on

**Would have caught:** Violations #1, #2

---

### Rule G-03: Spacing Token Enforcement — No Off-Scale Values
**What it does:** The spacing scale is `base-4` (multiples of 4px). All padding, margin, gap, and inset values must reference a `--space-*` primitive or a semantic/component token derived from it. Any value not on the scale (`18px`, `7px`, `13px`, etc.) is automatically rejected.

**The test:** `18 ÷ 4 = 4.5` — not a whole number. Rejected.

**Exception path:** If a new spacing step is genuinely needed, the developer opens a token request to the DSM. The DSM evaluates whether `--space-4` (16px) or `--space-5` (20px) can satisfy the design intent, or whether a new primitive step is warranted.

**Would have caught:** Violation #3

---

### Rule G-04: No Inline Styles Policy
**What it does:** `style=` attributes are banned in all production JSX/HTML. Zero exceptions. Inline styles:
- Override the entire token cascade
- Break dark mode
- Are unauditable for accessibility
- Cannot be overridden without `!important`

If a developer needs a "one-off" style for a specific element, the correct path is a dedicated class in the component CSS file that references a token.

**Enforcement:** ESLint rule `react/forbid-component-props` or a custom rule targeting `style` prop on all non-whitelisted elements.

**Would have caught:** Violation #4

---

### Rule G-05: Pre-Ship Component Checklist
Before any new or modified component is merged, the developer must self-audit:

```markdown
## Component Token Checklist
- [ ] Zero raw hex values — all colors reference `var(--color-*)`
- [ ] Zero magic number spacing — all padding/margin reference `var(--space-*)` or component tokens
- [ ] Zero inline `style=` attributes
- [ ] Font sizes from type scale only: `var(--text-*)`
- [ ] New colors (if any) registered in globals.css with director approval documented
- [ ] Contrast ratio documented for text-on-background combinations
- [ ] Dark mode behavior verified (semantic tokens swap automatically)
```

---

## Section D — How to Incorporate #D4A000 Correctly

If after director review the decision is made to **keep `#D4A000`** as a distinct color (rather than using the existing `--color-brand-accent: #C9953A`), the correct integration path follows the 3-level token architecture:

### Step 1 — Justify the Need (Director Approval)

Before any code change, document:
- **Why `#C9953A` is insufficient** — what visual distinction does `#D4A000` provide that the existing accent does not?
- **Where it will be used** — only the "Novedad" badge, or potentially other UI elements?
- **Contrast ratio** — `#D4A000` on `#FAF6F0` (page bg): approximately 2.8:1 — **FAILS WCAG AA** for normal text. Must be used as a large text or decorative element only, or paired with dark text on top.

### Step 2 — Register as a Primitive Token (Level 1)

Add to the `/* Color primitives */` block in `globals.css`:

```css
/* globals.css — TIER 1 PRIMITIVE TOKENS */
:root {
  /* ... existing primitives ... */

  /* Oro scale — expanded */
  --color-oro-400: #D4A000;   /* saturated gold — Novedad badge */
  --color-oro-500: #c9953a;   /* existing oro */
}
```

Naming convention: `--color-[family]-[shade]`. Since `#D4A000` is more saturated/lighter than `#C9953A`, it logically belongs at `oro-400` (lighter = lower number in the scale convention used by the system).

### Step 3 — Register as a Semantic Token (Level 2)

Add to the `/* TIER 2 — SEMANTIC TOKENS */` block:

```css
/* globals.css — TIER 2 SEMANTIC TOKENS */
:root {
  /* ... existing semantic tokens ... */

  /* Badge — Novedad variant */
  --color-badge-novedad-bg:   var(--color-oro-400);       /* #D4A000 */
  --color-badge-novedad-text: var(--color-crema-900);     /* #2C1A0E — required for contrast */
}
```

### Step 4 — Register as a Component Token (Level 3)

Add to the `/* TIER 3 — COMPONENT TOKENS */` block:

```css
/* globals.css — TIER 3 COMPONENT TOKENS */
:root {
  /* ... existing component tokens ... */

  /* Badge — Novedad */
  --badge-novedad-bg:           var(--color-badge-novedad-bg);
  --badge-novedad-text:         var(--color-badge-novedad-text);
  --badge-novedad-border-radius: var(--badge-border-radius);  /* reuse existing */
  --badge-novedad-padding:      var(--badge-padding);          /* reuse existing */
}
```

### Step 5 — Write the Component Using Only Level 3 Tokens

```css
/* components.css */
.badge--novedad {
  background-color: var(--badge-novedad-bg);
  color:            var(--badge-novedad-text);
  border-radius:    var(--badge-novedad-border-radius);
  padding:          var(--badge-novedad-padding);
  font-size:        var(--text-body-xs);
  font-weight:      700;
  letter-spacing:   0.06em;
  text-transform:   uppercase;
}
```

### Step 6 — Document Dark Mode Behavior

In the `[data-theme="dark"]` block (if dark mode is implemented), remap the semantic token:

```css
[data-theme="dark"] {
  /* The oro-400 is too bright on dark backgrounds; remap to a softer value */
  --color-badge-novedad-bg:   var(--color-oro-500);     /* fallback to existing accent */
  --color-badge-novedad-text: var(--color-crema-900);
}
```

### Step 7 — Document Contrast Audit

| Text on Background | Foreground | Background | Ratio | WCAG AA | WCAG AAA |
|---|---|---|---|---|---|
| Badge text on badge bg | `#2C1A0E` | `#D4A000` | ~7.2:1 | Pass | Pass |
| Badge text on page bg (if badge is transparent) | `#D4A000` | `#FAF6F0` | ~2.8:1 | Fail | Fail |

**Conclusion on contrast:** `#D4A000` can only be used as a **background color** with dark text (`#2C1A0E`) on top. It must **never** be used as text color on light backgrounds.

---

## Complete Corrected CSS — Summary

```css
/* ============================================================
   CORRECTED VIOLATIONS — La Nonna
   All four violations resolved to use token system correctly
   ============================================================ */


/* --- VIOLATION #1 FIX: .cta-button — use semantic color token --- */
.cta-button {
  /* BEFORE (wrong): color: #8B0000; */
  color: var(--color-brand-primary);          /* --color-terracota-500: #C4603A */
}

.cta-button:hover,
.cta-button:focus-visible {
  color: var(--color-brand-primary-dark);     /* --color-terracota-600: #9E4A28 */
}


/* --- VIOLATION #2 FIX: .badge-novedad — use token (existing accent or new registered token) --- */

/* Option A: Use existing accent token (no new color needed) */
.badge--novedad {
  /* BEFORE (wrong): background-color: #D4A000; */
  background-color: var(--color-brand-accent);      /* --color-oro-500: #C9953A */
  color:            var(--color-bg-inverse);         /* --color-crema-900: #2C1A0E */
  border-radius:    var(--badge-border-radius);
  padding:          var(--badge-padding);
  font-size:        var(--text-body-xs);
  font-weight:      700;
  letter-spacing:   0.06em;
  text-transform:   uppercase;
}

/* Option B: After director approval and proper token registration (see Section D) */
.badge--novedad {
  background-color: var(--badge-novedad-bg);         /* → --color-oro-400: #D4A000 */
  color:            var(--badge-novedad-text);        /* → --color-crema-900: #2C1A0E */
  border-radius:    var(--badge-novedad-border-radius);
  padding:          var(--badge-novedad-padding);
  font-size:        var(--text-body-xs);
  font-weight:      700;
  letter-spacing:   0.06em;
  text-transform:   uppercase;
}


/* --- VIOLATION #3 FIX: .card-nonna — use component padding token --- */
.card-nonna {
  /* BEFORE (wrong): padding: 18px; */
  padding: var(--card-padding);               /* → --space-6: 1.5rem (24px) */

  /* If the standard card-padding doesn't fit the design,
     use the closest on-scale value as a component-scoped override: */
  /* --card-nonna-padding: var(--space-5); */ /* → 1.25rem (20px) — requires DSM approval */
  /* padding: var(--card-nonna-padding); */
}


/* --- VIOLATION #4 FIX: Hero paragraph — remove inline style, use type scale token --- */
/*
  BEFORE (wrong):
  <p style="font-size: 15px">...</p>

  CORRECT HTML:
  <p class="hero__lead">...</p>
*/

.hero__lead {
  /* BEFORE (wrong): font-size: 15px (inline) */
  font-size:   var(--text-body-lg);      /* --size-lg: 1.125rem (18px) — hero lead copy */
  line-height: 1.7;
  color:       var(--color-text-secondary);
}

/* Alternative if body-md (16px) is more appropriate: */
.hero__body {
  font-size: var(--text-body-md);        /* --size-md: 1rem (16px) */
  color:     var(--color-text-secondary);
}
```

---

## Escalation Summary for Director

Four violations have been detected in the `frontend-developer` implementation for La Nonna. Two are CRITICAL (raw hex values, including an unregistered color) and two are HIGH (magic-number spacing, inline style with off-scale font size).

**Immediate actions required:**
1. **Block merge** of the affected components until violations are corrected
2. **Decision needed:** approve or reject `#D4A000` as a new palette entry (see Section D for proper incorporation path if approved)
3. **Process action:** implement the 5 governance rules from Section C before Phase 3 continues — particularly Stylelint token enforcement and the no-inline-styles ESLint rule

None of the four violations are acceptable under the current token contract. All have clear, low-effort corrections documented above.
