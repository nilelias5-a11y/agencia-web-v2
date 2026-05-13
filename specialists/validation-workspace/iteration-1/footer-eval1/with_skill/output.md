# Footer Architect Output — La Nonna

---

## DECISION LOG

### Pattern Selection: Pattern 3 — Contact-Forward Footer

La Nonna is a service business (restaurant) where conversion = booking a table or finding the address. Visitors reaching the footer are either:
1. Trying to find contact/location info before leaving
2. Looking for a reservation link they missed
3. Looking for social proof

Pattern 3 (Contact-Forward) is the correct choice. The restaurant has no online store (Pattern 4 is irrelevant), no content-heavy site (Pattern 5 is irrelevant), and enough pages that a full nav column adds value — so this implementation merges Pattern 3 with a navigation column, a newsletter block, and the legally required Spanish/EU elements.

### Social Platforms — 30-Day Rule Applied

| Platform | Last post | Include? | Reason |
|----------|-----------|----------|--------|
| Instagram @lanonna.gracia | 2 days ago | YES | Active, <30 days |
| Facebook @lanonna | 2023 | NO | Last post >2 years ago — dead profile destroys trust |

Result: Instagram only. No Facebook icon.

### Spain/EU Legal Requirements Applied

| Requirement | Basis | Included |
|-------------|-------|----------|
| Aviso Legal (Legal Notice) | LSSICE art. 10 — mandatory for Spanish websites | YES |
| Política de Privacidad | GDPR + LOPDGDD | YES |
| Política de Cookies | RGPD / Directiva ePrivacy (Spain transposed via LSSICE) | YES |
| GDPR newsletter consent checkbox | RGPD art. 6.1.a + art. 7 | YES |
| CIF/company reg | Only required for commercial entities — restaurant (physical, B2C) → recommended but not strictly mandatory on footer; flagged for client to add when CIF confirmed | FLAGGED |

Note: The existing `aviso-legal` page covers the legal notice and privacy/cookie info. Separate pages for privacy and cookies should be created before launch — flagged in component as `TODO`.

### Newsletter CTA Copy

Bad: "Suscríbete a nuestro newsletter"
Good: Value-specific, audience-aware

**Selected CTA (ES):** "Recibe nuestras novedades: menú del día, eventos especiales y recetas de la abuela"
**Selected CTA (EN):** "Get our updates: daily specials, events and recipes from the kitchen"

Reasoning: La Nonna's audience is neighborhood locals and repeat visitors. The value proposition is intimacy and insider access ("recetas de la abuela") — the same emotional hook that drives the brand. This is not a coupon-driven newsletter.

GDPR: Single checkbox with explicit consent copy required in Spain. No pre-checked boxes.

### Address Note

The task brief states C/ Verdi 23, but the project memory and existing translation files use C/ Verdi 28. The existing component also uses Verdi 28. The discrepancy in the task brief (23) vs. project data (28) is flagged — the component uses the confirmed project address (Verdi 28) pending client confirmation. Phone is also a placeholder per project memory.

---

## FOOTER SPEC — La Nonna

**Pattern:** 3 (Contact-Forward) + Navigation column + Newsletter
**Background color:** `var(--color-bg-inverse)` = `#2C1A0E` (crema-900 — matches existing site dark sections)
**Text color:** `rgba(250, 246, 240, 0.60)` for body / `var(--color-crema-100)` for headings
**Link color / hover:** `rgba(250, 246, 240, 0.60)` / `var(--color-crema-50)` (white)
**Accent color:** `var(--color-brand-primary)` = `#C4603A` (terracota-500) — for CTA button

**Navigation columns:**
- Column 1 "La Nonna": Carta, Nuestra Historia, Eventos Privados, Reservar mesa
- Column 2 "Visítanos": Address, Phone, Email, Hours, Google Maps link

**Newsletter:**
- Yes
- Platform: Resend (confirmed in task brief + project stack)
- CTA copy ES: "Recibe nuestras novedades: menú del día, eventos especiales y recetas de la abuela"
- CTA copy EN: "Get our updates: daily specials, events and recipes from the kitchen"
- Fields: Email only (conversion-optimized)
- GDPR: Consent checkbox required (Spain/EU)
- Success state: In-place confirmation, no page reload
- Error state: Inline validation message

