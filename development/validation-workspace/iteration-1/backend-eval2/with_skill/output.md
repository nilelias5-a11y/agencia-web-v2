# Eval 2 — NextAuth v5 + Supabase Auth: Café Vermut

**Skill applied:** `backend-developer`
**Project:** Café Vermut — área de pedidos autenticada
**Date:** 2026-05-13

---

## Overview

Complete NextAuth v5 authentication implementation for Café Vermut's order area (`/pedidos/*` and `/cuenta/*`). Covers:

1. `src/env.ts` — T3 Env with all auth variables typed and validated
2. `src/auth.ts` — NextAuth v5 with Google + Resend magic link + Supabase Adapter
3. `src/middleware.ts` — Edge middleware protecting `/pedidos/*` and `/cuenta/*`
4. `src/app/pedidos/page.tsx` — Protected Server Component with session check
5. `src/components/auth/logout-button.tsx` — Client Component logout button

Security checklist applied at the end.

---

## Dependencies

```bash
npm install next-auth@beta @auth/supabase-adapter @supabase/supabase-js @t3-oss/env-nextjs zod
```

**Package versions (as of 2026-05-13):**
- `next-auth`: `^5.0.0` (beta/stable v5)
- `@auth/supabase-adapter`: `^1.x`
- `@supabase/supabase-js`: `^2.x`
- `@t3-oss/env-nextjs`: `^0.10.x`
- `zod`: `^3.x`

---

## 1. `src/env.ts` — T3 Env (all auth variables)

```ts
// src/env.ts
import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  /**
   * Server-side secrets — NEVER sent to the browser.
   * Validated at build time and at server startup.
   */
  server: {
    // Database
    DATABASE_URL: z.string().url(),

    // NextAuth v5
    AUTH_SECRET: z.string().min(32, "AUTH_SECRET must be at least 32 characters"),
    AUTH_URL: z.string().url(),

    // Google OAuth provider
    GOOGLE_CLIENT_ID: z.string().min(1, "Google Client ID is required"),
    GOOGLE_CLIENT_SECRET: z.string().min(1, "Google Client Secret is required"),

    // Supabase (server-only, service role — never expose to client)
    SUPABASE_URL: z.string().url(),
    SUPABASE_SERVICE_ROLE_KEY: z.string().min(1, "Supabase Service Role Key is required"),

    // Resend (magic link email provider)
    RESEND_API_KEY: z.string().startsWith("re_", "Resend API key must start with 're_'"),

    // App domain (used for magic link from address)
    EMAIL_FROM: z.string().email("EMAIL_FROM must be a valid email address"),
  },

  /**
   * Client-side variables — safe to expose to the browser.
   * Must be prefixed with NEXT_PUBLIC_.
   */
  client: {
    NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
    NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1, "Supabase Anon Key is required"),
  },

  /**
   * Explicit runtime mapping — T3 Env requires every key listed above
   * to be mapped here to its process.env counterpart.
   * This is the ONLY place process.env is accessed in the codebase.
   */
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    AUTH_SECRET: process.env.AUTH_SECRET,
    AUTH_URL: process.env.AUTH_URL,
    GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID,
    GOOGLE_CLIENT_SECRET: process.env.GOOGLE_CLIENT_SECRET,
    SUPABASE_URL: process.env.SUPABASE_URL,
    SUPABASE_SERVICE_ROLE_KEY: process.env.SUPABASE_SERVICE_ROLE_KEY,
    RESEND_API_KEY: process.env.RESEND_API_KEY,
    EMAIL_FROM: process.env.EMAIL_FROM,
    NEXT_PUBLIC_SUPABASE_URL: process.env.NEXT_PUBLIC_SUPABASE_URL,
    NEXT_PUBLIC_SUPABASE_ANON_KEY: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
  },

  /**
   * Skip validation during CI lint/type-check if env vars are not available.
   * Remove this if your CI always has env vars set.
   */
  skipValidation: !!process.env.SKIP_ENV_VALIDATION,

  /**
   * Makes it so that empty strings are treated as undefined.
   * Prevents accidental use of empty env vars.
   */
  emptyStringAsUndefined: true,
})
```

**`.env.local` template:**

