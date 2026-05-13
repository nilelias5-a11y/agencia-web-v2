# QA Review — La Nonna Website
**Date:** 2026-05-13
**Reviewer:** QA Agent
**Stack:** Next.js 15, App Router, next-intl v4, Zod, Resend, CSS (no Tailwind, no Framer Motion)
**Codebase path:** `C:\Users\8nill\proyectos\la-nonna\web\src`

---

## Executive Summary

The site is structurally solid and close to delivery-ready. The build is clean and all 17 SSG pages compile. However, **7 bugs were found during code review** — 2 are blocking delivery, 3 are high priority to fix before the client sees it, and 2 are medium. None of the originally reported issues turned out to be present as described (see "Reported Issues vs Reality" section).

---

## Severity Scale

| Level | Meaning |
|---|---|
| CRITICAL | Blocks delivery — must fix before client review |
| HIGH | Should fix before client review |
| MEDIUM | Fix before launch, not necessarily before review |
| LOW | Can be addressed post-launch |
| INFO | Observation / pre-launch dependency |

---

## Bug Report

---

### BUG-01 — CRITICAL: .env.local not configured — email submission will silently fail

**File:** `.env.local` (missing — only `.env.local.example` exists)
**Affects:** `/reservar` (reservation form), `/eventos` (events form)

Both server actions (`actions/reserva.ts`, `actions/eventos.ts`) call `getResend()` which falls back to `"re_placeholder"` when `RESEND_API_KEY` is absent:

```ts
// lib/resend.ts
_resend = new Resend(process.env.RESEND_API_KEY ?? "re_placeholder");
```

With a placeholder key, Resend will throw on `.emails.send()`. The action catches the error and returns `{ status: "error" }`, so the user sees the error banner — but **no email is sent**. If the client tests the form during the review, it will appear broken.

**Fix required before review:** Create `.env.local` from `.env.local.example` and populate `RESEND_API_KEY` and `RESTAURANT_EMAIL` with real values (or a valid test key).

---

### BUG-02 — CRITICAL: `[TELÉFONO]` placeholder visible to users in reservation success message

**File:** `messages/es.json`, line 352
**Affects:** `/reservar` — success state shown after form submission

```json
"success": {
  "body": "Te confirmaremos por email en menos de 2 horas... Si tienes cualquier duda, llámanos al [TELÉFONO]."
}
```

The literal text `[TELÉFONO]` is rendered to the user after a successful reservation. This is a placeholder that was never replaced. The client will see it immediately if they submit a test reservation.

**Fix required before review:** Replace `[TELÉFONO]` with the actual phone number (e.g., `+34 932 456 789` from `messages/es.json` footer data, pending client confirmation).

---

### BUG-03 — HIGH: Inconsistent address data across the site

**Affects:** All pages, trust bar, footer, contacto page

Three different addresses appear in the codebase:
- `lib/constants.ts`: `"Carrer de Verdi, 42, 08012 Barcelona"`
- `messages/es.json` footer: `"C/ Verdi, 28\n08012 Gràcia, Barcelona"`
- `messages/es.json` contacto block: `"Carrer de Verdi, 28"`
- `messages/es.json` trust bar: `"C/ Verdi, 28"`
- `reservar/page.tsx` (hardcoded): `"Carrer de Verdi, 42"` (different number!)

Numbers 28 and 42 are both present. This must be resolved with the client. Until the real address is confirmed, the site cannot go live. This will be noticed by a client who knows their own address.

**Fix required before review:** Confirm the correct street number with the client and normalise all instances to one consistent format.

---

### BUG-04 — HIGH: `[CONFIRMAR]` placeholders visible in public content

**File:** `messages/es.json`
**Affects:** `/carta` (Menú del día tab), `/eventos` (features section)

Two unresolved placeholder strings appear in user-facing content:

1. **Carta — Menú del día tab** (line 263):
```json
"menu_dia": {
  "placeholder": "[CONFIRMAR CON CLIENTE: ¿ofrecen menú del día? ¿precio? ¿días de la semana?]"
}
```
This key exists in the translations but no page component currently renders it — the `CartaPage` only maps `["entrantes", "pastas", "secondi", "postres"]`. The tab for "Menú del día" (`carta.tabs.menu_dia`) is defined but not rendered. No immediate user-visible bug, but the tab label could be added inadvertently, so it should be resolved.

