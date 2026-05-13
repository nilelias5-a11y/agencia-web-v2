# UX Design — Ana López Fotógrafa

**Project type:** One-pager portfolio + contact  
**Client:** Ana López, fotógrafa freelance  
**Primary conversions:** (a) contacto directo, (b) reconocimiento de estilo a través del portfolio  
**Site model:** Single-page application with anchor navigation  
**Skill applied:** ux-designer  
**Date:** 2026-05-13  

---

## LEARNING SYSTEM — Context Note

No prior memory files found (`global-memory.md`, `ux-designer-memory.md`). This is the first project in this workspace. Decisions are based on established UX patterns for freelance photography portfolios. Learnings from this project should be written to `ux-designer-memory.md` after client approval.

**Applicable pattern knowledge (pre-loaded):**
- Portfolio one-pagers perform best when the hero is image-dominant and the CTA appears within the first viewport
- Photographers lose leads when the contact form is buried below the fold on mobile
- Gallery grids that use masonry or asymmetric layouts signal creative confidence; rigid grids signal corporate — mismatch for a freelance photographer
- Social proof (testimonials or client logos) placed directly above the contact form increases form submissions
- Bio sections placed mid-page (between gallery and contact) outperform bio-first ordering for photographers, because users qualify the work before they qualify the person

---

## PHASE 1 — Sitemap

### Sitemap — Ana López Fotógrafa

> NOTE: This is a one-pager. "Pages" are anchor sections within `/`. URL slugs reflect anchor IDs, not routes. Utility pages are real routes.

---

#### Primary Navigation (anchor links within `/`)

| Anchor | Section name | User goal on arrival | Primary CTA | Priority |
|--------|-------------|----------------------|-------------|----------|
| `/#inicio` | Hero / Portada | Understand who Ana is and whether her style matches the visitor's need | Scroll to portfolio | Critical |
| `/#portfolio` | Portfolio | See representative work, identify style, self-qualify | Contact Ana | Critical |
| `/#sobre-mi` | Sobre mí / Bio | Validate Ana as a professional, build trust | View portfolio / Contact | Important |
| `/#servicios` | Servicios | Understand what Ana offers and at what scope | Contact Ana | Important |
| `/#testimonios` | Testimonios | Social proof before committing to contact | Contact Ana | Important |
| `/#contacto` | Contacto | Submit inquiry | Send message | Critical |

---

#### Footer Navigation

| URL | Page name | Purpose |
|-----|-----------|---------|
| `/privacidad` | Política de Privacidad | Legal — GDPR compliance for contact form |
| `/cookies` | Política de Cookies | Legal — if analytics used |
| `mailto:ana@analopezfoto.com` | Email directo | Alternative contact path |
| Social links (Instagram, LinkedIn) | External profiles | Portfolio reinforcement |

---

#### Mobile Navigation

- Desktop: Horizontal nav bar with anchor links, sticky on scroll, transparent-to-solid on scroll
- Mobile: Hamburger icon (top-right) opens full-screen overlay menu with large tap targets (min 48px)
- Mobile menu order mirrors section order: Inicio → Portfolio → Sobre mí → Servicios → Testimonios → Contacto
- On mobile, nav collapses to logo + hamburger only; no inline links visible
- Active anchor state: nav link corresponding to current section gets underline indicator (intersection observer)

---

#### Hidden / Utility Pages

| URL | Purpose |
|-----|---------|
| `/exito` | Form submission confirmation page (redirect after successful send) |
| `/privacidad` | Privacy policy — linked from form and footer |
| `/404` | Not Found page with link back to `/#inicio` |

---

#### Page-level metadata

| Section | Primary user goal | Primary CTA | Priority |
|---------|------------------|-------------|----------|
| Hero `/#inicio` | Qualify aesthetic match immediately | "Ver portfolio" → `/#portfolio` | Critical |
| Portfolio `/#portfolio` | Browse work, assess style breadth | "Hablemos" → `/#contacto` | Critical |
| Sobre mí `/#sobre-mi` | Trust-building, humanization | "Ver mis trabajos" → `/#portfolio` or "Contacta conmigo" → `/#contacto` | Important |
| Servicios `/#servicios` | Understand offering scope | "Solicita presupuesto" → `/#contacto` | Important |
| Testimonios `/#testimonios` | Validate credibility via third-party | "Contactar" → `/#contacto` | Important |
| Contacto `/#contacto` | Submit inquiry | "Enviar mensaje" → form submit → `/exito` | Critical |

