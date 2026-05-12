# Schema.org/Recipe y Sitemap.xml automatico en Next.js 15

## Introduccion

Para que Google muestre tus recetas como **rich snippets** (con imagen, valoracion, tiempo de preparacion, etc.) necesitas dos cosas:

1. **JSON-LD con schema.org/Recipe** incrustado en cada pagina de receta.
2. **sitemap.xml dinamico** generado automaticamente para que Google descubra todas tus URLs.

A continuacion tienes la implementacion completa, lista para produccion.

---

## 1. Estructura del proyecto asumida

```
app/
  recetas/
    [slug]/
      page.tsx          <- pagina de detalle de receta
  sitemap.ts            <- sitemap dinamico (ruta especial Next.js)
  robots.ts             <- robots.txt (opcional pero recomendado)
lib/
  recipes.ts            <- funciones de acceso a datos
types/
  recipe.ts             <- tipos TypeScript
```

---

## 2. Tipos TypeScript para la receta

```typescript
// types/recipe.ts

export interface RecipeIngredient {
  quantity: string;   // "2 tazas"
  name: string;       // "harina"
}

export interface RecipeStep {
  order: number;
  text: string;
}

export interface Recipe {
  slug: string;
  name: string;
  description: string;
  image: string;           // URL absoluta
  author: string;
  datePublished: string;   // ISO 8601: "2024-03-15"
  dateModified?: string;
  prepTimeMinutes: number;
  cookTimeMinutes: number;
  totalTimeMinutes: number;
  servings: number;
  servingsUnit: string;    // "personas", "porciones"
  category: string;        // "Postre", "Entrante", etc.
  cuisine: string;         // "Italiana", "Española", etc.
  keywords: string[];
  ingredients: RecipeIngredient[];
  steps: RecipeStep[];
  rating?: {
    value: number;         // 4.5
    count: number;         // 128
  };
  calories?: number;
  difficulty: "Facil" | "Media" | "Dificil";
}
```

---

## 3. Funciones de acceso a datos

```typescript
// lib/recipes.ts
import { Recipe } from "@/types/recipe";

// En un proyecto real esto conectaria a tu BD, CMS o ficheros MDX.
// Aqui usamos datos mock para ilustrar la estructura.

const RECIPES: Recipe[] = [
  {
    slug: "pasta-carbonara-autentica",
    name: "Pasta Carbonara Autentica",
    description:
      "La receta clasica italiana de carbonara con guanciale, huevos y pecorino. Sin nata, tal como mandan los canones romanos.",
    image: "https://tu-dominio.com/images/carbonara.jpg",
    author: "Chef Marco",
    datePublished: "2024-01-10",
    dateModified: "2024-03-01",
    prepTimeMinutes: 10,
    cookTimeMinutes: 20,
    totalTimeMinutes: 30,
    servings: 4,
    servingsUnit: "personas",
    category: "Pasta",
    cuisine: "Italiana",
    keywords: ["carbonara", "pasta", "italiana", "guanciale", "pecorino"],
    ingredients: [
      { quantity: "400g", name: "espaguetis" },
      { quantity: "150g", name: "guanciale o panceta" },
      { quantity: "4", name: "yemas de huevo" },
      { quantity: "1", name: "huevo entero" },
      { quantity: "100g", name: "pecorino romano rallado" },
      { quantity: "al gusto", name: "pimienta negra molida" },
    ],
    steps: [
      { order: 1, text: "Cuece la pasta en abundante agua con sal hasta que este al dente." },
      { order: 2, text: "Mientras, saltea el guanciale en una sarten sin aceite hasta que este crujiente." },
      { order: 3, text: "Mezcla las yemas, el huevo entero y el pecorino en un bol. Añade pimienta generosamente." },
      { order: 4, text: "Escurre la pasta reservando un vaso del agua de coccion." },
      { order: 5, text: "Fuera del fuego, mezcla la pasta con el guanciale, añade la mezcla de huevo y remueve rapido añadiendo agua de coccion poco a poco hasta conseguir una salsa cremosa." },
    ],
    rating: { value: 4.8, count: 342 },
    calories: 620,
    difficulty: "Media",
  },
  // Añade mas recetas aqui...
];

export async function getAllRecipes(): Promise<Recipe[]> {
  // En produccion: await db.recipe.findMany() o fetch al CMS
  return RECIPES;
}

export async function getRecipeBySlug(slug: string): Promise<Recipe | null> {
  // En produccion: await db.recipe.findUnique({ where: { slug } })
  return RECIPES.find((r) => r.slug === slug) ?? null;
}
```

---

## 4. Componente JSON-LD para schema.org/Recipe

Este componente genera el bloque de datos estructurados que Google lee.

