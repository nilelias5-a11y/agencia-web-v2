# NextAuth v5 + Supabase Auth — Café Vermut

Complete implementation of NextAuth v5 authentication with Google provider, Resend magic link provider, Supabase Adapter, route protection via middleware, protected Server Component, and logout Client Component.

---

## Dependencies

```bash
npm install next-auth@beta @auth/supabase-adapter @supabase/supabase-js
npm install resend
# next-auth v5 already includes the Resend provider
```

---

## 1. `src/auth.ts` — NextAuth v5 Configuration

```typescript
// src/auth.ts
import NextAuth from "next-auth";
import Google from "next-auth/providers/google";
import Resend from "next-auth/providers/resend";
import { SupabaseAdapter } from "@auth/supabase-adapter";

export const { handlers, signIn, signOut, auth } = NextAuth({
  // Supabase adapter for session/user persistence
  adapter: SupabaseAdapter({
    url: process.env.SUPABASE_URL!,
    secret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  }),

  providers: [
    // Google OAuth provider
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      authorization: {
        params: {
          prompt: "consent",
          access_type: "offline",
          response_type: "code",
        },
      },
    }),

    // Resend magic link (email) provider
    Resend({
      apiKey: process.env.RESEND_API_KEY!,
      from: process.env.EMAIL_FROM ?? "Café Vermut <noreply@cafevermut.com>",
      // Custom email template for the magic link
      sendVerificationRequest: async ({ identifier: email, url, provider }) => {
        const { createTransport } = await import("nodemailer");
        // Use Resend's HTTP API directly
        const res = await fetch("https://api.resend.com/emails", {
          method: "POST",
          headers: {
            Authorization: `Bearer ${provider.apiKey}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            from: provider.from,
            to: email,
            subject: "Accede a Café Vermut — Tu enlace mágico",
            html: `
              <div style="font-family: sans-serif; max-width: 480px; margin: 0 auto; padding: 32px;">
                <h1 style="color: #1a1a1a; font-size: 24px; margin-bottom: 8px;">
                  Café Vermut
                </h1>
                <p style="color: #555; font-size: 16px; line-height: 1.5;">
                  Haz clic en el siguiente enlace para acceder a tu cuenta. 
                  El enlace expira en 10 minutos.
                </p>
                <a
                  href="${url}"
                  style="
                    display: inline-block;
                    margin-top: 24px;
                    padding: 14px 28px;
                    background-color: #b45309;
                    color: #fff;
                    text-decoration: none;
                    border-radius: 8px;
                    font-size: 16px;
                    font-weight: 600;
                  "
                >
                  Acceder ahora
                </a>
                <p style="color: #999; font-size: 13px; margin-top: 32px;">
                  Si no solicitaste este correo, puedes ignorarlo con total seguridad.
                </p>
              </div>
            `,
          }),
        });

        if (!res.ok) {
          const error = await res.json();
          throw new Error(
            `Error al enviar el magic link: ${error.message ?? res.statusText}`
          );
        }
      },
    }),
  ],

  // Session strategy: "database" because we use Supabase adapter
  session: {
    strategy: "database",
    // Session max age: 30 days
    maxAge: 30 * 24 * 60 * 60,
    // Update session activity every 24 hours
    updateAge: 24 * 60 * 60,
  },

  // Custom pages
  pages: {
    signIn: "/login",
    verifyRequest: "/login/verificar",
    error: "/login/error",
    signOut: "/",
  },

  callbacks: {
    // Expose user id in the session object
    async session({ session, user }) {
      if (session.user && user) {
        session.user.id = user.id;
      }
      return session;
    },

    // Control who can sign in
    async signIn({ user, account, profile }) {
      // Allow all sign-ins (add domain restrictions here if needed)
      // Example: only allow @cafevermut.com emails:
      // if (user.email && !user.email.endsWith("@cafevermut.com")) {
      //   return false;
      // }
      return true;
    },

    // Control redirects after sign-in
    async redirect({ url, baseUrl }) {
      // If the URL is relative, prefix with baseUrl
      if (url.startsWith("/")) return `${baseUrl}${url}`;
      // If the URL is on the same origin, allow it
      if (new URL(url).origin === baseUrl) return url;
      // Otherwise redirect to /pedidos (the protected area)
      return `${baseUrl}/pedidos`;
    },
  },

  // Enable debug logs in development
  debug: process.env.NODE_ENV === "development",

  // Trust the host header (required for some deployment environments)
  trustHost: true,
});
```

### Type augmentation for `session.user.id`

```typescript
// src/types/next-auth.d.ts
import type { DefaultSession } from "next-auth";