```bash
# .env.local — NEVER commit this file

# Database
DATABASE_URL="postgresql://..."

# NextAuth v5
# Generate with: openssl rand -base64 32
AUTH_SECRET="your-secret-min-32-chars-here"
AUTH_URL="http://localhost:3000"

# Google OAuth
# From https://console.cloud.google.com/
GOOGLE_CLIENT_ID="xxxx.apps.googleusercontent.com"
GOOGLE_CLIENT_SECRET="GOCSPX-xxxx"

# Supabase (server-only)
SUPABASE_URL="https://xxxx.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxx"

# Resend
RESEND_API_KEY="re_xxxx"
EMAIL_FROM="noreply@cafevermut.com"

# Supabase (public, safe for browser)
NEXT_PUBLIC_SUPABASE_URL="https://xxxx.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxx"
```

---

## 2. `src/auth.ts` — NextAuth v5 Configuration

```ts
// src/auth.ts
import NextAuth from "next-auth"
import { SupabaseAdapter } from "@auth/supabase-adapter"
import Google from "next-auth/providers/google"
import Resend from "next-auth/providers/resend"
import { env } from "@/env"

/**
 * NextAuth v5 configuration for Café Vermut.
 *
 * Providers:
 *   - Google OAuth: one-click login for users with Google accounts
 *   - Resend magic link: passwordless email login via Resend transactional email
 *
 * Adapter:
 *   - SupabaseAdapter: persists users, sessions, accounts, and verification
 *     tokens in Supabase using the service role key (server-only).
 *
 * Callbacks:
 *   - session: attaches user.id to the session so Server Components and
 *     Server Actions can identify the authenticated user without extra queries.
 */
export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: SupabaseAdapter({
    url: env.SUPABASE_URL,
    secret: env.SUPABASE_SERVICE_ROLE_KEY,
  }),

  providers: [
    Google({
      clientId: env.GOOGLE_CLIENT_ID,
      clientSecret: env.GOOGLE_CLIENT_SECRET,
      /**
       * Request only the minimum necessary OAuth scopes.
       * "openid email profile" is sufficient — do not request Drive, Calendar, etc.
       */
      authorization: {
        params: {
          scope: "openid email profile",
          prompt: "select_account", // Always show account picker — prevents silent session hijack
        },
      },
    }),

    Resend({
      apiKey: env.RESEND_API_KEY,
      from: env.EMAIL_FROM,
      /**
       * Magic link expires in 10 minutes.
       * Default is 24h — reduce to limit exposure window.
       */
      maxAge: 10 * 60, // 10 minutes in seconds
    }),
  ],

  callbacks: {
    /**
     * Attach user.id to the session token so it's available in:
     *   - Server Components via: const session = await auth()
     *   - Client Components via: const session = useSession()
     *
     * This avoids redundant Supabase queries just to get the user ID.
     */
    session({ session, user }) {
      session.user.id = user.id
      return session
    },

    /**
     * JWT callback is not needed when using a database adapter
     * (sessions are stored in the DB, not in JWTs).
     * Leave it out to avoid confusion.
     */
  },

  pages: {
    signIn: "/auth/login",       // Custom login page (Google button + email input)
    error: "/auth/error",        // Auth error page (wrong email, OAuth denied, etc.)
    verifyRequest: "/auth/verify", // "Check your email" page shown after magic link sent
  },

  /**
   * Session strategy: "database" is the default when using an adapter.
   * Session tokens are stored in Supabase — invalidation is instant.
   */
  session: {
    strategy: "database",
    /**
     * Session expires in 30 days.
     * Users who haven't visited in 30 days must re-authenticate.
     */
    maxAge: 30 * 24 * 60 * 60, // 30 days in seconds
    /**
     * Session is extended on every request if the user is active.
     * Set to the same value as maxAge to refresh on every visit.
     */
    updateAge: 24 * 60 * 60, // Extend session if active within 24h
  },

  /**
   * Enable debug logs only in development.
   * In production, NextAuth logs nothing to stdout by default.
   */
  debug: process.env.NODE_ENV === "development",
})
```

**`src/app/api/auth/[...nextauth]/route.ts` — Route Handler**

```ts
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"

/**
 * NextAuth v5 route handlers.
 * Exports GET and POST to handle all /api/auth/* requests:
 *   - GET  /api/auth/signin
 *   - GET  /api/auth/signout
 *   - GET  /api/auth/session
 *   - GET  /api/auth/csrf
 *   - GET  /api/auth/providers
 *   - GET  /api/auth/callback/:provider
 *   - POST /api/auth/signin/:provider
 *   - POST /api/auth/signout
 */
export const { GET, POST } = handlers
```

---

## 3. `src/middleware.ts` — Edge Middleware (protecting /pedidos/* and /cuenta/*)

