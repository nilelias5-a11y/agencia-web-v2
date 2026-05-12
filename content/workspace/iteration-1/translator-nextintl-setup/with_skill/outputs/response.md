# Configuración completa de next-intl en Next.js 15 App Router (ES + EN)

## Instalación

```bash
npm install next-intl
```

---

## Todos los archivos que necesitas crear

### Estructura final del proyecto

```
proyecto/
├── messages/
│   ├── es.json               ← textos en español
│   └── en.json               ← textos en inglés
├── src/
│   └── i18n/
│       ├── routing.ts        ← define locales y locale por defecto
│       ├── middleware.ts     ← NO, el middleware va en la raíz
│       ├── navigation.ts     ← Link, useRouter, usePathname type-safe
│       └── request.ts        ← carga mensajes en Server Components
├── app/
│   ├── layout.tsx            ← layout raíz (sin lógica de locale)
│   └── [locale]/
│       ├── layout.tsx        ← NextIntlClientProvider aquí
│       └── page.tsx          ← página de inicio
├── middleware.ts             ← en la raíz, junto a package.json
├── next.config.ts            ← envuelve con withNextIntl
└── global.d.ts               ← type safety para claves de traducción
```

---

## Archivo 1 — `src/i18n/routing.ts`

Define los locales disponibles y el locale por defecto. Es la fuente de verdad de toda la configuración.

```typescript
// src/i18n/routing.ts
import { defineRouting } from 'next-intl/routing'

export const routing = defineRouting({
  locales: ['es', 'en'],
  defaultLocale: 'es',
})
```

- `locales`: array con todos los idiomas que soporta la web.
- `defaultLocale`: idioma que se usa cuando no se puede detectar el del usuario (o cuando accede a `/` sin prefijo de idioma).
- Puedes añadir `pathnames` si quieres slugs traducidos (p. ej. `/sobre-nosotros` en ES y `/about` en EN), pero no es obligatorio para empezar.

---

## Archivo 2 — `middleware.ts` (en la raíz del proyecto)

Intercepta todas las peticiones HTTP y redirige al usuario al locale correcto según su configuración de navegador.

```typescript
// middleware.ts  ← raíz del proyecto, JUNTO a package.json
import createMiddleware from 'next-intl/middleware'
import { routing } from './src/i18n/routing'

export default createMiddleware(routing)

export const config = {
  matcher: ['/((?!api|_next|_vercel|.*\\..*).*)'],
}
```

**Por qué va en la raíz y no en `src/`:** Next.js solo reconoce `middleware.ts` en la raíz del proyecto (o en `src/` si usas la convención `src/middleware.ts`, pero siempre en la capa más alta de la app).

**El `matcher`** excluye rutas que no deben pasar por i18n:
- `api` → tus endpoints de API
- `_next` → assets internos de Next.js
- `_vercel` → infraestructura de Vercel
- `.*\\..*` → archivos estáticos (imágenes, fuentes, etc.)

---

## Archivo 3 — `src/i18n/navigation.ts`

Crea versiones type-safe de los hooks y componentes de navegación de Next.js, que preservan el locale automáticamente.

```typescript
// src/i18n/navigation.ts
import { createNavigation } from 'next-intl/navigation'
import { routing } from './routing'

export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing)
```

### IMPORTANTE: usa SIEMPRE este `Link`, nunca el de Next.js

| Lo que debes usar | Lo que NO debes usar |
|---|---|
| `import { Link } from '@/i18n/navigation'` | `import Link from 'next/link'` |
| `import { useRouter } from '@/i18n/navigation'` | `import { useRouter } from 'next/navigation'` |
| `import { usePathname } from '@/i18n/navigation'` | `import { usePathname } from 'next/navigation'` |

El `Link` de `@/i18n/navigation` añade automáticamente el prefijo de locale a todos los `href`. Si usas el de Next.js, los enlaces no tendrán locale y perderás el idioma al navegar.

**Ejemplo de uso correcto:**

