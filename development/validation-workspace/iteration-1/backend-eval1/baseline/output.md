# Eval 1 — Contact Form Server Action: Ana López Fotógrafa

Complete, runnable TypeScript implementation for Next.js 14+ App Router.

---

## File Structure

```
src/
  app/
    contacto/
      page.tsx                          # Page with ContactForm component
      actions.ts                        # Server Action (main file)
  components/
    contact/
      ContactForm.tsx                   # Client component form
  emails/
    ContactNotificationEmail.tsx        # Template: notification to Ana
    ContactConfirmationEmail.tsx        # Template: confirmation to user
  lib/
    rate-limit.ts                       # Upstash rate limiter helper
```

---

## Dependencies

```json
{
  "dependencies": {
    "@upstash/ratelimit": "^1.2.1",
    "@upstash/redis": "^1.34.0",
    "resend": "^3.5.0",
    "@react-email/components": "^0.0.22",
    "react-email": "^2.1.6",
    "zod": "^3.23.8",
    "react": "^18.3.1",
    "next": "^14.2.20"
  }
}
```

### Environment Variables

```env
# .env.local
UPSTASH_REDIS_REST_URL=https://your-redis.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-token
RESEND_API_KEY=re_your_api_key
ANA_EMAIL=ana@analopez.com
NEXT_PUBLIC_SITE_URL=https://analopez.com
```

---

## 1. Zod Schema — `src/app/contacto/schema.ts`

```typescript
import { z } from "zod";

export const sessionTypes = ["bodas", "retratos", "corporativo"] as const;
export type SessionType = (typeof sessionTypes)[number];

export const contactFormSchema = z.object({
  nombre: z
    .string()
    .min(2, "El nombre debe tener al menos 2 caracteres")
    .max(100, "El nombre no puede superar los 100 caracteres")
    .trim(),

  email: z
    .string()
    .email("Por favor, introduce un email válido")
    .max(254, "El email no puede superar los 254 caracteres")
    .toLowerCase()
    .trim(),

  tipoSesion: z.enum(sessionTypes, {
    errorMap: () => ({
      message: "Por favor, selecciona un tipo de sesión válido",
    }),
  }),

  mensaje: z
    .string()
    .min(10, "El mensaje debe tener al menos 10 caracteres")
    .max(2000, "El mensaje no puede superar los 2000 caracteres")
    .trim(),
});

export type ContactFormValues = z.infer<typeof contactFormSchema>;

// Shape returned to the client (no sensitive data leaks)
export type ContactFormState = {
  success: boolean;
  message: string;
  fieldErrors?: Partial<Record<keyof ContactFormValues, string[]>>;
};
```

---

## 2. Rate Limiter — `src/lib/rate-limit.ts`

```typescript
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

// Lazily initialised so Next.js build-time imports do not throw
let _ratelimit: Ratelimit | null = null;

function getRatelimit(): Ratelimit {
  if (_ratelimit) return _ratelimit;

  if (
    !process.env.UPSTASH_REDIS_REST_URL ||
    !process.env.UPSTASH_REDIS_REST_TOKEN
  ) {
    throw new Error(
      "Missing Upstash environment variables: UPSTASH_REDIS_REST_URL and UPSTASH_REDIS_REST_TOKEN are required."
    );
  }

  const redis = new Redis({
    url: process.env.UPSTASH_REDIS_REST_URL,
    token: process.env.UPSTASH_REDIS_REST_TOKEN,
  });

  _ratelimit = new Ratelimit({
    redis,
    // 3 submissions per 10-minute sliding window per IP
    limiter: Ratelimit.slidingWindow(3, "10 m"),
    analytics: true,
    prefix: "ratelimit:contact",
  });

  return _ratelimit;
}

export interface RateLimitResult {
  success: boolean;
  /** Remaining requests in the current window */
  remaining: number;
  /** Unix timestamp (ms) when the window resets */
  reset: number;
  /** How many seconds until reset */
  retryAfter: number;
}

/**
 * Check rate limit for a given identifier (IP address).
 * Falls back to a permissive result when env vars are missing (local dev).
 */
export async function checkRateLimit(
  identifier: string
): Promise<RateLimitResult> {
  try {
    const ratelimit = getRatelimit();
    const result = await ratelimit.limit(identifier);

    const retryAfter = Math.ceil((result.reset - Date.now()) / 1000);

    return {
      success: result.success,
      remaining: result.remaining,
      reset: result.reset,
      retryAfter: retryAfter > 0 ? retryAfter : 0,
    };
  } catch (error) {
    // In development without Upstash configured, allow all requests
    if (process.env.NODE_ENV === "development") {
      console.warn("[RateLimit] Upstash not configured, skipping in dev:", error);
      return { success: true, remaining: 99, reset: 0, retryAfter: 0 };
    }
    // In production, fail closed (deny) to be safe
    throw error;
  }
}
```

---

## 3. React Email Templates

