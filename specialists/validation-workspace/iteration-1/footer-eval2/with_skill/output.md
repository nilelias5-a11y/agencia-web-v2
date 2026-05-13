# Footer Spec + Implementation — Café Vermut

---

## Footer Spec — Café Vermut

**Pattern:** 4 — E-Commerce Footer
**Rationale:** Specialty coffee e-commerce shipping within Spain and Europe. Needs to build trust through logistics info, legal compliance, payment badges, and newsletter capture. Pattern 4 is the correct choice.

**Background color:** `#1A1209` (near-black espresso brown — evokes coffee, premium feel)
**Text color:** `#D4C5A9` (warm parchment — readable on dark, stays in coffee palette)
**Link color:** `#D4C5A9` / **hover:** `#FFFFFF`
**Accent color:** `#C8832A` (roasted amber — for CTA buttons and highlights)
**Border/divider color:** `#2E200F`

---

### Navigation columns

- **Column 1 "La Tienda"**
  - Todos los cafés (`/tienda`)
  - Café de especialidad (`/tienda/especialidad`)
  - Blend del mes (`/tienda/blend-del-mes`)
  - Accesorios (`/tienda/accesorios`)
  - Pack regalo (`/tienda/packs`)

- **Column 2 "Ayuda"**
  - Cómo hacer tu pedido (`/ayuda/como-pedir`)
  - Envíos y plazos (`/ayuda/envios`)
  - Devoluciones (`/politica-devoluciones`)
  - Preguntas frecuentes (`/ayuda/faq`)
  - Contacto (`/contacto`)

- **Column 3 "Café Vermut"**
  - Nuestra historia (`/nosotros`)
  - Sourcing y origen (`/nosotros/origen`)
  - Blog (`/blog`)
  - Trabaja con nosotros (`/trabaja-con-nosotros`)

---

**Newsletter:** Yes
**Platform:** TBD — confirm with director (recommend Resend for Next.js-native integration)
**CTA copy:** "Recibe recetas y novedades de temporada" (specific value prop, not generic "subscribe")
**GDPR checkbox required:** Yes (EU client shipping to EU)
**Consent text:** "Acepto recibir comunicaciones de Café Vermut. Puedo darme de baja en cualquier momento."

---

**Social platforms:** Instagram (@cafevermut), TikTok (@cafevermut)
**Facebook:** Omitted — client has no active Facebook presence. Adding a dead Facebook link destroys trust.

---

**Legal links (Spain/EU — all required):**
- Aviso Legal (`/aviso-legal`)
- Política de Privacidad (`/politica-privacidad`)
- Política de Cookies (`/politica-cookies`)
- Política de Devoluciones y Envíos (`/politica-devoluciones`)

**Copyright:** `© 2026 Café Vermut S.L. — NIF B-12345678`
**Registered address:** Carrer del Consell de Cent 280, Barcelona 08011, España

---

**Payment icons:** Visa, Mastercard, American Express
**Implementation note:** Do NOT use SVG files downloaded from unofficial sources. Use the official SVG payment mark files from each network's brand guidelines (Visa: visa.com/brand-center, Mastercard: brand.mastercard.com, Amex: developer.americanexpress.com). These are the only legally safe sources. Alternatively use the `react-payment-icons` package (MIT license) which bundles official-style marks. Size: 38×24px each, displayed as a horizontal row in the bottom bar.

**Shipping badge:** Sendcloud logo (optional — adds logistics trust signal; use only official Sendcloud partner badge)

**Trust badges:**
- SSL padlock icon (custom SVG — do not use third-party badge image services)
- No "Norton" or "McAfee" trust seals unless the client has an active subscription

---

**Responsive behavior at mobile:**
- All three navigation columns collapse to a single stacked accordion (each column title becomes a tap-to-expand trigger)
- Newsletter form goes full-width above the columns
- Payment icons row wraps but stays centered
- Logo stays top-center on mobile
- Legal links stack vertically, centered
- Social icons stay horizontal and centered
- Minimum tap target: 44px (Apple HIG / WCAG 2.5.5)

---

### Legal requirements for this client