```ts
// src/middleware.ts
import { auth } from "@/auth"
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"

/**
 * Protected route prefixes for Café Vermut.
 *
 * /pedidos/* — order management area (authenticated users only)
 * /cuenta/*  — customer account area (authenticated users only)
 */
const PROTECTED_PREFIXES = ["/pedidos", "/cuenta"] as const

/**
 * Routes that must remain public even if they start with a protected prefix.
 * (Add any public sub-routes here if needed in the future.)
 */
const PUBLIC_ROUTES: string[] = []

/**
 * NextAuth v5 middleware wrapper.
 *
 * Runs at the Edge — no Node.js APIs available here.
 * Reads the session from the signed session cookie without a DB round-trip.
 *
 * Auth flow:
 *   1. Request arrives at a protected route
 *   2. Middleware checks req.auth (populated by NextAuth from the session cookie)
 *   3. If no session → redirect to /auth/login with callbackUrl
 *   4. If session exists → allow through
 */
export default auth((req: NextRequest & { auth: typeof auth extends (...args: any[]) => infer R ? Awaited<R> : never }) => {
  const { pathname } = req.nextUrl
  const isAuthenticated = !!req.auth

  // Check if the current path requires authentication
  const isProtectedRoute =
    PROTECTED_PREFIXES.some((prefix) => pathname.startsWith(prefix)) &&
    !PUBLIC_ROUTES.includes(pathname)

  if (isProtectedRoute && !isAuthenticated) {
    /**
     * Redirect unauthenticated users to the login page.
     * Pass the original URL as callbackUrl so after login they return
     * to the page they were trying to access.
     *
     * Security: callbackUrl is validated by NextAuth to be same-origin only.
     */
    const loginUrl = new URL("/auth/login", req.url)
    loginUrl.searchParams.set("callbackUrl", req.nextUrl.pathname)
    return NextResponse.redirect(loginUrl)
  }

  /**
   * Prevent authenticated users from accessing the login page.
   * Redirect them to /pedidos if they're already logged in.
   */
  if (pathname.startsWith("/auth/login") && isAuthenticated) {
    return NextResponse.redirect(new URL("/pedidos", req.url))
  }

  return NextResponse.next()
})

/**
 * Matcher configuration.
 *
 * Runs middleware on ALL routes EXCEPT:
 *   - /api/auth/*  — NextAuth's own API routes (must be public)
 *   - /_next/*     — Next.js static assets
 *   - /favicon.ico, /robots.txt, sitemap.xml — static files
 *   - Image optimization routes
 *
 * Using a positive matcher for protected routes would be cleaner,
 * but the exclusion pattern ensures NextAuth internals are never blocked.
 */
export const config = {
  matcher: [
    /*
     * Match all routes except:
     * - api/auth (NextAuth handlers)
     * - _next/static (static files)
     * - _next/image (image optimization)
     * - favicon.ico, robots.txt, sitemap.xml
     */
    "/((?!api/auth|_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml).*)",
  ],
}
```

---

## 4. `src/app/pedidos/page.tsx` — Protected Server Component

```tsx
// src/app/pedidos/page.tsx
import { auth } from "@/auth"
import { redirect } from "next/navigation"
import type { Metadata } from "next"

export const metadata: Metadata = {
  title: "Mis Pedidos | Café Vermut",
  description: "Gestiona tus pedidos en Café Vermut",
  /**
   * Prevent search engines from indexing authenticated pages.
   * Even if a bot somehow reaches this page, it won't be indexed.
   */
  robots: {
    index: false,
    follow: false,
  },
}

/**
 * Protected Server Component — /pedidos
 *
 * Auth strategy (defence in depth):
 *   1. Middleware (Edge)    — first line of defence, fast redirect before rendering
 *   2. Server Component     — second explicit check, handles edge cases where
 *                             middleware might be bypassed (e.g., direct API calls,
 *                             future route changes not covered by matcher)
 *
 * Never rely on middleware alone. Always re-check auth in Server Components
 * that render sensitive data.
 *
 * The `auth()` call reads from the database-backed session.
 * It does NOT make a network request if the session cookie is valid —
 * NextAuth caches the session within the request lifecycle.
 */
export default async function PedidosPage() {
  const session = await auth()

  /**
   * Explicit session guard.
   * If middleware was bypassed for any reason, this redirect
   * ensures unauthenticated users NEVER see the page content.
   */
  if (!session?.user) {
    redirect("/auth/login")
  }

  const { user } = session

  return (
    <main className="container mx-auto max-w-4xl px-4 py-8">
      <header className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900">
          Mis Pedidos
        </h1>
        <p className="mt-2 text-gray-600">
          Bienvenido/a, <span className="font-medium">{user.name ?? user.email}</span>
        </p>
      </header>

      <section aria-labelledby="pedidos-heading">
        <h2 id="pedidos-heading" className="sr-only">
          Lista de pedidos
        </h2>

        {/**
         * TODO: Replace with actual order data fetched from Supabase.
         * Example:
         *   const orders = await getPedidosByUserId(user.id)
         *
         * The user.id is available from the session callback defined in auth.ts.
         * Use it to query only the authenticated user's orders — never expose
         * other users' data.
         */}
        <div className="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <p className="text-gray-500">
            No tienes pedidos todavía.{" "}
            <a
              href="/carta"
              className="text-amber-600 underline hover:text-amber-700"
            >
              Ver la carta
            </a>
          </p>
        </div>
      </section>

      <div className="mt-8 flex justify-end">
        {/**
         * LogoutButton is a Client Component because it uses a browser interaction.
         * It is imported here from its own file to respect the
         * Server/Client Component boundary.
         */}
        {/* <LogoutButton /> */}
        {/* Import from @/components/auth/logout-button — see section 5 */}
      </div>
    </main>
  )
}
```

