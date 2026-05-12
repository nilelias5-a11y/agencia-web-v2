# SEO: "fontanero urgente Madrid" — Guía completa de contenido + Next.js 15

---

## 1. Análisis de intención de búsqueda

**Keyword:** `fontanero urgente Madrid`

**Tipo de intención:** Transaccional con urgencia extrema. El usuario NO está investigando ni comparando: tiene un problema ahora mismo (fuga, tubería rota, inundación) y necesita a alguien que venga lo antes posible. Quiere ver un teléfono, disponibilidad 24h y confirmación de que llegan rápido.

**Formato que responde mejor:** Landing page de conversión (no un blog post). El CTA debe ser el elemento más visible de la página. No es el momento de educar; es el momento de convertir.

**Micro-momentos clave:** "¿Están disponibles ahora?", "¿Cuánto tardan?", "¿Cuánto cuesta?". La página debe resolver estas tres preguntas en el primer scroll.

---

## 2. Keyword Map

| Rol | Keyword |
|---|---|
| Principal | fontanero urgente Madrid |
| Secundaria 1 | fontanero 24 horas Madrid |
| Secundaria 2 | servicio de fontanería urgente Madrid |
| LSI / semántica | reparación de fugas Madrid, desatasco urgente Madrid, fontanero económico Madrid, fontanero a domicilio Madrid |
| Cola larga (FAQ) | cuánto cuesta un fontanero urgente en Madrid, fontanero urgente Madrid precio, fontanero urgente Madrid fin de semana |

---

## 3. Texto completo de la landing page

> **Nota de estructura:** se indica el nivel de encabezado entre corchetes. En el código real, estos se mapean a etiquetas HTML semánticas `<h1>`, `<h2>`, `<h3>`.

---

**[H1] Fontanero Urgente en Madrid — Disponible 24h, llegamos en menos de 60 minutos**

¿Tienes una fuga, una tubería rota o un desatasco que no puede esperar? Somos tu **fontanero urgente en Madrid** de confianza: operamos las 24 horas del día, los 365 días del año, incluidos festivos y fines de semana. Nuestros técnicos certificados están repartidos por toda la ciudad para garantizarte una respuesta en menos de 60 minutos. Llámanos ahora y resolvemos tu emergencia hoy.

📞 **[CTA principal: botón "Llamar ahora — 910 000 000"]**

---

**[H2] ¿Qué problemas resuelve nuestro servicio de fontanería urgente en Madrid?**

Atendemos cualquier avería doméstica o de comunidad sin cita previa:

- Fugas de agua en tuberías vistas o empotradas
- Roturas de tubo y reparación de bajantes
- Desatascos de inodoro, bañera, fregadero y colector general
- Averías en calentadores y termos eléctricos
- Grifería rota o con pérdidas
- Inundaciones y achiques de emergencia

**[H3] Cobertura en toda la Comunidad de Madrid**

Nuestros equipos se ubican estratégicamente en distritos como Centro, Salamanca, Vallecas, Carabanchel, Hortaleza y los principales municipios del área metropolitana (Getafe, Leganés, Alcorcón, Móstoles). Donde estés, llegamos.

---

**[H2] Fontanero 24 horas en Madrid: por qué elegirnos**

1. **Respuesta garantizada en menos de 60 minutos** — trabajamos con guardias de zona para minimizar el tiempo de desplazamiento.
2. **Presupuesto antes de empezar** — sin sorpresas en la factura final.
3. **Técnicos certificados y con seguro de responsabilidad civil** — tu hogar protegido.
4. **Reparación con garantía de 12 meses** — si la avería reaparece, volvemos sin coste adicional.
5. **Precios transparentes** — tarifa de urgencia nocturna y de festivo informada al llamar.

---

**[H2] ¿Cuánto cuesta un fontanero urgente en Madrid?**

El precio de un **servicio de fontanería urgente en Madrid** varía según el tipo de avería, el horario y los materiales necesarios. Como referencia orientativa:

| Tipo de servicio | Rango de precio aproximado |
|---|---|
| Desatasco sencillo | 60 € – 120 € |
| Reparación de fuga en tubería | 80 € – 200 € |
| Sustitución de grifo | 60 € – 150 € |
| Emergencia nocturna (22h–8h) | +30 € sobre tarifa base |
| Festivos y fines de semana | +25 € sobre tarifa base |

> Siempre informamos el precio exacto antes de comenzar el trabajo. Nunca cobramos el desplazamiento si aceptas el presupuesto.

---

**[H2] Preguntas frecuentes sobre fontanería urgente en Madrid**

**[H3] ¿Cuánto tardan en llegar?**

Nuestro tiempo medio de respuesta en Madrid capital es de 45 minutos. En municipios del área metropolitana puede oscilar entre 45 y 90 minutos dependiendo del tráfico y la disponibilidad del técnico más cercano.

**[H3] ¿Trabajan en fin de semana y festivos?**

Sí. El servicio es ininterrumpido: sábados, domingos, festivos nacionales y locales de la Comunidad de Madrid. La tarifa de fin de semana se aplica entre las 14h del sábado y las 8h del lunes.