```typescript
// components/RecipeJsonLd.tsx
import { Recipe } from "@/types/recipe";

function formatDuration(minutes: number): string {
  // Convierte minutos al formato ISO 8601 Duration que requiere schema.org
  // Ejemplo: 90 -> "PT1H30M", 30 -> "PT30M"
  const hours = Math.floor(minutes / 60);
  const mins = minutes % 60;
  let duration = "PT";
  if (hours > 0) duration += `${hours}H`;
  if (mins > 0) duration += `${mins}M`;
  return duration;
}

interface RecipeJsonLdProps {
  recipe: Recipe;
  siteUrl: string;
}

export function RecipeJsonLd({ recipe, siteUrl }: RecipeJsonLdProps) {
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "Recipe",
    name: recipe.name,
    description: recipe.description,
    image: [recipe.image],
    author: {
      "@type": "Person",
      name: recipe.author,
    },
    datePublished: recipe.datePublished,
    ...(recipe.dateModified && { dateModified: recipe.dateModified }),
    prepTime: formatDuration(recipe.prepTimeMinutes),
    cookTime: formatDuration(recipe.cookTimeMinutes),
    totalTime: formatDuration(recipe.totalTimeMinutes),
    recipeYield: `${recipe.servings} ${recipe.servingsUnit}`,
    recipeCategory: recipe.category,
    recipeCuisine: recipe.cuisine,
    keywords: recipe.keywords.join(", "),
    recipeIngredient: recipe.ingredients.map(
      (ing) => `${ing.quantity} ${ing.name}`
    ),
    recipeInstructions: recipe.steps.map((step) => ({
      "@type": "HowToStep",
      position: step.order,
      text: step.text,
    })),
    ...(recipe.rating && {
      aggregateRating: {
        "@type": "AggregateRating",
        ratingValue: recipe.rating.value,
        reviewCount: recipe.rating.count,
        bestRating: 5,
        worstRating: 1,
      },
    }),
    ...(recipe.calories && {
      nutrition: {
        "@type": "NutritionInformation",
        calories: `${recipe.calories} kcal`,
      },
    }),
    url: `${siteUrl}/recetas/${recipe.slug}`,
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd, null, 2) }}
    />
  );
}
```

---

## 5. Metadata dinamica + JSON-LD en la pagina de receta

```typescript
// app/recetas/[slug]/page.tsx
import { Metadata } from "next";
import { notFound } from "next/navigation";
import { getAllRecipes, getRecipeBySlug } from "@/lib/recipes";
import { RecipeJsonLd } from "@/components/RecipeJsonLd";

const SITE_URL = process.env.NEXT_PUBLIC_SITE_URL ?? "https://tu-dominio.com";

interface PageProps {
  params: Promise<{ slug: string }>;
}

// Genera rutas estaticas en build time (SSG)
export async function generateStaticParams() {
  const recipes = await getAllRecipes();
  return recipes.map((recipe) => ({ slug: recipe.slug }));
}

// Genera <title>, <meta description> y Open Graph dinamicamente
export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { slug } = await params;
  const recipe = await getRecipeBySlug(slug);

  if (!recipe) {
    return { title: "Receta no encontrada" };
  }

  return {
    title: `${recipe.name} | Mi Blog de Recetas`,
    description: recipe.description,
    keywords: recipe.keywords,
    openGraph: {
      title: recipe.name,
      description: recipe.description,
      images: [{ url: recipe.image, width: 1200, height: 630, alt: recipe.name }],
      type: "article",
      publishedTime: recipe.datePublished,
      modifiedTime: recipe.dateModified,
    },
    twitter: {
      card: "summary_large_image",
      title: recipe.name,
      description: recipe.description,
      images: [recipe.image],
    },
    alternates: {
      canonical: `${SITE_URL}/recetas/${slug}`,
    },
  };
}

// Componente de pagina
export default async function RecipePage({ params }: PageProps) {
  const { slug } = await params;
  const recipe = await getRecipeBySlug(slug);

  if (!recipe) {
    notFound();
  }

  return (
    <>
      {/* JSON-LD inyectado en el <head> mediante Next.js */}
      <RecipeJsonLd recipe={recipe} siteUrl={SITE_URL} />

      <article>
        <h1>{recipe.name}</h1>
        <p>{recipe.description}</p>

        {/* Datos de tiempo y dificultad */}
        <div className="recipe-meta">
          <span>Preparacion: {recipe.prepTimeMinutes} min</span>
          <span>Coccion: {recipe.cookTimeMinutes} min</span>
          <span>Total: {recipe.totalTimeMinutes} min</span>
          <span>Raciones: {recipe.servings} {recipe.servingsUnit}</span>
          <span>Dificultad: {recipe.difficulty}</span>
        </div>

        {recipe.rating && (
          <div className="recipe-rating">
            Valoracion: {recipe.rating.value}/5 ({recipe.rating.count} valoraciones)
          </div>
        )}

        {/* Ingredientes */}
        <section>
          <h2>Ingredientes</h2>
          <ul>
            {recipe.ingredients.map((ing, i) => (
              <li key={i}>
                <strong>{ing.quantity}</strong> {ing.name}
              </li>
            ))}
          </ul>
        </section>

        {/* Pasos */}
        <section>
          <h2>Elaboracion</h2>
          <ol>
            {recipe.steps.map((step) => (
              <li key={step.order}>{step.text}</li>
            ))}
          </ol>
        </section>
      </article>
    </>
  );
}
```