**Social platforms:** Instagram only (@lanonna.gracia) — Facebook excluded (30-day rule: last post 2023)

**Legal links:**
- Aviso Legal → `/aviso-legal`
- Política de Privacidad → `/privacidad` (TODO: create page — currently covered in aviso-legal)
- Política de Cookies → `/cookies` (TODO: create page — currently covered in aviso-legal)

**Copyright:** `© {year} La Nonna. Todos los derechos reservados.` — year auto-generated via JS

**Payment icons:** None (no online store)
**Trust badges:** None (restaurant, not B2B)

**Responsive behavior at mobile:**
- 4 columns collapse to single stack
- Newsletter section moves below contact info
- Social icons stay centered at bottom
- Legal links wrap naturally (flex-wrap)
- Address column stays on top (highest-value info for mobile visitors)

**Legal requirements for this client:**
- Country: Spain (EU)
- Required: Aviso Legal (LSSICE art. 10), Política de Privacidad (GDPR/LOPDGDD), Política de Cookies (Directiva ePrivacy)
- GDPR newsletter consent: explicit checkbox, unchecked by default, clear purpose statement
- CIF: Pending client confirmation — should be added to Aviso Legal page, not required on footer for B2C restaurant

---

## REACT/TYPESCRIPT COMPONENT

### File: `Footer.tsx`