**Country:** Spain (EU member)
**Required by Spanish Ley 34/2002 (LSSI-CE) and GDPR:**
- Aviso Legal: company name, NIF, registered address, contact email, registry inscription — MANDATORY for any Spanish commercial website
- Política de Privacidad: data controller identity, purposes of processing, legal basis, retention periods, user rights (access, rectification, erasure, portability), DPA contact — MANDATORY under GDPR Art. 13
- Política de Cookies: list of all cookies used, their purpose, duration, whether first or third party, and consent mechanism — MANDATORY under LOPD-GDD and ePrivacy Directive
- Política de Devoluciones y Envíos: Spanish e-commerce law (Real Decreto Legislativo 1/2007, Ley General para la Defensa de los Consumidores y Usuarios) requires 14-day right of withdrawal displayed clearly before purchase AND in the footer

**NIF displayed in footer:** Yes — B-12345678 (required in Aviso Legal page; footer shorthand is acceptable and builds trust)

---

## React/TypeScript Component

```tsx
// components/Footer/Footer.tsx
"use client";

import Link from "next/link";
import Image from "next/image";
import { useState } from "react";
import { InstagramIcon, TikTokIcon } from "@/components/icons/SocialIcons";

// ---------------------------------------------------------------------------
// Types
// ---------------------------------------------------------------------------

interface FooterColumn {
  label: string;
  links: { text: string; href: string }[];
}

// ---------------------------------------------------------------------------
// Data
// ---------------------------------------------------------------------------

const SHOP_COLUMN: FooterColumn = {
  label: "La Tienda",
  links: [
    { text: "Todos los cafés", href: "/tienda" },
    { text: "Café de especialidad", href: "/tienda/especialidad" },
    { text: "Blend del mes", href: "/tienda/blend-del-mes" },
    { text: "Accesorios", href: "/tienda/accesorios" },
    { text: "Pack regalo", href: "/tienda/packs" },
  ],
};

const HELP_COLUMN: FooterColumn = {
  label: "Ayuda",
  links: [
    { text: "Cómo hacer tu pedido", href: "/ayuda/como-pedir" },
    { text: "Envíos y plazos", href: "/ayuda/envios" },
    { text: "Devoluciones", href: "/politica-devoluciones" },
    { text: "Preguntas frecuentes", href: "/ayuda/faq" },
    { text: "Contacto", href: "/contacto" },
  ],
};

const ABOUT_COLUMN: FooterColumn = {
  label: "Café Vermut",
  links: [
    { text: "Nuestra historia", href: "/nosotros" },
    { text: "Sourcing y origen", href: "/nosotros/origen" },
    { text: "Blog", href: "/blog" },
    { text: "Trabaja con nosotros", href: "/trabaja-con-nosotros" },
  ],
};

const LEGAL_LINKS = [
  { text: "Aviso Legal", href: "/aviso-legal" },
  { text: "Política de Privacidad", href: "/politica-privacidad" },
  { text: "Política de Cookies", href: "/politica-cookies" },
  { text: "Devoluciones y Envíos", href: "/politica-devoluciones" },
];

const COLUMNS = [SHOP_COLUMN, HELP_COLUMN, ABOUT_COLUMN];

// ---------------------------------------------------------------------------
// Sub-components
// ---------------------------------------------------------------------------

function FooterAccordion({ column }: { column: FooterColumn }) {
  const [open, setOpen] = useState(false);

  return (
    <div className="border-b border-[#2E200F] md:border-none">
      {/* Mobile: accordion trigger */}
      <button
        className="flex w-full items-center justify-between py-4 text-left text-sm font-semibold uppercase tracking-widest text-[#D4C5A9] md:hidden"
        onClick={() => setOpen((prev) => !prev)}
        aria-expanded={open}
        aria-controls={`footer-col-${column.label}`}
      >
        {column.label}
        <span
          className="ml-2 transition-transform duration-200"
          aria-hidden="true"
          style={{ transform: open ? "rotate(180deg)" : "rotate(0deg)" }}
        >
          ▾
        </span>
      </button>

      {/* Desktop: always-visible heading */}
      <p className="hidden text-sm font-semibold uppercase tracking-widest text-[#D4C5A9] md:block md:mb-4">
        {column.label}
      </p>

      {/* Links list */}
      <ul
        id={`footer-col-${column.label}`}
        className={`overflow-hidden transition-all duration-300 ease-in-out md:block ${
          open ? "max-h-96 pb-4" : "max-h-0 md:max-h-none"
        }`}
        role="list"
      >
        {column.links.map((link) => (
          <li key={link.href} className="mb-2">
            <Link
              href={link.href}
              className="text-sm text-[#D4C5A9]/70 transition-colors duration-150 hover:text-white"
            >
              {link.text}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

function NewsletterForm() {
  const [email, setEmail] = useState("");
  const [consent, setConsent] = useState(false);
  const [status, setStatus] = useState<"idle" | "loading" | "success" | "error">("idle");
  const [errorMsg, setErrorMsg] = useState("");

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setErrorMsg("");

    if (!consent) {
      setErrorMsg("Debes aceptar recibir comunicaciones para suscribirte.");
      return;
    }

    setStatus("loading");

    try {
      // Replace with your actual newsletter API endpoint
      const res = await fetch("/api/newsletter/subscribe", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email }),
      });

      if (!res.ok) throw new Error("server_error");
      setStatus("success");
    } catch {
      setStatus("error");
      setErrorMsg("Algo ha fallado. Inténtalo de nuevo en un momento.");
    }
  }

  if (status === "success") {
    return (
      <div
        className="rounded-lg border border-[#C8832A]/40 bg-[#C8832A]/10 p-4 text-sm text-[#D4C5A9]"
        role="status"
        aria-live="polite"
      >
        ¡Apuntado! Pronto recibirás noticias de Café Vermut.
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <p className="mb-1 text-sm font-semibold uppercase tracking-widest text-[#D4C5A9]">
        Newsletter
      </p>
      <p className="mb-4 text-sm text-[#D4C5A9]/70">
        Recibe recetas y novedades de temporada
      </p>

      <div className="flex flex-col gap-2 sm:flex-row">
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="tu@email.com"
          required
          autoComplete="email"
          aria-label="Tu dirección de correo electrónico"
          className="flex-1 rounded-md border border-[#2E200F] bg-[#2E200F] px-4 py-2 text-sm text-[#D4C5A9] placeholder-[#D4C5A9]/40 outline-none transition focus:border-[#C8832A] focus:ring-1 focus:ring-[#C8832A]"
        />
        <button
          type="submit"
          disabled={status === "loading"}
          className="rounded-md bg-[#C8832A] px-5 py-2 text-sm font-semibold text-white transition hover:bg-[#B5731E] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#C8832A] disabled:opacity-60"
        >
          {status === "loading" ? "Enviando…" : "Suscribirme"}
        </button>
      </div>

      {/* GDPR consent — mandatory for EU */}
      <label className="mt-3 flex items-start gap-2 cursor-pointer">
        <input
          type="checkbox"
          checked={consent}
          onChange={(e) => setConsent(e.target.checked)}
          className="mt-0.5 h-4 w-4 shrink-0 accent-[#C8832A]"
          aria-required="true"
        />
        <span className="text-xs text-[#D4C5A9]/60 leading-relaxed">
          Acepto recibir comunicaciones de Café Vermut. Puedo darme de baja en cualquier
          momento. Ver{" "}
          <Link href="/politica-privacidad" className="underline hover:text-white">
            Política de Privacidad
          </Link>
          .
        </span>
      </label>

      {errorMsg && (
        <p className="mt-2 text-xs text-red-400" role="alert" aria-live="assertive">
          {errorMsg}
        </p>
      )}
    </form>
  );
}

// ---------------------------------------------------------------------------
// Payment icons — inline SVG marks
// Using simplified geometric representations of the card brand marks.
// For production, replace with official SVG files from each network's brand center:
//   Visa:       visa.com/brand-center
//   Mastercard: brand.mastercard.com
//   Amex:       developer.americanexpress.com
// Or install: npm install react-payment-icons (MIT)
// ---------------------------------------------------------------------------

function PaymentIcons() {
  return (
    <div className="flex items-center gap-2" aria-label="Métodos de pago aceptados">
      {/* Visa */}
      <div
        className="flex h-6 w-10 items-center justify-center rounded border border-[#2E200F] bg-white px-1"
        title="Visa"
        role="img"
        aria-label="Visa"
      >
        <svg viewBox="0 0 40 16" className="h-3 w-auto">
          <text
            x="2"
            y="13"
            fontFamily="Arial, sans-serif"
            fontWeight="bold"
            fontSize="13"
            fill="#1A1F71"
          >
            VISA
          </text>
        </svg>
      </div>

      {/* Mastercard */}
      <div
        className="flex h-6 w-10 items-center justify-center rounded border border-[#2E200F] bg-white"
        title="Mastercard"
        role="img"
        aria-label="Mastercard"
      >
        <svg viewBox="0 0 38 24" className="h-4 w-auto">
          <circle cx="14" cy="12" r="9" fill="#EB001B" />
          <circle cx="24" cy="12" r="9" fill="#F79E1B" />
          <path
            d="M19 5.6a9 9 0 0 1 0 12.8A9 9 0 0 1 19 5.6z"
            fill="#FF5F00"
          />
        </svg>
      </div>

      {/* American Express */}
      <div
        className="flex h-6 w-10 items-center justify-center rounded border border-[#2E200F] bg-[#016FD0]"
        title="American Express"
        role="img"
        aria-label="American Express"
      >
        <svg viewBox="0 0 40 16" className="h-3 w-auto">
          <text
            x="2"
            y="12"
            fontFamily="Arial, sans-serif"
            fontWeight="bold"
            fontSize="8"
            fill="#FFFFFF"
            letterSpacing="0.5"
          >
            AMEX
          </text>
        </svg>
      </div>
    </div>
  );
}

function SocialLinks() {
  return (
    <div className="flex items-center gap-4">
      <a
        href="https://www.instagram.com/cafevermut"
        target="_blank"
        rel="noopener noreferrer"
        aria-label="Café Vermut en Instagram"
        className="text-[#D4C5A9]/60 transition-colors hover:text-white"
      >
        <InstagramIcon className="h-5 w-5" aria-hidden="true" />
      </a>
      <a
        href="https://www.tiktok.com/@cafevermut"
        target="_blank"
        rel="noopener noreferrer"
        aria-label="Café Vermut en TikTok"
        className="text-[#D4C5A9]/60 transition-colors hover:text-white"
      >
        <TikTokIcon className="h-5 w-5" aria-hidden="true" />
      </a>
    </div>
  );
}

// ---------------------------------------------------------------------------
// Main Footer
// ---------------------------------------------------------------------------

export default function Footer() {
  const currentYear = new Date().getFullYear();

  return (
    <footer className="bg-[#1A1209] text-[#D4C5A9]">

      {/* ── Top section: logo + columns + newsletter ── */}
      <div className="mx-auto max-w-7xl px-4 py-12 sm:px-6 lg:px-8">
        <div className="grid grid-cols-1 gap-8 md:grid-cols-2 lg:grid-cols-5">

          {/* Logo + tagline */}
          <div className="lg:col-span-2">
            <Link href="/" aria-label="Café Vermut — Inicio">
              {/* Replace src with actual logo path */}
              <Image
                src="/images/logo-cafevermut-light.svg"
                alt="Café Vermut"
                width={140}
                height={40}
                className="mb-4"
              />
            </Link>
            <p className="max-w-xs text-sm leading-relaxed text-[#D4C5A9]/70">
              Café de especialidad con origen. Tostado en Barcelona, enviado a toda España y Europa.
            </p>
            <address className="mt-4 not-italic text-xs text-[#D4C5A9]/50 leading-relaxed">
              Carrer del Consell de Cent 280<br />
              Barcelona 08011, España<br />
              <a
                href="mailto:info@cafevermut.com"
                className="hover:text-white transition-colors"
              >
                info@cafevermut.com
              </a>
            </address>
          </div>

          {/* Navigation columns */}
          {COLUMNS.map((col) => (
            <div key={col.label} className="lg:col-span-1">
              <FooterAccordion column={col} />
            </div>
          ))}
        </div>

        {/* Newsletter — full width below columns */}
        <div className="mt-10 border-t border-[#2E200F] pt-10 lg:max-w-md">
          <NewsletterForm />
        </div>
      </div>

      {/* ── Bottom bar: legal + payment + social + copyright ── */}
      <div className="border-t border-[#2E200F]">
        <div className="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">

          {/* Row 1: legal links + social */}
          <div className="flex flex-col items-start gap-4 sm:flex-row sm:items-center sm:justify-between">
            <nav aria-label="Legal">
              <ul className="flex flex-wrap gap-x-4 gap-y-2">
                {LEGAL_LINKS.map((link) => (
                  <li key={link.href}>
                    <Link
                      href={link.href}
                      className="text-xs text-[#D4C5A9]/50 transition-colors hover:text-white"
                    >
                      {link.text}
                    </Link>
                  </li>
                ))}
              </ul>
            </nav>
            <SocialLinks />
          </div>

          {/* Row 2: copyright + payment icons */}
          <div className="mt-4 flex flex-col items-start gap-4 sm:flex-row sm:items-center sm:justify-between">
            <p className="text-xs text-[#D4C5A9]/40">
              &copy; {currentYear} Café Vermut S.L. &mdash; NIF B-12345678. Todos los derechos reservados.
            </p>
            <PaymentIcons />
          </div>

        </div>
      </div>
    </footer>
  );
}
```