### 3a. Notification to Ana — `src/emails/ContactNotificationEmail.tsx`

```tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Row,
  Section,
  Text,
  Font,
} from "@react-email/components";
import * as React from "react";
import type { SessionType } from "@/app/contacto/schema";

const SESSION_TYPE_LABELS: Record<SessionType, string> = {
  bodas: "Bodas",
  retratos: "Retratos",
  corporativo: "Corporativo",
};

interface ContactNotificationEmailProps {
  nombre: string;
  email: string;
  tipoSesion: SessionType;
  mensaje: string;
  submittedAt: string; // ISO string, formatted before passing
  ip?: string;
}

export function ContactNotificationEmail({
  nombre,
  email,
  tipoSesion,
  mensaje,
  submittedAt,
  ip,
}: ContactNotificationEmailProps) {
  const previewText = `Nuevo contacto de ${nombre} — ${SESSION_TYPE_LABELS[tipoSesion]}`;

  return (
    <Html lang="es" dir="ltr">
      <Head>
        <Font
          fontFamily="Inter"
          fallbackFontFamily="Arial"
          webFont={{
            url: "https://fonts.gstatic.com/s/inter/v13/UcCO3FwrK3iLTeHuS_fvQtMwCp50KnMw2boKoduKmMEVuLyfAZ9hiA.woff2",
            format: "woff2",
          }}
          fontWeight={400}
          fontStyle="normal"
        />
      </Head>
      <Preview>{previewText}</Preview>
      <Body style={styles.body}>
        <Container style={styles.container}>
          {/* Header */}
          <Section style={styles.header}>
            <Heading style={styles.headerTitle}>Ana López Fotografía</Heading>
            <Text style={styles.headerSubtitle}>Panel de contacto</Text>
          </Section>

          {/* Alert badge */}
          <Section style={styles.alertSection}>
            <Text style={styles.alertBadge}>Nueva solicitud recibida</Text>
          </Section>

          {/* Client details */}
          <Section style={styles.card}>
            <Heading as="h2" style={styles.sectionTitle}>
              Datos del cliente
            </Heading>
            <Hr style={styles.divider} />

            <Row>
              <Section style={styles.fieldRow}>
                <Text style={styles.fieldLabel}>Nombre</Text>
                <Text style={styles.fieldValue}>{nombre}</Text>
              </Section>
            </Row>

            <Row>
              <Section style={styles.fieldRow}>
                <Text style={styles.fieldLabel}>Email</Text>
                <Text style={styles.fieldValue}>
                  <a href={`mailto:${email}`} style={styles.link}>
                    {email}
                  </a>
                </Text>
              </Section>
            </Row>

            <Row>
              <Section style={styles.fieldRow}>
                <Text style={styles.fieldLabel}>Tipo de sesión</Text>
                <Text style={styles.fieldValueAccent}>
                  {SESSION_TYPE_LABELS[tipoSesion]}
                </Text>
              </Section>
            </Row>
          </Section>

          {/* Message */}
          <Section style={styles.card}>
            <Heading as="h2" style={styles.sectionTitle}>
              Mensaje
            </Heading>
            <Hr style={styles.divider} />
            <Text style={styles.messageText}>{mensaje}</Text>
          </Section>

          {/* Metadata */}
          <Section style={styles.metaSection}>
            <Text style={styles.metaText}>
              Recibido el: {submittedAt}
            </Text>
            {ip && (
              <Text style={styles.metaText}>IP: {ip}</Text>
            )}
          </Section>

          {/* Footer */}
          <Section style={styles.footer}>
            <Text style={styles.footerText}>
              Ana López Fotografía · Barcelona
            </Text>
            <Text style={styles.footerText}>
              Este email se generó automáticamente. No respondas a este mensaje;
              responde directamente al cliente haciendo clic en su email.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  );
}

// ─── Styles ────────────────────────────────────────────────────────────────

const styles = {
  body: {
    backgroundColor: "#f5f5f4",
    fontFamily: "Inter, Arial, sans-serif",
    margin: "0",
    padding: "0",
  },
  container: {
    backgroundColor: "#ffffff",
    margin: "0 auto",
    maxWidth: "600px",
    padding: "0",
    borderRadius: "8px",
    overflow: "hidden" as const,
    boxShadow: "0 1px 3px rgba(0,0,0,0.1)",
  },
  header: {
    backgroundColor: "#1a1a1a",
    padding: "32px 40px 24px",
    textAlign: "center" as const,
  },
  headerTitle: {
    color: "#ffffff",
    fontSize: "24px",
    fontWeight: "700",
    margin: "0 0 4px 0",
    letterSpacing: "-0.5px",
  },
  headerSubtitle: {
    color: "#a3a3a3",
    fontSize: "14px",
    margin: "0",
  },
  alertSection: {
    backgroundColor: "#fafaf9",
    padding: "16px 40px",
    borderBottom: "1px solid #e7e5e4",
  },
  alertBadge: {
    backgroundColor: "#f0fdf4",
    border: "1px solid #bbf7d0",
    borderRadius: "20px",
    color: "#166534",
    display: "inline-block",
    fontSize: "13px",
    fontWeight: "600",
    margin: "0",
    padding: "6px 16px",
    textAlign: "center" as const,
  },
  card: {
    padding: "28px 40px",
    borderBottom: "1px solid #f5f5f4",
  },
  sectionTitle: {
    color: "#1c1917",
    fontSize: "16px",
    fontWeight: "600",
    margin: "0 0 12px 0",
  },
  divider: {
    borderColor: "#e7e5e4",
    borderTop: "1px solid #e7e5e4",
    margin: "0 0 20px 0",
  },
  fieldRow: {
    marginBottom: "16px",
    padding: "0",
  },
  fieldLabel: {
    color: "#78716c",
    fontSize: "12px",
    fontWeight: "600",
    letterSpacing: "0.05em",
    margin: "0 0 2px 0",
    textTransform: "uppercase" as const,
  },
  fieldValue: {
    color: "#1c1917",
    fontSize: "15px",
    margin: "0",
  },
  fieldValueAccent: {
    backgroundColor: "#fef3c7",
    border: "1px solid #fde68a",
    borderRadius: "4px",
    color: "#92400e",
    display: "inline-block",
    fontSize: "14px",
    fontWeight: "600",
    margin: "0",
    padding: "2px 10px",
  },
  link: {
    color: "#0ea5e9",
    textDecoration: "underline",
  },
  messageText: {
    backgroundColor: "#fafaf9",
    border: "1px solid #e7e5e4",
    borderRadius: "6px",
    color: "#292524",
    fontSize: "15px",
    lineHeight: "1.6",
    margin: "0",
    padding: "16px",
    whiteSpace: "pre-wrap" as const,
  },
  metaSection: {
    backgroundColor: "#fafaf9",
    padding: "16px 40px",
    borderBottom: "1px solid #e7e5e4",
  },
  metaText: {
    color: "#a8a29e",
    fontSize: "12px",
    margin: "0 0 4px 0",
  },
  footer: {
    padding: "24px 40px",
  },
  footerText: {
    color: "#a8a29e",
    fontSize: "12px",
    lineHeight: "1.5",
    margin: "0 0 4px 0",
    textAlign: "center" as const,
  },
} as const;

export default ContactNotificationEmail;
```

