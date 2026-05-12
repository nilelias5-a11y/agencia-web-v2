# SEO Local: Posicionar "fontanero urgente Madrid" con Next.js 15

## 1. Análisis de la keyword y estrategia

**Keyword principal:** `fontanero urgente Madrid`
**Intención de búsqueda:** Transaccional + urgente — el usuario necesita un fontanero AHORA, no está buscando información.
**Variantes LSI a incluir:**
- fontanero 24 horas Madrid
- fontanero de urgencia Madrid
- fontanero urgente Madrid barato
- reparación urgente de fontanería Madrid
- fontanero urgente zona [barrio] Madrid (Vallecas, Carabanchel, Chamartín, etc.)
- avería de fontanería Madrid urgente
- servicio urgente fontanero Madrid precio

**Tipo de página:** Esta es una Landing Page de servicio, NO un artículo de blog. Para búsquedas transaccionales urgentes Google prefiere páginas de servicio directas con CTA claros.

---

## 2. Estructura del artículo/landing optimizada

### URL recomendada
```
/servicios/fontanero-urgente-madrid
```

### Estructura H1–H6

```
H1: Fontanero Urgente en Madrid — Disponible 24 Horas, 7 Días a la Semana

  [CTA principal: Llamar ahora / WhatsApp]

H2: ¿Por qué elegirnos como tu fontanero de urgencia en Madrid?
  H3: Respuesta en menos de 60 minutos
  H3: Presupuesto gratuito sin compromiso
  H3: Fontaneros certificados y con experiencia

H2: Servicios de fontanería urgente en Madrid
  H3: Fugas de agua urgentes
  H3: Desatascos de urgencia en Madrid
  H3: Reparación urgente de tuberías rotas
  H3: Averías en calentadores y termos
  H3: Inundaciones y daños por agua

H2: ¿Cuánto cuesta un fontanero urgente en Madrid?
  [Tabla de precios orientativos]

H2: Zonas de Madrid donde operamos
  [Lista de distritos/barrios]

H2: Cómo solicitar un fontanero urgente en Madrid
  [Pasos numerados: 1, 2, 3]

H2: Preguntas frecuentes sobre fontanería urgente en Madrid
  H3: ¿Cuánto tarda en llegar un fontanero urgente en Madrid?
  H3: ¿Trabajáis los fines de semana y festivos?
  H3: ¿Dais garantía por los trabajos realizados?

H2: Opiniones de clientes en Madrid
  [Schema Review]
```

---

## 3. Contenido del artículo completo

A continuación se muestra el contenido real que iría en la página. Está escrito para **conversión y SEO** simultáneamente.

---

### Bloque introductorio (justo debajo del H1)

> **¿Tienes una avería de fontanería en Madrid?** Llámanos ahora y un fontanero urgente estará en tu domicilio en menos de 60 minutos. Atendemos emergencias de fontanería en Madrid las 24 horas del día, los 365 días del año.

**Por qué esta intro funciona:**
- Repite la keyword en las primeras 100 palabras
- Tiene un verbo de acción claro ("Llámanos")
- Incluye el dato de valor principal (60 minutos)

---

### Sección de servicios (con keyword density natural)

Ejemplo de párrafo optimizado:

> Nuestro servicio de **fontanero urgente en Madrid** cubre todo tipo de averías: desde una simple fuga bajo el fregadero hasta una tubería rota que inunda toda la vivienda. Como empresa de **fontanería de urgencia en Madrid**, sabemos que cada minuto cuenta cuando hay agua de por medio.

---

### Tabla de precios (crucial para featured snippets)

| Servicio urgente | Precio orientativo |
|---|---|
| Visita de urgencia (desplazamiento) | 30–60 € |
| Detección de fugas de agua | desde 80 € |
| Desatasco urgente | desde 70 € |
| Reparación tubería rota | desde 90 € |
| Cambio de grifo urgente | desde 60 € |
| Avería calentador urgente | desde 80 € |

*Precios sin IVA. Presupuesto definitivo tras valoración in situ.*

---

### Sección de zonas (SEO local por barrios)

> Ofrecemos servicio de **fontanero urgente en Madrid** en todos los distritos de la ciudad: Centro, Salamanca, Chamartín, Tetuán, Chamberí, Fuencarral, Moncloa, Latina, Carabanchel, Arganzuela, Retiro, Salamanca, Moratalaz, Ciudad Lineal, Hortaleza, Vallecas, Vicálvaro, San Blas, Barajas y Usera.

**Por qué funciona:** Google usa esta sección para posicionar también búsquedas del tipo "fontanero urgente Carabanchel" o "fontanero urgente Vallecas".

---

### FAQ optimizado para PAA (People Also Ask)

