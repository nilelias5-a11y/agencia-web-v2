# Backend Eval 1 — Contact Form Server Action (WITH SKILL)

**Project:** Ana López Fotógrafa  
**Task:** Complete Server Action for contact form  
**Skill applied:** `backend-developer.md`  
**Date:** 2026-05-13

---

## Overview

Full implementation of the contact form backend for Ana López's photography website.  
Covers: T3 Env typed variables, Zod schema with session type enum, Upstash rate limiting (3 req / 10 min per IP), dual Resend emails with React Email templates, and secure error handling.

---

## File Structure

```
src/
├── env.ts                                    # T3 Env — typed environment variables
├── lib/
│   ├── ratelimit.ts                          # Upstash Ratelimit instance
│   ├── email.ts                              # Resend client + sendContactEmails()
│   └── validations/
│       └── contact.ts                        # Zod schema + inferred types
├── emails/
│   ├── contact-notification.tsx              # Email to Ana
│   └── contact-confirmation.tsx              # Email to the user
└── app/
    └── actions/
        └── contact.ts                        # Server Action (entry point)
```

---

## 1. `src/env.ts` — T3 Env

```typescript
// src/env.ts
import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  server: {
    RESEND_API_KEY: z.string().startsWith("re_"),
    UPSTASH_REDIS_REST_URL: z.string().url(),
    UPSTASH_REDIS_REST_TOKEN: z.string().min(1),
    CONTACT_EMAIL_TO: z.string().email(), // ana@analopez.com
    EMAIL_FROM_DOMAIN: z.string().min(1), // e.g. analopez.com
  },
  client: {
    // No client-side secrets for this feature
  },
  runtimeEnv: {
    RESEND_API_KEY: process.env.RESEND_API_KEY,
    UPSTASH_REDIS_REST_URL: process.env.UPSTASH_REDIS_REST_URL,
    UPSTASH_REDIS_REST_TOKEN: process.env.UPSTASH_REDIS_REST_TOKEN,
    CONTACT_EMAIL_TO: process.env.CONTACT_EMAIL_TO,
    EMAIL_FROM_DOMAIN: process.env.EMAIL_FROM_DOMAIN,
  },
})
```

**`.env.local` (reference — never commit):**
```
RESEND_API_KEY=re_xxxxxxxxxxxxxxxxxxxx
UPSTASH_REDIS_REST_URL=https://xxxxxxxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=AXxxxxxxxxxxxxxxxxxxxxxxxx
CONTACT_EMAIL_TO=ana@analopez.com
EMAIL_FROM_DOMAIN=analopez.com
```

---

## 2. `src/lib/validations/contact.ts` — Zod Schema

```typescript
// src/lib/validations/contact.ts
import { z } from "zod"

export const SESSION_TYPES = ["bodas", "retratos", "corporativo"] as const
export type SessionType = (typeof SESSION_TYPES)[number]

export const contactSchema = z.object({
  name: z
    .string()
    .min(2, "El nombre debe tener al menos 2 caracteres")
    .max(100, "El nombre no puede superar los 100 caracteres")
    .trim(),

  email: z
    .string()
    .email("Por favor, introduce un email válido")
    .toLowerCase()
    .trim(),

  sessionType: z.enum(SESSION_TYPES, {
    errorMap: () => ({
      message: "El tipo de sesión debe ser: bodas, retratos o corporativo",
    }),
  }),

  message: z
    .string()
    .min(10, "El mensaje debe tener al menos 10 caracteres")
    .max(2000, "El mensaje no puede superar los 2000 caracteres")
    .trim(),
})

export type ContactInput = z.infer<typeof contactSchema>

// Map for human-readable session type labels (used in emails)
export const SESSION_TYPE_LABELS: Record<SessionType, string> = {
  bodas: "Sesión de bodas",
  retratos: "Sesión de retratos",
  corporativo: "Sesión corporativa",
}
```

---

## 3. `src/lib/ratelimit.ts` — Upstash Rate Limiting