---

### 3b. Confirmation to User — `src/emails/ContactConfirmationEmail.tsx`

```tsx
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Img,
  Preview,
  Section,
  Text,
  Font,
} from "@react-email/components";
import * as React from "react";
import type { SessionType } from "@/app/contacto/schema";

const SESSION_TYPE_LABELS: Record<SessionType, string> = {
  bodas: "Bodas",
  retratos: "Retratos",
  corporativo: "Corporativo",
};

const SESSION_TYPE_DESCRIPTIONS: Record<SessionType, string> = {
  bodas:
    "Capturo cada momento de tu día especial con una mirada natural y emotiva.",
  retratos:
    "Sesiones personalizadas para mostrar tu esencia auténtica.",
  corporativo:
    "Imágenes profesionales que refuerzan la identidad de tu marca.",
};

interface ContactConfirmationEmailProps {
  nombre: string;
  tipoSesion: SessionType;
  siteUrl?: string;
}

export function ContactConfirmationEmail({
  nombre,
  tipoSesion,
  siteUrl = "https://analopez.com",
}: ContactConfirmationEmailProps) {
  const firstName = nombre.split(" ")[0];
  const previewText = `Hola ${firstName}, he recibido tu mensaje — Ana López Fotografía`;

  return (
    <Html lang="es" dir="ltr">
      <Head>
        <Font
          fontFamily="Inter"
          fallbackFontFamily="Arial"
          webFont={{
            url: "https://fonts.gstatic.com/s/inter/v13/UcCO3FwrK3iLTeHuS_fvQtMwCp50KnMw2boKoduKmMEVuLyfAZ9hiA.woff2",
            format: "woff2",
          }}
          fontWeight={400}
          fontStyle="normal"
        />
      </Head>
      <Preview>{previewText}</Preview>
      <Body style={styles.body}>
        <Container style={styles.container}>
          {/* Header */}
          <Section style={styles.header}>
            <Heading style={styles.headerTitle}>Ana López</Heading>
            <Text style={styles.headerSubtitle}>Fotografía · Barcelona</Text>
          </Section>

          {/* Hero message */}
          <Section style={styles.heroSection}>
            <Heading as="h1" style={styles.heroTitle}>
              ¡Gracias, {firstName}!
            </Heading>
            <Text style={styles.heroText}>
              He recibido tu mensaje y me alegra mucho que hayas contactado
              conmigo. Te responderé en un plazo máximo de{" "}
              <strong>48 horas</strong>.
            </Text>
          </Section>

          {/* What you requested */}
          <Section style={styles.card}>
            <Text style={styles.cardLabel}>Tu solicitud</Text>
            <Heading as="h2" style={styles.sessionTypeTitle}>
              Sesión de {SESSION_TYPE_LABELS[tipoSesion]}
            </Heading>
            <Text style={styles.sessionTypeDescription}>
              {SESSION_TYPE_DESCRIPTIONS[tipoSesion]}
            </Text>
          </Section>

          {/* What happens next */}
          <Section style={styles.stepsSection}>
            <Heading as="h2" style={styles.stepsTitle}>
              ¿Qué ocurre ahora?
            </Heading>
            <Hr style={styles.divider} />

            <Section style={styles.stepRow}>
              <Text style={styles.stepNumber}>01</Text>
              <Text style={styles.stepText}>
                <strong>Revisaré tu mensaje</strong> y estudiaré tu proyecto
                con atención.
              </Text>
            </Section>

            <Section style={styles.stepRow}>
              <Text style={styles.stepNumber}>02</Text>
              <Text style={styles.stepText}>
                <strong>Me pondré en contacto contigo</strong> para hablar de
                los detalles y resolver todas tus dudas.
              </Text>
            </Section>

            <Section style={styles.stepRow}>
              <Text style={styles.stepNumber}>03</Text>
              <Text style={styles.stepText}>
                <strong>Prepararemos juntos</strong> la sesión perfecta para ti.
              </Text>
            </Section>
          </Section>

          {/* CTA */}
          <Section style={styles.ctaSection}>
            <Text style={styles.ctaText}>
              Mientras tanto, puedes explorar mi portfolio:
            </Text>
            <Button href={`${siteUrl}/portfolio`} style={styles.ctaButton}>
              Ver mi trabajo
            </Button>
          </Section>

          <Hr style={styles.footerDivider} />

          {/* Footer */}
          <Section style={styles.footer}>
            <Text style={styles.footerName}>Ana López</Text>
            <Text style={styles.footerTagline}>
              Fotografía de bodas, retratos y corporativo · Barcelona
            </Text>
            <Text style={styles.footerContact}>
              <a href={`mailto:ana@analopez.com`} style={styles.footerLink}>
                ana@analopez.com
              </a>
            </Text>
            <Hr style={styles.footerDivider} />
            <Text style={styles.footerLegal}>
              Has recibido este email porque completaste el formulario de
              contacto en {siteUrl}. Si no fuiste tú, puedes ignorar este
              mensaje.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  );
}

// ─── Styles ────────────────────────────────────────────────────────────────

const styles = {
  body: {
    backgroundColor: "#f5f5f4",
    fontFamily: "Inter, Arial, sans-serif",
    margin: "0",
    padding: "0",
  },
  container: {
    backgroundColor: "#ffffff",
    margin: "0 auto",
    maxWidth: "600px",
    padding: "0",
    borderRadius: "8px",
    overflow: "hidden" as const,
    boxShadow: "0 1px 3px rgba(0,0,0,0.1)",
  },
  header: {
    backgroundColor: "#1a1a1a",
    padding: "28px 40px 20px",
    textAlign: "center" as const,
  },
  headerTitle: {
    color: "#ffffff",
    fontSize: "22px",
    fontWeight: "700",
    letterSpacing: "-0.3px",
    margin: "0 0 2px 0",
  },
  headerSubtitle: {
    color: "#a3a3a3",
    fontSize: "13px",
    margin: "0",
  },
  heroSection: {
    backgroundColor: "#fafaf9",
    padding: "40px 40px 32px",
    textAlign: "center" as const,
    borderBottom: "1px solid #e7e5e4",
  },
  heroTitle: {
    color: "#1c1917",
    fontSize: "28px",
    fontWeight: "700",
    letterSpacing: "-0.5px",
    margin: "0 0 16px 0",
  },
  heroText: {
    color: "#57534e",
    fontSize: "16px",
    lineHeight: "1.6",
    margin: "0",
  },
  card: {
    backgroundColor: "#fffbeb",
    border: "1px solid #fde68a",
    borderRadius: "8px",
    margin: "28px 40px",
    padding: "20px 24px",
  },
  cardLabel: {
    color: "#92400e",
    fontSize: "11px",
    fontWeight: "700",
    letterSpacing: "0.08em",
    margin: "0 0 6px 0",
    textTransform: "uppercase" as const,
  },
  sessionTypeTitle: {
    color: "#1c1917",
    fontSize: "20px",
    fontWeight: "700",
    margin: "0 0 8px 0",
  },
  sessionTypeDescription: {
    color: "#78716c",
    fontSize: "14px",
    lineHeight: "1.5",
    margin: "0",
  },
  stepsSection: {
    padding: "4px 40px 28px",
  },
  stepsTitle: {
    color: "#1c1917",
    fontSize: "16px",
    fontWeight: "600",
    margin: "0 0 12px 0",
  },
  divider: {
    borderColor: "#e7e5e4",
    borderTop: "1px solid #e7e5e4",
    margin: "0 0 20px 0",
  },
  stepRow: {
    display: "flex",
    marginBottom: "16px",
    padding: "0",
  },
  stepNumber: {
    color: "#d6d3d1",
    fontSize: "24px",
    fontWeight: "700",
    margin: "0 16px 0 0",
    minWidth: "36px",
  },
  stepText: {
    color: "#44403c",
    fontSize: "15px",
    lineHeight: "1.5",
    margin: "4px 0 0 0",
  },
  ctaSection: {
    backgroundColor: "#fafaf9",
    borderTop: "1px solid #e7e5e4",
    borderBottom: "1px solid #e7e5e4",
    padding: "28px 40px",
    textAlign: "center" as const,
  },
  ctaText: {
    color: "#57534e",
    fontSize: "15px",
    margin: "0 0 16px 0",
  },
  ctaButton: {
    backgroundColor: "#1a1a1a",
    borderRadius: "6px",
    color: "#ffffff",
    display: "inline-block",
    fontSize: "14px",
    fontWeight: "600",
    padding: "12px 28px",
    textDecoration: "none",
  },
  footer: {
    padding: "24px 40px",
    textAlign: "center" as const,
  },
  footerDivider: {
    borderColor: "#e7e5e4",
    borderTop: "1px solid #e7e5e4",
    margin: "0",
  },
  footerName: {
    color: "#1c1917",
    fontSize: "15px",
    fontWeight: "600",
    margin: "0 0 2px 0",
  },
  footerTagline: {
    color: "#78716c",
    fontSize: "13px",
    margin: "0 0 8px 0",
  },
  footerContact: {
    margin: "0 0 16px 0",
  },
  footerLink: {
    color: "#0ea5e9",
    fontSize: "13px",
    textDecoration: "underline",
  },
  footerLegal: {
    color: "#a8a29e",
    fontSize: "11px",
    lineHeight: "1.5",
    margin: "12px 0 0 0",
  },
} as const;

export default ContactConfirmationEmail;
```

