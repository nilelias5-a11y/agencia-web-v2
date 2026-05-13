# Footer Design — Café Vermut E-Commerce

## 1. Spanish E-Commerce Legal Requirements

For an e-commerce operating in Spain and shipping to the EU, the following legal pages are **mandatory** under Spanish and EU law:

| Page | Legal Basis |
|------|-------------|
| **Aviso Legal** (Legal Notice) | LSSI-CE Art. 10 — must include company name, NIF, registered address, contact email |
| **Política de Privacidad** (Privacy Policy) | RGPD (GDPR) + LOPDGDD — data controller identity, processing purposes, user rights |
| **Política de Cookies** (Cookies Policy) | LSSI-CE Art. 22 + AEPD guidelines — cookie types, consent management |
| **Términos y Condiciones de Compra** (General Terms & Conditions) | RDL 1/2007 (TRLGDCU) — pre-contractual info, right of withdrawal (14 days), delivery, returns |
| **Política de Devoluciones** (Returns Policy) | TRLGDCU Arts. 102–108 — 14-day no-fault return right, refund procedures |
| **Política de Envíos** (Shipping Policy) | TRLGDCU — delivery timescales, costs, geographic scope |

**NIF display:** Must appear in the footer or Aviso Legal per LSSI-CE Art. 10.

---

## 2. Footer Structure Rationale

### Layout: 4-column grid (desktop) → stacked (mobile)

```
┌─────────────────────────────────────────────────────────────┐
│  LOGO + tagline          │ Tienda       │ Legal    │ Contacto│
│  Short brand desc        │ Café molido  │ Aviso    │ Address │
│  Social icons (IG, TT)   │ Granos       │ Privac.  │ Email   │
│                          │ Cápsulas     │ Cookies  │ Horario │
│                          │ Accesorios   │ T&C      │         │
│                          │ Gift cards   │ Devoluc. │         │
│                          │              │ Envíos   │         │
├─────────────────────────────────────────────────────────────┤
│  Payment icons (Visa, MC, Amex, Stripe)   © 2026 NIF        │
└─────────────────────────────────────────────────────────────┘
```

**Column rationale:**
- **Col 1 (Brand):** Logo + 1-line brand copy + social links. Establishes identity, builds trust.
- **Col 2 (Tienda):** Direct product/category navigation. Reduces bounce from footer.
- **Col 3 (Legal):** All legally required pages. Grouped for scannability.
- **Col 4 (Contacto):** Physical presence signals trust for Spanish consumers. Required by LSSI-CE.

**Bottom bar:** Payment method icons (trust signals), copyright + NIF (LSSI compliance).

---

## 3. React/TypeScript Component