declare module "next-auth" {
  interface Session {
    user: {
      id: string;
    } & DefaultSession["user"];
  }
}
```

---

## 2. `src/middleware.ts` — Route Protection

```typescript
// src/middleware.ts
import { auth } from "@/auth";
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// The auth() helper from NextAuth v5 can wrap the middleware directly
export default auth((req) => {
  const { nextUrl, auth: session } = req;
  const isLoggedIn = !!session?.user;

  // Protected route patterns
  const isProtectedRoute =
    nextUrl.pathname.startsWith("/pedidos") ||
    nextUrl.pathname.startsWith("/cuenta");

  // Auth routes — redirect logged-in users away
  const isAuthRoute =
    nextUrl.pathname.startsWith("/login") ||
    nextUrl.pathname.startsWith("/api/auth");

  // Allow all /api/auth/* routes to pass through (NextAuth handlers)
  if (nextUrl.pathname.startsWith("/api/auth")) {
    return NextResponse.next();
  }

  // If the route is protected and the user is NOT logged in → redirect to /login
  if (isProtectedRoute && !isLoggedIn) {
    const loginUrl = new URL("/login", nextUrl.origin);
    // Preserve the original URL as a callbackUrl so we can redirect after login
    loginUrl.searchParams.set("callbackUrl", nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  // If user IS logged in and tries to access /login → redirect to /pedidos
  if (isAuthRoute && isLoggedIn && !nextUrl.pathname.startsWith("/api/auth")) {
    return NextResponse.redirect(new URL("/pedidos", nextUrl.origin));
  }

  return NextResponse.next();
});

// Matcher: run middleware on all routes EXCEPT static files and Next internals
export const config = {
  matcher: [
    /*
     * Match all request paths EXCEPT:
     * - _next/static  (static files)
     * - _next/image   (image optimization)
     * - favicon.ico
     * - public folder assets (images, fonts, etc.)
     */
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp|ico|css|js)$).*)",
  ],
};
```

---

## 3. Environment Variables

### `.env.local` (development)

```bash
# ─── NextAuth ────────────────────────────────────────────────────────────────
# Generate with: openssl rand -base64 32
AUTH_SECRET="<generated-secret-min-32-chars>"

# Public URL of the app (no trailing slash)
NEXTAUTH_URL="http://localhost:3000"
# In NextAuth v5 the env var is also read as AUTH_URL
AUTH_URL="http://localhost:3000"

# ─── Google OAuth ────────────────────────────────────────────────────────────
# Create credentials at: https://console.cloud.google.com/apis/credentials
# Authorized redirect URI: http://localhost:3000/api/auth/callback/google
GOOGLE_CLIENT_ID="<your-google-client-id>.apps.googleusercontent.com"
GOOGLE_CLIENT_SECRET="<your-google-client-secret>"

# ─── Resend (magic link email) ───────────────────────────────────────────────
# Get your API key at: https://resend.com/api-keys
RESEND_API_KEY="re_<your-resend-api-key>"
# Must be a verified domain in Resend
EMAIL_FROM="Café Vermut <noreply@cafevermut.com>"