---

## 4. Server Action — `src/app/contacto/actions.ts`

```typescript
"use server";

import { headers } from "next/headers";
import { render } from "@react-email/render";
import { Resend } from "resend";

import {
  contactFormSchema,
  type ContactFormState,
  type ContactFormValues,
} from "./schema";
import { checkRateLimit } from "@/lib/rate-limit";
import { ContactNotificationEmail } from "@/emails/ContactNotificationEmail";
import { ContactConfirmationEmail } from "@/emails/ContactConfirmationEmail";

// ─── Constants ─────────────────────────────────────────────────────────────

const ANA_EMAIL = process.env.ANA_EMAIL ?? "ana@analopez.com";
const FROM_NOTIFICATION = "Ana López Fotografía <noreply@analopez.com>";
const FROM_CONFIRMATION = "Ana López <ana@analopez.com>";
const SITE_URL = process.env.NEXT_PUBLIC_SITE_URL ?? "https://analopez.com";

// ─── Internal logger ───────────────────────────────────────────────────────

type LogLevel = "info" | "warn" | "error";

function log(level: LogLevel, event: string, data?: Record<string, unknown>) {
  const entry = {
    timestamp: new Date().toISOString(),
    level,
    event,
    ...data,
  };
  // In production replace console with your observability platform
  // (e.g. Axiom, Datadog, Pino transport, etc.)
  console[level === "info" ? "log" : level](JSON.stringify(entry));
}

// ─── Helpers ───────────────────────────────────────────────────────────────

/** Extract IP from Next.js headers, supporting common proxy headers. */
function getClientIp(headersList: Headers): string {
  return (
    headersList.get("x-real-ip") ??
    headersList.get("x-forwarded-for")?.split(",")[0]?.trim() ??
    "127.0.0.1"
  );
}

/** Format a Date to Spanish locale datetime string. */
function formatDateEs(date: Date): string {
  return new Intl.DateTimeFormat("es-ES", {
    day: "2-digit",
    month: "long",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    timeZone: "Europe/Madrid",
  }).format(date);
}

// ─── Email sending ─────────────────────────────────────────────────────────

/**
 * Send both transactional emails via Resend.
 * Errors here are caught in the parent action and logged internally.
 */
async function sendEmails(
  data: ContactFormValues,
  submittedAt: Date,
  ip: string
): Promise<void> {
  if (!process.env.RESEND_API_KEY) {
    throw new Error("RESEND_API_KEY is not configured.");
  }

  const resend = new Resend(process.env.RESEND_API_KEY);
  const submittedAtFormatted = formatDateEs(submittedAt);

  // Render email templates to HTML strings
  const notificationHtml = await render(
    ContactNotificationEmail({
      nombre: data.nombre,
      email: data.email,
      tipoSesion: data.tipoSesion,
      mensaje: data.mensaje,
      submittedAt: submittedAtFormatted,
      ip,
    })
  );

  const confirmationHtml = await render(
    ContactConfirmationEmail({
      nombre: data.nombre,
      tipoSesion: data.tipoSesion,
      siteUrl: SITE_URL,
    })
  );

  // Send both emails concurrently — fail together if either throws
  const [notificationResult, confirmationResult] = await Promise.allSettled([
    resend.emails.send({
      from: FROM_NOTIFICATION,
      to: ANA_EMAIL,
      replyTo: data.email,
      subject: `Nueva consulta de ${data.nombre} — Sesión de ${data.tipoSesion}`,
      html: notificationHtml,
    }),
    resend.emails.send({
      from: FROM_CONFIRMATION,
      to: data.email,
      subject: "He recibido tu mensaje — Ana López Fotografía",
      html: confirmationHtml,
    }),
  ]);

  // Log individual email results
  if (notificationResult.status === "rejected") {
    log("error", "email:notification:failed", {
      reason: String(notificationResult.reason),
    });
  } else if (notificationResult.value.error) {
    log("error", "email:notification:api_error", {
      error: notificationResult.value.error,
    });
  } else {
    log("info", "email:notification:sent", {
      id: notificationResult.value.data?.id,
    });
  }

  if (confirmationResult.status === "rejected") {
    log("error", "email:confirmation:failed", {
      reason: String(confirmationResult.reason),
    });
  } else if (confirmationResult.value.error) {
    log("error", "email:confirmation:api_error", {
      error: confirmationResult.value.error,
    });
  } else {
    log("info", "email:confirmation:sent", {
      id: confirmationResult.value.data?.id,
      recipient: data.email,
    });
  }

  // Surface error only if the critical notification to Ana failed
  const notifFailed =
    notificationResult.status === "rejected" ||
    !!notificationResult.value?.error;

  if (notifFailed) {
    throw new Error("Failed to deliver notification email.");
  }
}

// ─── Server Action ─────────────────────────────────────────────────────────

/**
 * submitContactForm — Next.js Server Action
 *
 * Security surface:
 * - Validates and sanitises all input with Zod (trim, lowercase email)
 * - Rate-limits by IP (3 req / 10 min, sliding window, Upstash Redis)
 * - Never surfaces internal error details to the client
 * - Logs structured internal errors for observability
 */
export async function submitContactForm(
  _prevState: ContactFormState,
  formData: FormData
): Promise<ContactFormState> {
  const submittedAt = new Date();
  const headersList = headers();
  const ip = getClientIp(headersList);

  log("info", "contact:form:received", { ip });

  // ── 1. Rate limiting ──────────────────────────────────────────────────

  let rateLimitResult;
  try {
    rateLimitResult = await checkRateLimit(ip);
  } catch (error) {
    log("error", "contact:ratelimit:error", {
      ip,
      error: error instanceof Error ? error.message : String(error),
      stack: error instanceof Error ? error.stack : undefined,
    });
    // Fail closed in production
    return {
      success: false,
      message:
        "Ha ocurrido un error al procesar tu solicitud. Por favor, inténtalo de nuevo más tarde.",
    };
  }

  if (!rateLimitResult.success) {
    log("warn", "contact:ratelimit:exceeded", {
      ip,
      retryAfter: rateLimitResult.retryAfter,
    });
    return {
      success: false,
      message: `Has enviado demasiados mensajes. Por favor, espera ${rateLimitResult.retryAfter} segundos antes de intentarlo de nuevo.`,
    };
  }

  // ── 2. Validate form data with Zod ────────────────────────────────────

  const rawData = {
    nombre: formData.get("nombre"),
    email: formData.get("email"),
    tipoSesion: formData.get("tipoSesion"),
    mensaje: formData.get("mensaje"),
  };

  const parseResult = contactFormSchema.safeParse(rawData);

  if (!parseResult.success) {
    const fieldErrors = parseResult.error.flatten().fieldErrors;
    log("warn", "contact:validation:failed", { ip, fieldErrors });
    return {
      success: false,
      message: "Por favor, corrige los errores del formulario.",
      fieldErrors: fieldErrors as ContactFormState["fieldErrors"],
    };
  }

  const validatedData = parseResult.data;

  // ── 3. Send emails ────────────────────────────────────────────────────

  try {
    await sendEmails(validatedData, submittedAt, ip);
  } catch (error) {
    log("error", "contact:email:unhandled", {
      ip,
      nombre: validatedData.nombre,
      email: validatedData.email,
      tipoSesion: validatedData.tipoSesion,
      error: error instanceof Error ? error.message : String(error),
      stack: error instanceof Error ? error.stack : undefined,
      submittedAt: submittedAt.toISOString(),
    });

    return {
      success: false,
      message:
        "Tu mensaje no ha podido enviarse por un error interno. Por favor, inténtalo de nuevo o contáctame directamente en ana@analopez.com.",
    };
  }

  // ── 4. Success ────────────────────────────────────────────────────────

  log("info", "contact:form:success", {
    ip,
    tipoSesion: validatedData.tipoSesion,
    // never log PII (nombre/email) in success events — use the email logs
  });

  return {
    success: true,
    message: `¡Gracias, ${validatedData.nombre.split(" ")[0]}! He recibido tu mensaje y te responderé en las próximas 48 horas.`,
  };
}
```

