# Manejo de Plurales, Variables y Rich Text con next-intl

## Introduccion

`next-intl` usa el estandar **ICU Message Format** para manejar estos tres casos de forma robusta. A continuacion se explica cada uno con ejemplos completos.

---

## 1. Variables simples — "Bienvenido, Juan"

### Archivo de mensajes (`messages/es.json`)

```json
{
  "greeting": "Bienvenido, {name}",
  "userInfo": "Hola {name}, tienes {count} notificaciones nuevas"
}
```

### Uso en componente

```tsx
// app/[locale]/dashboard/page.tsx
import { useTranslations } from 'next-intl';

export default function DashboardPage() {
  const t = useTranslations();
  const userName = "Juan";

  return (
    <div>
      <h1>{t('greeting', { name: userName })}</h1>
      <p>{t('userInfo', { name: userName, count: 5 })}</p>
    </div>
  );
}
```

**Resultado:** `Bienvenido, Juan`

---

## 2. Plurales — "Tienes 3 articulos en el carrito"

El formato ICU para plurales usa la clausula `{variable, plural, ...}`. En espanol necesitas manejar al menos `one` (singular) y `other` (plural), aunque puedes usar `=0` para cero.

### Archivo de mensajes (`messages/es.json`)

```json
{
  "cart": {
    "itemCount": "{count, plural, =0 {No tienes articulos en el carrito} one {Tienes # articulo en el carrito} other {Tienes # articulos en el carrito}}"
  }
}
```

> **Nota:** El simbolo `#` se reemplaza automaticamente por el valor de la variable `count`.

### Uso en componente

```tsx
// app/[locale]/cart/page.tsx
import { useTranslations } from 'next-intl';

interface CartPageProps {
  itemCount: number;
}

export default function CartPage({ itemCount }: CartPageProps) {
  const t = useTranslations('cart');

  return (
    <div>
      <p>{t('itemCount', { count: itemCount })}</p>
    </div>
  );
}
```

**Resultados segun el valor:**
- `count = 0` → `No tienes articulos en el carrito`
- `count = 1` → `Tienes 1 articulo en el carrito`
- `count = 3` → `Tienes 3 articulos en el carrito`

### Plurales en ingles (para comparacion)

```json
{
  "cart": {
    "itemCount": "{count, plural, =0 {Your cart is empty} one {You have # item in your cart} other {You have # items in your cart}}"
  }
}
```

### Plural con seleccion de genero (`select`)

```json
{
  "welcome": "{gender, select, male {Bienvenido} female {Bienvenida} other {Bienvenido/a}}, {name}"
}
```

```tsx
t('welcome', { gender: 'female', name: 'Maria' })
// Resultado: "Bienvenida, Maria"
```

---

## 3. Rich Text con enlaces — "Al registrarte aceptas los Terminos de servicio"

Para insertar elementos HTML o componentes React dentro de las traducciones, `next-intl` usa **rich text** mediante etiquetas personalizadas en el mensaje.

### Archivo de mensajes (`messages/es.json`)

```json
{
  "auth": {
    "termsAgreement": "Al registrarte aceptas los <termsLink>Terminos de servicio</termsLink> y la <privacyLink>Politica de privacidad</privacyLink>",
    "termsSimple": "Al registrarte aceptas los <link>Terminos de servicio</link>"
  }
}
```

### Uso en componente (Client Component)

```tsx
// app/[locale]/register/page.tsx
import { useTranslations } from 'next-intl';
import Link from 'next/link';

export default function RegisterPage() {
  const t = useTranslations('auth');

  return (
    <form>
      {/* Campos del formulario... */}

      <p className="text-sm text-gray-600">
        {t.rich('termsAgreement', {
          termsLink: (chunks) => (
            <Link href="/terminos" className="text-blue-600 underline hover:text-blue-800">
              {chunks}
            </Link>
          ),
          privacyLink: (chunks) => (
            <Link href="/privacidad" className="text-blue-600 underline hover:text-blue-800">
              {chunks}
            </Link>
          ),
        })}
      </p>
    </form>
  );
}
```

**Resultado renderizado:**
> Al registrarte aceptas los [Terminos de servicio](/) y la [Politica de privacidad](/)

### Rich Text en Server Components

```tsx
// app/[locale]/register/page.tsx (Server Component)
import { getTranslations } from 'next-intl/server';
import Link from 'next/link';

export default async function RegisterPage() {
  const t = await getTranslations('auth');

  return (
    <form>
      <p>
        {t.rich('termsAgreement', {
          termsLink: (chunks) => (
            <Link href="/terminos">{chunks}</Link>
          ),
          privacyLink: (chunks) => (
            <Link href="/privacidad">{chunks}</Link>
          ),
        })}
      </p>
    </form>
  );
}
```