# ─── Supabase ────────────────────────────────────────────────────────────────
# Found in: Supabase Dashboard → Settings → API
SUPABASE_URL="https://<project-ref>.supabase.co"
# IMPORTANT: use SERVICE_ROLE key (not anon key) for the adapter
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
# Anon key for client-side Supabase queries (if needed separately)
NEXT_PUBLIC_SUPABASE_URL="https://<project-ref>.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
```

### `.env.production` (Vercel / production)

```bash
# Same variables as above but with production values:
AUTH_SECRET="<different-production-secret>"
NEXTAUTH_URL="https://cafevermut.com"
AUTH_URL="https://cafevermut.com"
GOOGLE_CLIENT_ID="<same-or-different-client-id>"
GOOGLE_CLIENT_SECRET="<production-google-secret>"
RESEND_API_KEY="re_<production-resend-key>"
EMAIL_FROM="Café Vermut <noreply@cafevermut.com>"
SUPABASE_URL="https://<project-ref>.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
NEXT_PUBLIC_SUPABASE_URL="https://<project-ref>.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
```

### Notes on environment variables

- `AUTH_SECRET` is **required** in production. Generate with `openssl rand -base64 32`.
- In NextAuth v5, `AUTH_SECRET` replaces `NEXTAUTH_SECRET` (though the old name still works).
- `SUPABASE_SERVICE_ROLE_KEY` must be the **service role** key, NOT the anon key — the adapter needs it to write to the database bypassing RLS.
- Add `NEXTAUTH_URL` / `AUTH_URL` explicitly in all environments, especially when deployed behind a proxy.

---

## 4. Route Handler — `src/app/api/auth/[...nextauth]/route.ts`

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth";

// Export the GET and POST handlers from NextAuth v5
export const { GET, POST } = handlers;

// Ensure this route is always dynamically rendered
export const dynamic = "force-dynamic";
```

---

## 5. Protected Server Component — `src/app/pedidos/page.tsx`

```typescript
// src/app/pedidos/page.tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";
import { LogoutButton } from "@/components/auth/LogoutButton";

// This page is a React Server Component (RSC)
// It verifies the session server-side and redirects if not authenticated.
// The middleware already protects this route, but this double-check
// is best practice — defence in depth.

export default async function PedidosPage() {
  // auth() returns the session or null — it reads from the database via
  // the Supabase adapter, so it is always up to date.
  const session = await auth();

  // If somehow middleware was bypassed or session expired mid-render
  if (!session?.user) {
    redirect("/login?callbackUrl=/pedidos");
  }

  return (
    <main className="min-h-screen bg-stone-50">
      {/* ── Top bar ── */}
      <header className="flex items-center justify-between px-6 py-4 bg-white border-b border-stone-200 shadow-sm">
        <div className="flex items-center gap-3">
          {/* Café Vermut branding */}
          <span className="text-2xl font-bold text-amber-800 tracking-tight">
            Café Vermut
          </span>
          <span className="hidden sm:inline text-stone-400 text-sm font-medium">
            / Mis Pedidos
          </span>
        </div>

        {/* User info + logout */}
        <div className="flex items-center gap-4">
          <div className="hidden sm:flex flex-col items-end">
            <span className="text-sm font-semibold text-stone-800">
              {session.user.name ?? "Usuario"}
            </span>
            <span className="text-xs text-stone-500">{session.user.email}</span>
          </div>
          {session.user.image && (
            // eslint-disable-next-line @next/next/no-img-element
            <img
              src={session.user.image}
              alt={session.user.name ?? "Avatar"}
              className="w-9 h-9 rounded-full ring-2 ring-amber-300 object-cover"
            />
          )}
          {/* Client Component — handles sign-out */}
          <LogoutButton />
        </div>
      </header>

      {/* ── Page content ── */}
      <section className="max-w-4xl mx-auto px-6 py-10">
        <h1 className="text-3xl font-bold text-stone-900 mb-2">Mis Pedidos</h1>
        <p className="text-stone-500 mb-8">
          Bienvenido/a,{" "}
          <strong className="text-amber-800">
            {session.user.name ?? session.user.email}
          </strong>
          . Aquí puedes ver el historial y estado de todos tus pedidos.
        </p>

        {/* Orders list placeholder */}
        <div className="space-y-4">
          {/* In a real app, fetch orders from Supabase here: */}
          {/* const orders = await getOrdersByUserId(session.user.id); */}

          <div className="rounded-xl border border-stone-200 bg-white p-6 shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="font-semibold text-stone-800">Pedido #2024-001</p>
                <p className="text-sm text-stone-500 mt-0.5">
                  Vermut rojo, tabla de embutidos × 2
                </p>
              </div>
              <span className="inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-amber-100 text-amber-800">
                En preparación
              </span>
            </div>
            <p className="text-xs text-stone-400 mt-3">13 may 2026 · 13:45</p>
          </div>

          <div className="rounded-xl border border-stone-200 bg-white p-6 shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="font-semibold text-stone-800">Pedido #2024-000</p>
                <p className="text-sm text-stone-500 mt-0.5">
                  Croquetas de jamón × 1, Cerveza artesanal × 2
                </p>
              </div>
              <span className="inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-green-100 text-green-700">
                Entregado
              </span>
            </div>
            <p className="text-xs text-stone-400 mt-3">10 may 2026 · 20:12</p>
          </div>
        </div>
      </section>
    </main>
  );
}
```