```typescript
// src/lib/ratelimit.ts
import { Ratelimit } from "@upstash/ratelimit"
import { Redis } from "@upstash/redis"
import { env } from "@/env"

// Redis client — reads UPSTASH_REDIS_REST_URL and UPSTASH_REDIS_REST_TOKEN
// from env (T3 Env validated), not directly from process.env
const redis = new Redis({
  url: env.UPSTASH_REDIS_REST_URL,
  token: env.UPSTASH_REDIS_REST_TOKEN,
})

/**
 * Contact form rate limit:
 * Maximum 3 submissions per 10 minutes per IP address.
 * Uses sliding window to prevent burst abuse at window boundaries.
 */
export const contactRatelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(3, "10 m"),
  analytics: true,
  prefix: "ratelimit:contact",
})
```

---

## 4. `src/lib/email.ts` — Resend Client + Email Sender

```typescript
// src/lib/email.ts
import { Resend } from "resend"
import { env } from "@/env"
import { ContactNotificationEmail } from "@/emails/contact-notification"
import { ContactConfirmationEmail } from "@/emails/contact-confirmation"
import type { ContactInput } from "@/lib/validations/contact"
import { SESSION_TYPE_LABELS } from "@/lib/validations/contact"

export const resend = new Resend(env.RESEND_API_KEY)

interface SendContactEmailsResult {
  notificationSent: boolean
  confirmationSent: boolean
}

/**
 * Sends two emails concurrently:
 * 1. Notification to Ana López (new lead alert)
 * 2. Confirmation to the user (personalised receipt)
 *
 * Uses Promise.allSettled so one failure does not block the other.
 * All errors are logged internally; the caller receives a clean boolean result.
 */
export async function sendContactEmails(
  data: ContactInput
): Promise<SendContactEmailsResult> {
  const sessionLabel = SESSION_TYPE_LABELS[data.sessionType]
  const fromNotifications = `Ana López Fotografía <notificaciones@${env.EMAIL_FROM_DOMAIN}>`
  const fromConfirmations = `Ana López Fotografía <hola@${env.EMAIL_FROM_DOMAIN}>`

  const [notification, confirmation] = await Promise.allSettled([
    resend.emails.send({
      from: fromNotifications,
      to: [env.CONTACT_EMAIL_TO],
      subject: `Nueva consulta de ${data.name} — ${sessionLabel}`,
      react: ContactNotificationEmail({
        name: data.name,
        email: data.email,
        sessionType: data.sessionType,
        sessionLabel,
        message: data.message,
      }),
    }),
    resend.emails.send({
      from: fromConfirmations,
      to: [data.email],
      subject: `Gracias por contactarme, ${data.name}`,
      react: ContactConfirmationEmail({
        name: data.name,
        sessionLabel,
      }),
    }),
  ])

  if (notification.status === "rejected") {
    console.error(
      "[sendContactEmails] Notification email failed:",
      notification.reason
    )
  }

  if (confirmation.status === "rejected") {
    console.error(
      "[sendContactEmails] Confirmation email failed:",
      confirmation.reason
    )
  }

  return {
    notificationSent: notification.status === "fulfilled",
    confirmationSent: confirmation.status === "fulfilled",
  }
}
```

---

## 5. `src/emails/contact-notification.tsx` — Email to Ana

