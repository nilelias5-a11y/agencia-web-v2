# Schema.org/Recipe + Sitemap dinámico para blog de recetas en Next.js 15

---

## 1. Análisis de intención de búsqueda

**Intención:** Informacional-técnica. El usuario busca una guía de implementación ("cómo implemento") para dos objetivos concretos: rich snippets de recetas en Google y generación automática del sitemap. El formato adecuado es una guía técnica paso a paso con código listo para copiar.

**Formato que responde mejor:** Tutorial con bloques de código por sección, ordenados de lo más impactante (JSON-LD Recipe → sitemap → OG image) para que el usuario pueda implementar incrementalmente.

---

## 2. Keyword map

| Tipo | Keyword |
|---|---|
| Principal | schema.org Recipe Next.js |
| Secundaria | rich snippets recetas Google |
| LSI | JSON-LD recetas structured data |
| LSI | sitemap.xml Next.js App Router dinámico |
| LSI | generateMetadata receta Next.js |
| LSI | opengraph-image recetas Next.js 15 |

---

## 3. Implementación completa

### Estructura de archivos resultante

```
app/
├── recetas/
│   └── [slug]/
│       ├── page.tsx              ← generateMetadata + JSON-LD Recipe
│       └── opengraph-image.tsx   ← OG image dinámica con ImageResponse
├── sitemap.ts                    ← Sitemap dinámico (lee de DB/API)
└── robots.ts                     ← robots.txt
```

---

### Paso 1 — JSON-LD con @type: Recipe

El tipo `Recipe` de schema.org es la clave para activar los **rich snippets de recetas** en Google: tiempo de preparación, ingredientes, valoraciones y calorías aparecen directamente en los resultados. **No uses `BlogPosting`** para páginas de receta — Google no mostrará el rich result de receta si el tipo no es `Recipe`.

```typescript
// app/recetas/[slug]/page.tsx
import { Metadata } from 'next'
import { getReceta } from '@/lib/recetas'

// ─── generateMetadata ────────────────────────────────────────────────────────

export async function generateMetadata({
  params,
}: {
  params: { slug: string }
}): Promise<Metadata> {
  const receta = await getReceta(params.slug)

  return {
    title: `${receta.nombre} | Blog de Recetas`,
    description: receta.descripcionCorta, // 150-160 caracteres, incluye keyword
    alternates: {
      canonical: `https://tudominio.com/recetas/${params.slug}`,
    },
    openGraph: {
      title: receta.nombre,
      description: receta.descripcionCorta,
      url: `https://tudominio.com/recetas/${params.slug}`,
      type: 'article',
      publishedTime: receta.publicadoEn,
      modifiedTime: receta.actualizadoEn,
      images: [{ url: receta.imagenPrincipal, width: 1200, height: 630 }],
    },
    twitter: {
      card: 'summary_large_image',
      title: receta.nombre,
      description: receta.descripcionCorta,
      images: [receta.imagenPrincipal],
    },
  }
}

// ─── Page component ───────────────────────────────────────────────────────────