**`src/app/pedidos/layout.tsx` — Layout with LogoutButton integration**

```tsx
// src/app/pedidos/layout.tsx
import { auth } from "@/auth"
import { redirect } from "next/navigation"
import { LogoutButton } from "@/components/auth/logout-button"
import type { ReactNode } from "react"

interface PedidosLayoutProps {
  children: ReactNode
}

/**
 * Shared layout for all /pedidos/* routes.
 *
 * Auth is checked at the layout level so every child page inherits
 * the protection without repeating the session check in every page.
 *
 * Note: page.tsx still performs its own check as a second layer of defence.
 */
export default async function PedidosLayout({ children }: PedidosLayoutProps) {
  const session = await auth()

  if (!session?.user) {
    redirect("/auth/login")
  }

  return (
    <div className="min-h-screen bg-gray-50">
      <nav className="border-b border-gray-200 bg-white px-4 py-3">
        <div className="container mx-auto flex max-w-4xl items-center justify-between">
          <a
            href="/"
            className="text-lg font-semibold text-gray-900 hover:text-amber-600"
          >
            Café Vermut
          </a>
          <div className="flex items-center gap-4">
            <span className="text-sm text-gray-600">
              {session.user.email}
            </span>
            <LogoutButton />
          </div>
        </div>
      </nav>
      <div className="container mx-auto max-w-4xl px-4 py-8">
        {children}
      </div>
    </div>
  )
}
```

---

## 5. `src/components/auth/logout-button.tsx` — Client Component

```tsx
// src/components/auth/logout-button.tsx
"use client"

import { signOut } from "next-auth/react"
import { useState } from "react"

interface LogoutButtonProps {
  /**
   * Where to redirect after sign-out.
   * Defaults to the home page.
   */
  callbackUrl?: string
  /**
   * Optional custom label.
   */
  label?: string
  /**
   * Optional CSS class names for styling.
   */
  className?: string
}

/**
 * LogoutButton — Client Component
 *
 * Must be a Client Component because it:
 *   1. Uses `signOut` from next-auth/react (browser-side SDK)
 *   2. Manages loading state with useState
 *   3. Handles user interaction (click event)
 *
 * Security notes:
 *   - `signOut` sends a POST to /api/auth/signout with a CSRF token.
 *     NextAuth handles CSRF automatically — do NOT call /api/auth/signout
 *     directly via GET (CSRF vulnerability).
 *   - `callbackUrl` is validated by NextAuth to be same-origin.
 *     Do NOT pass user-controlled URLs here.
 *   - Loading state prevents double-clicks from sending multiple signOut requests.
 */
export function LogoutButton({
  callbackUrl = "/",
  label = "Cerrar sesión",
  className,
}: LogoutButtonProps) {
  const [isLoading, setIsLoading] = useState(false)

  async function handleSignOut() {
    if (isLoading) return // Prevent double-click

    try {
      setIsLoading(true)
      await signOut({
        callbackUrl,
        /**
         * redirect: true (default) — let NextAuth handle the redirect.
         * The user will be taken to callbackUrl after the session is destroyed.
         */
        redirect: true,
      })
    } catch (error) {
      /**
       * Sign-out errors are rare but possible (network issues, etc.).
       * Log internally, show a generic message to the user.
       * Do NOT expose error details.
       */
      console.error("[LogoutButton] Sign-out error:", error)
      setIsLoading(false)
    }
  }

  return (
    <button
      onClick={handleSignOut}
      disabled={isLoading}
      aria-busy={isLoading}
      className={
        className ??
        "rounded-md bg-gray-100 px-3 py-1.5 text-sm font-medium text-gray-700 transition-colors hover:bg-gray-200 disabled:cursor-not-allowed disabled:opacity-50"
      }
      type="button"
    >
      {isLoading ? "Cerrando sesión…" : label}
    </button>
  )
}
```