```tsx
// src/emails/contact-notification.tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Section,
  Text,
} from "@react-email/components"
import type { SessionType } from "@/lib/validations/contact"

interface ContactNotificationEmailProps {
  name: string
  email: string
  sessionType: SessionType
  sessionLabel: string
  message: string
}

export function ContactNotificationEmail({
  name,
  email,
  sessionType,
  sessionLabel,
  message,
}: ContactNotificationEmailProps) {
  return (
    <Html lang="es">
      <Head />
      <Preview>Nueva consulta de {name} — {sessionLabel}</Preview>
      <Body style={bodyStyle}>
        <Container style={containerStyle}>
          <Heading style={headingStyle}>
            📸 Nueva consulta de contacto
          </Heading>

          <Section style={sectionStyle}>
            <Text style={labelStyle}>Nombre</Text>
            <Text style={valueStyle}>{name}</Text>
          </Section>

          <Section style={sectionStyle}>
            <Text style={labelStyle}>Email</Text>
            <Text style={valueStyle}>{email}</Text>
          </Section>

          <Section style={sectionStyle}>
            <Text style={labelStyle}>Tipo de sesión</Text>
            <Text style={valueStyle}>{sessionLabel}</Text>
          </Section>

          <Hr style={hrStyle} />

          <Section style={sectionStyle}>
            <Text style={labelStyle}>Mensaje</Text>
            <Text style={{ ...valueStyle, whiteSpace: "pre-wrap" }}>
              {message}
            </Text>
          </Section>

          <Hr style={hrStyle} />

          <Text style={footerStyle}>
            Responde directamente a este email para contactar con {name} ({email}).
          </Text>
        </Container>
      </Body>
    </Html>
  )
}

// ─── Styles ──────────────────────────────────────────────────────────────────

const bodyStyle: React.CSSProperties = {
  backgroundColor: "#f4f4f5",
  fontFamily:
    '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
  margin: 0,
  padding: 0,
}

const containerStyle: React.CSSProperties = {
  backgroundColor: "#ffffff",
  borderRadius: "8px",
  margin: "40px auto",
  maxWidth: "600px",
  padding: "40px 48px",
}

const headingStyle: React.CSSProperties = {
  color: "#1a1a1a",
  fontSize: "24px",
  fontWeight: "700",
  marginBottom: "32px",
  marginTop: 0,
}

const sectionStyle: React.CSSProperties = {
  marginBottom: "16px",
}

const labelStyle: React.CSSProperties = {
  color: "#71717a",
  fontSize: "12px",
  fontWeight: "600",
  letterSpacing: "0.05em",
  margin: "0 0 4px 0",
  textTransform: "uppercase",
}

const valueStyle: React.CSSProperties = {
  color: "#1a1a1a",
  fontSize: "15px",
  margin: 0,
}

const hrStyle: React.CSSProperties = {
  borderColor: "#e4e4e7",
  margin: "24px 0",
}

const footerStyle: React.CSSProperties = {
  color: "#71717a",
  fontSize: "13px",
  margin: 0,
}
```

---

## 6. `src/emails/contact-confirmation.tsx` — Email to the User

```tsx
// src/emails/contact-confirmation.tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Text,
} from "@react-email/components"

interface ContactConfirmationEmailProps {
  name: string
  sessionLabel: string
}

export function ContactConfirmationEmail({
  name,
  sessionLabel,
}: ContactConfirmationEmailProps) {
  return (
    <Html lang="es">
      <Head />
      <Preview>He recibido tu consulta, {name} — te escribiré pronto</Preview>
      <Body style={bodyStyle}>
        <Container style={containerStyle}>
          <Heading style={headingStyle}>
            ¡Gracias, {name}!
          </Heading>

          <Text style={textStyle}>
            He recibido tu consulta sobre una <strong>{sessionLabel}</strong>.
            Me encantará conocer tu proyecto con más detalle.
          </Text>

          <Text style={textStyle}>
            Te responderé en un plazo de <strong>1–2 días hábiles</strong>.
            Si tienes cualquier pregunta urgente, puedes escribirme directamente
            a{" "}
            <a href="mailto:ana@analopez.com" style={linkStyle}>
              ana@analopez.com
            </a>
            .
          </Text>

          <Hr style={hrStyle} />

          <Text style={signatureStyle}>
            Con cariño,
            <br />
            <strong>Ana López</strong>
            <br />
            Fotógrafa · Barcelona
          </Text>
        </Container>
      </Body>
    </Html>
  )
}

// ─── Styles ──────────────────────────────────────────────────────────────────

const bodyStyle: React.CSSProperties = {
  backgroundColor: "#fafaf9",
  fontFamily:
    '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
  margin: 0,
  padding: 0,
}

const containerStyle: React.CSSProperties = {
  backgroundColor: "#ffffff",
  borderRadius: "8px",
  margin: "40px auto",
  maxWidth: "600px",
  padding: "40px 48px",
}

const headingStyle: React.CSSProperties = {
  color: "#1a1a1a",
  fontSize: "26px",
  fontWeight: "700",
  marginBottom: "24px",
  marginTop: 0,
}

const textStyle: React.CSSProperties = {
  color: "#3f3f46",
  fontSize: "15px",
  lineHeight: "1.6",
  marginBottom: "16px",
}

const linkStyle: React.CSSProperties = {
  color: "#a16207",
  textDecoration: "none",
}

const hrStyle: React.CSSProperties = {
  borderColor: "#e4e4e7",
  margin: "32px 0 24px",
}

const signatureStyle: React.CSSProperties = {
  color: "#71717a",
  fontSize: "14px",
  lineHeight: "1.6",
  margin: 0,
}
```