---

## 6. Logout Button — `src/components/auth/LogoutButton.tsx`

```typescript
// src/components/auth/LogoutButton.tsx
"use client";

import { signOut } from "next-auth/react";
import { useState } from "react";

interface LogoutButtonProps {
  /** Optional redirect URL after logout. Defaults to "/". */
  redirectTo?: string;
  /** Custom button label. */
  label?: string;
  /** Extra CSS classes. */
  className?: string;
}

export function LogoutButton({
  redirectTo = "/",
  label = "Cerrar sesión",
  className,
}: LogoutButtonProps) {
  const [isLoading, setIsLoading] = useState(false);

  const handleSignOut = async () => {
    try {
      setIsLoading(true);
      await signOut({
        // Redirect to the specified URL after logout
        callbackUrl: redirectTo,
        // Redirect client-side (set to false to do a full page reload)
        redirect: true,
      });
    } catch (error) {
      console.error("Error al cerrar sesión:", error);
      setIsLoading(false);
    }
  };

  return (
    <button
      onClick={handleSignOut}
      disabled={isLoading}
      aria-label="Cerrar sesión"
      className={
        className ??
        `
        inline-flex items-center gap-2 px-4 py-2
        rounded-lg text-sm font-semibold
        bg-amber-800 text-white
        hover:bg-amber-700 active:bg-amber-900
        disabled:opacity-60 disabled:cursor-not-allowed
        transition-colors duration-150
        focus:outline-none focus:ring-2 focus:ring-amber-500 focus:ring-offset-2
      `
      }
    >
      {isLoading ? (
        <>
          {/* Spinner */}
          <svg
            className="w-4 h-4 animate-spin"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            aria-hidden="true"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"
            />
          </svg>
          <span>Cerrando sesión…</span>
        </>
      ) : (
        <>
          {/* Logout icon */}
          <svg
            className="w-4 h-4"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            strokeWidth={2}
            stroke="currentColor"
            aria-hidden="true"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              d="M15.75 9V5.25A2.25 2.25 0 0013.5 3h-6a2.25 2.25 0 00-2.25 2.25v13.5A2.25 2.25 0 007.5 21h6a2.25 2.25 0 002.25-2.25V15M18 15l3-3m0 0l-3-3m3 3H9"
            />
          </svg>
          <span>{label}</span>
        </>
      )}
    </button>
  );
}
```

---

## 7. Login Page — `src/app/login/page.tsx`