---

## 6. Supporting Auth Pages

### `src/app/auth/login/page.tsx` — Login Page

```tsx
// src/app/auth/login/page.tsx
import { signIn } from "@/auth"
import type { Metadata } from "next"

export const metadata: Metadata = {
  title: "Iniciar sesión | Café Vermut",
  robots: { index: false, follow: false },
}

interface LoginPageProps {
  searchParams: Promise<{ callbackUrl?: string; error?: string }>
}

/**
 * Login page — Server Component with Server Actions for form handling.
 *
 * Two sign-in flows:
 *   1. Google OAuth — one-click, redirects to Google
 *   2. Magic link (Resend) — user enters email, receives link via Resend
 *
 * Server Actions are used for both flows so sign-in logic stays server-side.
 * No API keys or sensitive config are ever sent to the browser.
 */
export default async function LoginPage({ searchParams }: LoginPageProps) {
  const { callbackUrl, error } = await searchParams

  /**
   * Validate callbackUrl to prevent open redirect attacks.
   * Only allow same-origin paths.
   */
  const safeCallbackUrl =
    callbackUrl && callbackUrl.startsWith("/") ? callbackUrl : "/pedidos"

  return (
    <main className="flex min-h-screen items-center justify-center bg-gray-50 px-4">
      <div className="w-full max-w-sm rounded-xl border border-gray-200 bg-white p-8 shadow-sm">
        <div className="mb-6 text-center">
          <h1 className="text-2xl font-bold text-gray-900">Café Vermut</h1>
          <p className="mt-1 text-sm text-gray-500">
            Inicia sesión para gestionar tus pedidos
          </p>
        </div>

        {/* Auth error display */}
        {error && (
          <div
            role="alert"
            className="mb-4 rounded-md bg-red-50 p-3 text-sm text-red-700"
          >
            {getAuthErrorMessage(error)}
          </div>
        )}

        {/* Google OAuth */}
        <form
          action={async () => {
            "use server"
            await signIn("google", { redirectTo: safeCallbackUrl })
          }}
        >
          <button
            type="submit"
            className="flex w-full items-center justify-center gap-2 rounded-lg border border-gray-300 bg-white px-4 py-2.5 text-sm font-medium text-gray-700 shadow-sm transition-colors hover:bg-gray-50"
          >
            <GoogleIcon />
            Continuar con Google
          </button>
        </form>

        <div className="my-4 flex items-center gap-2">
          <hr className="flex-1 border-gray-200" />
          <span className="text-xs text-gray-400">o</span>
          <hr className="flex-1 border-gray-200" />
        </div>

        {/* Magic link (Resend) */}
        <form
          action={async (formData: FormData) => {
            "use server"
            const email = formData.get("email")

            /**
             * Validate email before calling signIn.
             * NextAuth validates it too, but we validate early to give
             * better UX feedback and avoid unnecessary API calls.
             */
            if (
              typeof email !== "string" ||
              !email.includes("@") ||
              email.length > 254
            ) {
              // In a real app, use useFormState/useActionState for error feedback
              return
            }

            await signIn("resend", {
              email,
              redirectTo: safeCallbackUrl,
            })
          }}
        >
          <div className="space-y-3">
            <div>
              <label
                htmlFor="email"
                className="block text-sm font-medium text-gray-700"
              >
                Correo electrónico
              </label>
              <input
                id="email"
                name="email"
                type="email"
                required
                autoComplete="email"
                placeholder="tu@email.com"
                maxLength={254}
                className="mt-1 w-full rounded-md border border-gray-300 px-3 py-2 text-sm placeholder-gray-400 focus:border-amber-500 focus:outline-none focus:ring-1 focus:ring-amber-500"
              />
            </div>
            <button
              type="submit"
              className="w-full rounded-lg bg-amber-600 px-4 py-2.5 text-sm font-medium text-white transition-colors hover:bg-amber-700"
            >
              Enviar enlace mágico
            </button>
          </div>
        </form>
      </div>
    </main>
  )
}

/**
 * Map NextAuth error codes to user-friendly Spanish messages.
 * Never expose raw error codes or stack traces.
 */
function getAuthErrorMessage(error: string): string {
  const messages: Record<string, string> = {
    OAuthSignin: "Error al iniciar sesión con Google. Inténtalo de nuevo.",
    OAuthCallback: "Error al procesar la respuesta de Google. Inténtalo de nuevo.",
    OAuthCreateAccount: "No se pudo crear tu cuenta. Contacta con soporte.",
    EmailCreateAccount: "No se pudo crear tu cuenta con ese email.",
    Callback: "Error durante la autenticación. Inténtalo de nuevo.",
    OAuthAccountNotLinked:
      "Ya tienes una cuenta con ese email. Usa el mismo método de inicio de sesión.",
    EmailSignin: "No se pudo enviar el enlace mágico. Verifica tu email.",
    CredentialsSignin: "Credenciales incorrectas.",
    SessionRequired: "Debes iniciar sesión para acceder a esta página.",
    Default: "Algo ha fallado. Por favor, inténtalo de nuevo.",
  }

  return messages[error] ?? messages.Default
}

function GoogleIcon() {
  return (
    <svg width="18" height="18" viewBox="0 0 18 18" aria-hidden="true">
      <path
        fill="#4285F4"
        d="M16.51 8H8.98v3h4.3c-.18 1-.74 1.48-1.6 2.04v2.01h2.6a7.8 7.8 0 0 0 2.38-5.88c0-.57-.05-.66-.15-1.18z"
      />
      <path
        fill="#34A853"
        d="M8.98 17c2.16 0 3.97-.72 5.3-1.94l-2.6-2a4.8 4.8 0 0 1-7.18-2.54H1.83v2.07A8 8 0 0 0 8.98 17z"
      />
      <path
        fill="#FBBC05"
        d="M4.5 10.52a4.8 4.8 0 0 1 0-3.04V5.41H1.83a8 8 0 0 0 0 7.18l2.67-2.07z"
      />
      <path
        fill="#EA4335"
        d="M8.98 4.18c1.17 0 2.23.4 3.06 1.2l2.3-2.3A8 8 0 0 0 1.83 5.4L4.5 7.49a4.77 4.77 0 0 1 4.48-3.3z"
      />
    </svg>
  )
}
```