Cada H3 de FAQ debe ser **exactamente** una pregunta que aparece en el bloque "La gente también pregunta" de Google para esta keyword.

**¿Cuánto tarda en llegar un fontanero urgente en Madrid?**
> Nuestro tiempo medio de respuesta para urgencias de fontanería en Madrid es de 45 a 60 minutos. El plazo exacto depende de tu ubicación dentro de la ciudad y del tráfico en el momento del aviso.

**¿Trabajáis los fines de semana y festivos en Madrid?**
> Sí. Nuestro servicio de fontanero urgente en Madrid opera los 365 días del año, incluidos fines de semana, festivos locales (San Isidro, etc.) y festivos nacionales. No hay recargo adicional por urgencia nocturna.

**¿Cuánto cuesta llamar a un fontanero urgente en Madrid?**
> La visita de urgencia tiene un coste de desplazamiento de entre 30 y 60 euros dependiendo del horario. Una vez en tu domicilio, te damos un presupuesto cerrado antes de empezar cualquier trabajo.

---

## 4. Implementación en Next.js 15 App Router

### Estructura de archivos

```
app/
  servicios/
    fontanero-urgente-madrid/
      page.tsx          ← La landing page
      layout.tsx        ← Opcional, para breadcrumbs
```

### `app/servicios/fontanero-urgente-madrid/page.tsx`