```typescript
import { Link } from '@/i18n/navigation'

// Genera /es/about o /en/about según el locale actual
<Link href="/about">Sobre nosotros</Link>
```

---

## Archivo 4 — `src/i18n/request.ts`

Carga los mensajes correctos en cada petición del servidor. next-intl lo llama internamente.

```typescript
// src/i18n/request.ts
import { getRequestConfig } from 'next-intl/server'
import { routing } from './routing'

export default getRequestConfig(async ({ requestLocale }) => {
  let locale = await requestLocale

  // Fallback al locale por defecto si el locale no es válido
  if (!locale || !routing.locales.includes(locale as any)) {
    locale = routing.defaultLocale
  }

  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default,
  }
})
```

---

## Archivo 5 — `next.config.ts`

Conecta next-intl con el compilador de Next.js.

```typescript
// next.config.ts
import { NextConfig } from 'next'
import createNextIntlPlugin from 'next-intl/plugin'

const withNextIntl = createNextIntlPlugin('./src/i18n/request.ts')

const nextConfig: NextConfig = {}

export default withNextIntl(nextConfig)
```

---

## Archivo 6 — `app/[locale]/layout.tsx`

Este es el layout que envuelve todas las páginas con locale. Aquí vive `NextIntlClientProvider`, que pasa los mensajes a los Client Components.

```typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl'
import { getMessages } from 'next-intl/server'
import { notFound } from 'next/navigation'
import { routing } from '@/i18n/routing'

// Genera las rutas estáticas para cada locale en build time
export function generateStaticParams() {
  return routing.locales.map((locale) => ({ locale }))
}

export default async function LocaleLayout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ locale: string }>  // En Next.js 15 params es una Promise
}) {
  const { locale } = await params

  // Si el locale no existe, muestra 404
  if (!routing.locales.includes(locale as any)) notFound()

  // Carga los mensajes del locale actual para pasarlos al Provider
  const messages = await getMessages()

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
```

**Por qué `getMessages()` en lugar de cargar el JSON directamente:** `getMessages()` obtiene los mensajes que ya cargó `request.ts` para esta petición, sin duplicar la carga. Es la forma correcta de pasarlos al Provider.

---

## Archivo 7 — `global.d.ts` (type safety)

Hace que TypeScript conozca todas las claves de traducción. Si escribes una clave que no existe, obtienes un error en tiempo de compilación.

```typescript
// global.d.ts (en la raíz del proyecto)
import es from './messages/es.json'

declare global {
  type IntlMessages = typeof es
}
```

---

## Archivos 8 y 9 — `messages/es.json` y `messages/en.json`

Organiza los mensajes por namespace (una sección por página + `Common` para elementos compartidos).

```json
// messages/es.json
{
  "Common": {
    "cta": "Contactar",
    "back": "Volver",
    "loading": "Cargando..."
  },
  "Home": {
    "hero": {
      "title": "Diseño web que convierte",
      "subtitle": "Creamos experiencias digitales para empresas",
      "cta": "Empieza tu proyecto"
    }
  },
  "About": {
    "title": "Sobre nosotros",
    "description": "Somos una agencia especializada en Next.js"
  }
}
```

```json
// messages/en.json
{
  "Common": {
    "cta": "Contact us",
    "back": "Back",
    "loading": "Loading..."
  },
  "Home": {
    "hero": {
      "title": "Web design that converts",
      "subtitle": "We create digital experiences for businesses",
      "cta": "Start your project"
    }
  },
  "About": {
    "title": "About us",
    "description": "We are an agency specialized in Next.js"
  }
}
```

**Reglas de organización:**
- `Common` para textos compartidos (nav, footer, botones globales)
- Un namespace por página: `Home`, `About`, `Blog`
- Claves en camelCase, nunca con guiones ni espacios
- Las claves deben ser idénticas en todos los idiomas — solo cambia el valor

---

## Distinción clave: `getTranslations()` vs `useTranslations()`

Esta es la diferencia más importante de next-intl. Confundirlas provoca errores en runtime.