---

## 6. Sitemap.xml dinamico

Next.js 15 genera el sitemap automaticamente a partir de un fichero `app/sitemap.ts`. Google lo encontrara en `/sitemap.xml`.

```typescript
// app/sitemap.ts
import { MetadataRoute } from "next";
import { getAllRecipes } from "@/lib/recipes";

const SITE_URL = process.env.NEXT_PUBLIC_SITE_URL ?? "https://tu-dominio.com";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const recipes = await getAllRecipes();

  // URLs de las recetas
  const recipeUrls: MetadataRoute.Sitemap = recipes.map((recipe) => ({
    url: `${SITE_URL}/recetas/${recipe.slug}`,
    lastModified: new Date(recipe.dateModified ?? recipe.datePublished),
    changeFrequency: "monthly",
    priority: 0.8,
  }));

  // URLs estaticas del sitio
  const staticUrls: MetadataRoute.Sitemap = [
    {
      url: SITE_URL,
      lastModified: new Date(),
      changeFrequency: "daily",
      priority: 1.0,
    },
    {
      url: `${SITE_URL}/recetas`,
      lastModified: new Date(),
      changeFrequency: "daily",
      priority: 0.9,
    },
    {
      url: `${SITE_URL}/sobre-mi`,
      lastModified: new Date("2024-01-01"),
      changeFrequency: "yearly",
      priority: 0.3,
    },
    {
      url: `${SITE_URL}/contacto`,
      lastModified: new Date("2024-01-01"),
      changeFrequency: "yearly",
      priority: 0.3,
    },
  ];

  return [...staticUrls, ...recipeUrls];
}
```

El fichero generado en `/sitemap.xml` tendra este aspecto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://tu-dominio.com</loc>
    <lastmod>2024-03-15T00:00:00.000Z</lastmod>
    <changefreq>daily</changefreq>
    <priority>1</priority>
  </url>
  <url>
    <loc>https://tu-dominio.com/recetas/pasta-carbonara-autentica</loc>
    <lastmod>2024-03-01T00:00:00.000Z</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <!-- ...mas recetas... -->
</urlset>
```

---

## 7. robots.txt (complemento esencial)

```typescript
// app/robots.ts
import { MetadataRoute } from "next";

const SITE_URL = process.env.NEXT_PUBLIC_SITE_URL ?? "https://tu-dominio.com";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: "/",
        disallow: ["/api/", "/admin/", "/_next/"],
      },
    ],
    sitemap: `${SITE_URL}/sitemap.xml`,
    host: SITE_URL,
  };
}
```

---

## 8. Variable de entorno necesaria

En `.env.local` (desarrollo) y en tu plataforma de despliegue (produccion):

```bash
NEXT_PUBLIC_SITE_URL=https://tu-dominio.com
```

**Importante:** usa siempre la URL con el protocolo (`https://`) y sin barra final.

---

## 9. Verificacion del rich snippet

Una vez desplegado, verifica que todo funciona:

1. **Google Rich Results Test** — https://search.google.com/test/rich-results
   Pega la URL de una receta y comprueba que detecta el tipo `Recipe` sin errores.

2. **Schema.org Validator** — https://validator.schema.org
   Valida el JSON-LD de forma mas detallada.

3. **Google Search Console** — Tras indexar, en "Mejoras" apareceran las recetas con datos estructurados. Puedes ver cuantas tienen errores o advertencias.

---

## 10. Enviar el sitemap a Google

Tras el primer despliegue:

1. Ve a **Google Search Console** > tu propiedad.
2. Seccion **"Sitemaps"**.
3. Introduce: `sitemap.xml`
4. Haz clic en **"Enviar"**.

Google empezara a rastrear el sitemap y a indexar las URLs con sus metadatos de receta.

---

## Resumen de ficheros creados/modificados

| Fichero | Proposito |
|---|---|
| `types/recipe.ts` | Tipos TypeScript de la entidad receta |
| `lib/recipes.ts` | Capa de acceso a datos (BD/CMS/MDX) |
| `components/RecipeJsonLd.tsx` | Componente que inyecta el JSON-LD |
| `app/recetas/[slug]/page.tsx` | Pagina de detalle con metadata + JSON-LD |
| `app/sitemap.ts` | Sitemap.xml dinamico generado por Next.js |
| `app/robots.ts` | robots.txt apuntando al sitemap |

Con esta implementacion, Google podra descubrir todas tus recetas a traves del sitemap y mostrarlas como rich snippets con valoraciones, tiempos de preparacion e imagen destacada en los resultados de busqueda.
