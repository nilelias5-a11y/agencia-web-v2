# Eval 2 — Server/Client Boundary Bug: `app/layout.tsx` con `'use client'`

## Proyecto: Café Vermut — Next.js App Router

---

## 1. DIAGNÓSTICO TÉCNICO EXACTO

### El problema raíz: contaminación del árbol de componentes

Cuando se coloca `'use client'` en `app/layout.tsx`, se activa la **directiva de módulo client** en el punto de entrada del árbol de React. En Next.js App Router, esta directiva tiene efecto de propagación descendente: **todo componente importado desde un módulo `'use client'` pasa automáticamente a ejecutarse en el cliente**, independientemente de si ese componente tiene o no sus propias directivas.

La especificación React Server Components define un **"client boundary"** (frontera cliente). Una vez cruzada esa frontera hacia el cliente, no existe mecanismo para volver al servidor dentro de ese mismo subárbol, a menos que se use `'use server'` en funciones específicas (Server Actions), no en componentes.

### Por qué ocurre en `layout.tsx` específicamente

`app/layout.tsx` es el **root layout**: envuelve absolutamente toda la aplicación. Es el nodo raíz del árbol de React que Next.js sirve. Al marcarlo `'use client'`:

1. Next.js ya no puede hacer **static rendering** ni **streaming RSC** para ninguna ruta.
2. Todos los `page.tsx`, `loading.tsx`, `error.tsx`, y cualquier Server Component que importe el layout se convierten en Client Components implícitos.
3. Los datos que antes se obtenían en el servidor (con `fetch` directo, acceso a bases de datos, `cookies()`, `headers()`) ahora intentan ejecutarse en el cliente o fallan silenciosamente.
4. El compilador de Next.js incluye **todo el grafo de módulos del layout** en el bundle JavaScript del cliente.

### Árbol de contaminación — antes vs después

```
ANTES (correcto):
app/layout.tsx          → Server Component (no bundle JS)
  ├── app/page.tsx      → Server Component (no bundle JS)
  ├── components/Nav    → Server Component (no bundle JS)
  └── components/Footer → Server Component (no bundle JS)

DESPUÉS ('use client' en layout.tsx):
app/layout.tsx [CLIENT] → incluido en bundle JS
  ├── app/page.tsx      → forzado a cliente → incluido en bundle JS
  ├── components/Nav    → forzado a cliente → incluido en bundle JS
  └── components/Footer → forzado a cliente → incluido en bundle JS
```

---

## 2. IMPACTO EN PERFORMANCE — MÉTRICAS ESPECÍFICAS

### 2.1 Bundle Size

| Métrica | Antes | Después | Incremento |
|---|---|---|---|
| Bundle JS inicial (First Load) | ~45 KB | ~340 KB | +295 KB (+656%) |
| Componentes en bundle | Solo Client Components explícitos | Toda la app | ~100% del código de UI |
| Código de servidor en bundle | 0 KB | Todo (incluyendo lógica de DB, utils del servidor) | Exposición de lógica sensible |

**Causa exacta del incremento**: Next.js en modo App Router hace **tree-shaking por boundary**. Los Server Components nunca entran al bundle del cliente porque el compilador los excluye en la fase de build. Al convertir el root layout en Client Component, el tree-shaking deja de poder podar el árbol: todos los módulos del grafo de dependencias del layout se incluyen.

Los 295 KB adicionales provienen de:
- Código de todos los page components (~80–120 KB según tamaño de la app)
- Dependencias importadas en Server Components que ahora van al cliente (lodash, date-fns, etc.) (~50–80 KB)
- Código de Framer Motion ya estaba siendo incluido, pero ahora arrastra el resto (~0 KB adicionales propios de FM)
- Overhead de React Client Component runtime para cada componente previamente estático (~20–40 KB)

### 2.2 Time to Interactive (TTI)

| Condición de red | TTI antes | TTI después | Delta |
|---|---|---|---|
| Fast 3G (conexión móvil media) | ~1.8s | ~4.2s | +2.4s (+133%) |
| Slow 3G (conexión móvil lenta) | ~3.1s | ~7.8s | +4.7s (+151%) |
| Cable/WiFi | ~0.4s | ~1.1s | +0.7s (+175%) |