```typescript
// src/app/login/page.tsx
import { signIn } from "@/auth";
import { redirect } from "next/navigation";
import { auth } from "@/auth";

interface LoginPageProps {
  searchParams: Promise<{ callbackUrl?: string; error?: string }>;
}

export default async function LoginPage({ searchParams }: LoginPageProps) {
  const { callbackUrl, error } = await searchParams;

  // If already authenticated, redirect to callbackUrl or /pedidos
  const session = await auth();
  if (session?.user) {
    redirect(callbackUrl ?? "/pedidos");
  }

  const errorMessages: Record<string, string> = {
    OAuthSignin: "Error al iniciar sesión con Google. Inténtalo de nuevo.",
    OAuthCallback: "Error en la respuesta de Google. Inténtalo de nuevo.",
    OAuthCreateAccount: "No se pudo crear la cuenta. Inténtalo de nuevo.",
    EmailCreateAccount: "No se pudo crear la cuenta. Inténtalo de nuevo.",
    Callback: "Error en el callback de autenticación.",
    OAuthAccountNotLinked:
      "Este correo ya está registrado con otro método. Usa el mismo método.",
    EmailSignin: "No se pudo enviar el correo. Verifica tu dirección.",
    CredentialsSignin: "Credenciales incorrectas.",
    SessionRequired: "Debes iniciar sesión para acceder.",
    default: "Ocurrió un error inesperado. Inténtalo de nuevo.",
  };

  const errorMessage = error
    ? (errorMessages[error] ?? errorMessages.default)
    : null;

  return (
    <main className="min-h-screen flex items-center justify-center bg-stone-100 px-4">
      <div className="w-full max-w-md">
        {/* Card */}
        <div className="bg-white rounded-2xl shadow-lg overflow-hidden">
          {/* Header */}
          <div className="bg-amber-800 px-8 py-10 text-center">
            <h1 className="text-3xl font-bold text-white tracking-tight">
              Café Vermut
            </h1>
            <p className="text-amber-200 text-sm mt-1">
              Accede a tu área personal
            </p>
          </div>

          {/* Body */}
          <div className="px-8 py-8 space-y-6">
            {/* Error banner */}
            {errorMessage && (
              <div
                role="alert"
                className="flex items-start gap-3 p-4 rounded-lg bg-red-50 border border-red-200 text-red-700 text-sm"
              >
                <svg
                  className="w-5 h-5 flex-shrink-0 mt-0.5"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                  aria-hidden="true"
                >
                  <path
                    fillRule="evenodd"
                    d="M10 18a8 8 0 100-16 8 8 0 000 16zm-.75-4.75a.75.75 0 001.5 0v-4.5a.75.75 0 00-1.5 0v4.5zm.75-7.25a1 1 0 110 2 1 1 0 010-2z"
                    clipRule="evenodd"
                  />
                </svg>
                <span>{errorMessage}</span>
              </div>
            )}

            {/* Google OAuth */}
            <form
              action={async () => {
                "use server";
                await signIn("google", {
                  redirectTo: callbackUrl ?? "/pedidos",
                });
              }}
            >
              <button
                type="submit"
                className="
                  w-full flex items-center justify-center gap-3
                  px-4 py-3 rounded-xl border-2 border-stone-200
                  bg-white hover:bg-stone-50 active:bg-stone-100
                  text-stone-800 font-semibold text-sm
                  transition-colors duration-150
                  focus:outline-none focus:ring-2 focus:ring-amber-500 focus:ring-offset-2
                "
              >
                {/* Google logo */}
                <svg
                  className="w-5 h-5"
                  viewBox="0 0 24 24"
                  xmlns="http://www.w3.org/2000/svg"
                  aria-hidden="true"
                >
                  <path
                    d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"
                    fill="#4285F4"
                  />
                  <path
                    d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"
                    fill="#34A853"
                  />
                  <path
                    d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"
                    fill="#FBBC05"
                  />
                  <path
                    d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"
                    fill="#EA4335"
                  />
                </svg>
                Continuar con Google
              </button>
            </form>

            {/* Divider */}
            <div className="flex items-center gap-4">
              <div className="flex-1 h-px bg-stone-200" />
              <span className="text-xs text-stone-400 font-medium">O</span>
              <div className="flex-1 h-px bg-stone-200" />
            </div>

            {/* Magic link (email) */}
            <form
              action={async (formData: FormData) => {
                "use server";
                const email = formData.get("email") as string;
                await signIn("resend", {
                  email,
                  redirectTo: callbackUrl ?? "/pedidos",
                });
              }}
              className="space-y-3"
            >
              <div>
                <label
                  htmlFor="email"
                  className="block text-sm font-medium text-stone-700 mb-1.5"
                >
                  Correo electrónico
                </label>
                <input
                  id="email"
                  name="email"
                  type="email"
                  required
                  autoComplete="email"
                  placeholder="tu@correo.com"
                  className="
                    w-full px-4 py-3 rounded-xl border border-stone-300
                    text-stone-900 text-sm placeholder-stone-400
                    focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent
                    transition-shadow duration-150
                  "
                />
              </div>
              <button
                type="submit"
                className="
                  w-full px-4 py-3 rounded-xl
                  bg-amber-800 hover:bg-amber-700 active:bg-amber-900
                  text-white font-semibold text-sm
                  transition-colors duration-150
                  focus:outline-none focus:ring-2 focus:ring-amber-500 focus:ring-offset-2
                "
              >
                Enviar enlace mágico
              </button>
            </form>
          </div>
        </div>

        <p className="text-center text-xs text-stone-400 mt-6">
          Al iniciar sesión aceptas nuestros{" "}
          <a href="/terminos" className="underline hover:text-stone-600">
            Términos de Uso
          </a>{" "}
          y{" "}
          <a href="/privacidad" className="underline hover:text-stone-600">
            Política de Privacidad
          </a>
          .
        </p>
      </div>
    </main>
  );
}
```