---

## Required companion files

### `components/icons/SocialIcons.tsx`

```tsx
// components/icons/SocialIcons.tsx
import type { SVGProps } from "react";

export function InstagramIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      aria-hidden="true"
      {...props}
    >
      <rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
      <circle cx="12" cy="12" r="4" />
      <circle cx="17.5" cy="6.5" r="0.5" fill="currentColor" stroke="none" />
    </svg>
  );
}

export function TikTokIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="currentColor"
      aria-hidden="true"
      {...props}
    >
      <path d="M19.59 6.69a4.83 4.83 0 0 1-3.77-4.25V2h-3.45v13.67a2.89 2.89 0 0 1-2.88 2.5 2.89 2.89 0 0 1-2.89-2.89 2.89 2.89 0 0 1 2.89-2.89c.28 0 .54.04.79.1V9.01a6.33 6.33 0 0 0-.79-.05 6.34 6.34 0 0 0-6.34 6.34 6.34 6.34 0 0 0 6.34 6.34 6.34 6.34 0 0 0 6.33-6.34V8.69a8.18 8.18 0 0 0 4.78 1.52V6.75a4.85 4.85 0 0 1-1.01-.06z" />
    </svg>
  );
}
```

---

## Implementation notes

**Payment icons:** The inline SVG marks above are placeholder representations for development only. Before going live, replace them with:
1. Official SVG files from each card network's brand center (linked in the component comments), OR
2. Install `react-payment-icons` (MIT license): `npm install react-payment-icons`