```tsx
// components/Footer.tsx
import Link from 'next/link'
import Image from 'next/image'

// Social Icons as inline SVGs (no extra dependency)
function InstagramIcon() {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="1.75"
      strokeLinecap="round"
      strokeLinejoin="round"
      aria-hidden="true"
    >
      <rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
      <circle cx="12" cy="12" r="4" />
      <circle cx="17.5" cy="6.5" r="0.5" fill="currentColor" stroke="none" />
    </svg>
  )
}

function TikTokIcon() {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 24 24"
      fill="currentColor"
      aria-hidden="true"
    >
      <path d="M19.59 6.69a4.83 4.83 0 0 1-3.77-4.25V2h-3.45v13.67a2.89 2.89 0 0 1-2.88 2.5 2.89 2.89 0 0 1-2.89-2.89 2.89 2.89 0 0 1 2.89-2.89c.28 0 .54.04.79.1V9.01a6.33 6.33 0 0 0-.79-.05 6.34 6.34 0 0 0-6.34 6.34 6.34 6.34 0 0 0 6.34 6.34 6.34 6.34 0 0 0 6.33-6.34V8.69a8.18 8.18 0 0 0 4.78 1.52V6.75a4.85 4.85 0 0 1-1.01-.06z" />
    </svg>
  )
}

// Payment method badge (simple text+SVG fallback — replace with actual SVG sprites)
function PaymentBadge({ label }: { label: string }) {
  return (
    <span className="inline-flex items-center justify-center rounded border border-stone-600 bg-stone-800 px-2 py-1 text-xs font-medium text-stone-300 tracking-wide">
      {label}
    </span>
  )
}

const shopLinks = [
  { href: '/tienda/cafe-molido', label: 'Café Molido' },
  { href: '/tienda/granos', label: 'En Grano' },
  { href: '/tienda/capsulas', label: 'Cápsulas' },
  { href: '/tienda/accesorios', label: 'Accesorios' },
  { href: '/tienda/gift-cards', label: 'Tarjetas Regalo' },
]

const legalLinks = [
  { href: '/aviso-legal', label: 'Aviso Legal' },
  { href: '/privacidad', label: 'Política de Privacidad' },
  { href: '/cookies', label: 'Política de Cookies' },
  { href: '/terminos-y-condiciones', label: 'Términos y Condiciones' },
  { href: '/devoluciones', label: 'Política de Devoluciones' },
  { href: '/envios', label: 'Política de Envíos' },
]

const CURRENT_YEAR = new Date().getFullYear()

export default function Footer() {
  return (
    <footer className="bg-stone-900 text-stone-300">
      {/* Main grid */}
      <div className="mx-auto max-w-7xl px-6 py-14 lg:px-8">
        <div className="grid grid-cols-1 gap-10 sm:grid-cols-2 lg:grid-cols-4">

          {/* Col 1 — Brand */}
          <div className="space-y-5">
            <Link href="/" aria-label="Café Vermut — inicio">
              {/* Replace src with your actual logo path */}
              <Image
                src="/images/logo-cafe-vermut-white.svg"
                alt="Café Vermut"
                width={140}
                height={36}
                className="h-9 w-auto"
              />
            </Link>
            <p className="text-sm leading-relaxed text-stone-400">
              Specialty coffee de origen único. Tostado en Barcelona, enviado a toda España y Europa.
            </p>
            {/* Social links */}
            <div className="flex items-center gap-4">
              <a
                href="https://instagram.com/cafevermut"
                target="_blank"
                rel="noopener noreferrer"
                aria-label="Café Vermut en Instagram"
                className="text-stone-400 transition-colors hover:text-amber-400"
              >
                <InstagramIcon />
              </a>
              <a
                href="https://tiktok.com/@cafevermut"
                target="_blank"
                rel="noopener noreferrer"
                aria-label="Café Vermut en TikTok"
                className="text-stone-400 transition-colors hover:text-amber-400"
              >
                <TikTokIcon />
              </a>
            </div>
          </div>

          {/* Col 2 — Tienda */}
          <div>
            <h3 className="mb-4 text-xs font-semibold uppercase tracking-widest text-stone-400">
              Tienda
            </h3>
            <ul className="space-y-2.5">
              {shopLinks.map(({ href, label }) => (
                <li key={href}>
                  <Link
                    href={href}
                    className="text-sm text-stone-300 transition-colors hover:text-amber-400"
                  >
                    {label}
                  </Link>
                </li>
              ))}
            </ul>
          </div>

          {/* Col 3 — Legal */}
          <div>
            <h3 className="mb-4 text-xs font-semibold uppercase tracking-widest text-stone-400">
              Legal
            </h3>
            <ul className="space-y-2.5">
              {legalLinks.map(({ href, label }) => (
                <li key={href}>
                  <Link
                    href={href}
                    className="text-sm text-stone-300 transition-colors hover:text-amber-400"
                  >
                    {label}
                  </Link>
                </li>
              ))}
            </ul>
          </div>

          {/* Col 4 — Contacto */}
          <div>
            <h3 className="mb-4 text-xs font-semibold uppercase tracking-widest text-stone-400">
              Contacto
            </h3>
            <address className="not-italic space-y-3 text-sm text-stone-400">
              <p>
                Carrer del Consell de Cent 280
                <br />
                08011 Barcelona, España
              </p>
              <p>
                <a
                  href="mailto:info@cafevermut.com"
                  className="transition-colors hover:text-amber-400"
                >
                  info@cafevermut.com
                </a>
              </p>
              <p className="text-stone-500 text-xs">
                Lun–Vie 9:00–18:00 (CET)
              </p>
            </address>
          </div>

        </div>
      </div>

      {/* Bottom bar */}
      <div className="border-t border-stone-800">
        <div className="mx-auto max-w-7xl px-6 py-5 lg:px-8">
          <div className="flex flex-col items-center gap-4 sm:flex-row sm:justify-between">

            {/* Copyright + NIF (LSSI-CE Art. 10 compliance) */}
            <p className="text-xs text-stone-500 text-center sm:text-left">
              © {CURRENT_YEAR} Café Vermut S.L. — NIF: B-12345678. Todos los derechos reservados.
            </p>

            {/* Payment methods */}
            <div className="flex items-center gap-2" aria-label="Métodos de pago aceptados">
              <PaymentBadge label="Visa" />
              <PaymentBadge label="Mastercard" />
              <PaymentBadge label="Amex" />
              {/* Stripe lockup */}
              <span className="inline-flex items-center gap-1 rounded border border-stone-600 bg-stone-800 px-2 py-1 text-xs font-medium text-stone-300 tracking-wide">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="30"
                  height="13"
                  viewBox="0 0 60 25"
                  aria-label="Stripe"
                  role="img"
                  fill="#6772e5"
                >
                  <path d="M5.45 9.82c0-.96.79-1.33 2.1-1.33 1.88 0 4.25.57 6.13 1.59V4.4C11.8 3.5 9.97 3.1 8.13 3.1 3.46 3.1.47 5.55.47 9.98c0 6.96 9.59 5.85 9.59 8.85 0 1.13-.99 1.5-2.36 1.5-2.04 0-4.65-.84-6.71-1.96v5.74c2.28 1 4.59 1.42 6.71 1.42 5.11 0 8.63-2.53 8.63-7.02-.03-7.52-9.88-6.18-9.88-8.69zm16.98-5.62l-6.37 1.36v4.28l-3.08.66v3.71l3.08-.66v8.03c0 3.36 2.23 4.51 5.31 4.51 1.53 0 2.76-.2 3.65-.56v-3.94c-.66.27-3.93 1.26-3.93-1.13v-7.3l3.93-.84V7.56l-2.59.64zm7.97 1.54l-.26-.82h-4.75V25h5.46V13.64c1.29-1.69 3.47-1.38 4.14-1.14V7.8c-.7-.26-3.23-.75-4.59 1.94zm7.14-5.76l-5.5 1.17V5.3l5.5-1.17v-4.15zm-5.5 4.07h5.5V25h-5.5V4.05zm18.12-.51c-2.01 0-3.31.95-4.04 1.6l-.27-1.27h-4.81V29l5.46-1.16.01-6.27c.76.55 1.87 1.33 3.62 1.33 3.66 0 6.99-2.95 6.99-9.45-.02-5.95-3.39-9.91-6.96-9.91zm-1.23 14.99c-1.2 0-1.91-.43-2.4-1.08l-.02-8.5c.53-.6 1.26-1.01 2.42-1.01 1.85 0 3.13 2.07 3.13 5.27 0 3.27-1.25 5.32-3.13 5.32z" />
                </svg>
              </span>
            </div>

          </div>
        </div>
      </div>
    </footer>
  )
}
```