---

## 7. `src/app/actions/contact.ts` — Server Action

```typescript
// src/app/actions/contact.ts
"use server"

import { headers } from "next/headers"
import { contactSchema } from "@/lib/validations/contact"
import { contactRatelimit } from "@/lib/ratelimit"
import { sendContactEmails } from "@/lib/email"
import type { ContactInput } from "@/lib/validations/contact"

// ─── Return type ─────────────────────────────────────────────────────────────

export type ContactActionResult =
  | { success: true }
  | { success: false; error: string; fieldErrors?: Partial<Record<keyof ContactInput, string[]>> }

// ─── Server Action ────────────────────────────────────────────────────────────

/**
 * submitContact — public Server Action for the Ana López photography contact form.
 *
 * Pipeline:
 *   1. Extract IP for rate limiting
 *   2. Apply sliding-window rate limit (3 req / 10 min per IP)
 *   3. Parse and validate form data with Zod
 *   4. Send dual emails via Resend (notification + confirmation)
 *   5. Return typed result — never expose internal errors to the client
 */
export async function submitContact(
  formData: FormData
): Promise<ContactActionResult> {
  // ── 1. Rate limiting ───────────────────────────────────────────────────────
  const headersList = await headers()
  const ip =
    headersList.get("x-forwarded-for")?.split(",")[0]?.trim() ??
    headersList.get("x-real-ip") ??
    "anonymous"

  let rateLimitResult: Awaited<ReturnType<typeof contactRatelimit.limit>>
  try {
    rateLimitResult = await contactRatelimit.limit(ip)
  } catch (error) {
    // If Upstash is unreachable, fail open with a log — do not block users
    console.error("[submitContact] Rate limit check failed:", error)
    rateLimitResult = { success: true, limit: 3, remaining: 0, reset: 0, pending: Promise.resolve() }
  }

  if (!rateLimitResult.success) {
    return {
      success: false,
      error: "Demasiadas solicitudes. Por favor, espera unos minutos antes de intentarlo de nuevo.",
    }
  }

  // ── 2. Input validation ────────────────────────────────────────────────────
  const raw = {
    name: formData.get("name"),
    email: formData.get("email"),
    sessionType: formData.get("sessionType"),
    message: formData.get("message"),
  }

  const result = contactSchema.safeParse(raw)

  if (!result.success) {
    return {
      success: false,
      error: "Por favor, revisa los campos del formulario.",
      fieldErrors: result.error.flatten().fieldErrors as Partial<
        Record<keyof ContactInput, string[]>
      >,
    }
  }

  const { name, email, sessionType, message } = result.data

  // ── 3. Send emails ─────────────────────────────────────────────────────────
  try {
    const emailResult = await sendContactEmails({ name, email, sessionType, message })

    // Log if notification to Ana failed — actionable for monitoring
    if (!emailResult.notificationSent) {
      console.error(
        "[submitContact] ALERT: Notification email to Ana not sent.",
        { name, email, sessionType }
      )
    }

    // Log if confirmation to user failed — lower severity
    if (!emailResult.confirmationSent) {
      console.warn(
        "[submitContact] Confirmation email to user not sent.",
        { email }
      )
    }

    // If the critical notification email failed, treat as hard error
    if (!emailResult.notificationSent) {
      return {
        success: false,
        error: "No hemos podido enviar tu mensaje. Por favor, inténtalo de nuevo o contacta directamente a ana@analopez.com.",
      }
    }

    return { success: true }
  } catch (error) {
    console.error("[submitContact] Unexpected error during email send:", error)
    return {
      success: false,
      error: "Ha ocurrido un error inesperado. Por favor, inténtalo de nuevo.",
    }
  }
}
```

---

## 8. Usage in a React Client Component (reference)

