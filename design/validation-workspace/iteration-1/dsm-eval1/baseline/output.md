# Café Vermut — globals.css (Eval 1 Baseline Output)

**Status:** Complete  
**Engineer:** Claude Sonnet 4.6  
**Date:** 2026-05-13  
**Tokens approved by:** brand-designer + ui-designer

---

## globals.css

```css
/* ============================================================
   CAFÉ VERMUT — GLOBAL STYLES
   Design System v1.0
   Architecture: Primitives → Semantic → Component tokens
   ============================================================ */

/* ============================================================
   1. PRIMITIVE TOKENS
   Raw values — never referenced directly in components.
   ============================================================ */

:root {
  /* --- Color Primitives --- */
  --primitive-sienna-50:  #F9EDE5;
  --primitive-sienna-100: #F2D5C1;
  --primitive-sienna-200: #E5AA8C;
  --primitive-sienna-300: #C97E52;
  --primitive-sienna-400: #A0522D;   /* brand-primary */
  --primitive-sienna-500: #8B4220;
  --primitive-sienna-600: #733515;
  --primitive-sienna-700: #5A280E;
  --primitive-sienna-800: #3D1A08;
  --primitive-sienna-900: #220E02;

  --primitive-olive-50:   #EEF1E8;
  --primitive-olive-100:  #D8DFC9;
  --primitive-olive-200:  #B4BE9C;
  --primitive-olive-300:  #8A9B6B;
  --primitive-olive-400:  #556B2F;   /* brand-secondary */
  --primitive-olive-500:  #465A27;
  --primitive-olive-600:  #374820;
  --primitive-olive-700:  #293718;
  --primitive-olive-800:  #1B2510;
  --primitive-olive-900:  #0E1408;

  --primitive-cream-50:   #FDFCF8;
  --primitive-cream-100:  #FAF7F0;   /* bg-base */
  --primitive-cream-200:  #F2EBD8;
  --primitive-cream-300:  #E8DAC0;
  --primitive-cream-400:  #D9C69E;

  --primitive-neutral-0:   #FFFFFF;
  --primitive-neutral-50:  #F7F7F7;
  --primitive-neutral-100: #EFEFEF;
  --primitive-neutral-200: #DEDEDE;
  --primitive-neutral-300: #C8C8C8;
  --primitive-neutral-400: #ADADAD;
  --primitive-neutral-500: #8C8C8C;
  --primitive-neutral-600: #636363;
  --primitive-neutral-700: #3D3D3D;
  --primitive-neutral-800: #2A2A2A;
  --primitive-neutral-900: #1A1A1A;   /* text-primary */
  --primitive-neutral-950: #0D0D0D;

  --primitive-feedback-success-light: #D4EDDA;
  --primitive-feedback-success:       #2D7A3A;
  --primitive-feedback-warning-light: #FFF3CD;
  --primitive-feedback-warning:       #856404;
  --primitive-feedback-error-light:   #F8D7DA;
  --primitive-feedback-error:         #842029;
  --primitive-feedback-info-light:    #D1ECF1;
  --primitive-feedback-info:          #0C5460;

  /* --- Spacing Primitives (4px base) --- */
  --primitive-space-0:    0;
  --primitive-space-1:    0.25rem;   /*  4px */
  --primitive-space-2:    0.5rem;    /*  8px */
  --primitive-space-3:    0.75rem;   /* 12px */
  --primitive-space-4:    1rem;      /* 16px */
  --primitive-space-5:    1.25rem;   /* 20px */
  --primitive-space-6:    1.5rem;    /* 24px */
  --primitive-space-8:    2rem;      /* 32px */
  --primitive-space-10:   2.5rem;    /* 40px */
  --primitive-space-12:   3rem;      /* 48px */
  --primitive-space-16:   4rem;      /* 64px */
  --primitive-space-20:   5rem;      /* 80px */
  --primitive-space-24:   6rem;      /* 96px */
  --primitive-space-32:   8rem;      /* 128px */
  --primitive-space-40:   10rem;     /* 160px */

  /* --- Radius Primitives --- */
  --primitive-radius-none:   0;
  --primitive-radius-xs:     2px;
  --primitive-radius-sm:     4px;
  --primitive-radius-md:     8px;
  --primitive-radius-lg:     12px;
  --primitive-radius-xl:     16px;
  --primitive-radius-2xl:    24px;
  --primitive-radius-full:   9999px;

  /* --- Shadow Primitives --- */
  --primitive-shadow-xs:  0 1px 2px 0 rgb(0 0 0 / 0.05);
  --primitive-shadow-sm:  0 1px 3px 0 rgb(0 0 0 / 0.10), 0 1px 2px -1px rgb(0 0 0 / 0.10);
  --primitive-shadow-md:  0 4px 6px -1px rgb(0 0 0 / 0.10), 0 2px 4px -2px rgb(0 0 0 / 0.10);
  --primitive-shadow-lg:  0 10px 15px -3px rgb(0 0 0 / 0.10), 0 4px 6px -4px rgb(0 0 0 / 0.10);
  --primitive-shadow-xl:  0 20px 25px -5px rgb(0 0 0 / 0.10), 0 8px 10px -6px rgb(0 0 0 / 0.10);
  --primitive-shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
  --primitive-shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.06);

  /* --- Duration Primitives --- */
  --primitive-duration-instant: 0ms;
  --primitive-duration-fast:    100ms;
  --primitive-duration-normal:  200ms;
  --primitive-duration-slow:    300ms;
  --primitive-duration-slower:  500ms;
  --primitive-duration-slowest: 800ms;

  /* --- Easing Primitives --- */
  --primitive-ease-linear:    linear;
  --primitive-ease-in:        cubic-bezier(0.4, 0, 1, 1);
  --primitive-ease-out:       cubic-bezier(0, 0, 0.2, 1);
  --primitive-ease-in-out:    cubic-bezier(0.4, 0, 0.2, 1);
  --primitive-ease-bounce:    cubic-bezier(0.34, 1.56, 0.64, 1);

  /* --- Font Family Primitives --- */
  --primitive-font-display:  'Playfair Display', Georgia, 'Times New Roman', serif;
  --primitive-font-body:     'Lato', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --primitive-font-mono:     'JetBrains Mono', 'Fira Code', 'Consolas', monospace;

  /* --- Font Weight Primitives --- */
  --primitive-weight-light:      300;
  --primitive-weight-regular:    400;
  --primitive-weight-medium:     500;
  --primitive-weight-semibold:   600;
  --primitive-weight-bold:       700;
  --primitive-weight-extrabold:  800;
  --primitive-weight-black:      900;

  /* --- Line Height Primitives --- */
  --primitive-leading-none:    1;
  --primitive-leading-tight:   1.25;
  --primitive-leading-snug:    1.375;
  --primitive-leading-normal:  1.5;
  --primitive-leading-relaxed: 1.625;
  --primitive-leading-loose:   2;

  /* --- Letter Spacing Primitives --- */
  --primitive-tracking-tighter: -0.05em;
  --primitive-tracking-tight:   -0.025em;
  --primitive-tracking-normal:   0em;
  --primitive-tracking-wide:     0.025em;
  --primitive-tracking-wider:    0.05em;
  --primitive-tracking-widest:   0.1em;

  /* --- Z-Index Primitives --- */
  --primitive-z-below:    -1;
  --primitive-z-base:      0;
  --primitive-z-raised:    10;
  --primitive-z-dropdown:  100;
  --primitive-z-sticky:    200;
  --primitive-z-overlay:   300;
  --primitive-z-modal:     400;
  --primitive-z-popover:   500;
  --primitive-z-toast:     600;
  --primitive-z-tooltip:   700;
}


/* ============================================================
   2. SEMANTIC TOKENS (Light Mode — default)
   Maps primitives to contextual meaning.
   ============================================================ */

:root {
  /* --- Brand --- */
  --color-brand-primary:          var(--primitive-sienna-400);
  --color-brand-primary-hover:    var(--primitive-sienna-500);
  --color-brand-primary-active:   var(--primitive-sienna-600);
  --color-brand-primary-subtle:   var(--primitive-sienna-50);
  --color-brand-primary-on:       var(--primitive-neutral-0);

  --color-brand-secondary:        var(--primitive-olive-400);
  --color-brand-secondary-hover:  var(--primitive-olive-500);
  --color-brand-secondary-active: var(--primitive-olive-600);
  --color-brand-secondary-subtle: var(--primitive-olive-50);
  --color-brand-secondary-on:     var(--primitive-neutral-0);

  /* --- Background --- */
  --color-bg-base:        var(--primitive-cream-100);
  --color-bg-elevated:    var(--primitive-neutral-0);
  --color-bg-sunken:      var(--primitive-cream-200);
  --color-bg-overlay:     rgb(26 26 26 / 0.5);
  --color-bg-inverse:     var(--primitive-neutral-900);

  /* --- Surface --- */
  --color-surface-default:  var(--primitive-neutral-0);
  --color-surface-raised:   var(--primitive-neutral-0);
  --color-surface-overlay:  var(--primitive-neutral-0);
  --color-surface-brand:    var(--primitive-sienna-50);
  --color-surface-brand-secondary: var(--primitive-olive-50);

  /* --- Text --- */
  --color-text-primary:    var(--primitive-neutral-900);
  --color-text-secondary:  var(--primitive-neutral-600);
  --color-text-tertiary:   var(--primitive-neutral-500);
  --color-text-disabled:   var(--primitive-neutral-400);
  --color-text-inverse:    var(--primitive-neutral-0);
  --color-text-brand:      var(--primitive-sienna-400);
  --color-text-brand-secondary: var(--primitive-olive-400);
  --color-text-on-brand:   var(--primitive-neutral-0);

  /* --- Border --- */
  --color-border-default:  var(--primitive-neutral-200);
  --color-border-subtle:   var(--primitive-neutral-100);
  --color-border-strong:   var(--primitive-neutral-400);
  --color-border-brand:    var(--primitive-sienna-400);
  --color-border-brand-secondary: var(--primitive-olive-400);
  --color-border-inverse:  var(--primitive-neutral-700);

  /* --- Feedback --- */
  --color-feedback-success-bg:   var(--primitive-feedback-success-light);
  --color-feedback-success-text: var(--primitive-feedback-success);
  --color-feedback-warning-bg:   var(--primitive-feedback-warning-light);
  --color-feedback-warning-text: var(--primitive-feedback-warning);
  --color-feedback-error-bg:     var(--primitive-feedback-error-light);
  --color-feedback-error-text:   var(--primitive-feedback-error);
  --color-feedback-info-bg:      var(--primitive-feedback-info-light);
  --color-feedback-info-text:    var(--primitive-feedback-info);

  /* --- Focus --- */
  --color-focus-ring:        var(--primitive-sienna-400);
  --color-focus-ring-offset: var(--primitive-cream-100);

  /* --- Semantic Spacing --- */
  --space-xs:   var(--primitive-space-1);
  --space-sm:   var(--primitive-space-2);
  --space-md:   var(--primitive-space-4);
  --space-lg:   var(--primitive-space-6);
  --space-xl:   var(--primitive-space-8);
  --space-2xl:  var(--primitive-space-12);
  --space-3xl:  var(--primitive-space-16);
  --space-4xl:  var(--primitive-space-20);
  --space-5xl:  var(--primitive-space-24);

  /* --- Semantic Radius --- */
  --radius-sm:   var(--primitive-radius-sm);
  --radius-md:   var(--primitive-radius-md);
  --radius-lg:   var(--primitive-radius-lg);
  --radius-full: var(--primitive-radius-full);

  /* --- Semantic Shadows --- */
  --shadow-sm:  var(--primitive-shadow-sm);
  --shadow-md:  var(--primitive-shadow-md);
  --shadow-lg:  var(--primitive-shadow-lg);
  --shadow-xl:  var(--primitive-shadow-xl);

  /* --- Semantic Typography --- */
  --font-display: var(--primitive-font-display);
  --font-body:    var(--primitive-font-body);
  --font-mono:    var(--primitive-font-mono);

  /* --- Semantic Motion --- */
  --duration-fast:   var(--primitive-duration-fast);
  --duration-normal: var(--primitive-duration-normal);
  --duration-slow:   var(--primitive-duration-slow);
  --ease-default:    var(--primitive-ease-in-out);
  --ease-enter:      var(--primitive-ease-out);
  --ease-exit:       var(--primitive-ease-in);
  --ease-spring:     var(--primitive-ease-bounce);

  /* --- Layout --- */
  --container-sm:   640px;
  --container-md:   768px;
  --container-lg:   1024px;
  --container-xl:   1280px;
  --container-2xl:  1440px;
  --container-padding-x: clamp(1rem, 4vw, 2.5rem);
}


/* ============================================================
   3. DARK MODE SEMANTIC TOKEN OVERRIDES
   ============================================================ */

@media (prefers-color-scheme: dark) {
  :root {
    /* Brand adjusts slightly for contrast on dark */
    --color-brand-primary:        var(--primitive-sienna-300);
    --color-brand-primary-hover:  var(--primitive-sienna-200);
    --color-brand-primary-active: var(--primitive-sienna-400);
    --color-brand-primary-subtle: var(--primitive-sienna-800);
    --color-brand-primary-on:     var(--primitive-neutral-950);

    --color-brand-secondary:        var(--primitive-olive-300);
    --color-brand-secondary-hover:  var(--primitive-olive-200);
    --color-brand-secondary-active: var(--primitive-olive-400);
    --color-brand-secondary-subtle: var(--primitive-olive-800);
    --color-brand-secondary-on:     var(--primitive-neutral-950);

    /* Background */
    --color-bg-base:     var(--primitive-neutral-950);
    --color-bg-elevated: var(--primitive-neutral-800);
    --color-bg-sunken:   #0A0A0A;
    --color-bg-overlay:  rgb(0 0 0 / 0.7);
    --color-bg-inverse:  var(--primitive-cream-100);

    /* Surface */
    --color-surface-default:  var(--primitive-neutral-800);
    --color-surface-raised:   var(--primitive-neutral-700);
    --color-surface-overlay:  var(--primitive-neutral-800);
    --color-surface-brand:    var(--primitive-sienna-800);
    --color-surface-brand-secondary: var(--primitive-olive-800);

    /* Text */
    --color-text-primary:    var(--primitive-neutral-50);
    --color-text-secondary:  var(--primitive-neutral-300);
    --color-text-tertiary:   var(--primitive-neutral-400);
    --color-text-disabled:   var(--primitive-neutral-600);
    --color-text-inverse:    var(--primitive-neutral-900);
    --color-text-brand:      var(--primitive-sienna-300);
    --color-text-brand-secondary: var(--primitive-olive-300);
    --color-text-on-brand:   var(--primitive-neutral-950);

    /* Border */
    --color-border-default:  var(--primitive-neutral-700);
    --color-border-subtle:   var(--primitive-neutral-800);
    --color-border-strong:   var(--primitive-neutral-500);
    --color-border-brand:    var(--primitive-sienna-300);
    --color-border-brand-secondary: var(--primitive-olive-300);
    --color-border-inverse:  var(--primitive-neutral-200);

    /* Shadows — more pronounced for dark elevation */
    --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.3), 0 1px 2px -1px rgb(0 0 0 / 0.3);
    --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.35), 0 2px 4px -2px rgb(0 0 0 / 0.35);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.4), 0 4px 6px -4px rgb(0 0 0 / 0.4);
    --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.45), 0 8px 10px -6px rgb(0 0 0 / 0.45);

    /* Focus ring on dark background */
    --color-focus-ring:        var(--primitive-sienna-300);
    --color-focus-ring-offset: var(--primitive-neutral-950);
  }
}

/* Data-attribute override for explicit dark/light toggle */
[data-theme="dark"] {
  --color-brand-primary:        var(--primitive-sienna-300);
  --color-brand-primary-hover:  var(--primitive-sienna-200);
  --color-brand-primary-active: var(--primitive-sienna-400);
  --color-brand-primary-subtle: var(--primitive-sienna-800);
  --color-brand-primary-on:     var(--primitive-neutral-950);

  --color-brand-secondary:        var(--primitive-olive-300);
  --color-brand-secondary-hover:  var(--primitive-olive-200);
  --color-brand-secondary-active: var(--primitive-olive-400);
  --color-brand-secondary-subtle: var(--primitive-olive-800);
  --color-brand-secondary-on:     var(--primitive-neutral-950);

  --color-bg-base:     var(--primitive-neutral-950);
  --color-bg-elevated: var(--primitive-neutral-800);
  --color-bg-sunken:   #0A0A0A;
  --color-bg-overlay:  rgb(0 0 0 / 0.7);
  --color-bg-inverse:  var(--primitive-cream-100);

  --color-surface-default:  var(--primitive-neutral-800);
  --color-surface-raised:   var(--primitive-neutral-700);
  --color-surface-overlay:  var(--primitive-neutral-800);
  --color-surface-brand:    var(--primitive-sienna-800);
  --color-surface-brand-secondary: var(--primitive-olive-800);

  --color-text-primary:    var(--primitive-neutral-50);
  --color-text-secondary:  var(--primitive-neutral-300);
  --color-text-tertiary:   var(--primitive-neutral-400);
  --color-text-disabled:   var(--primitive-neutral-600);
  --color-text-inverse:    var(--primitive-neutral-900);
  --color-text-brand:      var(--primitive-sienna-300);
  --color-text-brand-secondary: var(--primitive-olive-300);
  --color-text-on-brand:   var(--primitive-neutral-950);

  --color-border-default:  var(--primitive-neutral-700);
  --color-border-subtle:   var(--primitive-neutral-800);
  --color-border-strong:   var(--primitive-neutral-500);
  --color-border-brand:    var(--primitive-sienna-300);
  --color-border-brand-secondary: var(--primitive-olive-300);
  --color-border-inverse:  var(--primitive-neutral-200);

  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.3), 0 1px 2px -1px rgb(0 0 0 / 0.3);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.35), 0 2px 4px -2px rgb(0 0 0 / 0.35);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.4), 0 4px 6px -4px rgb(0 0 0 / 0.4);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.45), 0 8px 10px -6px rgb(0 0 0 / 0.45);

  --color-focus-ring:        var(--primitive-sienna-300);
  --color-focus-ring-offset: var(--primitive-neutral-950);
}

[data-theme="light"] {
  /* Explicit light mode — mirrors :root defaults */
  --color-brand-primary:        var(--primitive-sienna-400);
  --color-brand-primary-hover:  var(--primitive-sienna-500);
  --color-brand-primary-active: var(--primitive-sienna-600);
  --color-brand-primary-subtle: var(--primitive-sienna-50);
  --color-brand-primary-on:     var(--primitive-neutral-0);

  --color-brand-secondary:        var(--primitive-olive-400);
  --color-brand-secondary-hover:  var(--primitive-olive-500);
  --color-brand-secondary-active: var(--primitive-olive-600);
  --color-brand-secondary-subtle: var(--primitive-olive-50);
  --color-brand-secondary-on:     var(--primitive-neutral-0);

  --color-bg-base:     var(--primitive-cream-100);
  --color-bg-elevated: var(--primitive-neutral-0);
  --color-bg-sunken:   var(--primitive-cream-200);
  --color-bg-overlay:  rgb(26 26 26 / 0.5);
  --color-bg-inverse:  var(--primitive-neutral-900);

  --color-surface-default:  var(--primitive-neutral-0);
  --color-surface-raised:   var(--primitive-neutral-0);
  --color-surface-overlay:  var(--primitive-neutral-0);
  --color-surface-brand:    var(--primitive-sienna-50);
  --color-surface-brand-secondary: var(--primitive-olive-50);

  --color-text-primary:    var(--primitive-neutral-900);
  --color-text-secondary:  var(--primitive-neutral-600);
  --color-text-tertiary:   var(--primitive-neutral-500);
  --color-text-disabled:   var(--primitive-neutral-400);
  --color-text-inverse:    var(--primitive-neutral-0);
  --color-text-brand:      var(--primitive-sienna-400);
  --color-text-brand-secondary: var(--primitive-olive-400);
  --color-text-on-brand:   var(--primitive-neutral-0);

  --color-border-default:  var(--primitive-neutral-200);
  --color-border-subtle:   var(--primitive-neutral-100);
  --color-border-strong:   var(--primitive-neutral-400);
  --color-border-brand:    var(--primitive-sienna-400);
  --color-border-brand-secondary: var(--primitive-olive-400);
  --color-border-inverse:  var(--primitive-neutral-700);

  --shadow-sm: var(--primitive-shadow-sm);
  --shadow-md: var(--primitive-shadow-md);
  --shadow-lg: var(--primitive-shadow-lg);
  --shadow-xl: var(--primitive-shadow-xl);

  --color-focus-ring:        var(--primitive-sienna-400);
  --color-focus-ring-offset: var(--primitive-cream-100);
}


/* ============================================================
   4. COMPONENT TOKENS
   Scoped to specific UI patterns.
   ============================================================ */

:root {
  /* --- Button --- */
  --btn-bg-primary:          var(--color-brand-primary);
  --btn-bg-primary-hover:    var(--color-brand-primary-hover);
  --btn-bg-primary-active:   var(--color-brand-primary-active);
  --btn-text-primary:        var(--color-brand-primary-on);
  --btn-border-primary:      transparent;

  --btn-bg-secondary:        transparent;
  --btn-bg-secondary-hover:  var(--color-brand-primary-subtle);
  --btn-text-secondary:      var(--color-brand-primary);
  --btn-border-secondary:    var(--color-brand-primary);

  --btn-bg-ghost:            transparent;
  --btn-bg-ghost-hover:      var(--color-bg-sunken);
  --btn-text-ghost:          var(--color-text-secondary);
  --btn-border-ghost:        transparent;

  --btn-bg-disabled:         var(--color-border-default);
  --btn-text-disabled:       var(--color-text-disabled);

  --btn-radius:              var(--primitive-radius-sm);
  --btn-padding-x:           var(--primitive-space-5);
  --btn-padding-y:           var(--primitive-space-3);
  --btn-font-family:         var(--font-body);
  --btn-font-weight:         var(--primitive-weight-bold);
  --btn-letter-spacing:      var(--primitive-tracking-wider);
  --btn-font-size:           0.875rem;
  --btn-transition:          background-color var(--duration-fast) var(--ease-default),
                             border-color var(--duration-fast) var(--ease-default),
                             color var(--duration-fast) var(--ease-default),
                             box-shadow var(--duration-fast) var(--ease-default);

  /* --- Card --- */
  --card-bg:                 var(--color-surface-default);
  --card-border:             var(--color-border-subtle);
  --card-radius:             var(--primitive-radius-lg);
  --card-shadow:             var(--shadow-sm);
  --card-shadow-hover:       var(--shadow-md);
  --card-padding:            var(--primitive-space-6);

  /* --- Input --- */
  --input-bg:                var(--color-surface-default);
  --input-bg-disabled:       var(--color-bg-sunken);
  --input-border:            var(--color-border-default);
  --input-border-focus:      var(--color-brand-primary);
  --input-border-error:      var(--primitive-feedback-error);
  --input-text:              var(--color-text-primary);
  --input-placeholder:       var(--color-text-tertiary);
  --input-radius:            var(--primitive-radius-sm);
  --input-padding-x:         var(--primitive-space-4);
  --input-padding-y:         var(--primitive-space-3);

  /* --- Nav --- */
  --nav-bg:                  var(--color-bg-base);
  --nav-border:              var(--color-border-subtle);
  --nav-text:                var(--color-text-secondary);
  --nav-text-active:         var(--color-text-brand);
  --nav-height:              4rem;
  --nav-height-mobile:       3.5rem;

  /* --- Badge --- */
  --badge-radius:            var(--primitive-radius-full);
  --badge-padding-x:         var(--primitive-space-3);
  --badge-padding-y:         var(--primitive-space-1);
  --badge-font-size:         0.75rem;
  --badge-font-weight:       var(--primitive-weight-semibold);
  --badge-letter-spacing:    var(--primitive-tracking-wide);

  /* --- Divider --- */
  --divider-color:           var(--color-border-default);
  --divider-color-ornamental: var(--color-brand-primary);
}


/* ============================================================
   5. CSS RESET
   Modern reset based on Andy Bell's reset + Josh Comeau's reset.
   ============================================================ */

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Prevent font size inflation on mobile */
html {
  -moz-text-size-adjust: none;
  -webkit-text-size-adjust: none;
  text-size-adjust: none;
  /* Smooth scrolling — overridden by reduced-motion below */
  scroll-behavior: smooth;
  /* Improve text rendering */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

body {
  min-block-size: 100vh;
  /* Responsive font rendering */
  font-synthesis: none;
  /* Prevent overflow on small devices */
  overflow-x: hidden;
}

/* Remove default list styles for lists with role="list" */
ul[role="list"],
ol[role="list"] {
  list-style: none;
}

/* Set shorter line heights for headings */
h1, h2, h3, h4, h5, h6 {
  line-height: var(--primitive-leading-tight);
  text-wrap: balance;
}

/* Prevent orphans on body copy */
p, li, figcaption {
  text-wrap: pretty;
}

/* Remove default anchor styles */
a {
  color: inherit;
  text-decoration: inherit;
}

/* Responsive media */
img,
picture,
video,
canvas,
svg {
  display: block;
  max-width: 100%;
}

/* Inherit fonts for form elements */
input,
button,
textarea,
select,
optgroup {
  font: inherit;
  color: inherit;
}

/* Remove default textarea resize (vertical only is fine) */
textarea {
  resize: vertical;
}

/* Avoid text overflows */
p,
h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}

/* Remove default fieldset styles */
fieldset {
  border: none;
}

/* Remove built-in form typography styles */
button,
input,
select,
textarea {
  background-color: transparent;
  border: 1px solid var(--color-border-default);
}

/* Remove default button styles */
button {
  cursor: pointer;
  border: none;
  background: none;
}

/* Table reset */
table {
  border-collapse: collapse;
  border-spacing: 0;
}

/* Address */
address {
  font-style: normal;
}

/* Abbr underline */
abbr[title] {
  text-decoration: underline dotted;
  cursor: help;
}

/* Code */
code, kbd, samp, pre {
  font-family: var(--font-mono);
  font-size: 0.9em;
}

pre {
  overflow: auto;
  white-space: pre-wrap;
}

/* HR */
hr {
  border: none;
  border-top: 1px solid var(--divider-color);
  margin-block: var(--space-lg);
}

/* Hidden attribute */
[hidden] {
  display: none !important;
}

/* Remove animations for people who prefer reduced motion — see section 11 */
/* (defined separately to avoid cascade issues) */

/* Stacking context: create one for the root element */
#root,
#__next {
  isolation: isolate;
}


/* ============================================================
   6. BASE DOCUMENT STYLES
   ============================================================ */

html {
  font-size: 100%; /* 16px base */
}

body {
  background-color: var(--color-bg-base);
  color: var(--color-text-primary);
  font-family: var(--font-body);
  font-size: 1rem;
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-relaxed);
}

/* Base link styles (semantic, not reset) */
a:not([class]) {
  color: var(--color-text-brand);
  text-decoration: underline;
  text-underline-offset: 0.2em;
  transition: color var(--duration-fast) var(--ease-default);
}

a:not([class]):hover {
  color: var(--color-brand-primary-hover);
}

a:not([class]):visited {
  color: var(--primitive-sienna-600);
}

/* Selection */
::selection {
  background-color: var(--primitive-sienna-200);
  color: var(--primitive-sienna-900);
}


/* ============================================================
   7. TYPOGRAPHY UTILITIES
   Complete type scale with real clamp() values.
   Minimum viewport: 375px (23.4375rem)
   Maximum viewport: 1440px (90rem)
   ============================================================ */

/* --- Display headings (Playfair Display) --- */

.text-display-2xl {
  /* 72px → 120px */
  font-size: clamp(4.5rem, 2.864rem + 6.982vw, 7.5rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-none);
  letter-spacing: var(--primitive-tracking-tighter);
}

.text-display-xl {
  /* 56px → 96px */
  font-size: clamp(3.5rem, 2.273rem + 5.236vw, 6rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-none);
  letter-spacing: var(--primitive-tracking-tight);
}

.text-display-lg {
  /* 48px → 72px */
  font-size: clamp(3rem, 2.264rem + 3.142vw, 4.5rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-tight);
  letter-spacing: var(--primitive-tracking-tight);
}

/* --- Heading scale (Playfair Display) --- */

.text-h1 {
  /* 36px → 60px */
  font-size: clamp(2.25rem, 1.514rem + 3.142vw, 3.75rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-tight);
  letter-spacing: var(--primitive-tracking-tight);
}

.text-h2 {
  /* 30px → 48px */
  font-size: clamp(1.875rem, 1.321rem + 2.356vw, 3rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-snug);
  letter-spacing: var(--primitive-tracking-tight);
}

.text-h3 {
  /* 24px → 36px */
  font-size: clamp(1.5rem, 1.132rem + 1.571vw, 2.25rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-semibold);
  line-height: var(--primitive-leading-snug);
  letter-spacing: var(--primitive-tracking-normal);
}

.text-h4 {
  /* 20px → 28px */
  font-size: clamp(1.25rem, 1.004rem + 1.047vw, 1.75rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-semibold);
  line-height: var(--primitive-leading-snug);
  letter-spacing: var(--primitive-tracking-normal);
}

.text-h5 {
  /* 18px → 22px */
  font-size: clamp(1.125rem, 1.002rem + 0.524vw, 1.375rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-medium);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-normal);
}

.text-h6 {
  /* 16px → 18px */
  font-size: clamp(1rem, 0.938rem + 0.262vw, 1.125rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-medium);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-wide);
}

/* Apply heading classes to semantic elements by default */
h1 { font-size: clamp(2.25rem, 1.514rem + 3.142vw, 3.75rem);  font-family: var(--font-display); font-weight: var(--primitive-weight-bold);    line-height: var(--primitive-leading-tight); letter-spacing: var(--primitive-tracking-tight); }
h2 { font-size: clamp(1.875rem, 1.321rem + 2.356vw, 3rem);    font-family: var(--font-display); font-weight: var(--primitive-weight-bold);    line-height: var(--primitive-leading-snug);  letter-spacing: var(--primitive-tracking-tight); }
h3 { font-size: clamp(1.5rem, 1.132rem + 1.571vw, 2.25rem);   font-family: var(--font-display); font-weight: var(--primitive-weight-semibold); line-height: var(--primitive-leading-snug);  letter-spacing: var(--primitive-tracking-normal); }
h4 { font-size: clamp(1.25rem, 1.004rem + 1.047vw, 1.75rem);  font-family: var(--font-display); font-weight: var(--primitive-weight-semibold); line-height: var(--primitive-leading-snug);  letter-spacing: var(--primitive-tracking-normal); }
h5 { font-size: clamp(1.125rem, 1.002rem + 0.524vw, 1.375rem); font-family: var(--font-display); font-weight: var(--primitive-weight-medium);  line-height: var(--primitive-leading-normal); letter-spacing: var(--primitive-tracking-normal); }
h6 { font-size: clamp(1rem, 0.938rem + 0.262vw, 1.125rem);    font-family: var(--font-display); font-weight: var(--primitive-weight-medium);  line-height: var(--primitive-leading-normal); letter-spacing: var(--primitive-tracking-wide); }

/* --- Body scale (Lato) --- */

.text-body-xl {
  /* 20px → 24px */
  font-size: clamp(1.25rem, 1.127rem + 0.524vw, 1.5rem);
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-relaxed);
}

.text-body-lg {
  /* 18px → 20px */
  font-size: clamp(1.125rem, 1.063rem + 0.262vw, 1.25rem);
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-relaxed);
}

.text-body-md {
  /* 16px — base */
  font-size: 1rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-relaxed);
}

.text-body-sm {
  /* 14px */
  font-size: 0.875rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-normal);
}

.text-body-xs {
  /* 12px */
  font-size: 0.75rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-normal);
}

/* --- UI text (Lato — labels, caps, overlines) --- */

.text-label-lg {
  font-size: 0.875rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-semibold);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-wide);
}

.text-label-md {
  font-size: 0.8125rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-semibold);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-wide);
}

.text-label-sm {
  font-size: 0.75rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-semibold);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-wider);
}

.text-overline {
  font-size: 0.6875rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-bold);
  line-height: var(--primitive-leading-normal);
  letter-spacing: var(--primitive-tracking-widest);
  text-transform: uppercase;
}

.text-caption {
  font-size: 0.75rem;
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-regular);
  line-height: var(--primitive-leading-normal);
  color: var(--color-text-secondary);
}

/* --- Prose (editorial copy) --- */

.text-lead {
  /* 18px → 22px */
  font-size: clamp(1.125rem, 1.002rem + 0.524vw, 1.375rem);
  font-family: var(--font-body);
  font-weight: var(--primitive-weight-light);
  line-height: var(--primitive-leading-relaxed);
  color: var(--color-text-secondary);
}

.text-quote {
  /* 20px → 28px */
  font-size: clamp(1.25rem, 1.004rem + 1.047vw, 1.75rem);
  font-family: var(--font-display);
  font-weight: var(--primitive-weight-regular);
  font-style: italic;
  line-height: var(--primitive-leading-relaxed);
  color: var(--color-text-brand);
}

/* --- Modifier utilities --- */

.font-display  { font-family: var(--font-display); }
.font-body     { font-family: var(--font-body); }
.font-mono     { font-family: var(--font-mono); }

.font-light     { font-weight: var(--primitive-weight-light); }
.font-regular   { font-weight: var(--primitive-weight-regular); }
.font-medium    { font-weight: var(--primitive-weight-medium); }
.font-semibold  { font-weight: var(--primitive-weight-semibold); }
.font-bold      { font-weight: var(--primitive-weight-bold); }
.font-extrabold { font-weight: var(--primitive-weight-extrabold); }
.font-black     { font-weight: var(--primitive-weight-black); }

.text-italic   { font-style: italic; }
.text-normal   { font-style: normal; }
.text-balance  { text-wrap: balance; }
.text-pretty   { text-wrap: pretty; }
.text-truncate { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.text-uppercase { text-transform: uppercase; }
.text-lowercase { text-transform: lowercase; }
.text-capitalize { text-transform: capitalize; }

.text-left    { text-align: left; }
.text-center  { text-align: center; }
.text-right   { text-align: right; }

.text-color-primary   { color: var(--color-text-primary); }
.text-color-secondary { color: var(--color-text-secondary); }
.text-color-tertiary  { color: var(--color-text-tertiary); }
.text-color-disabled  { color: var(--color-text-disabled); }
.text-color-brand     { color: var(--color-text-brand); }
.text-color-inverse   { color: var(--color-text-inverse); }


/* ============================================================
   8. LAYOUT UTILITIES
   ============================================================ */

/* --- Container --- */

.container {
  width: 100%;
  max-width: var(--container-xl);  /* 1280px default */
  margin-inline: auto;
  padding-inline: var(--container-padding-x);
}

.container-sm  { max-width: var(--container-sm); }
.container-md  { max-width: var(--container-md); }
.container-lg  { max-width: var(--container-lg); }
.container-xl  { max-width: var(--container-xl); }
.container-2xl { max-width: var(--container-2xl); }
.container-full { max-width: 100%; }

/* --- Section --- */

.section {
  /* 48px → 120px vertical padding, fluid */
  padding-block: clamp(3rem, 1.909rem + 4.655vw, 7.5rem);
}

.section-sm {
  /* 32px → 64px */
  padding-block: clamp(2rem, 1.273rem + 3.107vw, 4rem);
}

.section-lg {
  /* 64px → 160px */
  padding-block: clamp(4rem, 1.818rem + 9.310vw, 10rem);
}

.section-hero {
  /* 80px → 200px */
  padding-block: clamp(5rem, 2.273rem + 11.643vw, 12.5rem);
}

/* --- Flow --- */

.flow > * + * {
  margin-block-start: var(--flow-space, var(--space-md));
}

.flow-tight > * + * {
  margin-block-start: var(--flow-space, var(--space-sm));
}

.flow-loose > * + * {
  margin-block-start: var(--flow-space, var(--space-xl));
}

/* --- Grid --- */

.grid {
  display: grid;
  gap: var(--grid-gap, var(--space-lg));
}

.grid-2 { grid-template-columns: repeat(2, 1fr); }
.grid-3 { grid-template-columns: repeat(3, 1fr); }
.grid-4 { grid-template-columns: repeat(4, 1fr); }

.grid-auto-sm { grid-template-columns: repeat(auto-fill, minmax(min(100%, 16rem), 1fr)); }
.grid-auto-md { grid-template-columns: repeat(auto-fill, minmax(min(100%, 22rem), 1fr)); }
.grid-auto-lg { grid-template-columns: repeat(auto-fill, minmax(min(100%, 30rem), 1fr)); }

/* --- Flex helpers --- */

.flex         { display: flex; }
.flex-col     { display: flex; flex-direction: column; }
.flex-center  { display: flex; align-items: center; justify-content: center; }
.flex-between { display: flex; align-items: center; justify-content: space-between; }
.flex-start   { display: flex; align-items: center; justify-content: flex-start; }
.flex-end     { display: flex; align-items: center; justify-content: flex-end; }
.flex-wrap    { flex-wrap: wrap; }
.flex-1       { flex: 1 1 0%; }
.flex-auto    { flex: 1 1 auto; }
.flex-none    { flex: none; }
.items-start   { align-items: flex-start; }
.items-center  { align-items: center; }
.items-end     { align-items: flex-end; }
.items-stretch { align-items: stretch; }

/* --- Spacing helpers --- */

.gap-xs  { gap: var(--space-xs); }
.gap-sm  { gap: var(--space-sm); }
.gap-md  { gap: var(--space-md); }
.gap-lg  { gap: var(--space-lg); }
.gap-xl  { gap: var(--space-xl); }
.gap-2xl { gap: var(--space-2xl); }

/* --- Stack (vertical rhythm) --- */

.stack      { display: flex; flex-direction: column; gap: var(--space-md); }
.stack-sm   { display: flex; flex-direction: column; gap: var(--space-sm); }
.stack-lg   { display: flex; flex-direction: column; gap: var(--space-xl); }

/* --- Cluster (horizontal wrap) --- */

.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-sm);
  align-items: center;
}

/* --- Sidebar layout --- */

.sidebar-layout {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-xl);
}

.sidebar-layout > :first-child {
  flex-basis: 300px;
  flex-grow: 1;
}

.sidebar-layout > :last-child {
  flex-basis: 0;
  flex-grow: 999;
  min-inline-size: 50%;
}

/* --- Cover (full-height centered content) --- */

.cover {
  display: flex;
  flex-direction: column;
  min-block-size: 100vh;
}

.cover > :only-child,
.cover > [data-cover-center] {
  margin-block: auto;
}

/* --- Bleed (full-width within a container) --- */

.bleed {
  margin-inline: calc(var(--container-padding-x) * -1);
}

/* --- Visibility --- */

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

.not-sr-only {
  position: static;
  width: auto;
  height: auto;
  padding: 0;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
}


/* ============================================================
   9. FOCUS & ACCESSIBILITY UTILITIES
   ============================================================ */

/* --- Default focus-visible ring --- */

:focus {
  outline: none;
}

:focus-visible {
  outline: 3px solid var(--color-focus-ring);
  outline-offset: 3px;
  border-radius: var(--radius-sm);
}

/* --- Interactive element base states --- */

[role="button"],
button,
a,
input,
select,
textarea,
[tabindex] {
  transition: outline var(--duration-fast) var(--ease-default);
}

/* Skip to main content link */
.skip-link {
  position: absolute;
  top: 0;
  left: 0;
  z-index: var(--primitive-z-toast);
  padding: var(--space-sm) var(--space-md);
  background-color: var(--color-bg-elevated);
  color: var(--color-text-primary);
  font-weight: var(--primitive-weight-bold);
  border-radius: 0 0 var(--radius-md) 0;
  border: 2px solid var(--color-border-brand);
  transform: translateY(-100%);
  transition: transform var(--duration-fast) var(--ease-out);
  text-decoration: none;
}

.skip-link:focus-visible {
  transform: translateY(0);
}

/* --- Disabled states --- */

[disabled],
[aria-disabled="true"] {
  cursor: not-allowed;
  opacity: 0.5;
  pointer-events: none;
}

[aria-disabled="true"][tabindex] {
  pointer-events: auto; /* Allow focus even when disabled for keyboard nav */
}

/* --- Current/selected states --- */

[aria-current="page"],
[aria-selected="true"] {
  color: var(--color-text-brand);
  font-weight: var(--primitive-weight-semibold);
}

/* --- Loading state --- */

[aria-busy="true"] {
  cursor: progress;
}

/* --- High-contrast mode adjustments --- */

@media (forced-colors: active) {
  :focus-visible {
    outline: 3px solid ButtonText;
    outline-offset: 3px;
  }

  .skip-link {
    border: 2px solid ButtonText;
  }
}

/* --- Keyboard navigation helpers --- */

.focus-within\:ring:focus-within {
  outline: 3px solid var(--color-focus-ring);
  outline-offset: 3px;
  border-radius: var(--radius-sm);
}

/* Interactive utility */
.interactive {
  cursor: pointer;
  transition:
    opacity var(--duration-fast) var(--ease-default),
    transform var(--duration-fast) var(--ease-default);
}

.interactive:hover  { opacity: 0.85; }
.interactive:active { transform: scale(0.98); }

/* --- ARIA live region utility --- */

.live-region {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  white-space: nowrap;
}


/* ============================================================
   10. ANIMATION & TRANSITION UTILITIES
   ============================================================ */

.transition-fast    { transition-duration: var(--duration-fast);   transition-timing-function: var(--ease-default); }
.transition-normal  { transition-duration: var(--duration-normal); transition-timing-function: var(--ease-default); }
.transition-slow    { transition-duration: var(--duration-slow);   transition-timing-function: var(--ease-default); }

.transition-colors  {
  transition-property: color, background-color, border-color, fill, stroke;
  transition-duration: var(--duration-fast);
  transition-timing-function: var(--ease-default);
}

.transition-transform {
  transition-property: transform;
  transition-duration: var(--duration-normal);
  transition-timing-function: var(--ease-default);
}

.transition-opacity {
  transition-property: opacity;
  transition-duration: var(--duration-normal);
  transition-timing-function: var(--ease-default);
}

.transition-shadow {
  transition-property: box-shadow;
  transition-duration: var(--duration-normal);
  transition-timing-function: var(--ease-default);
}

/* Entrance animations */
@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes slide-up {
  from { opacity: 0; transform: translateY(1.5rem); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes slide-down {
  from { opacity: 0; transform: translateY(-1.5rem); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes scale-in {
  from { opacity: 0; transform: scale(0.95); }
  to   { opacity: 1; transform: scale(1); }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.5; }
}

@keyframes shimmer {
  from { background-position: -200% center; }
  to   { background-position: 200% center; }
}

.animate-fade-in    { animation: fade-in    var(--duration-normal) var(--ease-enter) both; }
.animate-slide-up   { animation: slide-up   var(--duration-normal) var(--ease-enter) both; }
.animate-slide-down { animation: slide-down var(--duration-normal) var(--ease-enter) both; }
.animate-scale-in   { animation: scale-in   var(--duration-normal) var(--ease-enter) both; }
.animate-spin       { animation: spin var(--primitive-duration-slowest) var(--primitive-ease-linear) infinite; }
.animate-pulse      { animation: pulse 2s var(--primitive-ease-in-out) infinite; }


/* ============================================================
   11. REDUCED MOTION
   Must come after all animation/transition definitions
   to correctly override them.
   ============================================================ */

@media (prefers-reduced-motion: reduce) {
  /* Disable scroll behavior */
  html {
    scroll-behavior: auto;
  }

  /* Remove all transitions and animations */
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    transition-delay: 0ms !important;
    scroll-behavior: auto !important;
  }

  /* Allow essential animations at reduced intensity */
  .spinner,
  [role="progressbar"] {
    animation-duration: 2s !important;
    animation-iteration-count: infinite !important;
  }

  /* Disable parallax */
  .parallax {
    transform: none !important;
  }

  /* Skip link still needs transform for functionality */
  .skip-link {
    transition: none !important;
  }

  .skip-link:focus-visible {
    transform: translateY(0) !important;
    transition: none !important;
  }
}

/* Honour explicit user preferences when JavaScript toggles this */
[data-reduce-motion="true"] *,
[data-reduce-motion="true"] *::before,
[data-reduce-motion="true"] *::after {
  animation-duration: 0.01ms !important;
  animation-iteration-count: 1 !important;
  transition-duration: 0.01ms !important;
  transition-delay: 0ms !important;
}


/* ============================================================
   12. COMPONENT BASE STYLES
   Minimal, token-driven base — not a full component library.
   ============================================================ */

/* --- Button base --- */

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs);
  padding: var(--btn-padding-y) var(--btn-padding-x);
  font-family: var(--btn-font-family);
  font-weight: var(--btn-font-weight);
  font-size: var(--btn-font-size);
  letter-spacing: var(--btn-letter-spacing);
  text-transform: uppercase;
  border-radius: var(--btn-radius);
  border: 1px solid transparent;
  cursor: pointer;
  text-decoration: none;
  white-space: nowrap;
  transition: var(--btn-transition);
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

.btn:focus-visible {
  outline: 3px solid var(--color-focus-ring);
  outline-offset: 3px;
}

.btn-primary {
  background-color: var(--btn-bg-primary);
  color: var(--btn-text-primary);
  border-color: var(--btn-border-primary);
}

.btn-primary:hover:not([disabled]):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-primary-hover);
}

.btn-primary:active:not([disabled]):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-primary-active);
}

.btn-secondary {
  background-color: var(--btn-bg-secondary);
  color: var(--btn-text-secondary);
  border-color: var(--btn-border-secondary);
}

.btn-secondary:hover:not([disabled]):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-secondary-hover);
}

.btn-ghost {
  background-color: var(--btn-bg-ghost);
  color: var(--btn-text-ghost);
  border-color: var(--btn-border-ghost);
}

.btn-ghost:hover:not([disabled]):not([aria-disabled="true"]) {
  background-color: var(--btn-bg-ghost-hover);
}

.btn-sm {
  padding: var(--primitive-space-2) var(--primitive-space-4);
  font-size: 0.75rem;
}

.btn-lg {
  padding: var(--primitive-space-4) var(--primitive-space-8);
  font-size: 1rem;
}

/* --- Card base --- */

.card {
  background-color: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: var(--card-radius);
  box-shadow: var(--card-shadow);
  padding: var(--card-padding);
  transition: box-shadow var(--duration-normal) var(--ease-default);
}

.card-hover:hover {
  box-shadow: var(--card-shadow-hover);
}

/* --- Divider ornamental --- */

.divider-ornamental {
  display: flex;
  align-items: center;
  gap: var(--space-md);
  color: var(--divider-color-ornamental);
  margin-block: var(--space-xl);
}

.divider-ornamental::before,
.divider-ornamental::after {
  content: '';
  flex: 1;
  height: 1px;
  background-color: currentColor;
  opacity: 0.3;
}

/* --- Badge base --- */

.badge {
  display: inline-flex;
  align-items: center;
  padding: var(--badge-padding-y) var(--badge-padding-x);
  border-radius: var(--badge-radius);
  font-size: var(--badge-font-size);
  font-weight: var(--badge-font-weight);
  letter-spacing: var(--badge-letter-spacing);
  line-height: 1;
  text-transform: uppercase;
}

.badge-brand {
  background-color: var(--color-brand-primary-subtle);
  color: var(--color-brand-primary);
}

.badge-secondary {
  background-color: var(--color-brand-secondary-subtle);
  color: var(--color-brand-secondary);
}

/* --- Loading skeleton --- */

.skeleton {
  background: linear-gradient(
    90deg,
    var(--color-bg-sunken) 25%,
    var(--color-border-subtle) 50%,
    var(--color-bg-sunken) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: var(--radius-sm);
}
```