export default async function PaginaReceta({
  params,
}: {
  params: { slug: string }
}) {
  const receta = await getReceta(params.slug)

  // JSON-LD con @type: Recipe — activa rich snippets en Google
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Recipe',
    name: receta.nombre,
    description: receta.descripcionCorta,
    image: [receta.imagenPrincipal],
    author: {
      '@type': 'Person',
      name: receta.autor.nombre,
    },
    datePublished: receta.publicadoEn,
    dateModified: receta.actualizadoEn,
    prepTime: receta.tiempoPreparacion, // ISO 8601: "PT15M"
    cookTime: receta.tiempoCoccion,     // ISO 8601: "PT30M"
    totalTime: receta.tiempoTotal,      // ISO 8601: "PT45M"
    recipeYield: `${receta.raciones} personas`,
    recipeCategory: receta.categoria,   // "Postres", "Platos principales"...
    recipeCuisine: receta.cocina,       // "Mediterránea", "Italiana"...
    keywords: receta.etiquetas.join(', '),
    recipeIngredient: receta.ingredientes, // string[]
    recipeInstructions: receta.pasos.map((paso, i) => ({
      '@type': 'HowToStep',
      position: i + 1,
      name: paso.titulo,
      text: paso.descripcion,
      image: paso.imagen ?? undefined,
    })),
    nutrition: receta.nutricion
      ? {
          '@type': 'NutritionInformation',
          calories: `${receta.nutricion.calorias} cal`,
          proteinContent: `${receta.nutricion.proteinas}g`,
          fatContent: `${receta.nutricion.grasas}g`,
          carbohydrateContent: `${receta.nutricion.carbohidratos}g`,
        }
      : undefined,
    aggregateRating: receta.valoraciones?.total > 0
      ? {
          '@type': 'AggregateRating',
          ratingValue: receta.valoraciones.media,
          ratingCount: receta.valoraciones.total,
          bestRating: 5,
          worstRating: 1,
        }
      : undefined,
    video: receta.video
      ? {
          '@type': 'VideoObject',
          name: `Cómo hacer ${receta.nombre}`,
          description: receta.descripcionCorta,
          thumbnailUrl: receta.video.miniatura,
          contentUrl: receta.video.url,
          uploadDate: receta.publicadoEn,
          duration: receta.video.duracion, // ISO 8601: "PT3M20S"
        }
      : undefined,
  }

  return (
    <>
      {/* Script JSON-LD — inyectado en Server Component, nunca en Client Component */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />

      {/* Resto del componente de receta */}
      <article>
        <h1>{receta.nombre}</h1>
        <p>{receta.descripcionCorta}</p>
        {/* ... */}
      </article>
    </>
  )
}
```

**Campos obligatorios para que Google active el rich result de Recipe:**
- `name`
- `image` (array con al menos una imagen de 1200×630 px)
- `recipeIngredient`
- `recipeInstructions`

Los campos opcionales como `aggregateRating`, `nutrition` y `video` amplían el rich snippet pero no son obligatorios para que aparezca.

---

### Paso 2 — Sitemap dinámico en app/sitemap.ts

El sitemap se coloca en `app/sitemap.ts` (NO en `public/sitemap.xml` ni en `pages/`). Next.js genera automáticamente la ruta `/sitemap.xml` a partir de este archivo. La función es `async`, lo que permite leer desde tu base de datos o API en tiempo de build (o bajo demanda si usas ISR).

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next'
import { getAllRecetas } from '@/lib/recetas'
import { getAllCategorias } from '@/lib/categorias'

const BASE_URL = 'https://tudominio.com'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  // Leer recetas desde DB/API en tiempo de build
  const [recetas, categorias] = await Promise.all([
    getAllRecetas(),
    getAllCategorias(),
  ])

  // Entradas para cada receta individual
  const entradasRecetas: MetadataRoute.Sitemap = recetas.map((receta) => ({
    url: `${BASE_URL}/recetas/${receta.slug}`,
    lastModified: new Date(receta.actualizadoEn),
    changeFrequency: 'monthly',
    priority: 0.8,
  }))

  // Entradas para páginas de categoría
  const entradasCategorias: MetadataRoute.Sitemap = categorias.map((cat) => ({
    url: `${BASE_URL}/recetas/categoria/${cat.slug}`,
    lastModified: new Date(),
    changeFrequency: 'weekly',
    priority: 0.7,
  }))

  return [
    // Página de inicio — máxima prioridad
    {
      url: BASE_URL,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 1,
    },
    // Índice de recetas
    {
      url: `${BASE_URL}/recetas`,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.9,
    },
    // Categorías
    ...entradasCategorias,
    // Recetas individuales
    ...entradasRecetas,
  ]
}
```

**Notas clave:**
- `getAllRecetas()` y `getAllCategorias()` son tus funciones de acceso a datos (Prisma, fetch a CMS, etc.). La función `sitemap()` es async, así que pueden ser awaited directamente.
- Si tienes miles de recetas, considera el **sitemap index** (múltiples sitemaps) usando el parámetro `generateSitemaps()` de Next.js 15.
- El archivo se invalida automáticamente cuando revalidas las rutas con `revalidatePath` o `revalidateTag`.