---

## PHASE 2 — Conversion Flows

### Conversion Flow A — Primary: Contact Inquiry (Lead Generation)

**Trigger:** User arrives at `/#inicio` from Instagram bio link, Google search, or word-of-mouth referral  
**Goal:** User submits the contact form at `/#contacto`

```
Step 1: /#inicio (Hero)
        User sees → Full-viewport hero image + headline defining Ana's photographic style
        Decision point → "Does this style match what I need?"
        YES → Clicks "Ver portfolio" or scrolls naturally
        NO → Exits (no recovery possible if style is genuinely wrong)

Step 2: /#portfolio (Portfolio grid)
        User sees → Curated selection (12–18 images) in masonry grid, organized by category tabs
        Decision point → "Is the quality and style consistently what I want?"
        YES → Continues scrolling or clicks "Hablemos" CTA
        MAYBE → Scrolls to Bio to validate professionalism before committing

Step 3: /#sobre-mi (Bio)
        User sees → Professional photo of Ana, short biography, values, approach to photography
        Decision point → "Do I trust this person to handle my project?"
        YES → Scrolls to Servicios or jumps to Contacto
        UNCERTAIN → Reads Testimonios

Step 4: /#servicios (Services)
        User sees → Service categories, brief description, scope indicators (no prices)
        Decision point → "Does Ana do what I need?"
        YES → Scrolls toward Contacto
        NO → Exits

Step 5: /#testimonios (Social proof)
        User sees → 3–5 testimonials from past clients, with name and context
        Decision point → "Am I confident enough to reach out?"
        YES → Clicks CTA or scrolls to form

Step 6: /#contacto (Contact form)
        User sees → Short form (name, email, type of project, message, privacy checkbox)
        Action → Fills form, submits
        Conversion → Redirect to /exito
```

**Drop-off risks:**
- Hero: Style mismatch — unavoidable but minimized by strong hero image curation
- Portfolio: Cognitive overload if too many images — capped at 18 visible, "ver más" loads additional
- Contact form: Friction from too many fields — max 5 fields, all justified
- Contact form: Privacy anxiety — privacy link visible next to checkbox

**Recovery mechanisms:**
- Sticky nav: User can jump to `/#contacto` from any point in the page
- Sticky "Contactar" CTA button appears on mobile after scrolling past hero (floating pill button, bottom-right)
- Portfolio "Hablemos" CTA repeats after gallery — captures users who decide mid-browse
- Testimonios CTA directly above the form — last conversion push before the action

---

### Conversion Flow B — Secondary: Portfolio Recognition & Social Follow

**Trigger:** User arrives from Instagram or direct referral specifically to see work  
**Goal:** User follows Ana on Instagram or saves the website for later referral

```
Step 1: /#inicio → style impression formed
Step 2: /#portfolio → deep browse, possibly opens image lightbox
        Decision → "I want to see more / follow her work"
        → Clicks Instagram link in portfolio section or footer
        Secondary conversion achieved: Instagram follow
```

**Drop-off risks:** User browses without engaging if no Instagram CTA is visible near portfolio  
**Recovery:** Instagram icon/link placed at bottom of portfolio section and in footer

---

### Conversion Flow C — Re-entry (Returning Visitor Who Did Not Convert)

**Trigger:** User returns directly to `analopezfoto.com` after earlier visit  
**Goal:** Push toward contact this time

```
Step 1: Lands on /#inicio
Step 2: Jumps directly to /#contacto via sticky nav (returning user knows the site)
Step 3: Form submission → /exito
```

**Recovery mechanism:** No new content needed for re-entry — the form must be immediately reachable via navigation on every return visit. Sticky nav is the critical mechanism here.

---

## PHASE 3 — Section-by-Section Wireframes

## Landing Page — / (Single Page)

