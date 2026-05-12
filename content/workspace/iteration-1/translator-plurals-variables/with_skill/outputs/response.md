# Plurales, variables y rich text en next-intl

Estos tres casos — plural, variable e hipervínculo incrustado — tienen cada uno su propio patrón en next-intl. A continuación los ves todos juntos, aplicados a tus ejemplos concretos.

---

## 1. Archivos de mensajes (ICU format)

```json
// messages/es.json
{
  "Cart": {
    "itemCount": "{count, plural, =0 {Tu carrito está vacío} one {Tienes # artículo en el carrito} other {Tienes # artículos en el carrito}}"
  },
  "Auth": {
    "welcome": "Bienvenido, {name}",
    "registerTerms": "Al registrarte aceptas los <link>Términos de servicio</link>"
  }
}
```

```json
// messages/en.json
{
  "Cart": {
    "itemCount": "{count, plural, =0 {Your cart is empty} one {You have # item in your cart} other {You have # items in your cart}}"
  },
  "Auth": {
    "welcome": "Welcome, {name}",
    "registerTerms": "By signing up you accept our <link>Terms of service</link>"
  }
}
```

### Por qué este formato

| Parte | Significado |
|---|---|
| `=0` | Caso cero explícito — nunca muestra "0 artículos", sino un mensaje más natural |
| `one` | Singular (1 artículo) |
| `other` | Plural (2, 3, 4… artículos) |
| `#` | Se sustituye automáticamente por el valor de `count` |
| `{name}` | Variable de interpolación simple |
| `<link>...</link>` | Etiqueta personalizada para rich text — next-intl la interpreta pero NO la renderiza sola |

---

## 2. Plural con `=0` — componente del carrito

### Server Component

```typescript
// app/[locale]/cart/page.tsx
import { getTranslations } from 'next-intl/server'

export default async function CartPage() {
  const t = await getTranslations('Cart')
  const itemCount = 3 // viene de tu store o base de datos

  return (
    <main>
      <p>{t('itemCount', { count: itemCount })}</p>
      {/* count=0 → "Tu carrito está vacío"   */}
      {/* count=1 → "Tienes 1 artículo en el carrito"  */}
      {/* count=3 → "Tienes 3 artículos en el carrito" */}
    </main>
  )
}
```

### Client Component

```typescript
'use client'
import { useTranslations } from 'next-intl'

export function CartSummary({ itemCount }: { itemCount: number }) {
  const t = useTranslations('Cart')

  return (
    <div>
      <p>{t('itemCount', { count: itemCount })}</p>
    </div>
  )
}
```

La llamada es idéntica en ambos contextos — solo cambia cómo obtienes `t` (`getTranslations` vs `useTranslations`).

---

## 3. Variable de interpolación — saludo de bienvenida

```typescript
// Server Component
import { getTranslations } from 'next-intl/server'

export default async function Dashboard({ userName }: { userName: string }) {
  const t = await getTranslations('Auth')

  return <h1>{t('welcome', { name: userName })}</h1>
  // → "Bienvenido, Juan"
}
```

```typescript
// Client Component
'use client'
import { useTranslations } from 'next-intl'

export function WelcomeBanner({ userName }: { userName: string }) {
  const t = useTranslations('Auth')

  return <h1>{t('welcome', { name: userName })}</h1>
  // → "Bienvenido, Juan"
}
```

`{name}` en el JSON se sustituye por el valor del objeto que pasas como segundo argumento. El orden de las palabras puede diferir por idioma — por eso la variable va dentro del string, no concatenada.

---

## 4. Rich text con enlace — `t.rich()`

Este caso NO uses concatenación de strings ni `dangerouslySetInnerHTML`. `t.rich()` es la API correcta:

```typescript
// Server Component
import { getTranslations } from 'next-intl/server'
import Link from 'next/link'  // o el Link de @/i18n/navigation si necesitas i18n en la ruta

export default async function RegisterForm() {
  const t = await getTranslations('Auth')

  return (
    <form>
      {/* otros campos */}
      <p>
        {t.rich('registerTerms', {
          link: (chunks) => (
            <Link href="/terms" className="underline text-blue-600">
              {chunks}
            </Link>
          ),
        })}
      </p>
      {/* → "Al registrarte aceptas los Términos de servicio"
            donde "Términos de servicio" es un <a> real */}
    </form>
  )
}
```

```typescript
// Client Component
'use client'
import { useTranslations } from 'next-intl'
import { Link } from '@/i18n/navigation'

export function TermsNotice() {
  const t = useTranslations('Auth')

  return (
    <p>
      {t.rich('registerTerms', {
        link: (chunks) => (
          <Link href="/terms" className="underline text-blue-600">
            {chunks}
          </Link>
        ),
      })}
    </p>
  )
}
```

### Por qué `t.rich()` y no alternativas

| Enfoque | Problema |
|---|---|
| `"Acepto los " + t('terms')` | Rompe en idiomas donde el orden de palabras es distinto |
| `dangerouslySetInnerHTML` | Vulnerabilidad XSS; el HTML viene de los archivos de mensajes, que podrían modificarse |
| `t.rich()` | Seguro, el componente React lo controla tú; el idioma solo define qué texto va dentro del enlace |

---

## 5. Todo junto — componente de registro completo

```typescript
// components/RegisterPage.tsx (Client Component)
'use client'
import { useTranslations } from 'next-intl'
import { Link } from '@/i18n/navigation'

export function RegisterPage({ userName, cartCount }: { userName: string; cartCount: number }) {
  const tAuth = useTranslations('Auth')
  const tCart = useTranslations('Cart')

  return (
    <div>
      {/* Variable simple */}
      <h1>{tAuth('welcome', { name: userName })}</h1>

      {/* Plural con =0 */}
      <p>{tCart('itemCount', { count: cartCount })}</p>

      {/* Rich text con enlace */}
      <p>
        {tAuth.rich('registerTerms', {
          link: (chunks) => (
            <Link href="/terms" className="text-blue-600 underline">
              {chunks}
            </Link>
          ),
        })}
      </p>
    </div>
  )
}
```

---

## Resumen rápido

```
plural    → {count, plural, =0 {...} one {# artículo} other {# artículos}}
           t('key', { count: n })

variable  → {name} en el JSON
           t('key', { name: 'Juan' })

enlace    → <link>texto</link> en el JSON
           t.rich('key', { link: (chunks) => <a href="...">{chunks}</a> })
```

El caso `=0` es opcional en la especificación ICU pero **muy recomendado para UX** — evita mostrar "Tienes 0 artículos" cuando lo natural es "Tu carrito está vacío".