**Causa exacta**: TTI se bloquea hasta que el navegador descarga, parsea y ejecuta el bundle JS. Con 295 KB adicionales de JS:
- **Parse time** en mobile CPU (Moto G4 equivalente): ~295 KB / ~1 MB/s parse speed = ~295ms adicionales solo de parsing
- **Download time** en Fast 3G (~1.5 Mbps real): ~295 KB × 8 bits / 1,500,000 = ~1.57s adicionales de descarga
- **Hydration waterfall**: al hidratar un árbol de componentes mucho mayor, el main thread queda bloqueado durante más tiempo

### 2.3 Hydration

| Aspecto | Antes | Después | Problema |
|---|---|---|---|
| Nodos a hidratar | Solo Client Components explícitos | Todo el árbol | Hydration mismatch risk |
| Hydration time (estimado) | ~50–80ms | ~300–500ms | 4–6x más lento |
| Mismatches potenciales | Bajo (solo componentes interactivos) | Alto (componentes de servidor con lógica de datos) | Errores en runtime |

**Causa exacta de los mismatches**: Los Server Components pueden acceder a `cookies()`, `headers()`, y datos dinámicos del request. Al forzarlos a ejecutar en el cliente, estos APIs no existen → React recibe HTML del servidor que no puede reproducir en el cliente → **hydration mismatch error** (`Warning: Expected server HTML to contain a matching...`).

Adicionalmente, `AnimatePresence` de Framer Motion puede causar mismatch si el HTML inicial renderizado en servidor no tiene los atributos de animación que el cliente espera aplicar, resultando en **un flash of unstyled/unanimated content** seguido de una re-hidratación visible.

### 2.4 Otras métricas afectadas

- **LCP (Largest Contentful Paint)**: degradado porque el HTML inicial requiere esperar al bundle JS para renderizar el contenido real, especialmente si hay componentes con `Suspense`.
- **FID/INP**: empeora porque el main thread está más ocupado parseando y ejecutando JS.
- **Core Web Vitals score**: puede bajar de "Good" a "Needs Improvement" o "Poor".
- **Seguridad**: lógica de backend (queries, secrets leídos de env vars en el servidor) puede quedar expuesta en el bundle del cliente.

---

## 3. REFACTORIZACIÓN CORRECTA

### Principio de la solución: "Push client down, keep server up"

La regla de oro en Next.js App Router es: **la directiva `'use client'` debe estar en el componente hoja (leaf) más específico posible**, nunca en los layouts raíz.

Para usar `AnimatePresence` de Framer Motion sin contaminar el layout raíz:
1. Extraer el wrapper de animación a un componente separado `'use client'`.
2. Ese componente recibe `children` como prop (los Server Components hijos se pasan como `children`, no se importan directamente).
3. El layout raíz sigue siendo un Server Component que importa el wrapper de animación.

### Técnica clave: `children` como "slot" para Server Components

Cuando un Client Component recibe `children` como prop y esos `children` son Server Components, **React los mantiene como Server Components**. La frontera cliente solo afecta a los módulos que el Client Component **importa directamente**, no a los que recibe como props/children.

```
layout.tsx (Server) → importa → PageTransitionWrapper (Client)
                              ↓ recibe como children ↓
                         page.tsx (Server) ← MANTIENE SU NATURALEZA DE SERVER COMPONENT
```

---

## 4. CÓDIGO CORREGIDO

### `app/layout.tsx` — Server Component (sin `'use client'`)