**[H3] ¿Tienen garantía los trabajos?**

Todos nuestros trabajos de fontanería urgente en Madrid incluyen garantía escrita de 12 meses. Si la avería reaparece por el mismo motivo, regresamos sin cargo por mano de obra.

**[H3] ¿Puedo pagar con tarjeta?**

Sí, aceptamos efectivo, tarjeta de débito/crédito y transferencia bancaria. Emitimos factura en todos los casos.

---

**[H2] Solicita tu fontanero urgente ahora**

No dejes que una avería se convierta en un problema mayor. Cada minuto cuenta cuando hay agua por en medio. Llámanos o rellena el formulario y te llamamos nosotros en menos de 5 minutos.

📞 **[CTA principal: botón "Llamar ahora — 910 000 000"]**
📋 **[CTA secundario: "Solicitar presupuesto rápido"]**

---

## 4. generateMetadata() — Next.js 15 App Router

```typescript
// app/fontanero-urgente-madrid/page.tsx
import { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Fontanero Urgente Madrid 24h — Llegamos en menos de 60 min | TuFontanero',
  description:
    'Fontanero urgente en Madrid disponible 24 horas, todos los días del año. Fugas, desatascos y averías. Llegamos en menos de 60 minutos. Presupuesto gratis.',
  alternates: {
    canonical: 'https://www.tufontanero.es/fontanero-urgente-madrid',
  },
  openGraph: {
    title: 'Fontanero Urgente Madrid 24h — Llegamos en menos de 60 min',
    description:
      'Fontanero urgente en Madrid disponible 24 horas, todos los días del año. Fugas, desatascos y averías. Llegamos en menos de 60 minutos.',
    url: 'https://www.tufontanero.es/fontanero-urgente-madrid',
    type: 'website',
    locale: 'es_ES',
    images: [
      {
        url: 'https://www.tufontanero.es/og/fontanero-urgente-madrid.jpg',
        width: 1200,
        height: 630,
        alt: 'Fontanero urgente Madrid 24 horas',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Fontanero Urgente Madrid 24h — Llegamos en menos de 60 min',
    description:
      'Fontanero urgente en Madrid disponible 24 horas. Fugas, desatascos y averías resueltas hoy.',
    images: ['https://www.tufontanero.es/og/fontanero-urgente-madrid.jpg'],
  },
}
```

> **Por qué `export const metadata` y no `generateMetadata()`:** esta es una página estática (no necesita datos dinámicos en el momento del request), por lo que se usa el objeto `metadata` exportado directamente, que es la forma preferida en Next.js 15 para rutas estáticas. Si la página consumiera datos de CMS o base de datos para construir el título/descripción, se usaría `export async function generateMetadata()`.

**Notas críticas:**

- `alternates.canonical` evita que Google indexe versiones duplicadas (con `?utm_source=...`, paginación, etc.).
- El `title` tiene la keyword al inicio: Google trunca a ~60 caracteres por la derecha.
- La `description` tiene 158 caracteres: dentro del límite de 160. Incluye keyword + urgencia + CTA implícito ("Presupuesto gratis").

---

## 5. JSON-LD — LocalBusiness + FAQPage

Para una web de fontanería local, se combinan dos schemas:

- **`LocalBusiness` (subtipo `Plumber`):** indica a Google que es un negocio físico con área de servicio, horario y teléfono. Habilita el Knowledge Panel y el rich snippet de negocio local.
- **`FAQPage`:** captura las respuestas de la sección FAQ directamente en los resultados de búsqueda (featured snippet expandido).