```tsx
"use client";

import { useState, FormEvent } from "react";
import { useTranslations } from "next-intl";
import { Link } from "@/i18n/navigation";

// SVG icons inlined to avoid external dependency
function InstagramIcon() {
  return (
    <svg
      width="20"
      height="20"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="1.5"
      strokeLinecap="round"
      strokeLinejoin="round"
      aria-hidden="true"
    >
      <rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
      <circle cx="12" cy="12" r="4" />
      <circle cx="17.5" cy="6.5" r="0.5" fill="currentColor" stroke="none" />
    </svg>
  );
}

type NewsletterState = "idle" | "loading" | "success" | "error";

export default function Footer() {
  const t = useTranslations("footer");
  const year = new Date().getFullYear();

  const [email, setEmail] = useState("");
  const [consent, setConsent] = useState(false);
  const [newsletterState, setNewsletterState] = useState<NewsletterState>("idle");
  const [errorMessage, setErrorMessage] = useState("");

  async function handleNewsletterSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();

    // Client-side validation
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setErrorMessage(t("newsletter_error_email"));
      return;
    }
    if (!consent) {
      setErrorMessage(t("newsletter_error_consent"));
      return;
    }

    setErrorMessage("");
    setNewsletterState("loading");

    try {
      const res = await fetch("/api/newsletter", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email }),
      });

      if (!res.ok) throw new Error("Server error");
      setNewsletterState("success");
      setEmail("");
      setConsent(false);
    } catch {
      setNewsletterState("error");
      setErrorMessage(t("newsletter_error_server"));
    }
  }

  return (
    <footer className="footer" role="contentinfo">
      <div className="container">

        {/* ── Main Grid ── */}
        <div className="footer__grid">

          {/* Column 1: Brand */}
          <div className="footer__col footer__col--brand">
            <Link href="/" className="footer__logo" aria-label="La Nonna — inicio">
              <span className="footer__logo-text">La Nonna</span>
            </Link>
            <p className="footer__tagline">{t("tagline")}</p>

            {/* Social — Instagram only (30-day rule applied: Facebook excluded) */}
            <div className="footer__social" aria-label={t("social_label")}>
              <a
                href="https://www.instagram.com/lanonna.gracia"
                target="_blank"
                rel="noopener noreferrer"
                className="footer__social-link"
                aria-label="Instagram de La Nonna (@lanonna.gracia)"
              >
                <InstagramIcon />
              </a>
            </div>
          </div>

          {/* Column 2: Navigation */}
          <div className="footer__col">
            <p className="footer__col-title">{t("col_restaurant")}</p>
            <nav className="footer__links" aria-label={t("col_restaurant")}>
              <Link href="/carta" className="footer__link">{t("links.carta")}</Link>
              <Link href="/historia" className="footer__link">{t("links.historia")}</Link>
              <Link href="/eventos" className="footer__link">{t("links.eventos")}</Link>
              <Link href="/reservar" className="footer__link">{t("links.reservar")}</Link>
            </nav>
          </div>

          {/* Column 3: Contact */}
          <div className="footer__col">
            <p className="footer__col-title">{t("col_visit")}</p>
            <address className="footer__address">
              <span>C/ Verdi, 28</span>
              <span>08012 Gràcia, Barcelona</span>
              <a
                href={`tel:${t("phone").replace(/\s/g, "")}`}
                className="footer__address-link"
              >
                {t("phone")}
              </a>
              <a
                href={`mailto:${t("email")}`}
                className="footer__address-link"
              >
                {t("email")}
              </a>
              <a
                href="https://maps.google.com/?q=C+Verdi+28+Barcelona"
                target="_blank"
                rel="noopener noreferrer"
                className="footer__address-link footer__address-link--maps"
              >
                {t("maps_link")} ↗
              </a>
            </address>

            <p className="footer__col-title footer__col-title--hours">{t("hours_label")}</p>
            <div className="footer__hours">
              {(t.raw("hours") as string[]).map((line: string) => (
                <span key={line}>{line}</span>
              ))}
            </div>
          </div>

          {/* Column 4: Newsletter */}
          <div className="footer__col footer__col--newsletter">
            <p className="footer__col-title">{t("newsletter_title")}</p>
            <p className="footer__newsletter-desc">{t("newsletter_desc")}</p>

            {newsletterState === "success" ? (
              <div className="footer__newsletter-success" role="status" aria-live="polite">
                <span className="footer__newsletter-success-icon" aria-hidden="true">✓</span>
                <p>{t("newsletter_success")}</p>
              </div>
            ) : (
              <form
                className="footer__newsletter-form"
                onSubmit={handleNewsletterSubmit}
                noValidate
              >
                <div className="footer__newsletter-field">
                  <label htmlFor="footer-email" className="sr-only">
                    {t("newsletter_email_label")}
                  </label>
                  <input
                    id="footer-email"
                    type="email"
                    name="email"
                    value={email}
                    onChange={(e) => {
                      setEmail(e.target.value);
                      setErrorMessage("");
                      if (newsletterState === "error") setNewsletterState("idle");
                    }}
                    placeholder={t("newsletter_email_placeholder")}
                    className="footer__newsletter-input"
                    autoComplete="email"
                    required
                    aria-required="true"
                    aria-describedby={errorMessage ? "footer-newsletter-error" : undefined}
                    disabled={newsletterState === "loading"}
                  />
                  <button
                    type="submit"
                    className="footer__newsletter-btn"
                    disabled={newsletterState === "loading"}
                    aria-label={t("newsletter_cta")}
                  >
                    {newsletterState === "loading"
                      ? t("newsletter_sending")
                      : t("newsletter_cta")}
                  </button>
                </div>

                {/* GDPR consent — required for Spain/EU (RGPD art. 6.1.a) */}
                <label className="footer__newsletter-consent">
                  <input
                    type="checkbox"
                    checked={consent}
                    onChange={(e) => {
                      setConsent(e.target.checked);
                      setErrorMessage("");
                    }}
                    className="footer__newsletter-checkbox"
                    required
                    aria-required="true"
                  />
                  <span className="footer__newsletter-consent-text">
                    {t("newsletter_consent_prefix")}{" "}
                    <Link href="/privacidad" className="footer__newsletter-consent-link">
                      {t("newsletter_consent_link")}
                    </Link>
                  </span>
                </label>

                {errorMessage && (
                  <p
                    id="footer-newsletter-error"
                    className="footer__newsletter-error"
                    role="alert"
                    aria-live="assertive"
                  >
                    {errorMessage}
                  </p>
                )}
              </form>
            )}
          </div>

        </div>

        {/* ── Bottom Bar ── */}
        <div className="footer__bottom">
          <p className="footer__copy">
            © {year} La Nonna.{" "}
            <span className="footer__copy-rights">{t("copy_rights")}</span>
          </p>
          <nav className="footer__legal" aria-label={t("col_legal")}>
            <Link href="/aviso-legal" className="footer__legal-link">
              {t("legal_aviso")}
            </Link>
            {/* TODO: create /privacidad and /cookies pages before launch */}
            <Link href="/privacidad" className="footer__legal-link">
              {t("legal_privacidad")}
            </Link>
            <Link href="/cookies" className="footer__legal-link">
              {t("legal_cookies")}
            </Link>
          </nav>
        </div>

      </div>
    </footer>
  );
}
```