```tsx
import type { Metadata } from "next";
import { JsonLd } from "@/components/JsonLd"; // componente auxiliar (ver abajo)

// ─── generateMetadata ────────────────────────────────────────────────────────

export async function generateMetadata(): Promise<Metadata> {
  const title = "Fontanero Urgente en Madrid | 24h | Respuesta en 60 min";
  const description =
    "¿Avería de fontanería en Madrid? Fontanero urgente disponible 24 horas, 7 días a la semana. Respuesta en menos de 60 minutos. Presupuesto gratis. ☎ Llama ahora.";
  const canonicalUrl = "https://www.tufontaneria.es/servicios/fontanero-urgente-madrid";

  return {
    // ── Básico ──────────────────────────────────────────────────────────────
    title,
    description,
    keywords: [
      "fontanero urgente Madrid",
      "fontanero 24 horas Madrid",
      "fontanero de urgencia Madrid",
      "fontanería urgente Madrid",
      "fontanero urgente Madrid barato",
    ],

    // ── Canonical ────────────────────────────────────────────────────────────
    alternates: {
      canonical: canonicalUrl,
    },

    // ── Open Graph ──────────────────────────────────────────────────────────
    openGraph: {
      title,
      description,
      url: canonicalUrl,
      siteName: "Tu Fontanería Madrid",
      locale: "es_ES",
      type: "website",
      images: [
        {
          url: "https://www.tufontaneria.es/og/fontanero-urgente-madrid.jpg",
          width: 1200,
          height: 630,
          alt: "Fontanero urgente en Madrid — disponible 24h",
        },
      ],
    },

    // ── Twitter Card ─────────────────────────────────────────────────────────
    twitter: {
      card: "summary_large_image",
      title,
      description,
      images: ["https://www.tufontaneria.es/og/fontanero-urgente-madrid.jpg"],
    },

    // ── Robots ───────────────────────────────────────────────────────────────
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        "max-video-preview": -1,
        "max-image-preview": "large",
        "max-snippet": -1,
      },
    },

    // ── Otros ────────────────────────────────────────────────────────────────
    authors: [{ name: "Tu Fontanería Madrid" }],
    category: "Servicios del hogar",
  };
}

// ─── Schemas JSON-LD ─────────────────────────────────────────────────────────

const localBusinessSchema = {
  "@context": "https://schema.org",
  "@type": "Plumber",
  name: "Tu Fontanería Madrid",
  description:
    "Servicio de fontanero urgente en Madrid, disponible 24 horas al día, 7 días a la semana.",
  url: "https://www.tufontaneria.es",
  telephone: "+34-600-000-000",
  priceRange: "€€",
  address: {
    "@type": "PostalAddress",
    streetAddress: "Calle Ejemplo 1",
    addressLocality: "Madrid",
    postalCode: "28001",
    addressRegion: "Comunidad de Madrid",
    addressCountry: "ES",
  },
  geo: {
    "@type": "GeoCoordinates",
    latitude: 40.4168,
    longitude: -3.7038,
  },
  openingHoursSpecification: [
    {
      "@type": "OpeningHoursSpecification",
      dayOfWeek: [
        "Monday", "Tuesday", "Wednesday",
        "Thursday", "Friday", "Saturday", "Sunday",
      ],
      opens: "00:00",
      closes: "23:59",
    },
  ],
  areaServed: {
    "@type": "City",
    name: "Madrid",
  },
  serviceType: "Fontanería urgente",
  hasOfferCatalog: {
    "@type": "OfferCatalog",
    name: "Servicios de fontanería urgente",
    itemListElement: [
      {
        "@type": "Offer",
        itemOffered: {
          "@type": "Service",
          name: "Reparación urgente de fugas de agua en Madrid",
        },
      },
      {
        "@type": "Offer",
        itemOffered: {
          "@type": "Service",
          name: "Desatascos urgentes en Madrid",
        },
      },
    ],
  },
};

const faqSchema = {
  "@context": "https://schema.org",
  "@type": "FAQPage",
  mainEntity: [
    {
      "@type": "Question",
      name: "¿Cuánto tarda en llegar un fontanero urgente en Madrid?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "Nuestro tiempo medio de respuesta para urgencias de fontanería en Madrid es de 45 a 60 minutos.",
      },
    },
    {
      "@type": "Question",
      name: "¿Trabajáis los fines de semana y festivos en Madrid?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "Sí. Nuestro servicio de fontanero urgente en Madrid opera los 365 días del año, incluidos fines de semana y festivos.",
      },
    },
    {
      "@type": "Question",
      name: "¿Cuánto cuesta llamar a un fontanero urgente en Madrid?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "La visita de urgencia tiene un coste de desplazamiento de entre 30 y 60 euros. Presupuesto cerrado antes de empezar cualquier trabajo.",
      },
    },
  ],
};

// ─── Page Component ──────────────────────────────────────────────────────────

export default function FontaneroUrgenteMadridPage() {
  return (
    <>
      {/* JSON-LD Schemas */}
      <JsonLd data={localBusinessSchema} />
      <JsonLd data={faqSchema} />

      <main>
        {/* ── Hero / H1 ── */}
        <section aria-label="Servicio urgente">
          <h1>Fontanero Urgente en Madrid — 24 Horas, 7 Días a la Semana</h1>
          <p>
            ¿Tienes una avería de fontanería en Madrid? Llámanos ahora y un
            fontanero urgente estará en tu domicilio en{" "}
            <strong>menos de 60 minutos</strong>. Atendemos emergencias de
            fontanería en Madrid los 365 días del año.
          </p>
          <a href="tel:+34600000000" aria-label="Llamar a fontanero urgente Madrid">
            ☎ Llamar ahora — Urgencias 24h
          </a>
        </section>

        {/* ── Por qué elegirnos ── */}
        <section aria-labelledby="por-que">
          <h2 id="por-que">
            ¿Por qué elegirnos como tu fontanero de urgencia en Madrid?
          </h2>
          <h3>Respuesta en menos de 60 minutos</h3>
          <p>
            Contamos con fontaneros distribuidos por toda Madrid para garantizar
            la llegada más rápida posible a tu domicilio.
          </p>
          <h3>Presupuesto gratuito sin compromiso</h3>
          <p>
            Antes de empezar cualquier trabajo, te damos un presupuesto cerrado
            y detallado. Sin sorpresas en la factura final.
          </p>
          <h3>Fontaneros certificados y con experiencia</h3>
          <p>
            Todo nuestro equipo de <strong>fontanería urgente en Madrid</strong>{" "}
            cuenta con formación acreditada y más de 10 años de experiencia en
            el sector.
          </p>
        </section>

        {/* ── Servicios ── */}
        <section aria-labelledby="servicios">
          <h2 id="servicios">Servicios de fontanería urgente en Madrid</h2>
          {/* ... resto de servicios */}
        </section>

        {/* ── Precios ── */}
        <section aria-labelledby="precios">
          <h2 id="precios">¿Cuánto cuesta un fontanero urgente en Madrid?</h2>
          <table>
            <thead>
              <tr>
                <th scope="col">Servicio urgente</th>
                <th scope="col">Precio orientativo</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>Visita de urgencia (desplazamiento)</td>
                <td>30–60 €</td>
              </tr>
              <tr>
                <td>Detección de fugas de agua</td>
                <td>desde 80 €</td>
              </tr>
              <tr>
                <td>Desatasco urgente</td>
                <td>desde 70 €</td>
              </tr>
            </tbody>
          </table>
        </section>

        {/* ── FAQ ── */}
        <section aria-labelledby="faq">
          <h2 id="faq">
            Preguntas frecuentes sobre fontanería urgente en Madrid
          </h2>
          <dl>
            <dt>
              <h3>¿Cuánto tarda en llegar un fontanero urgente en Madrid?</h3>
            </dt>
            <dd>
              Nuestro tiempo medio de respuesta para urgencias de fontanería en
              Madrid es de 45 a 60 minutos. El plazo exacto depende de tu
              ubicación dentro de la ciudad.
            </dd>

            <dt>
              <h3>¿Trabajáis los fines de semana y festivos en Madrid?</h3>
            </dt>
            <dd>
              Sí. Nuestro servicio de fontanero urgente en Madrid opera los 365
              días del año, incluidos fines de semana y festivos. Sin recargo
              adicional por urgencia nocturna.
            </dd>

            <dt>
              <h3>¿Cuánto cuesta llamar a un fontanero urgente en Madrid?</h3>
            </dt>
            <dd>
              La visita de urgencia tiene un coste de desplazamiento de entre 30
              y 60 euros. Presupuesto cerrado antes de empezar cualquier trabajo.
            </dd>
          </dl>
        </section>
      </main>
    </>
  );
}
```