---

## 4. Implementation Notes

### File placement
```
components/
  Footer.tsx          ← the component above
public/
  images/
    logo-cafe-vermut-white.svg   ← add your SVG logo
```

### Usage in layout
```tsx
// app/layout.tsx
import Footer from '@/components/Footer'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="es">
      <body>
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  )
}
```

### Tailwind color palette used
| Token | Purpose |
|-------|---------|
| `stone-900` | Footer background |
| `stone-800` | Bottom bar cards, border |
| `stone-400` | Secondary text, column headers |
| `stone-300` | Primary link text |
| `stone-500` | Muted text (NIF, hours) |
| `amber-400` | Hover accent (brand warmth — adjust to your palette) |

### Payment icons — production recommendation
Replace the text `PaymentBadge` placeholders with proper SVG sprites from:
- [svgrepo.com/collection/payment-method-flat-icons](https://www.svgrepo.com/collection/payment-method-flat-icons/)
- Or the official Stripe brand assets (stripe.com/newsroom/brand)

### Cookies banner integration
The **Política de Cookies** link in the legal column should also trigger your cookie consent manager (e.g., `onClick={() => openCookieBanner()}`). Example:

```tsx
// In legalLinks, swap the cookies entry for a button/link combo:
<button
  onClick={() => window.__cookieConsent?.openPreferences()}
  className="text-sm text-stone-300 transition-colors hover:text-amber-400 text-left"
>
  Política de Cookies
</button>
```

### `lang="es"` — already in layout
The `<html lang="es">` attribute is required for Spanish-language WCAG compliance and helps screen readers use correct pronunciation.

---

## 5. Legal Pages Checklist (what each page must contain)

### Aviso Legal
- Denominación social: Café Vermut S.L.
- NIF: B-12345678
- Domicilio: Carrer del Consell de Cent 280, 08011 Barcelona
- Email: info@cafevermut.com
- Registro Mercantil: inscription data
- Actividad: venta online de café de especialidad

### Política de Privacidad
- Responsable del tratamiento + DPO contact
- Finalidades: gestión de pedidos, envío de newsletter (solo con consentimiento explícito), mejora del servicio
- Legitimación: ejecución del contrato (pedidos), consentimiento (marketing)
- Destinatarios: Stripe (procesador de pagos), transportistas, hosting (Vercel)
- Transferencias internacionales: Stripe (cláusulas contractuales tipo)
- Derechos ARCO+: acceso, rectificación, supresión, oposición, portabilidad, limitación
- Reclamación ante AEPD: aepd.es

### Términos y Condiciones
- Proceso de compra paso a paso
- Derecho de desistimiento: 14 días naturales desde recepción
- Excepciones al desistimiento: café sellado abierto
- Plazos de entrega: España peninsular 2–4 días, Europa 5–10 días
- Garantías legales (RDL 1/2007)
- Ley aplicable: ley española; fuero: juzgados de Barcelona