---

### Newsletter API Route: `app/api/newsletter/route.ts`

```ts
import { NextRequest, NextResponse } from "next/server";
import { Resend } from "resend";
import { z } from "zod";

const schema = z.object({
  email: z.string().email(),
});

// Lazy init — same pattern used by existing server actions in this project
function getResend() {
  if (!process.env.RESEND_API_KEY) throw new Error("RESEND_API_KEY not set");
  return new Resend(process.env.RESEND_API_KEY);
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { email } = schema.parse(body);

    const resend = getResend();

    // Option A: Send notification email to restaurant (simple, no list management)
    // Option B: Add to Resend audience (requires Resend audience feature)
    // Currently implementing Option A — can upgrade to audience when client confirms list strategy

    await resend.emails.send({
      from: "La Nonna Web <noreply@lanonna.es>",
      to: process.env.RESTAURANT_EMAIL ?? "info@lanonna.es",
      subject: "Nueva suscripción al boletín",
      text: `Nueva suscripción al boletín de La Nonna.\n\nEmail: ${email}\n\nFecha: ${new Date().toLocaleString("es-ES", { timeZone: "Europe/Madrid" })}`,
    });

    return NextResponse.json({ ok: true });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: "Email inválido" }, { status: 400 });
    }
    console.error("[newsletter] Error:", error);
    return NextResponse.json({ error: "Server error" }, { status: 500 });
  }
}
```

---

### CSS Additions to `components.css`