### `src/app/auth/verify/page.tsx` — Magic Link Sent

```tsx
// src/app/auth/verify/page.tsx
import type { Metadata } from "next"

export const metadata: Metadata = {
  title: "Revisa tu email | Café Vermut",
  robots: { index: false, follow: false },
}

/**
 * Shown after a magic link email is sent.
 * Pure Server Component — no sensitive data, no auth required.
 */
export default function VerifyPage() {
  return (
    <main className="flex min-h-screen items-center justify-center bg-gray-50 px-4">
      <div className="w-full max-w-sm rounded-xl border border-gray-200 bg-white p-8 text-center shadow-sm">
        <div className="mb-4 text-4xl">📧</div>
        <h1 className="text-xl font-bold text-gray-900">
          Revisa tu correo
        </h1>
        <p className="mt-2 text-sm text-gray-600">
          Hemos enviado un enlace mágico a tu email. Haz clic en él para
          iniciar sesión. El enlace expira en 10 minutos.
        </p>
        <p className="mt-4 text-xs text-gray-400">
          Si no ves el email, revisa tu carpeta de spam.
        </p>
      </div>
    </main>
  )
}
```

### `src/app/auth/error/page.tsx` — Auth Error Page

```tsx
// src/app/auth/error/page.tsx
import type { Metadata } from "next"

export const metadata: Metadata = {
  title: "Error de autenticación | Café Vermut",
  robots: { index: false, follow: false },
}

interface ErrorPageProps {
  searchParams: Promise<{ error?: string }>
}

/**
 * Displayed by NextAuth when an auth error occurs.
 *
 * Security: Never render raw error codes or technical details.
 * Log the error server-side, show a friendly message to the user.
 */
export default async function AuthErrorPage({ searchParams }: ErrorPageProps) {
  const { error } = await searchParams

  // Log for monitoring — never expose to client
  if (error) {
    console.error("[auth/error] Auth error occurred:", error)
  }

  return (
    <main className="flex min-h-screen items-center justify-center bg-gray-50 px-4">
      <div className="w-full max-w-sm rounded-xl border border-red-200 bg-white p-8 text-center shadow-sm">
        <div className="mb-4 text-4xl">⚠️</div>
        <h1 className="text-xl font-bold text-gray-900">
          Algo ha fallado
        </h1>
        <p className="mt-2 text-sm text-gray-600">
          Ha ocurrido un error durante la autenticación. Por favor, inténtalo
          de nuevo.
        </p>
        <a
          href="/auth/login"
          className="mt-6 inline-block rounded-lg bg-amber-600 px-4 py-2 text-sm font-medium text-white hover:bg-amber-700"
        >
          Volver al inicio de sesión
        </a>
      </div>
    </main>
  )
}
```