### Componente auxiliar `JsonLd`

Crea este componente en `components/JsonLd.tsx`:

```tsx
// components/JsonLd.tsx
type JsonLdProps = {
  data: Record<string, unknown>;
};

export function JsonLd({ data }: JsonLdProps) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}
```

---

## 5. Checklist SEO completa

### On-page
- [x] Keyword en H1 exacta
- [x] Keyword en los primeros 100 caracteres del body
- [x] Keyword en meta title (primeras palabras)
- [x] Keyword en meta description
- [x] Keyword en URL slug
- [x] Keyword en al menos un H2
- [x] Variantes LSI distribuidas por el contenido
- [x] Alt text de imágenes con keyword
- [x] Tabla de precios (activa featured snippets)
- [x] FAQ con preguntas exactas del PAA de Google

### Schema Markup
- [x] `LocalBusiness` (tipo `Plumber`) con `areaServed: Madrid`
- [x] `FAQPage` schema
- [x] `openingHoursSpecification` 24/7
- [x] `geo` coordinates de Madrid
- [x] `telephone` visible y clickable (tel: link)

### Next.js / Técnico
- [x] `generateMetadata` con title, description, canonical
- [x] Open Graph completo (imagen 1200x630)
- [x] Twitter Card
- [x] `robots` con `max-snippet: -1` para fragmentos enriquecidos
- [x] Canonical URL absoluta
- [x] Imágenes con Next.js `<Image>` con `priority` en el hero

### Core Web Vitals
```tsx
// En el hero image, usar priority para mejorar LCP
import Image from "next/image";

<Image
  src="/fontanero-urgente-madrid.webp"
  alt="Fontanero urgente en Madrid atendiendo una avería"
  width={800}
  height={450}
  priority  // ← mejora LCP directamente
/>
```

---

## 6. Estrategia de link building local

Para reforzar la keyword "fontanero urgente Madrid" necesitas señales locales:

1. **Google Business Profile** — Asegúrate de que la categoría sea "Fontanero" y añade el servicio "Fontanería urgente" con la keyword en la descripción.

2. **Citas NAP consistentes** — Nombre, Dirección y Teléfono idénticos en:
   - Páginas amarillas
   - Yelp España
   - Habitissimo
   - Cronoshare
   - TuCasa.com

3. **Enlaces desde directorios locales de Madrid** — Busca directorios de empresas de Madrid (Ayuntamiento, cámaras de comercio, etc.).

4. **Reviews en Google** — Pide a cada cliente satisfecho que deje una review mencionando "fontanero urgente Madrid" de forma natural.

---

## 7. Medición del resultado

Configura estos eventos en Google Analytics 4 para medir conversiones:

```tsx
// app/servicios/fontanero-urgente-madrid/page.tsx
"use client";

// Tracking de click en CTA de llamada
function trackCallClick() {
  if (typeof window !== "undefined" && window.gtag) {
    window.gtag("event", "generate_lead", {
      event_category: "fontanero_urgente_madrid",
      event_label: "cta_llamada_hero",
      value: 1,
    });
  }
}

// En el JSX:
<a
  href="tel:+34600000000"
  onClick={trackCallClick}
>
  ☎ Llamar ahora
</a>
```

---

## Resumen ejecutivo

| Elemento | Implementación |
|---|---|
| URL | `/servicios/fontanero-urgente-madrid` |
| H1 | Keyword exacta + propuesta de valor |
| Meta title | Keyword al inicio + diferenciador (24h) |
| Meta description | Keyword + beneficio + CTA |
| Schemas | LocalBusiness (Plumber) + FAQPage |
| Contenido | 800–1200 palabras, keyword density ~1.5% |
| CTA principal | Teléfono visible en hero y sticky |
| Zonas | Lista de todos los distritos de Madrid |

El tiempo estimado para aparecer en las primeras posiciones para "fontanero urgente Madrid" es de **3 a 6 meses**, dependiendo de la autoridad de dominio actual del sitio y la estrategia de link building que se implemente paralelamente.