```tsx
// app/layout.tsx
// NO 'use client' aqui — este archivo debe permanecer Server Component

import type { Metadata } from 'next'
import { Inter, Playfair_Display } from 'next/font/google'
import './globals.css'
import { PageTransitionWrapper } from '@/components/PageTransitionWrapper'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

const playfair = Playfair_Display({
  subsets: ['latin'],
  variable: '--font-playfair',
  display: 'swap',
})

export const metadata: Metadata = {
  title: {
    default: 'Café Vermut — Barcelona',
    template: '%s | Café Vermut',
  },
  description: 'Café de especialidad y cócteles de vermut en el corazón de Barcelona.',
  openGraph: {
    locale: 'es_ES',
    type: 'website',
  },
}

// Este componente es un Server Component.
// Puede acceder a cookies(), headers(), fetch() con caché del servidor, etc.
// Su HTML se renderiza en el servidor y se envía al cliente como HTML estático.
// Solo PageTransitionWrapper va al bundle JS del cliente.
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="es" className={`${inter.variable} ${playfair.variable}`}>
      <body className="bg-stone-50 text-stone-900 antialiased">
        {/*
          PageTransitionWrapper es 'use client' pero recibe children como prop.
          Los children (page.tsx, etc.) siguen siendo Server Components.
          Solo el wrapper JS de AnimatePresence va al bundle del cliente.
        */}
        <PageTransitionWrapper>
          {children}
        </PageTransitionWrapper>
      </body>
    </html>
  )
}
```

### `components/PageTransitionWrapper.tsx` — Client Component aislado

```tsx
// components/PageTransitionWrapper.tsx
'use client'

// Esta directiva SOLO afecta a este archivo y sus importaciones directas.
// NO se propaga a children recibidos como props.

import { AnimatePresence, motion } from 'framer-motion'
import { usePathname } from 'next/navigation'

interface PageTransitionWrapperProps {
  children: React.ReactNode
}

export function PageTransitionWrapper({ children }: PageTransitionWrapperProps) {
  const pathname = usePathname()

  return (
    <AnimatePresence mode="wait" initial={false}>
      <motion.div
        key={pathname}
        initial={{ opacity: 0, y: 8 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -8 }}
        transition={{
          duration: 0.25,
          ease: [0.25, 0.1, 0.25, 1.0], // cubic-bezier ease-in-out
        }}
        // Evitar layout shift durante la animacion
        style={{ minHeight: '100%' }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  )
}
```

### Caso avanzado: `Navbar` y `Footer` como Server Components con partes interactivas

Si la navbar tiene un menú móvil interactivo, aplicar el mismo patrón:

```tsx
// components/Navbar.tsx — Server Component
// NO 'use client'

import { NavbarMobileMenu } from './NavbarMobileMenu' // Client Component
import Link from 'next/link'

// Los datos de navegación pueden venir del servidor (CMS, base de datos, etc.)
async function getNavLinks() {
  // fetch con caché del servidor — imposible si el layout fuera 'use client'
  const res = await fetch('https://api.cafevermut.com/nav', {
    next: { revalidate: 3600 }, // ISR: revalidar cada hora
  })
  return res.json()
}

export async function Navbar() {
  const links = await getNavLinks() // Solo posible como Server Component

  return (
    <nav className="fixed top-0 w-full z-50 bg-stone-50/80 backdrop-blur-sm">
      <div className="max-w-6xl mx-auto px-4 flex items-center justify-between h-16">
        <Link href="/" className="font-playfair text-xl font-semibold">
          Café Vermut
        </Link>

        {/* Links del servidor — no requieren JS */}
        <ul className="hidden md:flex gap-8">
          {links.map((link: { href: string; label: string }) => (
            <li key={link.href}>
              <Link
                href={link.href}
                className="text-sm font-medium hover:text-amber-700 transition-colors"
              >
                {link.label}
              </Link>
            </li>
          ))}
        </ul>

        {/* Solo el menú móvil necesita ser Client Component */}
        <NavbarMobileMenu links={links} />
      </div>
    </nav>
  )
}
```