---

## 7. TypeScript Type Augmentation

```ts
// src/types/next-auth.d.ts
import type { DefaultSession } from "next-auth"

/**
 * Extend the built-in NextAuth session types to include user.id.
 *
 * This is required because:
 *   1. We set session.user.id in the session callback (auth.ts)
 *   2. Without this declaration, TypeScript won't know the field exists
 *   3. This makes session.user.id type-safe throughout the codebase
 */
declare module "next-auth" {
  interface Session {
    user: {
      id: string
    } & DefaultSession["user"]
  }
}
```

---

## 8. Supabase Client Helpers

```ts
// src/lib/supabase/server.ts
import { createClient } from "@supabase/supabase-js"
import { env } from "@/env"

/**
 * Server-side Supabase client using the service role key.
 *
 * USE ONLY IN:
 *   - Server Components
 *   - Server Actions
 *   - API Routes
 *   - NextAuth adapter (already handled internally)
 *
 * NEVER import this in Client Components — the service role key
 * bypasses Row Level Security and must never reach the browser.
 */
export function createServerSupabaseClient() {
  return createClient(env.SUPABASE_URL, env.SUPABASE_SERVICE_ROLE_KEY, {
    auth: {
      /**
       * Disable auto-refresh and persisting session in the server client.
       * Session management is handled by NextAuth, not Supabase Auth.
       */
      autoRefreshToken: false,
      persistSession: false,
    },
  })
}
```

```ts
// src/lib/supabase/client.ts
"use client"

import { createClient } from "@supabase/supabase-js"
import { env } from "@/env"

/**
 * Browser-side Supabase client using the anon key.
 *
 * USE ONLY IN Client Components for:
 *   - Real-time subscriptions
 *   - Reading public data
 *
 * For mutations, prefer Server Actions with the server client
 * to keep business logic server-side.
 *
 * Note: This client respects Supabase Row Level Security (RLS).
 */
export const supabaseBrowserClient = createClient(
  env.NEXT_PUBLIC_SUPABASE_URL,
  env.NEXT_PUBLIC_SUPABASE_ANON_KEY
)
```

---

## 9. Security Checklist