**Page purpose:** Convert photography prospects into qualified leads while establishing Ana's visual identity and professional credibility  
**Primary CTA:** Submit contact form  
**Entry sources:** Instagram bio link, Google search (name + "fotógrafa"), direct referrals, word of mouth

---

### Section 1 — Hero / Portada (`#inicio`)

**Layout:** Full-viewport (100vh), full-bleed background image, content overlay centered or bottom-left aligned  
**Content:**
- Background: Single hero photograph (Ana's best/most representative work) — fills entire viewport, no whitespace margins
- Logo / Name: "Ana López" — typeset as wordmark, top-left, white or light tone depending on image
- Navigation bar: Horizontal anchor links (Portfolio · Sobre mí · Servicios · Contacto), top-right, white text, transparent background
- Headline: Short identity statement — approximately 40–60 characters — e.g., "Fotografía que cuenta historias reales." — tone: confident, personal, evocative
- Subheadline (optional): 1 line, ~80 characters — clarifies specialty — e.g., "Retratos, bodas y sesiones de marca en Barcelona."
- Primary CTA button: "Ver portfolio" — links to `#portfolio` — white filled button or outlined, high contrast on image

**Primary CTA:** "Ver portfolio" → `/#portfolio`  
**Secondary CTA:** Scroll indicator arrow (animated) → implicit scroll to next section

**Mobile behavior:**
- Hero image remains full-viewport; portrait crops preferred for mobile (object-position: top)
- Logo scales down, remains top-left
- Hamburger replaces nav links, top-right
- Headline font size reduces (fluid type: clamp 2rem–4rem)
- CTA button remains visible and tappable (min 48px height, min 200px width)
- Subheadline may be hidden on smallest screens (below 375px) to avoid crowding
- Scroll indicator remains visible

**Animation hint:** Hero image fade-in on load (300ms delay, 600ms duration). Headline and CTA slide up on load (staggered, 400ms offset from image). No parallax on mobile.

**Conversion notes:** The hero's sole job is the aesthetic handshake — it must answer "is this person's style right for me?" in under 3 seconds. The CTA leads deeper rather than to contact, because users who haven't seen work yet are not ready to commit. Forcing contact before showing work increases bounce rate.

**Accessibility:**
- Hero image has `role="img"` and `aria-label` describing the photograph (e.g., "Retrato en exteriores de una pareja en el barrio de Gràcia, Barcelona")
- CTA button has descriptive label, not just "click here"
- Navigation links have `aria-label="Navegación principal"`
- Color contrast of text overlay meets WCAG AA (4.5:1 minimum) — use semi-transparent dark scrim behind text if needed
- Skip navigation link: `<a href="#portfolio" class="skip-link">Ir al contenido principal</a>` — visually hidden, visible on focus

---

### Section 2 — Portfolio (`#portfolio`)

**Layout:** Masonry grid (2 columns desktop, 1–2 columns mobile) with category filter tabs above  
**Content:**
- Section heading: "Portfolio" — left-aligned, minimal — approximately 10 characters
- Filter tabs: pill-style tabs for categories — e.g., "Todos · Retratos · Bodas · Marca · Eventos" — default state: "Todos" selected
- Image grid: 12 images visible on load; masonry layout (variable heights); images are the sole content — no captions visible by default
- Each image: hover state reveals overlay with brief project name (e.g., "Boda civil · 2025") and a subtle zoom effect
- Lightbox: clicking any image opens full-screen lightbox with navigation arrows, project name, close button
- "Ver más trabajos" button: below grid — loads 6 additional images (progressive disclosure); hidden once all loaded
- Instagram CTA: below grid — "Sígueme en Instagram para ver más" — text link with Instagram icon

**Primary CTA:** "Hablemos" button — appears at bottom of section, after all images — links to `#contacto`  
**Secondary CTA:** "Sígueme en Instagram" text link — opens Instagram in new tab

**Mobile behavior:**
- Filter tabs: horizontally scrollable row (overflow-x: auto, no scrollbar visible), snaps to active tab
- Grid: 2-column masonry on screens ≥ 375px; single column on screens < 375px
- Hover overlays become tap: first tap shows overlay, second tap opens lightbox
- Lightbox: full-screen with swipe-to-navigate gesture support
- "Hablemos" CTA remains full-width button
- Instagram CTA remains below

**Animation hint:** Grid items fade in staggered on section entry (intersection observer, 80ms delay between items). Filter tab switch: crossfade outgoing/incoming images (200ms). Lightbox: scale-up from thumbnail position.

**Conversion notes:** Masonry signals creativity and variety. Category filters allow users to self-select relevant work (a wedding client needs to see weddings, not brand photography). The "Hablemos" CTA after the gallery is the primary conversion trigger — users who scroll through the full portfolio are warm leads. Instagram link captures users not yet ready to commit but wanting to stay connected.

**Accessibility:**
- Each image has descriptive `alt` text (e.g., "Retrato de novia en jardín, luz natural, Barcelona 2025")
- Lightbox traps focus while open; closes on Escape key
- Lightbox navigation arrows have `aria-label="Imagen anterior"` / `aria-label="Imagen siguiente"`
- Filter tabs use `role="tablist"` and `role="tab"` with `aria-selected` state
- Keyboard users can navigate the grid with Tab; Enter/Space opens lightbox
- "Ver más" button has `aria-label="Cargar más imágenes del portfolio"`

---

### Section 3 — Sobre mí (`#sobre-mi`)

**Layout:** Split 50-50 — left: professional photo of Ana; right: bio text block  
**Content:**
- Left column: Portrait photo of Ana — professional but warm, not corporate — full-height of section, object-fit cover
- Right column:
  - Section eyebrow label: "Sobre mí" — small caps, muted color
  - Headline: "Hola, soy Ana." — ~15 characters — warm, direct tone
  - Body copy: 3 short paragraphs (~250 words total):
    - Para. 1: Background — how she came to photography, what drives her
    - Para. 2: Style and approach — what makes her work distinctive
    - Para. 3: Personal details — location (Barcelona), availability, what types of projects she loves
  - Secondary CTA: "Contacta conmigo" → `#contacto` — ghost button style

**Primary CTA:** "Contacta conmigo" → `#contacto`  
**Secondary CTA:** None additional

**Mobile behavior:**
- Layout stacks vertically: photo on top (aspect ratio 4:3, full-width), text block below
- Photo height fixed at ~280px on mobile (not full-height of section)
- All three paragraphs remain visible — no truncation
- CTA button becomes full-width

**Animation hint:** On section entry, left image slides in from left (translateX -40px → 0, 500ms ease-out), right text block fades in with slight upward drift (400ms, 100ms delay).

**Conversion notes:** The bio is placed after the portfolio intentionally — users have already seen the work and are now validating the person behind it. Headline uses first name and direct address to humanize immediately. Three paragraphs cover the three trust questions: who, why, and for whom. Ghost CTA is softer than the primary filled button — appropriate here since the harder conversion push is saved for after testimonials.

**Accessibility:**
- Ana's photo has `alt` text: "Ana López, fotógrafa freelance en Barcelona" — do not use decorative alt="" since the image is trust-building content
- Body text minimum 16px (1rem), line-height ≥ 1.6
- Section heading level: `<h2>` for "Sobre mí"
- CTA button contrast meets WCAG AA

---

### Section 4 — Servicios (`#servicios`)

**Layout:** Centered content block, 3-column card grid (service categories)  
**Content:**
- Section heading: "Servicios" — centered, `<h2>`
- Subheading: 1 line — "Aquí tienes un resumen de lo que ofrezco. Si tienes algo en mente diferente, escríbeme." — ~100 characters — conversational, open
- Service cards (3 cards):
  - Card 1: **Retratos** — icon or small representative image — 2–3 sentence description — scope: individuals, couples, families — no price
  - Card 2: **Bodas & Eventos** — icon or image — 2–3 sentence description — scope: ceremonies, social events — no price
  - Card 3: **Fotografía de Marca** — icon or image — 2–3 sentence description — scope: businesses, products, personal branding — no price
- Note below cards: "No encuentras lo que buscas? Escríbeme y lo hablamos." — text link to `#contacto`
- Primary CTA: "Solicita presupuesto" → `#contacto` — filled button, centered below cards

**Primary CTA:** "Solicita presupuesto" → `#contacto`  
**Secondary CTA:** Text link "Escríbeme" → `#contacto`

**Mobile behavior:**
- 3-column card grid collapses to single column (cards stack vertically)
- Each card: full-width, compact padding (16px)
- Icons/images remain visible
- CTA button full-width

**Animation hint:** Cards fade in staggered on section entry (100ms offset each). No motion beyond fade.

**Conversion notes:** No prices listed — this is deliberate. Prices create friction before the lead has formed intent; Ana can discuss scope and pricing in the contact conversation. Cards are kept brief — the goal is to confirm "she does what I need" not to exhaustively describe methodology. The open-ended note below cards ("si tienes algo diferente") reduces hesitation from users whose project doesn't fit neatly into one category.

**Accessibility:**
- Service cards use `<article>` elements or `<li>` items in a `<ul>`
- If icons are used: decorative icons get `aria-hidden="true"`; if icons carry meaning, they get `aria-label`
- Card headings: `<h3>` (children of `<h2>` Servicios)
- Text contrast on card backgrounds meets WCAG AA

---

### Section 5 — Testimonios (`#testimonios`)

**Layout:** Centered, single-column testimonial carousel or static stacked cards (3 testimonials visible)  
**Content:**
- Section heading: "Lo que dicen" — centered, `<h2>`
- 3–5 testimonial blocks, each containing:
  - Pull quote: 2–4 sentences (~100–150 characters) — specific, results-focused — in quotation marks
  - Client name: First name + last initial (e.g., "Marta G.") — no full names without client consent
  - Context: 1 line — e.g., "Boda civil · Tarragona, 2024" or "Sesión de marca · Barcelona, 2025"
  - Optional: 5-star visual rating (decorative, not interactive)
- If carousel: visible navigation dots below, left/right arrow controls

**Primary CTA:** "Contactar" filled button — centered below testimonials → `#contacto`

**Mobile behavior:**
- Carousel preferred on mobile: one testimonial visible at a time, swipe to navigate
- Arrow controls remain tappable (min 44px touch target)
- Navigation dots visible below
- CTA button full-width, appears below carousel

**Animation hint:** Section fades in on entry. If carousel: slide/crossfade transition (300ms ease). Static: no animation beyond section entry fade.

**Conversion notes:** This section is the final trust-building moment before the contact form. Placement directly above the form is intentional — warm the user's confidence immediately before asking for action. Testimonials should be specific and reference outcomes (e.g., "Las fotos de nuestra boda son exactamente lo que imaginábamos") rather than generic praise ("¡Muy profesional!"). The CTA here is a direct and confident push — "Contactar" — because by this point the user has seen work, bio, services, and social proof.

**Accessibility:**
- Carousel has `aria-live="polite"` to announce slide changes to screen readers
- Each testimonial in carousel: `aria-label="Testimonio X de Y"` on the containing element
- Navigation arrows: `aria-label="Testimonio anterior"` / `aria-label="Siguiente testimonio"`
- Star ratings (if used): `aria-label="5 estrellas"` on the container, individual stars `aria-hidden="true"`
- Do not use autoplay on carousel — no unexpected content changes

---

### Section 6 — Contacto (`#contacto`)

**Layout:** Centered, max-width 600px, vertically stacked form  
**Content:**
- Section heading: "Hablemos" — `<h2>` — warm, not transactional
- Subheading: 1–2 lines — "Cuéntame tu proyecto y te respondo en menos de 48 horas." — ~80 characters — sets response expectation
- Contact form fields (5 fields maximum):
  1. **Nombre** — text input — required — placeholder: "Tu nombre"
  2. **Email** — email input — required — placeholder: "tu@email.com"
  3. **Tipo de proyecto** — select dropdown — required — options: "Retrato · Boda o evento · Fotografía de marca · Otro"
  4. **Mensaje** — textarea (4 rows) — required — placeholder: "Cuéntame qué tienes en mente..."
  5. **Privacidad checkbox** — required — label: "He leído y acepto la [Política de Privacidad]" — link opens `/privacidad` in new tab
- Submit button: "Enviar mensaje" — full-width — primary style
- Alternative contact line below form: "O escríbeme directamente a [ana@analopezfoto.com]" — mailto link

**Primary CTA:** "Enviar mensaje" → form submission → redirect `/exito`  
**Secondary CTA:** Direct email link `mailto:ana@analopezfoto.com`

**Mobile behavior:**
- Form is already single-column — no layout change needed
- All inputs full-width
- Submit button full-width, min 52px height for easy tap
- Floating "Contactar" pill button (bottom-right, position: fixed) that was visible from portfolio section onwards — disappears when user reaches this section (to avoid redundancy)
- Keyboard: on mobile, tapping an input scrolls form into view correctly (avoid `scroll-margin-top` issues with sticky nav)

**Animation hint:** Section fades in on entry. Form fields: no animation (stability is preferred for form interactions). Submit button: loading spinner state while sending. No animation on success (redirect to `/exito` is the feedback).

**Conversion notes:** 5 fields only — each is strictly necessary. Name and email are minimum viable. Project type helps Ana prioritize and personalize responses (and pre-qualifies the lead). The message is required — open-ended, low-friction placeholder. Checkbox is required for GDPR compliance. Response time promise in the subheading ("48 horas") reduces hesitation about whether the inquiry will be ignored. The direct email link is a fallback for users who distrust forms or prefer direct communication — don't hide it.

**Accessibility:**
- All inputs have explicit `<label>` elements — `for` attribute matching input `id` — no placeholder-only labeling
- Required fields have `aria-required="true"` and visual asterisk with `<span aria-hidden="true">*</span>` plus screen-reader text "(obligatorio)"
- Error messages: inline below each field on blur — role="alert" or `aria-live="polite"` — message: "Por favor, introduce un [nombre / email válido / etc.]"
- Submit button: `aria-label="Enviar formulario de contacto"` in addition to visible label
- Privacy link: opens in new tab — `rel="noopener noreferrer"` — announced to screen readers with "(abre en nueva pestaña)"
- Form group uses `<fieldset>` and `<legend>` if grouped, or individual labeled inputs
- On successful validation before submit: no screen reader announcement needed (redirect handles it)

---

### Section 7 — Footer

**Layout:** Full-width, dark background (or light — matches brand), centered or left-right split  
**Content:**
- Left: "Ana López" wordmark / logo — small version
- Center: Footer anchor links — Portfolio · Sobre mí · Contacto (abbreviated set)
- Right: Social icons — Instagram, LinkedIn (if applicable) — open in new tab
- Bottom row (smaller): "© 2026 Ana López · [Política de Privacidad] · [Política de Cookies]"
- Optional: "Diseñado por [Agencia]" credit — very small, low-contrast

**Primary CTA:** None — footer is navigation/legal only  
**Secondary CTA:** Social icons as passive exit points

**Mobile behavior:**
- Single-column stack: Logo → Social icons → Anchor links → Legal links
- All elements centered
- Adequate padding (min 24px vertical) so legal links are tappable

**Animation hint:** None — footer is static.

**Conversion notes:** Footer is not a conversion section. Its job is to provide legal links (GDPR requirement given the contact form), offer a final navigation shortcut, and reinforce brand identity. Keep it minimal.

**Accessibility:**
- Footer landmark: `<footer role="contentinfo">`
- Social links: `aria-label="Instagram de Ana López (abre en nueva pestaña)"` etc.
- Legal links: sufficient color contrast even if subdued styling
- Copyright text: not required to meet contrast threshold (it's decorative/legal boilerplate) — but prefer AA compliance regardless

---

### Utility Page — /exito (Form Confirmation)

**Layout:** Centered, minimal — logo + success message + CTA home  
**Content:**
- Logo / Name
- Heading: "¡Mensaje enviado!" — `<h1>`
- Body: "Gracias, [Nombre si available]. Te responderé en menos de 48 horas." — ~80 characters
- CTA: "Volver al inicio" → `/` — text or ghost button

**Mobile behavior:** Identical layout — already single-column  
**Accessibility:** `<h1>` confirms action completed; focus moved to heading on redirect for screen readers

---

**Page Notes:**
- Smooth scroll behavior is global (`scroll-behavior: smooth` in CSS; respect `prefers-reduced-motion`)
- Intersection Observer is used to: (a) update active nav link on scroll, (b) trigger section animations, (c) show/hide floating mobile CTA
- All section headings follow heading hierarchy: `<h1>` = Ana López (hero wordmark or visually as page title), `<h2>` = section names, `<h3>` = card/item titles within sections
- No autoplay video or audio — photography site, image-only hero
- Page title: `<title>Ana López — Fotógrafa Freelance en Barcelona</title>`
- Meta description: ~155 characters — includes location, specialty, CTA signal

---

## PHASE 4 — Global Interaction Principles

### Navigation Behavior

- **Desktop:** Sticky navigation bar — transparent over hero (text white with text-shadow), transitions to solid background (white or dark, per brand palette) after scrolling past 80% of hero height — transition: background 200ms ease
- **Mobile:** Hamburger icon (3 horizontal lines, top-right, 44×44px touch target) — opens full-screen overlay menu — overlay covers entire viewport, dark semi-transparent background — menu items centered vertically, large type (24px min), easy to tap
- **Active state:** Nav anchor link corresponding to the section currently in viewport receives an underline or accent-color indicator — powered by Intersection Observer with threshold 0.5
- **Scroll progress:** No scroll progress bar — adds visual noise for a portfolio aesthetic; the sticky nav + active state is sufficient orientation aid

---

### Scroll Behavior

- **Smooth scroll:** Yes — `scroll-behavior: smooth` on `<html>` element; additionally respect `prefers-reduced-motion: reduce` — if reduced motion preferred, disable smooth scroll and all transition animations
- **Section snapping:** No — snapping interrupts natural browsing of portfolio images and disrupts lightbox interactions; linear scroll only
- **Lazy loading:** Images — `loading="lazy"` on all portfolio grid images; hero image is `loading="eager"` (above fold); LCP image is preloaded via `<link rel="preload">`
- **Back-to-top:** Yes — appears after user scrolls past `#sobre-mi` (approximately 150% of viewport height) — floating button, bottom-right corner, below the mobile "Contactar" pill on mobile — icon: up arrow with `aria-label="Volver al inicio"` — disappears when user reaches `#contacto` (no longer needed)

---

### Forms

- **Validation:** On blur (field-by-field as user leaves each input) + final check on submit — do not validate while user is actively typing (frustrating UX)
- **Error messages:** Inline below each field — appear immediately on blur if invalid — color: error red with icon — text: specific and actionable (e.g., "El email no tiene un formato válido" not "Campo incorrecto")
- **Success state:** Page redirect to `/exito` — do not use inline success message on same page (risks user re-submitting form or confusion about what happened)
- **Required field indicator:** Asterisk (`*`) next to label — legend at top of form: "Los campos marcados con * son obligatorios" — screen reader text included

---

### Links & CTAs

- **External links:** Open in new tab (`target="_blank"`) — applies to: social media links, privacy policy link in form, any external portfolio platforms — always `rel="noopener noreferrer"`
- **Anchor links (internal):** Same tab — smooth scroll to section
- **CTA hierarchy:**
  - **Primary (filled button):** Main action of section — e.g., "Enviar mensaje", "Hablemos", "Solicita presupuesto" — used once per section maximum
  - **Secondary (ghost/outlined button):** Supporting action — e.g., "Contacta conmigo" in bio — used when two actions are both meaningful but one is softer
  - **Text link:** Tertiary — e.g., "Sígueme en Instagram", "Escríbeme directamente" — used for low-friction passive actions
  - **Floating pill (mobile):** Fixed positioned, persistent "Contactar" button — bottom-right — appears after hero on mobile only — primary style but condensed — disappears at `#contacto`
- **Hover states (universal rule):** All interactive elements (buttons, links, images in grid) have visible hover state — minimum: color shift or underline for text links; scale(1.02) + shadow elevation for cards; opacity overlay for images — transition: 150–200ms ease — no hover state on touch devices (use `:hover` media query guard or pointer-coarse detection)

---

### One-Pager Specific Interaction Principles

These principles apply uniquely to single-page scroll sites and supplement the global rules above:

1. **Navigation is orientation, not destination:** In a one-pager, the nav links do not navigate to pages — they orient the user within a long scroll. The active state must update in real time as the user scrolls, not only on click.

2. **Every CTA within the page leads to `#contacto`:** There are no internal links to other pages (except legal utilities). All CTAs must anchor to `#contacto` or to a section that leads there. Avoid dead-end sections.

3. **Section identity:** Each section must be visually distinguishable from the next — alternating background tone (white / light gray, or light / dark) prevents scroll fatigue and helps users mentally segment content.

4. **Mobile floating CTA:** Because mobile users cannot see the full nav easily, a persistent floating "Contactar" button ensures the conversion path is always reachable regardless of scroll position. This is the single most important mobile-specific pattern for portfolio one-pagers.

5. **Lightbox focus management:** When the portfolio lightbox opens, keyboard focus must move into the lightbox and be trapped there. When lightbox closes, focus must return to the image that triggered it. This is not optional — keyboard and screen reader users cannot access the gallery without this.

6. **No section autoplay or forced scroll:** Do not implement auto-advance carousels, auto-playing video, or forced section scroll snapping. Photography audiences browse at their own pace — interrupting that pace increases bounce rate.

7. **URL hash updates:** As the user scrolls through sections, update the browser URL hash (e.g., `/#portfolio`, `/#sobre-mi`) using History API — enables sharing of specific sections and back-button navigation within the page.

---

### Accessibility Defaults (Global)

- **Focus ring:** Always visible — never `outline: none` without a custom focus indicator — use a 2px solid accent-color ring with 2px offset — ensure visibility on all background colors
- **Skip navigation:** Yes — required — `<a href="#portfolio" class="skip-link">Saltar al contenido principal</a>` — first focusable element in `<body>` — visually hidden (`clip`, `position: absolute`) but visible on `:focus` — moves to `#portfolio` to skip the hero navigation
- **Keyboard tab order:** Follows DOM order — no `tabindex` manipulation except:
  - Lightbox (trap focus within, index 0 on first focusable element inside)
  - Floating mobile CTA (tabindex="-1" when `#contacto` section is visible, to avoid duplication with the actual form CTA)
  - Mobile nav overlay (trap focus within when open, return to hamburger button on close)
- **Color contrast:** All body text ≥ 4.5:1 contrast ratio (WCAG AA). Large text / UI components ≥ 3:1. Test all text-over-image combinations with a dark scrim layer.
- **Touch targets:** Minimum 44×44px for all tappable elements on mobile
- **Motion:** All CSS transitions and JavaScript animations check `prefers-reduced-motion: reduce` — if set, disable transforms and fades; keep only opacity fades at ≤ 100ms (effectively instant)
- **Images:** All informational images have descriptive `alt` text. Decorative images: `alt=""`. Portfolio images: descriptive alt text describing subject, setting, and year — critical for SEO and screen reader users who are evaluating style verbally.
- **Language:** `<html lang="es">` — important for correct screen reader pronunciation
- **Form labels:** Every input has an explicit `<label>` — no `aria-label` substitutions unless absolutely necessary — explicit labels are the most robust solution

---

## Post-Project Memory Notes

*(To be written to `ux-designer-memory.md` after client approval)*

- **Project type:** Portfolio one-pager, freelance photographer, Barcelona
- **Section order approved:** Hero → Portfolio → Bio → Servicios → Testimonios → Contacto → Footer
- **Key structural decision:** Portfolio placed before Bio — users qualify work before person; increases lead quality
- **Conversion mechanism:** Floating mobile CTA (pill button, bottom-right) is the primary mobile conversion driver — hero CTA alone is insufficient on mobile
- **Lightbox:** Required for photography portfolio — implement with focus trap, keyboard navigation, swipe gesture
- **Form length:** 5 fields maximum — name, email, project type, message, privacy — tested minimum for lead qualification without friction
- **CTA pattern:** Three-tier (filled → ghost → text-link) used across page — primary CTA once per section, not repeated
- **Recovery mechanisms:** Sticky nav + floating pill + CTA repeated at portfolio bottom + testimonials CTA immediately before form
- **No prices:** Deliberate — photographer pricing is scope-dependent; premature pricing creates false barriers