```typescript
// Para blogs con >50.000 URLs: sitemap paginado con generateSitemaps()
// app/sitemap.ts (versión con paginación)
import { MetadataRoute } from 'next'
import { getRecetasPaginadas, contarRecetas } from '@/lib/recetas'

const RECETAS_POR_SITEMAP = 1000

export async function generateSitemaps() {
  const total = await contarRecetas()
  const paginas = Math.ceil(total / RECETAS_POR_SITEMAP)
  return Array.from({ length: paginas }, (_, i) => ({ id: i }))
}

export default async function sitemap({
  id,
}: {
  id: number
}): Promise<MetadataRoute.Sitemap> {
  const recetas = await getRecetasPaginadas({
    skip: id * RECETAS_POR_SITEMAP,
    take: RECETAS_POR_SITEMAP,
  })

  return recetas.map((receta) => ({
    url: `https://tudominio.com/recetas/${receta.slug}`,
    lastModified: new Date(receta.actualizadoEn),
    changeFrequency: 'monthly' as const,
    priority: 0.8,
  }))
}
```

---

### Paso 3 — OG Image dinámica con opengraph-image.tsx

Coloca el archivo en la misma carpeta que `page.tsx`. Next.js lo detecta automáticamente y lo sirve como imagen Open Graph para esa ruta y sus subrutas.

```typescript
// app/recetas/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'
import { getReceta } from '@/lib/recetas'

export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

