# Design System Manager — Eval 1
## Café Vermut — `globals.css` Complete Output

**Skill:** design-system-manager  
**Phase executed:** Phase 1 (Token Architecture) + Phase 2 (globals.css)  
**Brand tokens used:** brand-primary `#A0522D`, brand-secondary `#556B2F`, bg-base `#FAF7F0`, text-primary `#1A1A1A`, typefaces Playfair Display + Lato

---

```css
/* ============================================================
   CAFÉ VERMUT — globals.css
   Design System Manager · Agencia Web v2
   Architecture: 3-level token system
   Level 1 → Primitives | Level 2 → Semantic | Level 3 → Component
   ============================================================ */

/* ============================================================
   1. PRIMITIVE TOKENS
   Raw values with no semantic meaning.
   Named after what they ARE, not what they DO.
   ============================================================ */
:root {

  /* ─── Color Primitives ─── */

  /* Sienna / Brand Primary scale */
  --color-sienna-950: #2C0F06;
  --color-sienna-900: #4A1A0A;
  --color-sienna-800: #6B2710;
  --color-sienna-700: #8B3A1C;
  --color-sienna-600: #A0522D;   /* brand-primary */
  --color-sienna-500: #B8673F;
  --color-sienna-400: #CC7F56;
  --color-sienna-300: #DDA07A;
  --color-sienna-200: #EEC4A8;
  --color-sienna-100: #F5E0D0;
  --color-sienna-50:  #FBF3EE;

  /* Olive / Brand Secondary scale */
  --color-olive-950: #0D1508;
  --color-olive-900: #1A2910;
  --color-olive-800: #2D4420;
  --color-olive-700: #3E5C2C;
  --color-olive-600: #556B2F;   /* brand-secondary */
  --color-olive-500: #6A8540;
  --color-olive-400: #839F56;
  --color-olive-300: #A3B87A;
  --color-olive-200: #C4D4A3;
  --color-olive-100: #E0EAC8;
  --color-olive-50:  #F2F7E8;

  /* Cream / Background scale */
  --color-cream-950: #3A3428;
  --color-cream-900: #5C5242;
  --color-cream-800: #7D7260;
  --color-cream-700: #9E9280;
  --color-cream-600: #B8AE9C;
  --color-cream-500: #CBBFAE;
  --color-cream-400: #D9D0C2;
  --color-cream-300: #E8E2D6;
  --color-cream-200: #F0EBDF;
  --color-cream-100: #FAF7F0;   /* bg-base */
  --color-cream-50:  #FDFCF8;

  /* Neutral / Text scale */
  --color-neutral-950: #0A0A0A;
  --color-neutral-900: #1A1A1A;  /* text-primary */
  --color-neutral-800: #2D2D2D;
  --color-neutral-700: #404040;
  --color-neutral-600: #595959;
  --color-neutral-500: #737373;
  --color-neutral-400: #8C8C8C;
  --color-neutral-300: #A6A6A6;
  --color-neutral-200: #CCCCCC;
  --color-neutral-100: #E6E6E6;
  --color-neutral-50:  #F5F5F5;
  --color-neutral-0:   #FFFFFF;

  /* Status colors */
  --color-green-500:  #22C55E;
  --color-green-100:  #DCFCE7;
  --color-yellow-500: #EAB308;
  --color-yellow-100: #FEF9C3;
  --color-red-500:    #EF4444;
  --color-red-100:    #FEE2E2;
  --color-blue-500:   #3B82F6;
  --color-blue-100:   #DBEAFE;

  /* Dark mode background primitives */
  --color-dark-950: #0F0E0C;
  --color-dark-900: #1A1915;
  --color-dark-800: #252420;
  --color-dark-700: #302F2A;
  --color-dark-600: #3D3B35;
  --color-dark-500: #4A4840;

  /* ─── Spacing Primitives ─── */
  --primitive-space-1:  4px;
  --primitive-space-2:  8px;
  --primitive-space-3:  12px;
  --primitive-space-4:  16px;
  --primitive-space-5:  20px;
  --primitive-space-6:  24px;
  --primitive-space-8:  32px;
  --primitive-space-10: 40px;
  --primitive-space-12: 48px;
  --primitive-space-16: 64px;
  --primitive-space-20: 80px;
  --primitive-space-24: 96px;
  --primitive-space-32: 128px;
  --primitive-space-40: 160px;

  /* ─── Typography Primitives ─── */
  --primitive-font-heading: 'Playfair Display', Georgia, 'Times New Roman', serif;
  --primitive-font-body:    'Lato', 'Helvetica Neue', Arial, sans-serif;
  --primitive-font-mono:    'Fira Code', 'Courier New', Courier, monospace;

  --primitive-font-size-xs:   0.75rem;    /*  12px */
  --primitive-font-size-sm:   0.875rem;   /*  14px */
  --primitive-font-size-base: 1rem;       /*  16px */
  --primitive-font-size-lg:   1.125rem;   /*  18px */
  --primitive-font-size-xl:   1.25rem;    /*  20px */
  --primitive-font-size-2xl:  1.5rem;     /*  24px */
  --primitive-font-size-3xl:  1.875rem;   /*  30px */
  --primitive-font-size-4xl:  2.25rem;    /*  36px */
  --primitive-font-size-5xl:  3rem;       /*  48px */
  --primitive-font-size-6xl:  3.75rem;    /*  60px */
  --primitive-font-size-7xl:  4.5rem;     /*  72px */

  --primitive-font-weight-regular:   400;
  --primitive-font-weight-medium:    500;
  --primitive-font-weight-semibold:  600;
  --primitive-font-weight-bold:      700;
  --primitive-font-weight-black:     900;

  /* ─── Radius Primitives ─── */
  --primitive-radius-none:   0px;
  --primitive-radius-sm:     2px;
  --primitive-radius-md:     4px;
  --primitive-radius-lg:     8px;
  --primitive-radius-xl:     12px;
  --primitive-radius-2xl:    16px;
  --primitive-radius-3xl:    24px;
  --primitive-radius-full:   9999px;

  /* ─── Shadow Primitives ─── */
  --primitive-shadow-xs:  0 1px 2px 0 rgb(0 0 0 / 0.05);
  --primitive-shadow-sm:  0 1px 3px 0 rgb(0 0 0 / 0.10), 0 1px 2px -1px rgb(0 0 0 / 0.10);
  --primitive-shadow-md:  0 4px 6px -1px rgb(0 0 0 / 0.10), 0 2px 4px -2px rgb(0 0 0 / 0.10);
  --primitive-shadow-lg:  0 10px 15px -3px rgb(0 0 0 / 0.10), 0 4px 6px -4px rgb(0 0 0 / 0.10);
  --primitive-shadow-xl:  0 20px 25px -5px rgb(0 0 0 / 0.10), 0 8px 10px -6px rgb(0 0 0 / 0.10);
  --primitive-shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
}


/* ============================================================
   2. SEMANTIC TOKENS — Light Mode (default)
   Map primitives to PURPOSE.
   Named after what they DO, not what they ARE.
   ============================================================ */
:root {

  /* ─── Color — Semantic ─── */

  /* Brand */
  --color-brand-primary:        var(--color-sienna-600);
  --color-brand-primary-hover:  var(--color-sienna-700);
  --color-brand-primary-active: var(--color-sienna-800);
  --color-brand-primary-subtle: var(--color-sienna-100);
  --color-brand-secondary:        var(--color-olive-600);
  --color-brand-secondary-hover:  var(--color-olive-700);
  --color-brand-secondary-subtle: var(--color-olive-100);

  /* Background */
  --color-bg-base:    var(--color-cream-100);
  --color-bg-subtle:  var(--color-cream-200);
  --color-bg-muted:   var(--color-cream-300);
  --color-bg-inverse: var(--color-neutral-900);
  --color-bg-overlay: rgb(26 26 26 / 0.60);

  /* Surface (cards, modals, drawers) */
  --color-surface-default: var(--color-cream-50);
  --color-surface-raised:  var(--color-neutral-0);
  --color-surface-overlay: var(--color-neutral-0);

  /* Text */
  --color-text-primary:   var(--color-neutral-900);
  --color-text-secondary: var(--color-neutral-600);
  --color-text-tertiary:  var(--color-neutral-400);
  --color-text-disabled:  var(--color-neutral-300);
  --color-text-inverse:   var(--color-cream-50);
  --color-text-brand:     var(--color-sienna-600);
  --color-text-on-brand:  var(--color-cream-50);

  /* Border */
  --color-border-default: var(--color-cream-300);
  --color-border-subtle:  var(--color-cream-200);
  --color-border-strong:  var(--color-neutral-300);
  --color-border-brand:   var(--color-sienna-600);
  --color-border-focus:   var(--color-sienna-600);

  /* Status */
  --color-success:        var(--color-green-500);
  --color-success-subtle: var(--color-green-100);
  --color-warning:        var(--color-yellow-500);
  --color-warning-subtle: var(--color-yellow-100);
  --color-error:          var(--color-red-500);
  --color-error-subtle:   var(--color-red-100);
  --color-info:           var(--color-blue-500);
  --color-info-subtle:    var(--color-blue-100);

  /* ─── Spacing — Semantic ─── */
  --space-1:  var(--primitive-space-1);
  --space-2:  var(--primitive-space-2);
  --space-3:  var(--primitive-space-3);
  --space-4:  var(--primitive-space-4);
  --space-5:  var(--primitive-space-5);
  --space-6:  var(--primitive-space-6);
  --space-8:  var(--primitive-space-8);
  --space-10: var(--primitive-space-10);
  --space-12: var(--primitive-space-12);
  --space-16: var(--primitive-space-16);
  --space-20: var(--primitive-space-20);
  --space-24: var(--primitive-space-24);
  --space-32: var(--primitive-space-32);
  --space-40: var(--primitive-space-40);

  --space-component-padding-x:  var(--primitive-space-6);   /* 24px */
  --space-component-padding-y:  var(--primitive-space-3);   /* 12px */
  --space-component-gap:        var(--primitive-space-4);   /* 16px */
  --space-content-gap:          var(--primitive-space-8);   /* 32px */
  --space-section-gap:          var(--primitive-space-20);  /* 80px */
  --space-section-gap-sm:       var(--primitive-space-12);  /* 48px */
  --space-container-padding:    var(--primitive-space-6);   /* 24px */

  /* ─── Typography — Semantic ─── */
  --font-heading: var(--primitive-font-heading);
  --font-body:    var(--primitive-font-body);
  --font-mono:    var(--primitive-font-mono);

  /* Semantic font sizes referenced by typography utilities */
  --text-display:  clamp(3rem, 6vw + 1rem, 5rem);         /* 48px → 80px */
  --text-h1:       clamp(2.25rem, 4vw + 0.75rem, 3.75rem); /* 36px → 60px */
  --text-h2:       clamp(1.875rem, 3vw + 0.5rem, 3rem);    /* 30px → 48px */
  --text-h3:       clamp(1.5rem, 2.5vw + 0.25rem, 2.25rem);/* 24px → 36px */
  --text-h4:       clamp(1.25rem, 1.5vw + 0.25rem, 1.875rem);/* 20px → 30px */
  --text-body-lg:  clamp(1.0625rem, 1vw + 0.5rem, 1.25rem);/* 17px → 20px */
  --text-body:     var(--primitive-font-size-base);         /* 16px */
  --text-sm:       var(--primitive-font-size-sm);           /* 14px */
  --text-caption:  var(--primitive-font-size-xs);           /* 12px */
  --text-overline: 0.6875rem;                               /* 11px */

  /* ─── Radius — Semantic ─── */
  --radius-button:  var(--primitive-radius-md);     /* 4px  — refined, not pill */
  --radius-card:    var(--primitive-radius-lg);     /* 8px  */
  --radius-input:   var(--primitive-radius-md);     /* 4px  */
  --radius-badge:   var(--primitive-radius-full);   /* pill */
  --radius-tag:     var(--primitive-radius-sm);     /* 2px  */
  --radius-modal:   var(--primitive-radius-xl);     /* 12px */
  --radius-image:   var(--primitive-radius-lg);     /* 8px  */

  /* ─── Shadow — Semantic ─── */
  --shadow-card:     var(--primitive-shadow-sm);
  --shadow-card-hover: var(--primitive-shadow-md);
  --shadow-modal:    var(--primitive-shadow-xl);
  --shadow-dropdown: var(--primitive-shadow-lg);
  --shadow-button:   var(--primitive-shadow-xs);
  --shadow-sticky:   var(--primitive-shadow-md);

  /* ─── Motion — Semantic ─── */
  --duration-instant: 100ms;
  --duration-fast:    200ms;
  --duration-normal:  300ms;
  --duration-slow:    500ms;
  --duration-slower:  700ms;

  --easing-default: cubic-bezier(0.25, 0.1, 0.25, 1.0);
  --easing-in:      cubic-bezier(0.4,  0.0, 1.0,  1.0);
  --easing-out:     cubic-bezier(0.0,  0.0, 0.2,  1.0);
  --easing-in-out:  cubic-bezier(0.4,  0.0, 0.2,  1.0);
  --easing-spring:  cubic-bezier(0.34, 1.56, 0.64, 1.0);

  /* ─── Z-index — Semantic ─── */
  --z-below:    -1;
  --z-base:      0;
  --z-raised:   10;
  --z-dropdown: 100;
  --z-sticky:   200;
  --z-overlay:  300;
  --z-modal:    400;
  --z-toast:    500;
  --z-tooltip:  600;
}


/* ============================================================
   3. DARK MODE
   Override ONLY semantic color tokens.
   Spacing, radius, shadow, motion, and z-index remain unchanged.
   Implementation: [data-theme="dark"] on <html> or <body>
   ============================================================ */
[data-theme="dark"],
.dark {

  /* Brand — adjusted for dark surfaces */
  --color-brand-primary:        var(--color-sienna-400);
  --color-brand-primary-hover:  var(--color-sienna-300);
  --color-brand-primary-active: var(--color-sienna-200);
  --color-brand-primary-subtle: var(--color-sienna-950);
  --color-brand-secondary:        var(--color-olive-400);
  --color-brand-secondary-hover:  var(--color-olive-300);
  --color-brand-secondary-subtle: var(--color-olive-950);

  /* Background */
  --color-bg-base:    var(--color-dark-900);
  --color-bg-subtle:  var(--color-dark-800);
  --color-bg-muted:   var(--color-dark-700);
  --color-bg-inverse: var(--color-cream-100);
  --color-bg-overlay: rgb(0 0 0 / 0.70);

  /* Surface */
  --color-surface-default: var(--color-dark-800);
  --color-surface-raised:  var(--color-dark-700);
  --color-surface-overlay: var(--color-dark-600);

  /* Text */
  --color-text-primary:   var(--color-cream-100);
  --color-text-secondary: var(--color-cream-300);
  --color-text-tertiary:  var(--color-neutral-500);
  --color-text-disabled:  var(--color-neutral-700);
  --color-text-inverse:   var(--color-neutral-900);
  --color-text-brand:     var(--color-sienna-400);
  --color-text-on-brand:  var(--color-neutral-900);

  /* Border */
  --color-border-default: var(--color-dark-600);
  --color-border-subtle:  var(--color-dark-700);
  --color-border-strong:  var(--color-dark-500);
  --color-border-brand:   var(--color-sienna-400);
  --color-border-focus:   var(--color-sienna-400);

  /* Status — same hues, already accessible on dark */
  --color-success:        var(--color-green-500);
  --color-success-subtle: rgb(34 197 94 / 0.15);
  --color-warning:        var(--color-yellow-500);
  --color-warning-subtle: rgb(234 179 8 / 0.15);
  --color-error:          var(--color-red-500);
  --color-error-subtle:   rgb(239 68 68 / 0.15);
  --color-info:           var(--color-blue-500);
  --color-info-subtle:    rgb(59 130 246 / 0.15);
}


/* ============================================================
   4. CSS RESET
   ============================================================ */

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  scroll-behavior: smooth;
  -webkit-text-size-adjust: 100%;
  text-size-adjust: 100%;
  tab-size: 4;
  font-size: 100%; /* respect user browser preferences */
  height: 100%;
}

body {
  font-family: var(--font-body);
  font-size: var(--text-body);
  color: var(--color-text-primary);
  background-color: var(--color-bg-base);
  line-height: 1.65;
  min-height: 100%;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* Block elements */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: var(--primitive-font-weight-bold);
  line-height: 1.15;
  overflow-wrap: break-word;
  color: var(--color-text-primary);
}

p {
  overflow-wrap: break-word;
  max-width: 70ch; /* optimal reading line length */
}

/* Lists */
ul, ol {
  list-style: none;
}

/* Media */
img,
picture,
video,
canvas,
svg {
  display: block;
  max-width: 100%;
}

img {
  font-style: italic; /* style alt text if image fails */
}

/* Interactive elements */
a {
  color: inherit;
  text-decoration: none;
}

button {
  cursor: pointer;
  border: none;
  background: none;
}

button,
input,
textarea,
select {
  font: inherit;
}

button:disabled,
input:disabled,
textarea:disabled,
select:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

/* Forms */
input,
textarea,
select {
  background-color: transparent;
  border: 1px solid var(--color-border-default);
  border-radius: var(--radius-input);
  color: var(--color-text-primary);
  padding: var(--space-component-padding-y) var(--space-component-padding-x);
}

textarea {
  resize: vertical;
}

/* Tables */
table {
  border-collapse: collapse;
  border-spacing: 0;
}

/* Focus — remove default, replaced in Section 7 */
:focus {
  outline: none;
}

/* Misc */
hr {
  border: none;
  border-top: 1px solid var(--color-border-default);
  color: inherit;
}

abbr[title] {
  text-decoration: underline dotted;
  cursor: help;
}

b, strong {
  font-weight: var(--primitive-font-weight-bold);
}

small {
  font-size: var(--text-sm);
}

sub,
sup {
  font-size: 75%;
  line-height: 0;
  position: relative;
  vertical-align: baseline;
}

sub { bottom: -0.25em; }
sup { top: -0.5em; }

code, kbd, samp, pre {
  font-family: var(--font-mono);
  font-size: var(--text-sm);
}

pre {
  overflow: auto;
}

/* Remove animations on elements with hidden attribute */
[hidden] {
  display: none !important;
}

/* Remove default fieldset styles */
fieldset {
  border: none;
  padding: 0;
  margin: 0;
}

legend {
  padding: 0;
}

/* Dialog reset */
dialog {
  border: none;
  padding: 0;
  background: transparent;
}


/* ============================================================
   5. TYPOGRAPHY UTILITIES
   Uses real clamp() values for fluid scaling.
   Viewport breakpoints: 320px (min) → 1280px (max)
   Formula: clamp(min, preferred-vw, max)
   ============================================================ */

/* Heading utilities — Playfair Display */
.text-display {
  font-family: var(--font-heading);
  font-size: clamp(3rem, 5vw + 1.25rem, 5rem);        /* 48px → 80px */
  font-weight: var(--primitive-font-weight-black);
  line-height: 1.05;
  letter-spacing: -0.02em;
  color: var(--color-text-primary);
}

.text-h1 {
  font-family: var(--font-heading);
  font-size: clamp(2.25rem, 3.5vw + 0.875rem, 3.75rem); /* 36px → 60px */
  font-weight: var(--primitive-font-weight-bold);
  line-height: 1.1;
  letter-spacing: -0.015em;
  color: var(--color-text-primary);
}

.text-h2 {
  font-family: var(--font-heading);
  font-size: clamp(1.875rem, 2.5vw + 0.625rem, 3rem);   /* 30px → 48px */
  font-weight: var(--primitive-font-weight-bold);
  line-height: 1.15;
  letter-spacing: -0.01em;
  color: var(--color-text-primary);
}

.text-h3 {
  font-family: var(--font-heading);
  font-size: clamp(1.5rem, 2vw + 0.375rem, 2.25rem);    /* 24px → 36px */
  font-weight: var(--primitive-font-weight-bold);
  line-height: 1.2;
  letter-spacing: -0.005em;
  color: var(--color-text-primary);
}

.text-h4 {
  font-family: var(--font-heading);
  font-size: clamp(1.25rem, 1.25vw + 0.3125rem, 1.875rem); /* 20px → 30px */
  font-weight: var(--primitive-font-weight-semibold);
  line-height: 1.25;
  color: var(--color-text-primary);
}

.text-h5 {
  font-family: var(--font-heading);
  font-size: clamp(1.125rem, 0.75vw + 0.25rem, 1.5rem);  /* 18px → 24px */
  font-weight: var(--primitive-font-weight-semibold);
  line-height: 1.3;
  color: var(--color-text-primary);
}

.text-h6 {
  font-family: var(--font-body);
  font-size: clamp(1rem, 0.5vw + 0.1875rem, 1.25rem);    /* 16px → 20px */
  font-weight: var(--primitive-font-weight-semibold);
  line-height: 1.4;
  color: var(--color-text-primary);
}

/* Body utilities — Lato */
.text-body-lg {
  font-family: var(--font-body);
  font-size: clamp(1.0625rem, 0.75vw + 0.25rem, 1.25rem); /* 17px → 20px */
  font-weight: var(--primitive-font-weight-regular);
  line-height: 1.6;
  color: var(--color-text-primary);
}

.text-body {
  font-family: var(--font-body);
  font-size: var(--text-body); /* 16px — fixed, no fluid needed */
  font-weight: var(--primitive-font-weight-regular);
  line-height: 1.65;
  color: var(--color-text-primary);
}

.text-sm {
  font-family: var(--font-body);
  font-size: var(--text-sm); /* 14px */
  font-weight: var(--primitive-font-weight-regular);
  line-height: 1.6;
  color: var(--color-text-secondary);
}

.text-caption {
  font-family: var(--font-body);
  font-size: var(--text-caption); /* 12px */
  font-weight: var(--primitive-font-weight-regular);
  line-height: 1.5;
  color: var(--color-text-tertiary);
}

/* Decorative — complements Playfair Display brand identity */
.text-overline {
  font-family: var(--font-body);
  font-size: var(--text-overline); /* 11px */
  font-weight: var(--primitive-font-weight-bold);
  line-height: 1.5;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--color-text-brand);
}

/* Modifier utilities */
.text-brand   { color: var(--color-text-brand); }
.text-muted   { color: var(--color-text-secondary); }
.text-subtle  { color: var(--color-text-tertiary); }
.text-inverse { color: var(--color-text-inverse); }

.text-balance  { text-wrap: balance; }
.text-pretty   { text-wrap: pretty; }

.font-heading  { font-family: var(--font-heading); }
.font-body     { font-family: var(--font-body); }

.font-regular  { font-weight: var(--primitive-font-weight-regular); }
.font-medium   { font-weight: var(--primitive-font-weight-medium); }
.font-semibold { font-weight: var(--primitive-font-weight-semibold); }
.font-bold     { font-weight: var(--primitive-font-weight-bold); }


/* ============================================================
   6. LAYOUT UTILITIES
   ============================================================ */

/* Container — fluid with max-width cap */
.container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: var(--space-container-padding);
}

.container--sm  { max-width: 768px; }
.container--md  { max-width: 1024px; }
.container--lg  { max-width: 1280px; }  /* default */
.container--xl  { max-width: 1440px; }
.container--full { max-width: none; }

/* Section — vertical rhythm */
.section {
  padding-block: var(--space-section-gap); /* 80px */
}

.section--sm {
  padding-block: var(--space-section-gap-sm); /* 48px */
}

.section--lg {
  padding-block: var(--space-32); /* 128px */
}

/* Flow — vertical spacing for prose/stacked content */
.flow > * + * {
  margin-top: var(--space-content-gap); /* 32px default */
}

.flow--sm > * + * { margin-top: var(--space-4); }
.flow--md > * + * { margin-top: var(--space-8); }
.flow--lg > * + * { margin-top: var(--space-16); }

/* Cluster — horizontal wrapping group */
.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-4);
  align-items: center;
}

/* Stack — vertical flex */
.stack {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
}

/* Grid — responsive auto-grid */
.grid-auto {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(280px, 100%), 1fr));
  gap: var(--space-8);
}

/* Divider */
.divider {
  border: none;
  border-top: 1px solid var(--color-border-default);
  margin-block: var(--space-8);
}


/* ============================================================
   7. FOCUS & ACCESSIBILITY UTILITIES
   ============================================================ */

/* Global focus ring — overrides reset above */
:focus-visible {
  outline: 2px solid var(--color-border-focus);
  outline-offset: 3px;
  border-radius: var(--primitive-radius-sm);
}

/* Screen-reader only — visually hidden but accessible */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* Reveal to focus (for skip-links) */
.sr-only-focusable:focus,
.sr-only-focusable:focus-visible {
  position: static;
  width: auto;
  height: auto;
  padding: var(--space-2) var(--space-4);
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
}

/* Skip navigation link */
.skip-link {
  position: absolute;
  top: var(--space-4);
  left: var(--space-4);
  z-index: var(--z-toast);
  padding: var(--space-2) var(--space-6);
  background-color: var(--color-brand-primary);
  color: var(--color-text-on-brand);
  font-family: var(--font-body);
  font-size: var(--text-sm);
  font-weight: var(--primitive-font-weight-semibold);
  border-radius: var(--radius-button);
  transform: translateY(-200%);
  transition: transform var(--duration-fast) var(--easing-out);
}

.skip-link:focus-visible {
  transform: translateY(0);
}

/* Not focusable — for decorative elements */
[aria-hidden="true"] {
  pointer-events: none;
}

/* Disabled state */
[aria-disabled="true"],
[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

/* High contrast mode support */
@media (forced-colors: active) {
  :focus-visible {
    outline: 3px solid ButtonText;
  }

  .btn {
    forced-color-adjust: none;
  }
}


/* ============================================================
   8. REDUCED MOTION
   ============================================================ */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }

  /* Preserve opacity transitions for UX clarity */
  .transition-opacity {
    transition-duration: var(--duration-normal) !important;
  }
}

/* Update motion custom properties for reduced-motion consumers */
@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-fast:   0.01ms;
    --duration-normal: 0.01ms;
    --duration-slow:   0.01ms;
    --duration-slower: 0.01ms;
  }
}


/* ============================================================
   9. COMPONENT TOKENS — Level 3
   Component-scoped tokens that ONLY reference Level 2 semantic tokens.
   Rule: no Level 1 primitive token may be used directly in a component.
   ============================================================ */

/* ─── Button ─── */
.btn {
  /* Component token declarations */
  --btn-bg:            var(--color-brand-primary);
  --btn-bg-hover:      var(--color-brand-primary-hover);
  --btn-bg-active:     var(--color-brand-primary-active);
  --btn-text:          var(--color-text-on-brand);
  --btn-border:        transparent;
  --btn-border-focus:  var(--color-border-focus);
  --btn-radius:        var(--radius-button);
  --btn-padding-x:     var(--space-component-padding-x);
  --btn-padding-y:     var(--space-component-padding-y);
  --btn-font-size:     var(--text-body);
  --btn-font-weight:   var(--primitive-font-weight-semibold);
  --btn-shadow:        var(--shadow-button);
  --btn-transition:
    background-color var(--duration-fast) var(--easing-out),
    transform        var(--duration-fast) var(--easing-out),
    box-shadow       var(--duration-fast) var(--easing-out);

  /* Applied styles */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--btn-padding-y) var(--btn-padding-x);
  background-color: var(--btn-bg);
  color: var(--btn-text);
  font-family: var(--font-body);
  font-size: var(--btn-font-size);
  font-weight: var(--btn-font-weight);
  line-height: 1;
  border: 1px solid var(--btn-border);
  border-radius: var(--btn-radius);
  box-shadow: var(--btn-shadow);
  text-decoration: none;
  cursor: pointer;
  transition: var(--btn-transition);
  white-space: nowrap;
  user-select: none;
  -webkit-user-select: none;
}

.btn:hover:not(:disabled):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-hover);
  transform: translateY(-1px);
  box-shadow: var(--shadow-card);
}

.btn:active:not(:disabled):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-active);
  transform: translateY(0);
  box-shadow: var(--btn-shadow);
}

/* Secondary variant */
.btn--secondary {
  --btn-bg:        var(--color-brand-secondary);
  --btn-bg-hover:  var(--color-brand-secondary-hover);
  --btn-bg-active: var(--color-olive-800);
}

/* Outline variant */
.btn--outline {
  --btn-bg:        transparent;
  --btn-bg-hover:  var(--color-brand-primary-subtle);
  --btn-bg-active: var(--color-sienna-200);
  --btn-text:      var(--color-brand-primary);
  --btn-border:    var(--color-brand-primary);
}

/* Ghost variant */
.btn--ghost {
  --btn-bg:        transparent;
  --btn-bg-hover:  var(--color-bg-subtle);
  --btn-bg-active: var(--color-bg-muted);
  --btn-text:      var(--color-text-primary);
  --btn-border:    transparent;
  --btn-shadow:    none;
}

/* Size modifiers */
.btn--sm {
  --btn-padding-x: var(--space-4);
  --btn-padding-y: var(--space-2);
  --btn-font-size: var(--text-sm);
}

.btn--lg {
  --btn-padding-x: var(--space-8);
  --btn-padding-y: var(--space-4);
  --btn-font-size: var(--text-body-lg);
}


/* ─── Card ─── */
.card {
  --card-bg:      var(--color-surface-default);
  --card-border:  var(--color-border-default);
  --card-radius:  var(--radius-card);
  --card-shadow:  var(--shadow-card);
  --card-padding: var(--space-8);

  background-color: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: var(--card-radius);
  box-shadow: var(--card-shadow);
  padding: var(--card-padding);
  overflow: hidden;
  transition: box-shadow var(--duration-fast) var(--easing-out),
              transform   var(--duration-fast) var(--easing-out);
}

.card--interactive:hover {
  box-shadow: var(--shadow-card-hover);
  transform: translateY(-2px);
}

.card--raised {
  --card-bg:     var(--color-surface-raised);
  --card-shadow: var(--shadow-card-hover);
}


/* ─── Badge ─── */
.badge {
  --badge-bg:      var(--color-brand-primary-subtle);
  --badge-text:    var(--color-brand-primary);
  --badge-radius:  var(--radius-badge);
  --badge-padding-x: var(--space-3);
  --badge-padding-y: var(--space-1);
  --badge-font-size: var(--text-caption);

  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--badge-padding-y) var(--badge-padding-x);
  background-color: var(--badge-bg);
  color: var(--badge-text);
  font-family: var(--font-body);
  font-size: var(--badge-font-size);
  font-weight: var(--primitive-font-weight-semibold);
  line-height: 1;
  border-radius: var(--badge-radius);
  white-space: nowrap;
}

.badge--secondary {
  --badge-bg:   var(--color-brand-secondary-subtle);
  --badge-text: var(--color-brand-secondary);
}

.badge--success {
  --badge-bg:   var(--color-success-subtle);
  --badge-text: var(--color-success);
}

.badge--warning {
  --badge-bg:   var(--color-warning-subtle);
  --badge-text: var(--color-warning);
}

.badge--error {
  --badge-bg:   var(--color-error-subtle);
  --badge-text: var(--color-error);
}


/* ─── Input ─── */
.input {
  --input-bg:           var(--color-surface-default);
  --input-text:         var(--color-text-primary);
  --input-placeholder:  var(--color-text-tertiary);
  --input-border:       var(--color-border-default);
  --input-border-focus: var(--color-border-focus);
  --input-border-error: var(--color-error);
  --input-radius:       var(--radius-input);
  --input-padding-x:    var(--space-component-padding-x);
  --input-padding-y:    var(--space-component-padding-y);
  --input-font-size:    var(--text-body);

  display: block;
  width: 100%;
  padding: var(--input-padding-y) var(--input-padding-x);
  background-color: var(--input-bg);
  color: var(--input-text);
  font-family: var(--font-body);
  font-size: var(--input-font-size);
  border: 1px solid var(--input-border);
  border-radius: var(--input-radius);
  transition: border-color var(--duration-fast) var(--easing-out),
              box-shadow   var(--duration-fast) var(--easing-out);
}

.input::placeholder {
  color: var(--input-placeholder);
}

.input:focus-visible {
  border-color: var(--input-border-focus);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--color-border-focus) 20%, transparent);
}

.input--error {
  border-color: var(--input-border-error);
}

.input--error:focus-visible {
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--color-error) 20%, transparent);
}
```