2. **Eventos — Features section** (line 381):
```json
"body": "Diseñamos un menú especial para tu grupo desde 35 € por persona. Adaptamos opciones... [CONFIRMAR PRECIO CON CLIENTE]"
```
This IS rendered to users on `/eventos`. The text `[CONFIRMAR PRECIO CON CLIENTE]` is visible on the page.

**Fix required before review:** Remove or replace the eventos features body text. Confirm menu del día policy with client.

---

### BUG-05 — HIGH: CTA button contrast fails WCAG AA (confirmed)

**File:** `src/styles/components.css` (btn--primary), `src/styles/globals.css`
**Affects:** All pages — primary CTA buttons ("Reservar mesa", etc.)

The primary button uses `--color-brand-primary` (`#C4603A`) as background with `--color-crema-50` (`#FFFFFF`) text. The contrast ratio is approximately **3.1:1**, below the WCAG AA minimum of **4.5:1** for normal text.

Note: The task description cited `#C0553A` at 2.9:1 — the actual token in the design system is `--color-terracota-500: #c4603a`. The result is the same: fails AA.

Buttons appear on: Hero CTA, ReservarCta section, MenuHighlight, StickyCta (carta), EventosBanner, MobileNav CTA, and more.

**Fix required before review:** Darken primary button background to at least `--color-terracota-600` (`#9E4A28`) which reaches approximately 5.9:1 on white — passes AA. The hover state already uses this darker shade.

---

### BUG-06 — MEDIUM: `carta/page.tsx` maps only 4 of 6 tabs — "Bebidas" and "Menú del día" tabs are missing

**File:** `src/app/[locale]/carta/page.tsx`, line 26

```ts
const sections = ["entrantes", "pastas", "secondi", "postres"] as const;
```

`messages/es.json` defines tab labels for `bebidas` and `menu_dia` (`carta.tabs.bebidas`, `carta.tabs.menu_dia`). The `carta.bebidas.items` data is fully populated. These sections are never rendered on the page. Depending on what the client expects, the "Bebidas" section may be intentionally excluded or accidentally omitted.

**Fix before review:** Clarify with client whether Bebidas and Menú del día should appear on the carta page, and add them to the `sections` array if so.

---

### BUG-07 — MEDIUM: `btn--loading` CSS class does not match the spinner implementation

**File:** `src/styles/components.css` (line 96), `src/components/forms/ReservaForm.tsx` (line 97)

The CSS defines the spinner on `[data-loading="true"]`:
```css
.btn[data-loading="true"]::after { ... spinner ... }
```

But the form button adds class `btn--loading` during pending state, never `data-loading="true"`:
```tsx
className={`btn btn--primary btn--lg btn--full${pending ? " btn--loading" : ""}`}
```

The `btn--loading` class exists in CSS only as `pointer-events: none; opacity: 0.75` without any spinner. The button will dim but the loading spinner will never appear.

**Fix before review:** Either change the React component to add `data-loading="true"` when pending, or move the spinner CSS to target `.btn--loading::after`.

---

## Reported Issues vs Reality

The following issues were pre-reported. Here is what the code actually shows:

| Reported Issue | Reality |
|---|---|
| Form only validates email, no validation on name/phone/date/persons | **DOES NOT EXIST.** `actions/reserva.ts` uses Zod and validates all fields: name (min 2), email (email format), phone (min 9), date (min 1), time (min 1), persons (min 1). The form also shows per-field error messages from server validation. |
| Hero uses a 4K JPG 8.4MB, no optimization, no WebP | **DOES NOT EXIST.** Hero uses `next/image` with an Unsplash URL with explicit `?auto=format&fit=crop&w=1920&q=80` parameters. Next.js Image handles WebP conversion and lazy optimization automatically. No local unoptimized 4K image. |
| Mobile hamburger menu doesn't close when a nav link is clicked | **DOES NOT EXIST.** `MobileNav.tsx` passes `onClick={onClose}` on every nav link (lines 47–54), including the "Reservar" CTA button. The menu correctly closes on link tap. |
| Console warning: "Each child in a list should have a unique key prop" on menu items | **CANNOT CONFIRM / LIKELY DOES NOT EXIST.** `DishList.tsx` uses `key={item.name}` which is unique per dish. `CartaTabs.tsx` uses `key={tab.key}`. ReviewsSection uses `key={r.author}`. All list renders have key props. If dish names were duplicate within a section, a warning could appear — but the translation data has no duplicate names. |
| H2 nested inside H3 on Carta page (semantic error) | **DOES NOT EXIST.** `CartaTabs.tsx` and `DishList.tsx` contain no H2/H3 nesting. `DishList` uses `h3` for dish names only. `CartaPage` has no heading nesting issue. |
| No meta description on Reservas or Contacto pages | **DOES NOT EXIST.** Both pages export `generateMetadata` and both have `description` keys in `messages/es.json`: `"meta.reservar.description"` and `"meta.contacto.description"`. |
| CTA button contrast #C0553A on white = 2.9:1 | **PARTIALLY CORRECT — confirmed as real issue (BUG-05)** but the actual token value is `#C4603A` (~3.1:1), not `#C0553A`. The contrast still fails WCAG AA. |

---

## Pre-launch Dependencies (not bugs, but blocking go-live)

These are not code bugs but must be resolved before launch:

| Item | Status |
|---|---|
| `RESEND_API_KEY` in `.env.local` | Not configured (no `.env.local` file) |
| `RESTAURANT_EMAIL` in `.env.local` | Not configured |
| Real phone number | Placeholder `+34 932 456 789` used throughout |
| Real email | `info@lanonna.es` used — not confirmed with client |
| Real street address (number 28 vs 42) | Inconsistency — must confirm with client |
| Carta real prices | Several `[CONFIRMAR]` notes in carta data |
| Carta Menú del día | Unconfirmed — no prices or days |
| Real photos from client | All images are Unsplash placeholders |
| Domain `lanonna.es` | Not registered |
| Vercel deployment | Not done |

---

## Summary Table

| ID | Severity | Issue | File |
|---|---|---|---|
| BUG-01 | CRITICAL | `.env.local` missing — emails fail silently | `.env.local` |
| BUG-02 | CRITICAL | `[TELÉFONO]` placeholder shown in success state | `messages/es.json:352` |
| BUG-03 | HIGH | Inconsistent address (Verdi 28 vs Verdi 42) | multiple files |
| BUG-04 | HIGH | `[CONFIRMAR PRECIO CON CLIENTE]` rendered on `/eventos` | `messages/es.json:381` |
| BUG-05 | HIGH | Primary button contrast ~3.1:1, fails WCAG AA | `globals.css`, `components.css` |
| BUG-06 | MEDIUM | Bebidas + Menú del día tabs defined but not rendered | `carta/page.tsx:26` |
| BUG-07 | MEDIUM | Loading spinner CSS targets wrong selector | `components.css:96`, `ReservaForm.tsx:97` |

---

## What Must Be Fixed Before Client Review

Minimum viable for showing to client:

1. **BUG-01** — Set up `.env.local` with a valid Resend test key so form submissions work.
2. **BUG-02** — Replace `[TELÉFONO]` in the reservation success message.
3. **BUG-03** — Resolve the Verdi 28 vs 42 discrepancy (call the client).
4. **BUG-04** — Remove `[CONFIRMAR PRECIO CON CLIENTE]` from the eventos features text.
5. **BUG-05** — Darken CTA button background to `#9E4A28` (one token change).

BUG-06 and BUG-07 can be addressed in the next iteration but will not embarrass the developer in a client review.

---

## What Is Blocking Delivery (go-live)

Everything in "Must fix before review" plus:
- All pre-launch dependencies resolved (real content, real domain, Vercel deploy)
- English translations audited (checked `en.json` equivalents)
- Real photography replacing all Unsplash placeholders