// Sin reexportar alt desde aquí: Next.js lo coge de generateMetadata
export default async function OgImageReceta({
  params,
}: {
  params: { slug: string }
}) {
  const receta = await getReceta(params.slug)

  // Carga la fuente personalizada (opcional pero recomendado para branding)
  // const fontData = await fetch(new URL('/fonts/Inter-Bold.ttf', import.meta.url)).then(r => r.arrayBuffer())

  return new ImageResponse(
    (
      <div
        style={{
          background: 'linear-gradient(135deg, #1a1a2e 0%, #16213e 60%, #0f3460 100%)',
          width: '100%',
          height: '100%',
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'flex-end',
          padding: '60px 80px',
          position: 'relative',
        }}
      >
        {/* Imagen de fondo de la receta (semitransparente) */}
        {receta.imagenPrincipal && (
          // eslint-disable-next-line @next/next/no-img-element
          <img
            src={receta.imagenPrincipal}
            alt=""
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: '100%',
              objectFit: 'cover',
              opacity: 0.3,
            }}
          />
        )}

        {/* Overlay para legibilidad */}
        <div
          style={{
            position: 'absolute',
            bottom: 0,
            left: 0,
            right: 0,
            height: '70%',
            background: 'linear-gradient(to top, rgba(0,0,0,0.85), transparent)',
          }}
        />

        {/* Categoría — keyword secundaria visible */}
        <p
          style={{
            color: '#f97316',
            fontSize: 22,
            fontWeight: 600,
            margin: '0 0 12px',
            textTransform: 'uppercase',
            letterSpacing: '0.1em',
            position: 'relative',
          }}
        >
          {receta.categoria}
        </p>

        {/* Nombre de la receta — H1 visual */}
        <h1
          style={{
            color: '#ffffff',
            fontSize: receta.nombre.length > 40 ? 52 : 64,
            fontWeight: 700,
            lineHeight: 1.1,
            margin: '0 0 24px',
            maxWidth: '80%',
            position: 'relative',
          }}
        >
          {receta.nombre}
        </h1>

        {/* Metadatos rápidos: tiempo + raciones */}
        <div
          style={{
            display: 'flex',
            gap: 32,
            color: '#cbd5e1',
            fontSize: 20,
            position: 'relative',
          }}
        >
          <span>⏱ {receta.tiempoTotalLegible}</span>
          <span>👥 {receta.raciones} personas</span>
          {receta.valoraciones?.media && (
            <span>⭐ {receta.valoraciones.media.toFixed(1)}</span>
          )}
        </div>

        {/* Logo/marca del blog en esquina superior derecha */}
        <p
          style={{
            position: 'absolute',
            top: 40,
            right: 80,
            color: '#94a3b8',
            fontSize: 18,
            fontWeight: 500,
          }}
        >
          tublogderecetas.com
        </p>
      </div>
    ),
    {
      ...size,
      // fonts: [{ name: 'Inter', data: fontData, weight: 700 }],
    }
  )
}
```

**Por qué `opengraph-image.tsx` y no una imagen estática:**
- La imagen se genera en el servidor con los datos reales de cada receta (nombre, categoría, valoración), así cada receta tiene su propia OG image única.
- No necesitas configurar nada en `generateMetadata` — Next.js detecta el archivo y añade automáticamente los meta tags `og:image` y `twitter:image`.
- El endpoint queda en `/recetas/[slug]/opengraph-image` y es cacheado por Next.js.

---

### Paso 4 — robots.ts (opcional pero recomendado)

```typescript
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/admin/', '/api/', '/draft/'],
      },
    ],
    sitemap: 'https://tudominio.com/sitemap.xml',
  }
}
```

---

## 4. Bloque generateMetadata() listo para copiar

```typescript
export async function generateMetadata({
  params,
}: {
  params: { slug: string }
}): Promise<Metadata> {
  const receta = await getReceta(params.slug)

  return {
    title: `Receta de ${receta.nombre} | Tu Blog de Recetas`,
    description: `Aprende a preparar ${receta.nombre} en ${receta.tiempoTotalLegible}. Receta paso a paso con ingredientes, trucos y valoraciones. ¡Empieza ya!`,
    alternates: {
      canonical: `https://tudominio.com/recetas/${params.slug}`,
    },
    openGraph: {
      title: receta.nombre,
      description: `Receta de ${receta.nombre} — ${receta.tiempoTotalLegible} · ${receta.raciones} personas`,
      url: `https://tudominio.com/recetas/${params.slug}`,
      type: 'article',
      publishedTime: receta.publicadoEn,
      modifiedTime: receta.actualizadoEn,
      // og:image lo gestiona opengraph-image.tsx automáticamente
    },
    twitter: {
      card: 'summary_large_image',
      title: receta.nombre,
      description: `Receta de ${receta.nombre} — ${receta.tiempoTotalLegible}`,
      // twitter:image lo gestiona opengraph-image.tsx automáticamente
    },
  }
}
```

---

## 5. JSON-LD — Tipo correcto: Recipe (no BlogPosting)

| Campo | Por qué es importante | Ejemplo de valor |
|---|---|---|
| `@type: Recipe` | Activa el rich result de receta en Google | `"Recipe"` |
| `image` | Obligatorio para el rich result; mínimo 1200×630 | `["https://…/carbonara.jpg"]` |
| `recipeIngredient` | Google lo muestra en el resultado expandido | `["200g pasta", "3 huevos"]` |
| `recipeInstructions` | HowToStep mejora la comprensión de Google | Array de `HowToStep` |
| `prepTime` / `cookTime` | Se muestran como badges en los resultados | `"PT15M"` / `"PT30M"` |
| `aggregateRating` | Activa las estrellas en los resultados — alto CTR | `{ ratingValue: 4.8, ratingCount: 312 }` |
| `nutrition` | Visible en el panel lateral de Google | `{ calories: "520 cal" }` |

---

## 6. Checklist SEO on-page — 10 puntos

- [x] **@type: Recipe** en el JSON-LD (no BlogPosting, no Article)
- [x] **Imagen de la receta** incluida en `image[]` del JSON-LD con dimensiones >= 1200×630
- [x] **recipeIngredient y recipeInstructions** presentes (campos obligatorios para rich result)
- [x] **Tiempo en ISO 8601** (`PT15M`, `PT45M`) — Google no procesa formatos de texto libre
- [x] **generateMetadata con canonical** declarado explícitamente en cada página de receta
- [x] **Meta description de 150-160 caracteres** con keyword + micro-CTA
- [x] **sitemap.ts en app/** (no en `public/` ni `pages/`) con función `async` que lee de DB
- [x] **opengraph-image.tsx junto a page.tsx** — genera OG image única por receta con `ImageResponse`
- [x] **robots.ts** apunta al sitemap y bloquea `/admin/` y `/api/`
- [x] **JSON-LD en Server Component** — nunca en Client Component (los scripts de LD+JSON deben estar en el HTML inicial para que Google los indexe sin JS)

---

## Verificación post-implementación

1. **Google Rich Results Test**: https://search.google.com/test/rich-results — pega la URL de una receta y verifica que detecta el tipo `Recipe` y todos los campos obligatorios.
2. **Google Search Console** → Mejoras → Recetas: confirma que las páginas están siendo procesadas sin errores de schema.
3. **Sitemap**: accede a `https://tudominio.com/sitemap.xml` directamente en el navegador para confirmar que Next.js lo sirve correctamente.
4. **OG Image**: usa https://opengraph.xyz/ o el debugger de LinkedIn para previsualizar cómo se ve la OG image generada por `opengraph-image.tsx`.