---

## 5. Client Form Component — `src/components/contact/ContactForm.tsx`

```tsx
"use client";

import { useActionState, useEffect, useRef } from "react";
import { submitContactForm } from "@/app/contacto/actions";
import type { ContactFormState } from "@/app/contacto/schema";
import { sessionTypes } from "@/app/contacto/schema";

const SESSION_LABELS: Record<(typeof sessionTypes)[number], string> = {
  bodas: "Bodas",
  retratos: "Retratos",
  corporativo: "Corporativo",
};

const initialState: ContactFormState = {
  success: false,
  message: "",
};

export function ContactForm() {
  const [state, formAction, isPending] = useActionState(
    submitContactForm,
    initialState
  );

  const formRef = useRef<HTMLFormElement>(null);

  // Reset form fields on successful submission
  useEffect(() => {
    if (state.success) {
      formRef.current?.reset();
    }
  }, [state.success]);

  return (
    <form ref={formRef} action={formAction} noValidate className="space-y-6">
      {/* Global feedback */}
      {state.message && (
        <div
          role="alert"
          aria-live="polite"
          className={`rounded-lg px-4 py-3 text-sm font-medium ${
            state.success
              ? "bg-green-50 text-green-800 ring-1 ring-green-200"
              : "bg-red-50 text-red-800 ring-1 ring-red-200"
          }`}
        >
          {state.message}
        </div>
      )}

      {/* Nombre */}
      <div>
        <label
          htmlFor="nombre"
          className="block text-sm font-medium text-stone-700"
        >
          Nombre completo <span aria-hidden="true">*</span>
        </label>
        <input
          id="nombre"
          name="nombre"
          type="text"
          autoComplete="name"
          required
          aria-required="true"
          aria-describedby={
            state.fieldErrors?.nombre ? "nombre-error" : undefined
          }
          aria-invalid={!!state.fieldErrors?.nombre}
          disabled={isPending}
          className="mt-1 block w-full rounded-md border border-stone-300 px-3 py-2 text-stone-900 placeholder-stone-400 shadow-sm focus:border-stone-500 focus:outline-none focus:ring-1 focus:ring-stone-500 disabled:opacity-60"
          placeholder="María García"
        />
        {state.fieldErrors?.nombre && (
          <p id="nombre-error" role="alert" className="mt-1 text-sm text-red-600">
            {state.fieldErrors.nombre[0]}
          </p>
        )}
      </div>

      {/* Email */}
      <div>
        <label
          htmlFor="email"
          className="block text-sm font-medium text-stone-700"
        >
          Email <span aria-hidden="true">*</span>
        </label>
        <input
          id="email"
          name="email"
          type="email"
          autoComplete="email"
          required
          aria-required="true"
          aria-describedby={
            state.fieldErrors?.email ? "email-error" : undefined
          }
          aria-invalid={!!state.fieldErrors?.email}
          disabled={isPending}
          className="mt-1 block w-full rounded-md border border-stone-300 px-3 py-2 text-stone-900 placeholder-stone-400 shadow-sm focus:border-stone-500 focus:outline-none focus:ring-1 focus:ring-stone-500 disabled:opacity-60"
          placeholder="maria@ejemplo.com"
        />
        {state.fieldErrors?.email && (
          <p id="email-error" role="alert" className="mt-1 text-sm text-red-600">
            {state.fieldErrors.email[0]}
          </p>
        )}
      </div>

      {/* Tipo de sesión */}
      <div>
        <fieldset>
          <legend className="block text-sm font-medium text-stone-700">
            Tipo de sesión <span aria-hidden="true">*</span>
          </legend>
          <div className="mt-2 flex flex-wrap gap-3">
            {sessionTypes.map((type) => (
              <label
                key={type}
                className="relative flex cursor-pointer items-center gap-2 rounded-lg border border-stone-300 px-4 py-2.5 text-sm font-medium text-stone-700 transition-colors hover:border-stone-500 has-[:checked]:border-stone-800 has-[:checked]:bg-stone-800 has-[:checked]:text-white"
              >
                <input
                  type="radio"
                  name="tipoSesion"
                  value={type}
                  required
                  aria-required="true"
                  disabled={isPending}
                  className="sr-only"
                />
                {SESSION_LABELS[type]}
              </label>
            ))}
          </div>
          {state.fieldErrors?.tipoSesion && (
            <p role="alert" className="mt-1 text-sm text-red-600">
              {state.fieldErrors.tipoSesion[0]}
            </p>
          )}
        </fieldset>
      </div>

      {/* Mensaje */}
      <div>
        <label
          htmlFor="mensaje"
          className="block text-sm font-medium text-stone-700"
        >
          Mensaje <span aria-hidden="true">*</span>
        </label>
        <textarea
          id="mensaje"
          name="mensaje"
          rows={5}
          required
          aria-required="true"
          aria-describedby={
            state.fieldErrors?.mensaje ? "mensaje-error" : undefined
          }
          aria-invalid={!!state.fieldErrors?.mensaje}
          disabled={isPending}
          className="mt-1 block w-full rounded-md border border-stone-300 px-3 py-2 text-stone-900 placeholder-stone-400 shadow-sm focus:border-stone-500 focus:outline-none focus:ring-1 focus:ring-stone-500 disabled:opacity-60"
          placeholder="Cuéntame sobre tu sesión, fechas, ubicación..."
        />
        {state.fieldErrors?.mensaje && (
          <p id="mensaje-error" role="alert" className="mt-1 text-sm text-red-600">
            {state.fieldErrors.mensaje[0]}
          </p>
        )}
      </div>

      {/* Submit */}
      <button
        type="submit"
        disabled={isPending}
        aria-disabled={isPending}
        className="w-full rounded-md bg-stone-900 px-6 py-3 text-sm font-semibold text-white shadow-sm transition-colors hover:bg-stone-700 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-stone-900 disabled:cursor-not-allowed disabled:opacity-60"
      >
        {isPending ? "Enviando..." : "Enviar mensaje"}
      </button>
    </form>
  );
}
```