---

## 8. Magic Link Verify Page — `src/app/login/verificar/page.tsx`

```typescript
// src/app/login/verificar/page.tsx

export default function VerifyRequestPage() {
  return (
    <main className="min-h-screen flex items-center justify-center bg-stone-100 px-4">
      <div className="w-full max-w-sm text-center">
        <div className="bg-white rounded-2xl shadow-lg px-8 py-10 space-y-5">
          {/* Email icon */}
          <div className="flex justify-center">
            <div className="w-16 h-16 rounded-full bg-amber-100 flex items-center justify-center">
              <svg
                className="w-8 h-8 text-amber-800"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                strokeWidth={1.5}
                stroke="currentColor"
                aria-hidden="true"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  d="M21.75 6.75v10.5a2.25 2.25 0 01-2.25 2.25h-15a2.25 2.25 0 01-2.25-2.25V6.75m19.5 0A2.25 2.25 0 0019.5 4.5h-15a2.25 2.25 0 00-2.25 2.25m19.5 0v.243a2.25 2.25 0 01-1.07 1.916l-7.5 4.615a2.25 2.25 0 01-2.36 0L3.32 8.91a2.25 2.25 0 01-1.07-1.916V6.75"
                />
              </svg>
            </div>
          </div>

          <div>
            <h1 className="text-xl font-bold text-stone-900">
              Revisa tu correo
            </h1>
            <p className="text-stone-500 text-sm mt-2 leading-relaxed">
              Hemos enviado un enlace mágico a tu correo. Haz clic en él para
              iniciar sesión. El enlace expira en 10 minutos.
            </p>
          </div>

          <p className="text-xs text-stone-400">
            ¿No lo ves? Revisa la carpeta de spam o{" "}
            <a href="/login" className="text-amber-800 underline font-medium">
              solicita otro enlace
            </a>
            .
          </p>
        </div>
      </div>
    </main>
  );
}
```

---

## 9. Supabase Database Schema

The `@auth/supabase-adapter` requires specific tables. Run this SQL in the Supabase SQL Editor:

```sql
-- NextAuth v5 + Supabase Adapter required tables
-- Run this in your Supabase SQL Editor

create table if not exists "next_auth"."users" (
  id uuid not null default gen_random_uuid(),
  name text,
  email text,
  "emailVerified" timestamptz,
  image text,
  primary key (id)
);

create table if not exists "next_auth"."accounts" (
  id uuid not null default gen_random_uuid(),
  "userId" uuid not null references "next_auth"."users"(id) on delete cascade,
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
  primary key (id),
  unique (provider, "providerAccountId")
);

create table if not exists "next_auth"."sessions" (
  id uuid not null default gen_random_uuid(),
  "sessionToken" text not null,
  "userId" uuid not null references "next_auth"."users"(id) on delete cascade,
  expires timestamptz not null,
  primary key (id),
  unique ("sessionToken")
);

create table if not exists "next_auth"."verification_tokens" (
  identifier text not null,
  token text not null,
  expires timestamptz not null,
  primary key (token),
  unique (identifier, token)
);

-- Grant access to the service role
grant usage on schema "next_auth" to service_role;
grant all privileges on all tables in schema "next_auth" to service_role;
grant all privileges on all sequences in schema "next_auth" to service_role;
```

---

## 10. `next.config.ts` — Relevant Configuration

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Allow images from Google and Gravatar (for user avatars)
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "lh3.googleusercontent.com",
        pathname: "/**",
      },
      {
        protocol: "https",
        hostname: "avatars.githubusercontent.com",
        pathname: "/**",
      },
      {
        protocol: "https",
        hostname: "*.googleusercontent.com",
        pathname: "/**",
      },
    ],
  },

  // Required for NextAuth v5 with edge middleware
  experimental: {
    // serverComponentsExternalPackages ensures Supabase adapter works in RSC
  },
};

export default nextConfig;
```

---

## File Structure Summary

```
src/
├── auth.ts                                    ← NextAuth v5 config (Google + Resend + Supabase)
├── middleware.ts                               ← Route protection (/pedidos/*, /cuenta/*)
├── types/
│   └── next-auth.d.ts                         ← Session type augmentation
├── app/
│   ├── api/
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts                   ← NextAuth route handler
│   ├── login/
│   │   ├── page.tsx                           ← Login page (Google + magic link)
│   │   └── verificar/
│   │       └── page.tsx                       ← "Check your email" page
│   └── pedidos/
│       └── page.tsx                           ← Protected Server Component
└── components/
    └── auth/
        └── LogoutButton.tsx                   ← Logout Client Component
.env.local                                     ← Environment variables (dev)
```

---

## Key Implementation Notes

### NextAuth v5 vs v4 Differences

| Concern | v4 | v5 |
|---|---|---|
| Config file | `pages/api/auth/[...nextauth].ts` | `src/auth.ts` + `src/app/api/auth/[...nextauth]/route.ts` |
| Secret env var | `NEXTAUTH_SECRET` | `AUTH_SECRET` (both work) |
| URL env var | `NEXTAUTH_URL` | `AUTH_URL` (both work) |
| Middleware | Manual JWT decode | `auth()` wrapper |
| Session in RSC | `getServerSession(authOptions)` | `await auth()` |
| Client sign-out | `signOut()` from `next-auth/client` | `signOut()` from `next-auth/react` |

### Security Checklist

- `AUTH_SECRET` is mandatory in production (minimum 32 characters).
- `SUPABASE_SERVICE_ROLE_KEY` is never exposed to the client — it is only used server-side in the adapter.
- The middleware matcher excludes static assets to avoid unnecessary auth checks.
- The protected page (`/pedidos/page.tsx`) performs a **second** session check as defence in depth, in case middleware is misconfigured.
- Magic link tokens are single-use and expire in 10 minutes (NextAuth default).
- Google OAuth uses `prompt: "consent"` to always show the account chooser.