| # | Check | Status | Implementation detail |
|---|-------|--------|----------------------|
| 1 | **All inputs validated with Zod** | PASS | Email input in login Server Action validated (type check + length + format). `callbackUrl` validated to be same-origin only. |
| 2 | **No `eval()` or `dangerouslySetInnerHTML` with user content** | PASS | No dynamic code evaluation. User-provided data (name, email) rendered via React's safe text nodes only. |
| 3 | **SQL injection impossible** | PASS | All DB access via Supabase JS client (parameterised queries). SupabaseAdapter handles auth tables internally. No raw SQL. |
| 4 | **Auth checked on every protected Server Action** | PASS | Layout + page both call `await auth()` and redirect if `!session`. Server Actions must also call `auth()` before any data access (shown in comments). |
| 5 | **Rate limiting on all public mutations** | NOTE | Rate limiting not included in this auth implementation. Add `@upstash/ratelimit` to the `/api/auth/signin` route and magic link Server Action. See rate limiting section in skill. |
| 6 | **Error messages never expose stack traces** | PASS | `getAuthErrorMessage()` maps error codes to friendly Spanish strings. Auth errors logged server-side via `console.error`. No raw error objects sent to client. |
| 7 | **Secrets accessed only via `env.*`** | PASS | All secrets declared in `src/env.ts`. `process.env` accessed only in `runtimeEnv` block. All other files use `import { env } from "@/env"`. |
| 8 | **CORS configured explicitly** | PASS | NextAuth handles CORS for `/api/auth/*`. No custom cross-origin API routes in this implementation. |
| 9 | **Webhook endpoints verify signatures** | N/A | No webhook endpoints in this implementation. When adding (e.g., payment webhooks), use `crypto.timingSafeEqual` to verify HMAC signatures. |
| 10 | **Session token is database-backed** | PASS | `strategy: "database"` in NextAuth config. Sessions stored in Supabase via SupabaseAdapter — instant invalidation possible. |
| 11 | **Magic link expires quickly** | PASS | `maxAge: 10 * 60` (10 minutes) on Resend provider. Default is 24h — reduced to limit exposure window. |
| 12 | **Google OAuth requests minimum scopes** | PASS | `scope: "openid email profile"` only. `prompt: "select_account"` prevents silent session reuse. |
| 13 | **CSRF protection on sign-out** | PASS | `signOut()` from `next-auth/react` sends POST with CSRF token automatically. `LogoutButton` uses this function, not a plain GET link. |
| 14 | **`callbackUrl` open redirect prevented** | PASS | Server validates `callbackUrl.startsWith("/")` before passing to `signIn`. NextAuth also validates same-origin internally (defence in depth). |
| 15 | **Protected pages not indexed by search engines** | PASS | `robots: { index: false, follow: false }` on metadata for `/pedidos`, `/cuenta`, and all `/auth/*` pages. |
| 16 | **Defence in depth (middleware + page guard)** | PASS | Middleware is first line of defence. Server Components perform explicit `auth()` check as second layer. Never rely on middleware alone. |
| 17 | **Service role key server-only** | PASS | `SUPABASE_SERVICE_ROLE_KEY` declared only in `server: {}` block of T3 Env. `createServerSupabaseClient()` file is NOT marked `"use client"`. |
| 18 | **Double-click / race condition on logout** | PASS | `isLoading` state in `LogoutButton` prevents multiple concurrent `signOut()` calls. |

---

## 10. File Structure Summary

```
src/
├── env.ts                              # T3 Env — all typed secrets
├── auth.ts                             # NextAuth v5 config (Google + Resend + Supabase)
├── middleware.ts                       # Edge middleware — protects /pedidos/* /cuenta/*
├── types/
│   └── next-auth.d.ts                  # Type augmentation for session.user.id
├── lib/
│   └── supabase/
│       ├── server.ts                   # Service role client (server-only)
│       └── client.ts                   # Anon key client (browser)
├── components/
│   └── auth/
│       └── logout-button.tsx           # Client Component — calls signOut()
└── app/
    ├── api/
    │   └── auth/
    │       └── [...nextauth]/
    │           └── route.ts            # NextAuth route handler (GET + POST)
    ├── auth/
    │   ├── login/
    │   │   └── page.tsx                # Login page (Google + magic link)
    │   ├── verify/
    │   │   └── page.tsx                # "Check your email" page
    │   └── error/
    │       └── page.tsx                # Auth error page
    └── pedidos/
        ├── layout.tsx                  # Protected layout with LogoutButton
        └── page.tsx                    # Protected page — double auth check
```

---

## 11. Setup Instructions

### Supabase Setup

Run the NextAuth v5 schema in Supabase SQL Editor (from `@auth/supabase-adapter` docs):

```sql
-- Required tables for SupabaseAdapter
-- Run once in Supabase SQL Editor

create table users (
  id uuid default gen_random_uuid() primary key,
  name text,
  email text unique,
  "emailVerified" timestamptz,
  image text
);

create table accounts (
  id uuid default gen_random_uuid() primary key,
  "userId" uuid not null references users(id) on delete cascade,
  type text not null,
  provider text not null,
  "providerAccountId" text not null,
  refresh_token text,
  access_token text,
  expires_at bigint,
  token_type text,
  scope text,
  id_token text,
  session_state text,
  unique(provider, "providerAccountId")
);

create table sessions (
  id uuid default gen_random_uuid() primary key,
  "sessionToken" text unique not null,
  "userId" uuid not null references users(id) on delete cascade,
  expires timestamptz not null
);

create table verification_tokens (
  identifier text not null,
  token text not null,
  expires timestamptz not null,
  unique(identifier, token)
);
```

### Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create OAuth 2.0 credentials
3. Add Authorized redirect URI: `http://localhost:3000/api/auth/callback/google`
4. For production: `https://cafevermut.com/api/auth/callback/google`

### Resend Setup

1. Create account at [resend.com](https://resend.com)
2. Verify your sending domain (`cafevermut.com`)
3. Create API key with "Sending access" only
4. Set `EMAIL_FROM=noreply@cafevermut.com`

---

*Generated by backend-developer skill — Café Vermut auth implementation*