---

## Clamp() Values Reference

All `clamp()` values use the formula:

```
clamp(min, preferred, max)
preferred = min + (max - min) * (100vw - minVW) / (maxVW - minVW)
```

- **minVW** = 375px = 23.4375rem  
- **maxVW** = 1440px = 90rem  

| Class | Min | Max | Preferred (approx) |
|---|---|---|---|
| `.text-display-2xl` | 72px | 120px | `2.864rem + 6.982vw` |
| `.text-display-xl` | 56px | 96px | `2.273rem + 5.236vw` |
| `.text-display-lg` | 48px | 72px | `2.264rem + 3.142vw` |
| `.text-h1` | 36px | 60px | `1.514rem + 3.142vw` |
| `.text-h2` | 30px | 48px | `1.321rem + 2.356vw` |
| `.text-h3` | 24px | 36px | `1.132rem + 1.571vw` |
| `.text-h4` | 20px | 28px | `1.004rem + 1.047vw` |
| `.text-h5` | 18px | 22px | `1.002rem + 0.524vw` |
| `.text-h6` | 16px | 18px | `0.938rem + 0.262vw` |
| `.section` | 48px | 120px | `1.909rem + 4.655vw` |
| `.section-sm` | 32px | 64px | `1.273rem + 3.107vw` |
| `.section-lg` | 64px | 160px | `1.818rem + 9.310vw` |
| `.section-hero` | 80px | 200px | `2.273rem + 11.643vw` |

---

## Usage Notes

### Font loading (add to `<head>`)
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;0,800;0,900;1,400;1,700&family=Lato:ital,wght@0,300;0,400;0,700;0,900;1,300;1,400&display=swap" rel="stylesheet">
```

### Dark mode toggle (JavaScript)
```js
document.documentElement.dataset.theme = 'dark'; // or 'light'
```

### Skip link (add as first element in `<body>`)
```html
<a href="#main-content" class="skip-link">Skip to main content</a>
<main id="main-content">...</main>
```

### Next.js (app router) import
```js
// app/layout.tsx
import './globals.css';
```