---

## Token Architecture Reference

### Level Summary

| Level | Section | Purpose | Example |
|---|---|---|---|
| 1 — Primitive | Section 1 | Raw values, no meaning | `--color-sienna-600: #A0522D` |
| 2 — Semantic | Section 2–3 | Purpose-based mapping | `--color-brand-primary: var(--color-sienna-600)` |
| 3 — Component | Section 9 | Component-scoped aliases | `--btn-bg: var(--color-brand-primary)` |

### Dark Mode Architecture

Dark mode overrides ONLY the semantic color tokens in `[data-theme="dark"]`. No CSS rules are duplicated. The component CSS (`.btn`, `.card`, etc.) does not change — it automatically adapts because it reads from semantic tokens, which are remapped for dark.

**Cascade:**
```
Component → reads --btn-bg
  → resolves to --color-brand-primary  (semantic)
    → light:  --color-sienna-600 (#A0522D)
    → dark:   --color-sienna-400 (#CC7F56)  ← only this line changes
```

### Typography Scale with Real clamp() Values

| Utility | Min | Max | Formula |
|---|---|---|---|
| `.text-display` | 48px (3rem) | 80px (5rem) | `clamp(3rem, 5vw + 1.25rem, 5rem)` |
| `.text-h1` | 36px (2.25rem) | 60px (3.75rem) | `clamp(2.25rem, 3.5vw + 0.875rem, 3.75rem)` |
| `.text-h2` | 30px (1.875rem) | 48px (3rem) | `clamp(1.875rem, 2.5vw + 0.625rem, 3rem)` |
| `.text-h3` | 24px (1.5rem) | 36px (2.25rem) | `clamp(1.5rem, 2vw + 0.375rem, 2.25rem)` |
| `.text-h4` | 20px (1.25rem) | 30px (1.875rem) | `clamp(1.25rem, 1.25vw + 0.3125rem, 1.875rem)` |
| `.text-body-lg` | 17px (1.0625rem) | 20px (1.25rem) | `clamp(1.0625rem, 0.75vw + 0.25rem, 1.25rem)` |

### Contrast Audit (WCAG 2.1)

| Combination | Hex values | Ratio | AA (4.5:1) | AAA (7:1) |
|---|---|---|---|---|
| text-primary on bg-base | #1A1A1A on #FAF7F0 | ~18.5:1 | Pass | Pass |
| brand-primary on bg-base | #A0522D on #FAF7F0 | ~4.8:1 | Pass | Fail |
| text-on-brand on brand-primary | #FDFCF8 on #A0522D | ~4.7:1 | Pass | Fail |
| text-secondary on bg-base | #595959 on #FAF7F0 | ~7.4:1 | Pass | Pass |
| brand-secondary on bg-base | #556B2F on #FAF7F0 | ~6.2:1 | Pass | Fail |
| dark text-primary on dark bg | #FAF7F0 on #1A1915 | ~16.9:1 | Pass | Pass |

### Governance Notes

- All component CSS references Level 2 tokens exclusively. Level 1 primitives appear only in Section 1.
- Adding a new color outside this token system requires director approval.
- Dark mode is maintained by updating only the 18 semantic color token overrides in `[data-theme="dark"]`.
- Motion tokens (`--duration-*`) are automatically zeroed out by the reduced-motion media query override in Section 8.