### Rich Text con formato HTML basico

```json
{
  "importantNotice": "Este campo es <strong>obligatorio</strong> y debe tener al menos <em>{minLength} caracteres</em>"
}
```

```tsx
t.rich('importantNotice', {
  minLength: 8,
  strong: (chunks) => <strong className="font-bold">{chunks}</strong>,
  em: (chunks) => <em className="italic">{chunks}</em>,
})
```

---

## 4. Combinacion de plurales + variables + rich text

Puedes combinar todos los casos en un solo mensaje:

### Archivo de mensajes

```json
{
  "cart": {
    "summary": "{count, plural, =0 {Tu carrito esta vacio. <link>Ver productos</link>} one {Tienes # articulo. <link>Ir al carrito</link>} other {Tienes # articulos. <link>Ir al carrito</link>}}"
  }
}
```

### Componente

```tsx
import { useTranslations } from 'next-intl';
import Link from 'next/link';

function CartSummary({ count }: { count: number }) {
  const t = useTranslations('cart');

  return (
    <p>
      {t.rich('summary', {
        count,
        link: (chunks) => (
          <Link href="/carrito" className="text-blue-600 underline">
            {chunks}
          </Link>
        ),
      })}
    </p>
  );
}
```

---

## 5. Estructura de archivos recomendada

```
messages/
  es.json          # Espanol (idioma por defecto)
  en.json          # Ingles
  ca.json          # Catalan

i18n/
  routing.ts       # Configuracion de locales y rutas
  request.ts       # Configuracion del servidor

next.config.ts     # Plugin de next-intl
```

### `messages/es.json` completo con todos los casos

```json
{
  "greeting": {
    "welcome": "Bienvenido, {name}",
    "welcomeGendered": "{gender, select, male {Bienvenido} female {Bienvenida} other {Bienvenido/a}}, {name}"
  },
  "cart": {
    "itemCount": "{count, plural, =0 {No tienes articulos en el carrito} one {Tienes # articulo en el carrito} other {Tienes # articulos en el carrito}}",
    "summary": "{count, plural, =0 {Tu carrito esta vacio} one {Tienes # articulo} other {Tienes # articulos}}"
  },
  "auth": {
    "termsAgreement": "Al registrarte aceptas los <termsLink>Terminos de servicio</termsLink> y la <privacyLink>Politica de privacidad</privacyLink>",
    "importantField": "Este campo es <strong>obligatorio</strong>"
  }
}
```

---

## 6. Reglas de pluralizacion por idioma

`next-intl` usa la libreria CLDR internamente, por lo que las reglas de plural son correctas por idioma automaticamente:

| Idioma  | Categorias de plural         |
|---------|------------------------------|
| Espanol | `one`, `other`               |
| Ingles  | `one`, `other`               |
| Arabe   | `zero`, `one`, `two`, `few`, `many`, `other` |
| Ruso    | `one`, `few`, `many`, `other` |
| Catalan | `one`, `other`               |

Solo necesitas definir las categorias que tu idioma utiliza.

---

## 7. Errores comunes y como evitarlos

### Error: usar `t()` en lugar de `t.rich()` para HTML

```tsx
// MAL - renderiza el HTML como texto plano
t('termsAgreement', {
  termsLink: '<a href="/terminos">Terminos</a>'
})

// BIEN - renderiza el componente React correctamente
t.rich('termsAgreement', {
  termsLink: (chunks) => <a href="/terminos">{chunks}</a>
})
```

### Error: olvidar la variable `count` en plurales

```tsx
// MAL - falta la variable count
t('itemCount')

// BIEN
t('itemCount', { count: cartItems.length })
```

### Error: usar `=1` en lugar de `one` para el singular

```json
// PUEDE FUNCIONAR pero no es el estandar ICU recomendado
"{count, plural, =1 {# articulo} other {# articulos}}"

// MEJOR - usar la categoria de plural estandar
"{count, plural, one {# articulo} other {# articulos}}"
```

---

## Resumen rapido

| Caso | Sintaxis ICU | Metodo |
|------|-------------|--------|
| Variable simple | `{name}` | `t('key', { name })` |
| Plural | `{count, plural, one {...} other {...}}` | `t('key', { count })` |
| Seleccion | `{val, select, a {...} b {...} other {...}}` | `t('key', { val })` |
| Rich text / HTML | `<tag>texto</tag>` | `t.rich('key', { tag: (c) => <Tag>{c}</Tag> })` |
| Combinado | Mezcla de los anteriores | `t.rich('key', { count, tag: ... })` |