---

## 6. Page — `src/app/contacto/page.tsx`

```tsx
import { ContactForm } from "@/components/contact/ContactForm";
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Contacto — Ana López Fotografía",
  description:
    "¿Tienes una sesión en mente? Escríbeme y hablamos de tu proyecto fotográfico.",
};

export default function ContactoPage() {
  return (
    <main className="mx-auto max-w-2xl px-4 py-16 sm:px-6 lg:px-8">
      <div className="mb-10 text-center">
        <h1 className="text-4xl font-bold tracking-tight text-stone-900 sm:text-5xl">
          Hablemos
        </h1>
        <p className="mt-4 text-lg text-stone-600">
          Cuéntame sobre tu sesión. Te responderé en menos de 48 horas.
        </p>
      </div>

      <div className="rounded-2xl border border-stone-200 bg-white p-8 shadow-sm">
        <ContactForm />
      </div>
    </main>
  );
}
```

---

## 7. Design Decisions & Notes

### Rate Limiting
- Uses `Ratelimit.slidingWindow(3, "10 m")` — more accurate than a fixed window; prevents bursts at window edges.
- The identifier is the real client IP (extracted from `x-real-ip` or `x-forwarded-for`), so limits are per-user and not per-server.
- Dev-mode fallback: when Upstash env vars are absent, the limiter is bypassed with a console warning instead of crashing the dev server.
- Production fail-closed: if Redis throws unexpectedly in production, the action returns a generic error rather than allowing unlimited submissions.