### `getTranslations()` — Server Components únicamente

- Es una función **async**
- Disponible en cualquier Server Component, layout, page, o Server Action
- **No necesita** `NextIntlClientProvider` (funciona sin el Provider)
- Importar desde `'next-intl/server'`

```typescript
// app/[locale]/page.tsx  ← Server Component (sin 'use client')
import { getTranslations } from 'next-intl/server'

export default async function HomePage() {
  // async → await obligatorio
  const t = await getTranslations('Home')

  return (
    <main>
      <h1>{t('hero.title')}</h1>
      <p>{t('hero.subtitle')}</p>
      <a href="/contact">{t('hero.cta')}</a>
    </main>
  )
}
```

### `useTranslations()` — Client Components únicamente

- Es un **hook de React** (sync, sin await)
- Solo funciona dentro de Client Components (`'use client'`)
- **Requiere** que el componente esté envuelto en `NextIntlClientProvider`
- Importar desde `'next-intl'` (sin `/server`)

```typescript
// components/HeroSection.tsx  ← Client Component
'use client'
import { useTranslations } from 'next-intl'

export function HeroSection() {
  // sync → sin await
  const t = useTranslations('Home')

  return (
    <section>
      <h1>{t('hero.title')}</h1>
      <p>{t('hero.subtitle')}</p>
    </section>
  )
}
```

### Tabla resumen

| | `getTranslations()` | `useTranslations()` |
|---|---|---|
| Tipo de componente | Server Component | Client Component |
| Sintaxis | `const t = await getTranslations('NS')` | `const t = useTranslations('NS')` |
| Importación | `from 'next-intl/server'` | `from 'next-intl'` |
| Necesita Provider | No | Sí (NextIntlClientProvider) |
| Puede usar hooks React | No | Sí |
| Recomendado para | Pages, layouts, SEO | Formularios, animaciones, estado |

**Regla práctica:** Por defecto, usa `getTranslations()` en Server Components. Solo usa `useTranslations()` cuando el componente ya necesite ser Client Component por otro motivo (useState, useEffect, event handlers, etc.).

---

## Bonus — Selector de idioma

Para que el usuario pueda cambiar de idioma, usa `useRouter` de tu `navigation.ts`:

```typescript
// components/LanguageSwitcher.tsx
'use client'
import { useRouter, usePathname } from '@/i18n/navigation'
import { routing } from '@/i18n/routing'

export function LanguageSwitcher({ currentLocale }: { currentLocale: string }) {
  const router = useRouter()
  const pathname = usePathname()

  return (
    <div>
      {routing.locales.map((locale) => (
        <button
          key={locale}
          onClick={() => router.replace(pathname, { locale })}
          disabled={locale === currentLocale}
        >
          {locale === 'es' ? 'ES' : 'EN'}
        </button>
      ))}
    </div>
  )
}
```

`router.replace(pathname, { locale })` cambia el idioma manteniendo la misma ruta (p. ej. de `/es/about` a `/en/about`).

---

## Checklist de configuración

- [ ] `npm install next-intl`
- [ ] `src/i18n/routing.ts` — `defineRouting` con locales y defaultLocale
- [ ] `middleware.ts` (raíz) — `createMiddleware(routing)` con matcher
- [ ] `src/i18n/navigation.ts` — `createNavigation` — usar SIEMPRE este `Link`
- [ ] `src/i18n/request.ts` — `getRequestConfig` con carga de mensajes
- [ ] `next.config.ts` — `withNextIntl` apuntando a `request.ts`
- [ ] `app/[locale]/layout.tsx` — `NextIntlClientProvider` + `getMessages()`
- [ ] `global.d.ts` — `IntlMessages = typeof es` para type safety
- [ ] `messages/es.json` y `messages/en.json` con la misma estructura de claves
- [ ] `generateStaticParams` en el layout de `[locale]`
- [ ] Todos los `Link` del proyecto usan `@/i18n/navigation`, no `next/link`