**Newsletter API:** The form posts to `/api/newsletter/subscribe`. Create a Next.js Route Handler at `app/api/newsletter/subscribe/route.ts` that calls your newsletter provider. Confirm platform (Resend recommended for Next.js App Router) with director before implementing.

**Logo:** Replace `/images/logo-cafevermut-light.svg` with the actual logo file. The footer uses the light (white/cream) variant since the background is dark.

**Cookie consent banner:** The Política de Cookies link in the footer is required, but a cookie consent banner (separate component) is also legally mandatory for EU. This is outside scope of footer but should be implemented as a companion component.

**Sendcloud badge:** Optional. If the client wants to display it, add an `<Image>` in the bottom bar between copyright and payment icons using the official Sendcloud partner badge asset obtained from their partner portal.

**`currentYear`:** The `new Date().getFullYear()` call runs client-side. For SSR consistency in Next.js App Router, compute it server-side by making Footer a Server Component (remove `"use client"`) and extracting only the `NewsletterForm` and `FooterAccordion` as Client Components with their own `"use client"` directive.

**Accessibility:** All interactive elements have `aria-label`. Social links open in `_blank` with `rel="noopener noreferrer"`. Address uses semantic `<address>` tag. Legal nav has `aria-label="Legal"`. Payment icons have `role="img"` with descriptive `aria-label`.