### Error Handling
- Three distinct error layers: rate limit failure, Zod validation failure, email send failure.
- Internal logs are structured JSON with `timestamp`, `level`, `event`, and contextual fields — drop-in compatible with Axiom/Datadog/Pino.
- PII (nombre, email) is **never** logged in success events to avoid sensitive data in log storage; only logged on failure events where it is operationally necessary for debugging.
- The client always receives a generic, user-friendly message — internal error details never leak.
- The notification email to Ana is treated as the critical path: if it fails, the action returns an error. The confirmation to the user is best-effort (failure is logged but does not block success).

### Email Templates
- Built with `@react-email/components` for cross-client compatibility (`<Body>`, `<Container>`, `<Section>`, `<Button>` etc. render to table-based HTML).
- All styles are inline objects (required for email clients); no Tailwind inside templates.
- `render()` from `@react-email/render` produces the final HTML string on the server before passing to Resend.
- The `FROM_CONFIRMATION` sender (`ana@analopez.com`) is used for the user-facing email so replies from users land in Ana's inbox naturally.

### Zod Schema
- `.trim()` on string fields prevents whitespace-only submissions and normalises input before storage/emails.
- `.toLowerCase()` on email ensures case-insensitive deduplication if needed later.
- `z.enum(sessionTypes)` with a `const` tuple guarantees TypeScript and runtime agree on allowed values.
- `fieldErrors` in the return type is `Partial<Record<keyof ContactFormValues, string[]>>` — Zod's `flatten()` shape — so the client component can map errors directly to form fields.

### `useActionState`
- Uses the React 19 / Next.js 14 `useActionState` hook (previously `useFormState`).
- `isPending` disables all inputs and the submit button during the async round-trip, preventing double submissions.
- `aria-*` attributes (`aria-invalid`, `aria-describedby`, `aria-live`) ensure screen-reader accessibility for both inline field errors and the global status message.
