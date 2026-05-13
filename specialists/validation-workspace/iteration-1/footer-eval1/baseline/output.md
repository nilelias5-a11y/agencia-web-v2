# Footer Design Spec & Component — La Nonna Restaurant

## 1. Design Rationale

### Pattern: "Warm Minimal" 3-Column + Full-Width Bottom Bar

**Why this pattern:**
- A restaurant footer should feel inviting, not corporate. Three columns at medium-width breakpoints (stacked on mobile) balance information density with whiteness.
- Full-width bottom bar for legal links separates commercial content from compliance text, which is both cleaner and legally advisable.
- Dark background (warm near-black, not pure #000) anchors the page and lets the hero imagery breathe above it.

### What to include (and what to omit)

| Element | Decision | Reason |
|---|---|---|
| Address + map link | YES | Core for a local restaurant; Google Maps deep link drives foot traffic |
| Phone | YES | Reservations are still often made by phone in Spain |
| Opening hours | YES | Reduces support friction; high-value info for tourists |
| Instagram @lanonna.gracia | YES | Active (post 2 days ago); highest ROI social for food |
| Facebook @lanonna | NO | Last post 2023 — showing a dead feed damages trust |
| Newsletter signup | YES | Client explicitly wants it; low-friction inline form |
| Online ordering / delivery links | NO | No online store |
| Navigation links | MINIMAL | Just anchor links to key sections (Menú, Reservas, Nosotros, Contacto) — avoids duplicating the main nav while keeping the footer crawlable |
| Logo / brand mark | YES | Reinforces brand identity at the natural reading endpoint |

### Legal elements required for a Spanish restaurant website

Under Spanish law (LSSI-CE, RGPD / GDPR, and Ley Orgánica 3/2018 — LOPDGDD) a commercial website must provide:

1. **Aviso Legal** — legal notice identifying the data controller: company name, CIF/NIF, registered address, and contact email. Mandatory per Art. 10 LSSI-CE.
2. **Política de Privacidad** — privacy policy explaining what personal data is collected (newsletter emails, contact forms, analytics cookies), legal basis, retention period, and data-subject rights (RGPD Art. 13/14).
3. **Política de Cookies** — cookie policy with a cookie banner on first visit. Required by LSSI-CE Art. 22.2 and AEPD guidelines. Must list each cookie category and allow granular consent.
4. **Copyright notice** — not strictly mandatory by law, but standard practice and recommended.

The newsletter signup must include:
- A checkbox (unchecked by default) confirming acceptance of the privacy policy.
- RGPD double opt-in confirmation email is best practice and reduces AEPD risk.

---

## 2. Visual Spec

```
┌─────────────────────────────────────────────────────────────────┐
│  [Logo / La Nonna wordmark]                                     │
│  Cocina italiana en el corazón de Gràcia                       │
│  [Instagram icon]                                               │
│                                                                 │
│  ┌─────────────────┐ ┌──────────────────┐ ┌─────────────────┐  │
│  │  VISÍTANOS       │ │  HORARIOS        │ │  NEWSLETTER     │  │
│  │  C/ Verdi 23     │ │  Lun–Jue 13–16h │ │  Tu mesa, tu   │  │
│  │  08012 Barcelona │ │        20–23h   │ │  historia.     │  │
│  │  Gràcia          │ │  Vie–Dom 13–16h │ │                │  │
│  │                  │ │        20–24h   │ │  [email input] │  │
│  │  ☎ 93 XXX XXXX  │ │  Lunes cerrado  │ │  [Suscribirme] │  │
│  │                  │ │                 │ │  ☐ He leído... │  │
│  │  [Ver en Maps]   │ │                 │ │                │  │
│  └─────────────────┘ └──────────────────┘ └─────────────────┘  │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  © 2024 La Nonna · Aviso Legal · Privacidad · Cookies          │
└─────────────────────────────────────────────────────────────────┘
```

**Color palette (Tailwind tokens):**
- Background: `stone-900` (#1c1917) — warm near-black, not cold gray
- Text primary: `stone-100` (#f5f5f4)
- Text secondary: `stone-400` (#a8a29e)
- Accent: `amber-500` (#f59e0b) — for links, button, and icon hover
- Border separator: `stone-700` (#44403c)
- Newsletter button: `amber-500` bg with `stone-900` text

**Typography:**
- Headings (column labels): `text-xs tracking-widest uppercase font-semibold text-stone-400`
- Body: `text-sm text-stone-300`
- Legal bar: `text-xs text-stone-500`

---

## 3. React/TypeScript Component

```tsx
// components/footer/Footer.tsx
"use client";

import React, { useState } from "react";
import Link from "next/link";
import Image from "next/image";

// ---------------------------------------------------------------------------
// Sub-components
// ---------------------------------------------------------------------------

function InstagramIcon({ className }: { className?: string }) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      className={className}
      aria-hidden="true"
    >
      <rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
      <circle cx="12" cy="12" r="4" />
      <circle cx="17.5" cy="6.5" r="0.5" fill="currentColor" stroke="none" />
    </svg>
  );
}

function MapPinIcon({ className }: { className?: string }) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      className={className}
      aria-hidden="true"
    >
      <path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7z" />
      <circle cx="12" cy="9" r="2.5" />
    </svg>
  );
}

function PhoneIcon({ className }: { className?: string }) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      className={className}
      aria-hidden="true"
    >
      <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07A19.5 19.5 0 0 1 4.17 12 19.79 19.79 0 0 1 1.1 3.37 2 2 0 0 1 3.07 1h3a2 2 0 0 1 2 1.72c.127.96.361 1.903.7 2.81a2 2 0 0 1-.45 2.11L7.09 8.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45c.907.339 1.85.573 2.81.7A2 2 0 0 1 21 16l.92.92z" />
    </svg>
  );
}

// ---------------------------------------------------------------------------
// Column: Contact & Location
// ---------------------------------------------------------------------------

function ContactColumn() {
  const googleMapsUrl =
    "https://www.google.com/maps/search/?api=1&query=C%2F+Verdi+23%2C+08012+Barcelona";

  return (
    <div>
      <h3 className="text-xs tracking-widest uppercase font-semibold text-stone-400 mb-4">
        Visítanos
      </h3>
      <ul className="space-y-3 text-sm text-stone-300">
        <li className="flex items-start gap-2">
          <MapPinIcon className="w-4 h-4 mt-0.5 shrink-0 text-amber-500" />
          <span>
            C/ Verdi 23
            <br />
            08012 Barcelona
            <br />
            <span className="text-stone-400">Gràcia</span>
          </span>
        </li>
        <li className="flex items-center gap-2">
          <PhoneIcon className="w-4 h-4 shrink-0 text-amber-500" />
          <a
            href="tel:+3493XXXXXXX"
            className="hover:text-amber-400 transition-colors"
          >
            93 XXX XXXX
          </a>
        </li>
        <li>
          <a
            href={googleMapsUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="inline-flex items-center gap-1 text-amber-400 hover:text-amber-300 transition-colors text-xs font-medium mt-1"
          >
            Ver en Google Maps
            <span aria-hidden="true">→</span>
          </a>
        </li>
      </ul>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Column: Opening Hours
// ---------------------------------------------------------------------------

const SCHEDULE: Array<{ days: string; hours: string[] }> = [
  { days: "Martes – Jueves", hours: ["13:00 – 16:00", "20:00 – 23:00"] },
  { days: "Viernes – Domingo", hours: ["13:00 – 16:00", "20:00 – 24:00"] },
  { days: "Lunes", hours: ["Cerrado"] },
];

function HoursColumn() {
  return (
    <div>
      <h3 className="text-xs tracking-widest uppercase font-semibold text-stone-400 mb-4">
        Horarios
      </h3>
      <ul className="space-y-3 text-sm">
        {SCHEDULE.map(({ days, hours }) => (
          <li key={days}>
            <span className="text-stone-300 font-medium">{days}</span>
            <div className="text-stone-400 text-xs mt-0.5 space-y-0.5">
              {hours.map((h) => (
                <div key={h}>{h}</div>
              ))}
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Column: Newsletter
// ---------------------------------------------------------------------------

type NewsletterStatus = "idle" | "loading" | "success" | "error";

function NewsletterColumn() {
  const [email, setEmail] = useState("");
  const [consent, setConsent] = useState(false);
  const [status, setStatus] = useState<NewsletterStatus>("idle");
  const [errorMsg, setErrorMsg] = useState("");

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setErrorMsg("");

    if (!consent) {
      setErrorMsg("Debes aceptar la política de privacidad para suscribirte.");
      return;
    }

    setStatus("loading");

    try {
      // Replace with your actual newsletter API endpoint (Mailchimp, Brevo, etc.)
      const res = await fetch("/api/newsletter/subscribe", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email }),
      });

      if (!res.ok) throw new Error("subscription_failed");
      setStatus("success");
      setEmail("");
      setConsent(false);
    } catch {
      setStatus("error");
      setErrorMsg("Ha ocurrido un error. Inténtalo de nuevo.");
    }
  }

  return (
    <div>
      <h3 className="text-xs tracking-widest uppercase font-semibold text-stone-400 mb-4">
        Newsletter
      </h3>
      <p className="text-sm text-stone-300 mb-4 leading-relaxed">
        Menús especiales, eventos y novedades de la nonna directamente en tu
        bandeja de entrada.
      </p>

      {status === "success" ? (
        <p className="text-sm text-amber-400 font-medium">
          ¡Gracias! Revisa tu correo para confirmar la suscripción.
        </p>
      ) : (
        <form onSubmit={handleSubmit} noValidate>
          <div className="flex flex-col gap-2">
            <label htmlFor="footer-newsletter-email" className="sr-only">
              Dirección de correo electrónico
            </label>
            <input
              id="footer-newsletter-email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              autoComplete="email"
              placeholder="tu@email.com"
              className="
                w-full rounded-md px-3 py-2 text-sm
                bg-stone-800 border border-stone-700
                text-stone-100 placeholder-stone-500
                focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent
                transition
              "
            />
            <button
              type="submit"
              disabled={status === "loading"}
              className="
                w-full rounded-md px-4 py-2 text-sm font-semibold
                bg-amber-500 text-stone-900
                hover:bg-amber-400 active:bg-amber-600
                disabled:opacity-60 disabled:cursor-not-allowed
                transition-colors
              "
            >
              {status === "loading" ? "Enviando…" : "Suscribirme"}
            </button>
          </div>

          {/* RGPD consent — required, unchecked by default */}
          <label className="flex items-start gap-2 mt-3 cursor-pointer group">
            <input
              type="checkbox"
              checked={consent}
              onChange={(e) => setConsent(e.target.checked)}
              className="mt-0.5 h-3.5 w-3.5 shrink-0 accent-amber-500 cursor-pointer"
            />
            <span className="text-xs text-stone-400 group-hover:text-stone-300 transition-colors leading-relaxed">
              He leído y acepto la{" "}
              <Link
                href="/privacidad"
                className="underline underline-offset-2 hover:text-amber-400 transition-colors"
              >
                Política de Privacidad
              </Link>
              . Puedo darme de baja en cualquier momento.
            </span>
          </label>

          {errorMsg && (
            <p role="alert" className="text-xs text-red-400 mt-2">
              {errorMsg}
            </p>
          )}
        </form>
      )}
    </div>
  );
}

// ---------------------------------------------------------------------------
// Main Footer
// ---------------------------------------------------------------------------

const NAV_LINKS = [
  { label: "Menú", href: "/#menu" },
  { label: "Reservas", href: "/#reservas" },
  { label: "Nosotros", href: "/#nosotros" },
  { label: "Contacto", href: "/#contacto" },
];

const LEGAL_LINKS = [
  { label: "Aviso Legal", href: "/aviso-legal" },
  { label: "Política de Privacidad", href: "/privacidad" },
  { label: "Política de Cookies", href: "/cookies" },
];

const CURRENT_YEAR = new Date().getFullYear();

export default function Footer() {
  return (
    <footer className="bg-stone-900 text-stone-100">
      {/* ── Main grid ── */}
      <div className="mx-auto max-w-6xl px-6 pt-14 pb-10">
        {/* Brand row */}
        <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-6 mb-10">
          {/* Logo / wordmark */}
          <div>
            {/*
              Replace the <span> below with an <Image> once you have the logo asset:
              <Image src="/images/logo-lanonna-light.svg" alt="La Nonna" width={140} height={40} />
            */}
            <span className="font-serif text-2xl tracking-wide text-stone-100">
              La Nonna
            </span>
            <p className="text-xs text-stone-400 mt-1">
              Cocina italiana en el corazón de Gràcia
            </p>
          </div>

          {/* Social + nav */}
          <div className="flex flex-col sm:items-end gap-4">
            {/* Instagram only — Facebook excluded (abandoned) */}
            <a
              href="https://www.instagram.com/lanonna.gracia"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Síguenos en Instagram @lanonna.gracia"
              className="flex items-center gap-2 text-stone-400 hover:text-amber-400 transition-colors text-sm group"
            >
              <InstagramIcon className="w-5 h-5 group-hover:scale-110 transition-transform" />
              <span>@lanonna.gracia</span>
            </a>

            {/* Secondary nav */}
            <nav aria-label="Navegación del pie de página">
              <ul className="flex flex-wrap gap-4 sm:justify-end">
                {NAV_LINKS.map(({ label, href }) => (
                  <li key={href}>
                    <Link
                      href={href}
                      className="text-xs text-stone-400 hover:text-amber-400 transition-colors"
                    >
                      {label}
                    </Link>
                  </li>
                ))}
              </ul>
            </nav>
          </div>
        </div>

        {/* Divider */}
        <hr className="border-stone-700 mb-10" />

        {/* Three-column info grid */}
        <div className="grid grid-cols-1 gap-10 sm:grid-cols-2 lg:grid-cols-3">
          <ContactColumn />
          <HoursColumn />
          <NewsletterColumn />
        </div>
      </div>

      {/* ── Legal bar ── */}
      <div className="border-t border-stone-800">
        <div className="mx-auto max-w-6xl px-6 py-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
          <p className="text-xs text-stone-500">
            © {CURRENT_YEAR} La Nonna · Todos los derechos reservados
          </p>
          <ul className="flex flex-wrap gap-4">
            {LEGAL_LINKS.map(({ label, href }) => (
              <li key={href}>
                <Link
                  href={href}
                  className="text-xs text-stone-500 hover:text-stone-300 transition-colors"
                >
                  {label}
                </Link>
              </li>
            ))}
          </ul>
        </div>
      </div>
    </footer>
  );
}
```

---

## 4. Implementation Notes

### File placement
```
src/
  components/
    footer/
      Footer.tsx        ← main component (above)
  app/
    layout.tsx          ← import and render <Footer /> here
    aviso-legal/
      page.tsx          ← required by law
    privacidad/
      page.tsx          ← required by law
    cookies/
      page.tsx          ← required by law
    api/
      newsletter/
        subscribe/
          route.ts      ← POST handler for the newsletter form
```

### Newsletter API route skeleton
```ts
// app/api/newsletter/subscribe/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const { email } = await req.json();

  if (!email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    return NextResponse.json({ error: "invalid_email" }, { status: 400 });
  }

  // TODO: integrate with Brevo / Mailchimp / Resend audiences
  // Remember: implement RGPD double opt-in confirmation email

  return NextResponse.json({ ok: true });
}
```

### Accessibility checklist
- Footer landmark: `<footer>` element (implicit `contentinfo` role).
- Secondary nav wrapped in `<nav aria-label="Navegación del pie de página">`.
- Newsletter input has an explicit `<label>` (visually hidden with `sr-only`).
- Instagram link has descriptive `aria-label`.
- Error messages use `role="alert"` for screen reader announcement.
- Color contrast: `stone-300` on `stone-900` passes WCAG AA (7.5:1). `amber-500` links on `stone-900` pass AA (4.7:1).

### Tailwind config — ensure `stone` palette is available
Tailwind v3 includes `stone` by default. If using a custom preset, verify `stone` is not excluded.

### Why Facebook was excluded
The client's Facebook page (@lanonna) has had no activity since 2023. Linking to a dormant social profile creates a negative first impression and signals to users (and search engines) that the business may be closed or neglected. If the client reactivates the page, add a `FacebookIcon` alongside the Instagram link using the same pattern.

### Opening hours assumption
The hours shown are placeholder values that must be confirmed with the client before launch. The `SCHEDULE` constant is intentionally extracted so a non-developer can update it without touching JSX.

---

## 5. Legal pages minimum viable content (Spain)

### /aviso-legal
Must include: denominación social, CIF/NIF, domicilio social, email de contacto, and datos del registro mercantil if applicable (LSSI-CE Art. 10).

### /privacidad
Must include: responsable del tratamiento, finalidades, base jurídica, destinatarios, plazo de conservación, derechos del interesado (acceso, rectificación, supresión, portabilidad, oposición), y derecho a reclamar ante la AEPD.

### /cookies
Must list each cookie by name or category (necesarias, analíticas, marketing), provider, duration, and purpose. Must be integrated with a consent management platform (CMP) or a custom banner that blocks non-essential cookies until consent is given. The AEPD (Spain's data protection authority) actively audits cookie compliance.