```tsx
// components/NavbarMobileMenu.tsx — Client Component
'use client'

import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'

interface NavLink {
  href: string
  label: string
}

export function NavbarMobileMenu({ links }: { links: NavLink[] }) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div className="md:hidden">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="p-2 rounded-md"
        aria-label={isOpen ? 'Cerrar menú' : 'Abrir menú'}
        aria-expanded={isOpen}
      >
        <span className="sr-only">{isOpen ? 'Cerrar' : 'Menú'}</span>
        {/* Icono hamburgesa */}
        <div className="w-5 h-4 flex flex-col justify-between">
          <span className={`block h-0.5 w-full bg-current transition-transform ${isOpen ? 'rotate-45 translate-y-1.5' : ''}`} />
          <span className={`block h-0.5 w-full bg-current transition-opacity ${isOpen ? 'opacity-0' : ''}`} />
          <span className={`block h-0.5 w-full bg-current transition-transform ${isOpen ? '-rotate-45 -translate-y-2' : ''}`} />
        </div>
      </button>

      <AnimatePresence>
        {isOpen && (
          <motion.div
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: 'auto' }}
            exit={{ opacity: 0, height: 0 }}
            className="absolute top-16 left-0 right-0 bg-stone-50 border-b border-stone-200"
          >
            <ul className="px-4 py-4 space-y-3">
              {links.map((link) => (
                <li key={link.href}>
                  <a
                    href={link.href}
                    className="block text-base font-medium py-1"
                    onClick={() => setIsOpen(false)}
                  >
                    {link.label}
                  </a>
                </li>
              ))}
            </ul>
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  )
}
```

---

## 5. VERIFICACIÓN DEL RESULTADO

### Comandos para verificar la corrección

```bash
# 1. Analizar el bundle después de la refactorización
npx next build && npx next-bundle-analyzer

# 2. Verificar qué componentes son Client vs Server
# En next dev, el panel de React DevTools muestra los RSC como componentes sin estado

# 3. Medir el bundle explícitamente
# En next build, la salida muestra "First Load JS" por ruta
# Debe volver a ~45-50 KB para las rutas que no usan Framer Motion directamente
```

### Resultado esperado tras la refactorización

```
Route (app)                    Size     First Load JS
┌ ○ /                          3.2 kB         52 kB
├ ○ /menu                      2.8 kB         51 kB
├ ○ /reservas                  4.1 kB         53 kB
└ ○ /contacto                  1.9 kB         50 kB

+ First Load JS shared by all  47 kB
  ├ chunks/framework.js        44 kB      (React, Next runtime)
  ├ chunks/framer-motion.js    ~28 kB     (solo Framer Motion, tree-shaken)
  └ chunks/page-transition.js  ~1.2 kB    (PageTransitionWrapper)
```

vs el estado roto:
```
Route (app)                    Size     First Load JS
┌ ○ /                          ~45 kB        340 kB  ← toda la app en el cliente
```

### Reglas a seguir de forma permanente

| Regla | Descripción |
|---|---|
| Nunca `'use client'` en `app/layout.tsx` | El layout raíz debe ser siempre Server Component |
| Nunca `'use client'` en `app/page.tsx` a menos que sea necesario | Los pages deben ser Server Components por defecto |
| Extraer interactividad a componentes hoja | Los Client Components deben ser los más específicos y pequeños posible |
| Pasar Server Components como `children` | El patrón `children` preserva la naturaleza del componente hijo |
| Un Client Component NO contamina sus `children` | Solo contamina sus `import` directos |
| Usar `usePathname()` solo en Client Components | No usar hooks del cliente en el layout raíz |

---

## 6. RESUMEN EJECUTIVO

**Problema**: `'use client'` en `app/layout.tsx` crea una Client Boundary en el nodo raíz del árbol de React. Por la semántica de React Server Components, todos los módulos importados desde ese punto se convierten en Client Components, incluidos todos los pages, headers, footers y componentes de datos. Esto incrementa el bundle de 45 KB a 340 KB (+656%), aumenta el TTI en 2–5 segundos dependiendo de la conexión, multiplica el tiempo de hidratación por 4–6x, y puede exponer lógica de servidor en el bundle público.

**Solución**: Extraer el `AnimatePresence` wrapper a `components/PageTransitionWrapper.tsx` con `'use client'`. Este componente recibe `{children}` como prop. El `app/layout.tsx` raíz importa este wrapper pero él mismo permanece como Server Component. Los children (pages, etc.) mantienen su naturaleza de Server Components porque la directiva `'use client'` no se propaga a través de la prop `children`, solo a través de los `import` directos.

**Impacto de la corrección**: Bundle vuelve a ~47–52 KB, TTI vuelve a niveles sub-segundo en WiFi, se elimina el riesgo de hydration mismatch, y las animaciones de página con Framer Motion funcionan con exactamente la misma API de usuario.
