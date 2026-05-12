# Guía completa: next-intl en Next.js 15 App Router (inglés + español)

## Instalación

```bash
npm install next-intl
```

---

## Estructura de archivos que necesitas crear

```
├── messages/
│   ├── en.json
│   └── es.json
├── src/
│   └── app/
│       └── [locale]/
│           ├── layout.tsx
│           └── page.tsx
├── i18n/
│   ├── routing.ts
│   └── request.ts
├── middleware.ts
└── next.config.ts
```

---

## 1. `messages/en.json`

```json
{
  "HomePage": {
    "title": "Welcome",
    "description": "This is the home page."
  },
  "Navigation": {
    "home": "Home",
    "about": "About"
  }
}
```

---

## 2. `messages/es.json`

```json
{
  "HomePage": {
    "title": "Bienvenido",
    "description": "Esta es la página de inicio."
  },
  "Navigation": {
    "home": "Inicio",
    "about": "Acerca de"
  }
}
```

---

## 3. `i18n/routing.ts`

Define los idiomas soportados y el locale por defecto.

```typescript
import { defineRouting } from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['en', 'es'],
  defaultLocale: 'es'
});
```

---

## 4. `i18n/request.ts`

Configura cómo se cargan los mensajes en el servidor.

```typescript
import { getRequestConfig } from 'next-intl/server';
import { routing } from './routing';

export default getRequestConfig(async ({ requestLocale }) => {
  // Obtiene el locale de la request
  let locale = await requestLocale;

  // Asegura que el locale sea válido; si no, usa el por defecto
  if (!locale || !routing.locales.includes(locale as any)) {
    locale = routing.defaultLocale;
  }

  return {
    locale,
    messages: (await import(`../messages/${locale}.json`)).default
  };
});
```

---

## 5. `middleware.ts`

El middleware intercepta todas las rutas y redirige al locale correcto automáticamente.

```typescript
import createMiddleware from 'next-intl/middleware';
import { routing } from './i18n/routing';

export default createMiddleware(routing);

export const config = {
  // Aplica el middleware a todas las rutas excepto archivos estáticos y rutas internas de Next.js
  matcher: [
    '/',
    '/(en|es)/:path*',
    '/((?!_next|_vercel|.*\\..*).*)'
  ]
};
```

---

## 6. `next.config.ts`

Integra el plugin de next-intl con Next.js.

```typescript
import type { NextConfig } from 'next';
import createNextIntlPlugin from 'next-intl/plugin';

const withNextIntl = createNextIntlPlugin('./i18n/request.ts');

const nextConfig: NextConfig = {
  // Aquí van el resto de tus opciones de Next.js
};

export default withNextIntl(nextConfig);
```

> Si usas `next.config.js` en lugar de `.ts`:
> ```javascript
> const createNextIntlPlugin = require('next-intl/plugin');
> const withNextIntl = createNextIntlPlugin('./i18n/request.ts');
> module.exports = withNextIntl({});
> ```

---

## 7. `src/app/[locale]/layout.tsx`

El layout raíz debe envolver la aplicación con el provider de next-intl.

```typescript
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';
import { notFound } from 'next/navigation';
import { routing } from '@/i18n/routing';

export function generateStaticParams() {
  return routing.locales.map((locale) => ({ locale }));
}

interface LocaleLayoutProps {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
}

export default async function LocaleLayout({
  children,
  params
}: LocaleLayoutProps) {
  const { locale } = await params;

  // Valida que el locale sea uno de los soportados
  if (!routing.locales.includes(locale as any)) {
    notFound();
  }

  // Carga los mensajes en el servidor para pasarlos al cliente
  const messages = await getMessages();

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

---

## 8. `src/app/[locale]/page.tsx`

Ejemplo de página usando traducciones.

```typescript
import { useTranslations } from 'next-intl';

export default function HomePage() {
  const t = useTranslations('HomePage');

  return (
    <main>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </main>
  );
}
```

---

## 9. Navegar entre idiomas (cambiar de locale)

Usa los helpers de next-intl para la navegación localizada. Crea `src/navigation.ts`:

```typescript
import { createNavigation } from 'next-intl/navigation';
import { routing } from '@/i18n/routing';

export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing);
```

### Selector de idioma (componente cliente)

```typescript
'use client';

import { useRouter, usePathname } from '@/navigation';
import { useLocale } from 'next-intl';

export default function LocaleSwitcher() {
  const router = useRouter();
  const pathname = usePathname();
  const locale = useLocale();

  const switchLocale = (newLocale: string) => {
    router.replace(pathname, { locale: newLocale });
  };

  return (
    <div>
      <button
        onClick={() => switchLocale('es')}
        disabled={locale === 'es'}
      >
        Español
      </button>
      <button
        onClick={() => switchLocale('en')}
        disabled={locale === 'en'}
      >
        English
      </button>
    </div>
  );
}
```

---

## 10. Usar `Link` localizado

En lugar de `next/link`, usa el `Link` de tu `navigation.ts` para que los enlaces incluyan automáticamente el locale:

```typescript
import { Link } from '@/navigation';

export default function Nav() {
  return (
    <nav>
      <Link href="/">Inicio</Link>
      <Link href="/about">Acerca de</Link>
    </nav>
  );
}
```

---

## Comportamiento del router con el middleware

| URL visitada | Resultado |
|---|---|
| `/` | Redirige a `/es/` (defaultLocale) |
| `/en/` | Muestra la home en inglés |
| `/es/about` | Muestra "about" en español |
| `/fr/` | 404 (locale no soportado) |

---

## Traducciones en componentes cliente

En componentes con `'use client'`, `useTranslations` funciona igual siempre que el componente esté dentro del `NextIntlClientProvider`:

```typescript
'use client';

import { useTranslations } from 'next-intl';

export default function ContactButton() {
  const t = useTranslations('Navigation');

  return <button>{t('home')}</button>;
}
```

---

## Metadatos SEO localizados

Para generar metadatos traducidos en una página:

```typescript
import { getTranslations } from 'next-intl/server';

export async function generateMetadata({
  params
}: {
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  const t = await getTranslations({ locale, namespace: 'HomePage' });

  return {
    title: t('title'),
    description: t('description')
  };
}
```

---

## Resumen de archivos y su función

| Archivo | Función |
|---|---|
| `messages/en.json` | Traducciones en inglés |
| `messages/es.json` | Traducciones en español |
| `i18n/routing.ts` | Define locales y defaultLocale |
| `i18n/request.ts` | Carga mensajes por request en el servidor |
| `middleware.ts` | Detecta y redirige al locale correcto |
| `next.config.ts` | Activa el plugin de next-intl |
| `app/[locale]/layout.tsx` | Envuelve la app con el provider |
| `app/[locale]/page.tsx` | Página con traducciones |
| `src/navigation.ts` | Helpers de navegación localizada |

---

## Notas importantes para Next.js 15

- En Next.js 15, los `params` de layouts y páginas son **Promises** — siempre usa `await params`.
- El archivo `i18n/request.ts` debe usar `requestLocale` (no `locale`) como argumento del callback, que también es una Promise.
- `getMessages()` en el layout servidor evita pasar los mensajes manualmente como prop.
- Usa `createNavigation` (Next.js 15) en lugar del antiguo `createSharedPathnamesNavigation`.