```css
/* ─────────────────────────────────────────
   Footer — full layout including newsletter
   ───────────────────────────────────────── */

/* Override existing grid to 4 columns on large screens */
@media (min-width: 1024px) {
  .footer__grid {
    grid-template-columns: 1.5fr 1fr 1.2fr 1.5fr;
  }
}

/* Brand column */
.footer__col--brand { display: flex; flex-direction: column; gap: var(--space-4); }

.footer__logo { text-decoration: none; }
.footer__logo-text {
  font-family: var(--font-display);
  font-size: var(--text-body-lg);
  font-weight: 600;
  color: var(--color-crema-100);
  letter-spacing: 0.02em;
}

.footer__tagline {
  font-size: var(--text-body-sm);
  color: rgba(250, 246, 240, 0.60);
  font-style: italic;
  margin: 0;
  line-height: 1.5;
}

/* Social icons */
.footer__social { display: flex; gap: var(--space-3); margin-top: var(--space-2); }
.footer__social-link {
  color: rgba(250, 246, 240, 0.50);
  text-decoration: none;
  transition: color var(--duration-fast);
  display: flex;
  align-items: center;
}
.footer__social-link:hover { color: var(--color-crema-50); }

/* Hours block */
.footer__col-title--hours { margin-top: var(--space-6); }
.footer__hours {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  font-size: var(--text-body-sm);
  color: rgba(250, 246, 240, 0.60);
  font-style: normal;
}

/* Address link variant */
.footer__address-link {
  display: block;
  color: rgba(250, 246, 240, 0.60);
  text-decoration: none;
  transition: color var(--duration-fast);
  font-size: var(--text-body-sm);
}
.footer__address-link:hover { color: var(--color-crema-50); }
.footer__address-link--maps {
  color: rgba(250, 246, 240, 0.45);
  font-size: var(--text-body-xs);
  margin-top: var(--space-2);
}

/* Newsletter column */
.footer__col--newsletter { display: flex; flex-direction: column; gap: var(--space-3); }

.footer__newsletter-desc {
  font-size: var(--text-body-sm);
  color: rgba(250, 246, 240, 0.60);
  line-height: 1.6;
  margin: 0;
}

/* Newsletter form */
.footer__newsletter-form { display: flex; flex-direction: column; gap: var(--space-3); }

.footer__newsletter-field { display: flex; flex-direction: column; gap: var(--space-2); }
@media (min-width: 480px) {
  .footer__newsletter-field { flex-direction: row; }
}

.footer__newsletter-input {
  flex: 1;
  padding: var(--space-3) var(--space-4);
  background: rgba(250, 246, 240, 0.08);
  border: 1px solid rgba(250, 246, 240, 0.20);
  border-radius: 4px;
  color: var(--color-crema-100);
  font-size: var(--text-body-sm);
  font-family: inherit;
  transition: border-color var(--duration-fast);
  min-width: 0;
}
.footer__newsletter-input::placeholder { color: rgba(250, 246, 240, 0.35); }
.footer__newsletter-input:focus {
  outline: none;
  border-color: rgba(250, 246, 240, 0.50);
}
.footer__newsletter-input:disabled { opacity: 0.5; cursor: not-allowed; }

.footer__newsletter-btn {
  padding: var(--space-3) var(--space-5);
  background: var(--color-brand-primary);
  color: var(--color-crema-100);
  border: none;
  border-radius: 4px;
  font-size: var(--text-body-sm);
  font-family: inherit;
  font-weight: 500;
  cursor: pointer;
  transition: background var(--duration-fast);
  white-space: nowrap;
}
.footer__newsletter-btn:hover:not(:disabled) {
  background: var(--color-brand-primary-dark);
}
.footer__newsletter-btn:disabled { opacity: 0.6; cursor: not-allowed; }

/* GDPR consent */
.footer__newsletter-consent {
  display: flex;
  align-items: flex-start;
  gap: var(--space-2);
  cursor: pointer;
}
.footer__newsletter-checkbox {
  margin-top: 2px;
  flex-shrink: 0;
  accent-color: var(--color-brand-primary);
  width: 14px;
  height: 14px;
  cursor: pointer;
}
.footer__newsletter-consent-text {
  font-size: var(--text-body-xs);
  color: rgba(250, 246, 240, 0.45);
  line-height: 1.5;
}
.footer__newsletter-consent-link {
  color: rgba(250, 246, 240, 0.60);
  text-decoration: underline;
  transition: color var(--duration-fast);
}
.footer__newsletter-consent-link:hover { color: var(--color-crema-50); }

/* Error state */
.footer__newsletter-error {
  font-size: var(--text-body-xs);
  color: #e8907a; /* terracota-300 — readable on dark bg */
  margin: 0;
  line-height: 1.4;
}

/* Success state */
.footer__newsletter-success {
  display: flex;
  align-items: flex-start;
  gap: var(--space-3);
  padding: var(--space-4);
  background: rgba(92, 107, 58, 0.20); /* oliva subtle */
  border: 1px solid rgba(92, 107, 58, 0.40);
  border-radius: 4px;
}
.footer__newsletter-success-icon {
  color: var(--color-oliva-300);
  font-size: var(--text-body-md);
  line-height: 1;
  flex-shrink: 0;
}
.footer__newsletter-success p {
  font-size: var(--text-body-sm);
  color: rgba(250, 246, 240, 0.75);
  margin: 0;
  line-height: 1.5;
}
```

---

### Translation Additions — `messages/es.json` (footer section)

Add the following keys to the existing `footer` object:

```json
"footer": {
  "tagline":        "Cocina italiana de familia desde 1985",
  "col_restaurant": "El restaurante",
  "col_visit":      "Visítanos",
  "col_legal":      "Legal",
  "social_label":   "Redes sociales",
  "links": {
    "carta":    "Carta",
    "historia": "Nuestra Historia",
    "eventos":  "Eventos Privados",
    "reservar": "Reservar mesa"
  },
  "phone":        "+34 93 XXX XXXX",
  "email":        "info@lanonna.es",
  "hours_label":  "Horarios",
  "hours": [
    "Mar–Dom: 13:00–16:00",
    "Mar–Dom: 20:00–23:30",
    "Lunes: cerrado"
  ],
  "maps_link": "Ver en Google Maps",
  "newsletter_title":           "Novedades de La Nonna",
  "newsletter_desc":            "Recibe nuestras novedades: menú del día, eventos especiales y recetas de la abuela",
  "newsletter_email_label":     "Tu dirección de email",
  "newsletter_email_placeholder": "tu@email.com",
  "newsletter_cta":             "Suscribirme",
  "newsletter_sending":         "Enviando...",
  "newsletter_success":         "¡Gracias! Te avisaremos cuando haya algo especial.",
  "newsletter_error_email":     "Introduce un email válido",
  "newsletter_error_consent":   "Debes aceptar la política de privacidad para suscribirte",
  "newsletter_error_server":    "Ha habido un error. Inténtalo de nuevo.",
  "newsletter_consent_prefix":  "He leído y acepto la",
  "newsletter_consent_link":    "política de privacidad",
  "legal_aviso":      "Aviso legal",
  "legal_privacidad": "Política de privacidad",
  "legal_cookies":    "Política de cookies",
  "copy_rights":      "Todos los derechos reservados."
}
```

### Translation Additions — `messages/en.json` (footer section)

```json
"footer": {
  "tagline":        "Artisan Italian cooking since 1985",
  "col_restaurant": "The restaurant",
  "col_visit":      "Visit us",
  "col_legal":      "Legal",
  "social_label":   "Social media",
  "links": {
    "carta":    "Menu",
    "historia": "Our Story",
    "eventos":  "Private Events",
    "reservar": "Book a Table"
  },
  "phone":        "+34 93 XXX XXXX",
  "email":        "info@lanonna.es",
  "hours_label":  "Opening hours",
  "hours": [
    "Tue–Sun: 1:00–4:00 pm",
    "Tue–Sun: 8:00–11:30 pm",
    "Monday: closed"
  ],
  "maps_link": "View on Google Maps",
  "newsletter_title":             "La Nonna updates",
  "newsletter_desc":              "Get our updates: daily specials, private events and recipes from the kitchen",
  "newsletter_email_label":       "Your email address",
  "newsletter_email_placeholder": "you@email.com",
  "newsletter_cta":               "Subscribe",
  "newsletter_sending":           "Sending...",
  "newsletter_success":           "Thank you! We'll let you know when something special is on.",
  "newsletter_error_email":       "Please enter a valid email",
  "newsletter_error_consent":     "You must accept the privacy policy to subscribe",
  "newsletter_error_server":      "Something went wrong. Please try again.",
  "newsletter_consent_prefix":    "I have read and accept the",
  "newsletter_consent_link":      "privacy policy",
  "legal_aviso":      "Legal notice",
  "legal_privacidad": "Privacy policy",
  "legal_cookies":    "Cookie policy",
  "copy_rights":      "All rights reserved."
}
```

---

## PRE-LAUNCH CHECKLIST

- [ ] Confirm real phone number with client (replace `+34 93 XXX XXXX`)
- [ ] Confirm real address — task brief says Verdi 23, project memory says Verdi 28
- [ ] Add `RESEND_API_KEY` and `RESTAURANT_EMAIL` to `.env.local` and Vercel env vars
- [ ] Create `/privacidad` (Privacy Policy) page — currently only covered in `/aviso-legal`
- [ ] Create `/cookies` (Cookie Policy) page — same note
- [ ] Consider upgrading newsletter to Resend Audiences for list management (instead of email notification)
- [ ] Add CIF to `/aviso-legal` once confirmed with client (recommended for LSSICE compliance)
- [ ] Verify Instagram handle — task brief says `@lanonna.gracia`, contacto page says `@lanonna_gracia`

---

## INTEGRATION INSTRUCTIONS

1. Replace the existing `Footer.tsx` at `/src/components/layout/Footer.tsx` with the component above
2. Add the CSS block to `/src/styles/components.css` — place after the existing footer block (line ~884)
3. Merge the translation keys into `messages/es.json` and `messages/en.json`
4. Create `app/api/newsletter/route.ts` with the API route above
5. Run `npm run build` to verify TypeScript types and SSG compilation