```tsx
// src/components/contact-form.tsx
"use client"

import { useActionState } from "react"
import { submitContact } from "@/app/actions/contact"
import type { ContactActionResult } from "@/app/actions/contact"
import { SESSION_TYPES, SESSION_TYPE_LABELS } from "@/lib/validations/contact"

const initialState: ContactActionResult | null = null

export function ContactForm() {
  const [state, formAction, isPending] = useActionState(
    submitContact,
    initialState
  )

  if (state?.success) {
    return (
      <p className="text-green-700">
        ¡Mensaje enviado! Te confirmaré por email en breve.
      </p>
    )
  }

  return (
    <form action={formAction} noValidate>
      {/* Name */}
      <div>
        <label htmlFor="name">Nombre</label>
        <input id="name" name="name" type="text" required />
        {!state?.success && state?.fieldErrors?.name && (
          <p className="text-red-600">{state.fieldErrors.name[0]}</p>
        )}
      </div>

      {/* Email */}
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" required />
        {!state?.success && state?.fieldErrors?.email && (
          <p className="text-red-600">{state.fieldErrors.email[0]}</p>
        )}
      </div>

      {/* Session type */}
      <div>
        <label htmlFor="sessionType">Tipo de sesión</label>
        <select id="sessionType" name="sessionType" required>
          <option value="">Selecciona una opción</option>
          {SESSION_TYPES.map((type) => (
            <option key={type} value={type}>
              {SESSION_TYPE_LABELS[type]}
            </option>
          ))}
        </select>
        {!state?.success && state?.fieldErrors?.sessionType && (
          <p className="text-red-600">{state.fieldErrors.sessionType[0]}</p>
        )}
      </div>

      {/* Message */}
      <div>
        <label htmlFor="message">Mensaje</label>
        <textarea id="message" name="message" required rows={5} />
        {!state?.success && state?.fieldErrors?.message && (
          <p className="text-red-600">{state.fieldErrors.message[0]}</p>
        )}
      </div>

      {/* Global error */}
      {state && !state.success && !state.fieldErrors && (
        <p className="text-red-600">{state.error}</p>
      )}

      <button type="submit" disabled={isPending}>
        {isPending ? "Enviando…" : "Enviar mensaje"}
      </button>
    </form>
  )
}
```

---

## 9. Required Dependencies

```bash
# Install all required packages
pnpm add @t3-oss/env-nextjs zod resend @react-email/components @upstash/ratelimit @upstash/redis
```

| Package | Version (min) | Purpose |
|---|---|---|
| `@t3-oss/env-nextjs` | ^0.10 | Type-safe environment variables |
| `zod` | ^3.23 | Input validation |
| `resend` | ^4.0 | Email API client |
| `@react-email/components` | ^0.0.22 | Email template primitives |
| `@upstash/ratelimit` | ^2.0 | Sliding window rate limiting |
| `@upstash/redis` | ^1.31 | Redis client for Upstash |

---

## 10. Security Checklist

- [x] All inputs validated with Zod before any processing
- [x] Rate limiting (3/10 min per IP) via Upstash sliding window
- [x] IP extracted from `x-forwarded-for` header (first IP to prevent spoofing)
- [x] Secrets accessed exclusively via `env.*` — zero bare `process.env` calls
- [x] Internal errors logged with `[submitContact]` prefix for searchability
- [x] Generic error messages returned to the client — no stack traces exposed
- [x] `Promise.allSettled` ensures one email failure does not crash the other
- [x] Rate limit fail-open strategy prevents Upstash outage from blocking users
- [x] Notification email failure treated as hard error (critical for the business)
- [x] Confirmation email failure treated as soft warning (user still submitted successfully only if notification went through)

---

## Design Decisions

**Why fail-open on rate limit?**  
If Upstash is unreachable, it is better to accept a legitimate submission than to block all users. The outage is logged for alerting.

**Why treat notification failure as a hard error?**  
Ana needs to receive every lead. If her notification is not delivered, the submission is silently lost from her perspective. We surface this to the user so they can try again or use the direct email.

**Why `Promise.allSettled` instead of `Promise.all`?**  
`Promise.all` would cancel both if one fails. With `allSettled`, the confirmation still reaches the user even if the notification has a transient failure, and vice versa.

**Why `sessionType` as an enum (not free text)?**  
Prevents injection, ensures consistent filtering/routing logic, and makes email templates type-safe without defensive checks.