```typescript
// app/fontanero-urgente-madrid/page.tsx
export default function FontaneroUrgenteMadridPage() {
  const localBusinessJsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Plumber',               // subtipo de LocalBusiness específico para fontaneros
    name: 'TuFontanero Madrid',
    url: 'https://www.tufontanero.es',
    telephone: '+34910000000',
    email: 'urgencias@tufontanero.es',
    description:
      'Fontanero urgente en Madrid disponible 24 horas los 365 días del año. Fugas, desatascos y averías resueltas en menos de 60 minutos.',
    image: 'https://www.tufontanero.es/logo.png',
    priceRange: '€€',
    currenciesAccepted: 'EUR',
    paymentAccepted: 'Cash, Credit Card, Bank Transfer',
    openingHours: 'Mo-Su 00:00-23:59',  // abierto 24h todos los días
    areaServed: {
      '@type': 'City',
      name: 'Madrid',
    },
    address: {
      '@type': 'PostalAddress',
      addressLocality: 'Madrid',
      addressRegion: 'Community of Madrid',
      addressCountry: 'ES',
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: 40.4168,
      longitude: -3.7038,
    },
    sameAs: [
      'https://www.facebook.com/tufontaneromadrid',
      'https://www.instagram.com/tufontaneromadrid',
    ],
    hasOfferCatalog: {
      '@type': 'OfferCatalog',
      name: 'Servicios de fontanería urgente en Madrid',
      itemListElement: [
        {
          '@type': 'Offer',
          itemOffered: {
            '@type': 'Service',
            name: 'Reparación de fugas urgente',
          },
        },
        {
          '@type': 'Offer',
          itemOffered: {
            '@type': 'Service',
            name: 'Desatasco urgente',
          },
        },
        {
          '@type': 'Offer',
          itemOffered: {
            '@type': 'Service',
            name: 'Fontanero 24 horas Madrid',
          },
        },
      ],
    },
  }

  const faqJsonLd = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: [
      {
        '@type': 'Question',
        name: '¿Cuánto tarda en llegar un fontanero urgente en Madrid?',
        acceptedAnswer: {
          '@type': 'Answer',
          text: 'Nuestro tiempo medio de respuesta en Madrid capital es de 45 minutos. En municipios del área metropolitana puede oscilar entre 45 y 90 minutos dependiendo del tráfico.',
        },
      },
      {
        '@type': 'Question',
        name: '¿Cuánto cuesta un fontanero urgente en Madrid?',
        acceptedAnswer: {
          '@type': 'Answer',
          text: 'El precio depende del tipo de avería. Un desatasco sencillo parte desde 60 €; una reparación de fuga, desde 80 €. Las urgencias nocturnas (22h–8h) tienen un suplemento de 30 € sobre la tarifa base. Siempre informamos el precio exacto antes de empezar.',
        },
      },
      {
        '@type': 'Question',
        name: '¿Trabajan en fin de semana y festivos?',
        acceptedAnswer: {
          '@type': 'Answer',
          text: 'Sí. El servicio de fontanería urgente en Madrid es ininterrumpido: sábados, domingos y festivos nacionales y locales de la Comunidad de Madrid.',
        },
      },
      {
        '@type': 'Question',
        name: '¿Tienen garantía los trabajos de fontanería?',
        acceptedAnswer: {
          '@type': 'Answer',
          text: 'Todos los trabajos incluyen garantía escrita de 12 meses. Si la misma avería reaparece, volvemos sin coste adicional de mano de obra.',
        },
      },
    ],
  }

  return (
    <>
      {/* JSON-LD: LocalBusiness (Plumber) */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(localBusinessJsonLd) }}
      />
      {/* JSON-LD: FAQPage */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(faqJsonLd) }}
      />

      {/* Contenido de la página */}
      <main>
        <h1>Fontanero Urgente en Madrid — Disponible 24h, llegamos en menos de 60 minutos</h1>
        {/* ... resto del JSX ... */}
      </main>
    </>
  )
}
```

**Por qué dos scripts JSON-LD:** Google permite múltiples bloques `<script type="application/ld+json">` en la misma página. Separarlos mejora la legibilidad del código y permite validar cada schema de forma independiente en [schema.org/validator](https://validator.schema.org).

---

## 6. Checklist SEO on-page

| # | Check | Estado |
|---|---|---|
| 1 | Keyword principal en H1 | ✅ "Fontanero Urgente en Madrid" en el H1 |
| 2 | Keyword en el primer párrafo (primeras 100 palabras) | ✅ Aparece en la segunda frase del primer párrafo |
| 3 | Keyword en al menos un H2 | ✅ Aparece en "¿Qué problemas resuelve nuestro servicio de fontanería urgente en Madrid?" |
| 4 | Meta description con keyword y CTA, 150-160 caracteres | ✅ 158 caracteres, incluye keyword y "Presupuesto gratis" |
| 5 | Canonical URL declarada | ✅ `alternates.canonical` en generateMetadata |
| 6 | JSON-LD LocalBusiness (Plumber) implementado | ✅ Con teléfono, horario 24h, areaServed y hasOfferCatalog |
| 7 | JSON-LD FAQPage implementado | ✅ 4 preguntas de cola larga alineadas con People Also Ask |
| 8 | Keywords secundarias repartidas naturalmente | ✅ "fontanero 24 horas Madrid", "desatasco urgente", "fontanero a domicilio" |
| 9 | Open Graph completo (título, descripción, imagen, URL, tipo) | ✅ Incluye todos los campos requeridos |
| 10 | Intención de búsqueda no mezclada (solo transaccional) | ✅ Ningún párrafo informativo que distraiga del CTA |

---

## Notas de implementación adicionales

**Ruta recomendada:** `/fontanero-urgente-madrid` (slug con keyword exacta, sin stop words innecesarias).

**Imágenes:** Añade `alt` descriptivos con keyword a las imágenes del equipo y los trabajos: `alt="Fontanero urgente reparando fuga en Madrid"`.

**Core Web Vitals:** Para una landing de conversión con CTA visible, asegúrate de que el LCP (Largest Contentful Paint) sea el H1 o el botón principal, no una imagen pesada. Usa `priority` en el componente `<Image>` de Next.js para la imagen hero.

**Google Business Profile:** El JSON-LD LocalBusiness debe ser consistente con los datos de tu ficha de Google Business Profile (nombre, teléfono, horario). La inconsistencia entre ambas fuentes perjudica la confianza de Google en el negocio.
