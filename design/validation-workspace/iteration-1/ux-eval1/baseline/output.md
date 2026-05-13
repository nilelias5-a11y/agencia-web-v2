# Café Vermut — UX Architecture
### Eval 1: Sitemap · Conversion Flow · Homepage Wireframe
**Date:** 2026-05-13
**Brief summary:** Cafetería artesanal barcelonesa en Gràcia. Vende café molido online. Objetivos: atraer clientes locales + ventas online de café. Presupuesto €2.000. 4-5 páginas principales.

---

## PARTE 1 — SITEMAP COMPLETO

### Estructura general

```
/ (Homepage)
├── /tienda
│   ├── /tienda/[slug-producto]
│   └── /tienda/carrito
│       └── /tienda/checkout
│           └── /tienda/confirmacion
├── /cafe
├── /nosotros
└── /contacto
```

### Tabla de páginas principales

| # | Página | URL | Objetivo principal | CTA primario | Prioridad |
|---|--------|-----|--------------------|--------------|-----------|
| 1 | Homepage | `/` | Capturar atención, presentar marca, redirigir tráfico a tienda y local | "Comprar café" (→ /tienda) | CRÍTICA |
| 2 | Tienda / Catálogo | `/tienda` | Mostrar el catálogo de cafés molidos, facilitar elección | "Añadir al carrito" | CRÍTICA |
| 3 | Detalle de producto | `/tienda/[slug]` | Convencer de la compra con información sensorial, origen, notas de cata | "Añadir al carrito" | CRÍTICA |
| 4 | Nuestro café | `/cafe` | Educar sobre origen, proceso de tueste, diferenciadores; reforzar confianza | "Ver la tienda" (→ /tienda) | ALTA |
| 5 | Nosotros | `/nosotros` | Humanizar la marca, conectar emocionalmente con el cliente local | "Visítanos" / mapa | MEDIA |
| 6 | Contacto | `/contacto` | Canal directo para pedidos especiales, consultas, colaboraciones | "Enviar mensaje" | MEDIA |

### Páginas de soporte (flujo de compra, sin navegación principal)

| # | Página | URL | Objetivo | CTA primario | Prioridad |
|---|--------|-----|----------|--------------|-----------|
| 7 | Carrito | `/tienda/carrito` | Revisar pedido antes de pagar | "Ir al checkout" | CRÍTICA |
| 8 | Checkout | `/tienda/checkout` | Recoger datos de envío y pago | "Confirmar y pagar" | CRÍTICA |
| 9 | Confirmación | `/tienda/confirmacion` | Cerrar ciclo, generar confianza postcompra, fidelizar | "Seguir comprando" / "Comparte tu café" | ALTA |

### Notas sobre el sitemap

- Las páginas de carrito, checkout y confirmación no aparecen en la navegación global, pero son parte del flujo crítico.
- La página `/cafe` actúa como puente educativo: convierte visitantes con dudas en compradores informados.
- `/nosotros` refuerza el posicionamiento local (Gràcia, Barcelona) y es clave para SEO local.
- Dado el presupuesto de €2.000, se descarta blog, área de cuenta de usuario y páginas de colecciones múltiples. El catálogo es plano (una sola colección).

---

## PARTE 2 — FLUJO DE CONVERSIÓN PRINCIPAL

### Flujo: Compra online de café molido
**Desde:** Homepage  
**Hasta:** Confirmación de pedido

```
┌─────────────────────────────────────────────────────────────────┐
│  PASO 1 — HOMEPAGE (/)                                          │
│  El usuario llega (tráfico orgánico, redes, boca-a-boca)        │
│                                                                 │
│  Acción esperada: Clic en CTA "Comprar café" en hero section    │
│  o en módulo de producto destacado                              │
│                                                                 │
│  Puntos de fuga: Usuario abandona sin acción                    │
│  Mitigación: Hero directo, carga rápida, prueba social visible  │
└──────────────────────┬──────────────────────────────────────────┘
                       │ CTA "Comprar café" o clic en producto
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASO 2 — CATÁLOGO (/tienda)                                    │
│  El usuario ve los cafés disponibles (2-4 SKUs en fase inicial) │
│                                                                 │
│  Información clave visible: nombre, origen, intensidad,         │
│  precio, formato (250g / 500g), foto del packaging              │
│                                                                 │
│  Acción esperada: Clic en producto para ver detalle             │
│  o "Añadir al carrito" directo desde la tarjeta                 │
│                                                                 │
│  Puntos de fuga: No encuentra lo que busca, precio percibido    │
│  Mitigación: Filtro simple (intensidad), precio visible         │
└──────────────────────┬──────────────────────────────────────────┘
                       │ Clic en producto
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASO 3 — DETALLE DE PRODUCTO (/tienda/[slug])                  │
│  El usuario evalúa el café: origen, notas de cata, proceso      │
│  de tueste, cómo prepararlo                                     │
│                                                                 │
│  Selección de variante: formato (250g / 500g) y molido          │
│  (grano entero / molido para cafetera italiana / molido fino)   │
│                                                                 │
│  Acción esperada: Seleccionar variante + "Añadir al carrito"    │
│                                                                 │
│  Puntos de fuga: Duda sobre el tipo de molido                   │
│  Mitigación: Guía de molido inline (acordeón o tooltip)         │
└──────────────────────┬──────────────────────────────────────────┘
                       │ "Añadir al carrito"
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASO 4 — CARRITO (/tienda/carrito)                             │
│  Mini-resumen del pedido: producto, variante, cantidad, precio  │
│                                                                 │
│  Funciones: modificar cantidad, eliminar artículo,              │
│  ver subtotal + gastos de envío estimados                       │
│                                                                 │
│  Incentivo: Banner "Envío gratis a partir de €25"               │
│  (si el pedido es menor, motiva a añadir más)                   │
│                                                                 │
│  Acción esperada: Clic en "Ir al checkout"                      │
│                                                                 │
│  Puntos de fuga: Reconsideración del precio total               │
│  Mitigación: Cálculo transparente, sin costes ocultos           │
└──────────────────────┬──────────────────────────────────────────┘
                       │ "Ir al checkout"
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASO 5 — CHECKOUT (/tienda/checkout)                           │
│  Formulario en una sola página, pasos lineales:                 │
│                                                                 │
│  5a. Datos de contacto: email, nombre, teléfono (opcional)      │
│  5b. Dirección de envío: calle, número, CP, ciudad              │
│      (autocompletado por CP para agilizar)                      │
│  5c. Método de pago: Stripe Elements (tarjeta) +                │
│      Bizum (si el presupuesto lo permite en v1)                 │
│  5d. Resumen del pedido (sidebar o accordion final)             │
│                                                                 │
│  Indicadores de confianza: candado SSL, logos Visa/Mastercard,  │
│  política de devoluciones resumida (3 líneas)                   │
│                                                                 │
│  Acción esperada: "Confirmar y pagar"                           │
│                                                                 │
│  Puntos de fuga: Fricción en formulario, desconfianza en pago   │
│  Mitigación: Menos campos posible, validación inline, badges    │
└──────────────────────┬──────────────────────────────────────────┘
                       │ Pago procesado con éxito
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  PASO 6 — CONFIRMACIÓN (/tienda/confirmacion)                   │
│  Mensaje de éxito claro: "¡Tu café está en camino!"             │
│                                                                 │
│  Contenido:                                                     │
│  - Número de pedido y resumen                                   │
│  - Email de confirmación enviado a [email]                      │
│  - Plazo de entrega estimado (ej. 2-4 días hábiles)             │
│  - Botón "Seguir comprando" (→ /tienda)                         │
│  - Módulo de redes sociales: "Comparte tu amor por el café"     │
│  - Dirección del local: "¿Estás en Barcelona? Visítanos"        │
│                                                                 │
│  Objetivo secundario: fidelización y amplificación social       │
└─────────────────────────────────────────────────────────────────┘
```

### Métricas clave del flujo (KPIs a trackear desde el día 1)

| Paso | Métrica | Benchmark inicial |
|------|---------|-------------------|
| Homepage → Tienda | CTR del hero CTA | > 15% |
| Tienda → Producto | Click-through a PDP | > 40% |
| Producto → Carrito | Add-to-cart rate | > 25% |
| Carrito → Checkout | Checkout initiation | > 60% |
| Checkout → Confirmación | Checkout completion | > 70% |
| **Conversión global** | Visitas → Compras | **> 2%** |

---

## PARTE 3 — WIREFRAME TEXTUAL DE LA HOMEPAGE

**URL:** `/`  
**Objetivo:** Primera impresión de marca, captura de atención, desvío a tienda y generación de confianza local.  
**Dispositivo de referencia:** Mobile-first (>60% tráfico esperado desde móvil), adaptado a desktop.

---

### [HEADER — Sticky, visible en scroll]

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [Logo "Café Vermut" — tipografía serif, izquierda]                        │
│                                                                            │
│  NAV DESKTOP:  Nuestro café   Tienda   Nosotros   Contacto                │
│                                                    [CTA: Comprar café →]   │
│                                                                            │
│  NAV MOBILE:   [Logo]                          [Icono hamburguesa]         │
│                                                [Icono carrito + badge]     │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Header transparente sobre el hero, se vuelve blanco/crema al hacer scroll.
- El CTA "Comprar café" en desktop usa color acento (terracota o verde oliva).
- Badge del carrito muestra número de ítems cuando hay productos añadidos.
- En mobile, el menú hamburguesa despliega overlay con los mismos links.

---

### [SECCIÓN 1 — HERO]
**Objetivo:** Impacto visual, transmitir esencia de marca artesanal, activar el CTA de compra.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   [Fotografía de pantalla completa — fondo: granos de café sobre mesa     │
│    de madera rústica, luz natural. Tono cálido, artesanal. Sin personas.  │
│    Overlay sutil oscuro para legibilidad del texto]                        │
│                                                                            │
│                                                                            │
│              CAFÉ DE ORIGEN,                                               │
│              MOLIDO EN GRÀCIA.                                             │
│                                                                            │
│         Tueste artesanal cada semana. Enviado a tu puerta.                │
│                                                                            │
│                  [ Comprar café →  ]   [ Ver el local ]                   │
│                   (botón primario)      (link secundario)                  │
│                                                                            │
│                                                                            │
│   [Indicador de scroll: flecha o punto animado hacia abajo]               │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Headline: máximo 5 palabras por línea. Serif bold. Tamaño grande (48-64px desktop, 36px mobile).
- Subhead: una sola frase. Sans-serif regular. Color blanco o crema.
- Botón primario: relleno sólido, color acento. Botón secundario: solo borde (ghost).
- La fotografía es el activo visual más importante. Debe ser propia o licenciada de alta calidad.
- Altura del hero: 90vh en desktop, 70vh en mobile (deja ver el inicio de la siguiente sección).

---

### [SECCIÓN 2 — PROPUESTA DE VALOR / TRES PILARES]
**Objetivo:** Comunicar en 6 segundos qué hace especial a Café Vermut. Responder "¿por qué comprar aquí?".

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│      [ICONO: grano de café]    [ICONO: mano/artesano]    [ICONO: caja]   │
│                                                                            │
│       CAFÉ DE ORIGEN           TUESTE SEMANAL           ENVÍO EN 48H      │
│       ─────────────           ───────────────          ────────────────   │
│       Seleccionamos            Tostamos en              Pedido hoy,        │
│       fincas pequeñas          pequeños lotes           en tu puerta       │
│       de América y             cada semana              pasado mañana.     │
│       África.                  en Gràcia.                                  │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Layout: 3 columnas en desktop, stack vertical en mobile.
- Iconos: línea fina, estilo minimalista. Mismo trazo que la tipografía.
- Fondo: blanco o crema muy claro. Rompe visualmente con el hero oscuro.
- Texto muy corto. Máximo 2-3 líneas por pilar.

---

### [SECCIÓN 3 — PRODUCTOS DESTACADOS]
**Objetivo:** Mostrar 2-3 cafés estrella, generar el primer impulso de compra.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   Nuestros cafés                            [ Ver todos los cafés → ]     │
│                                                                            │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐        │
│  │                  │  │                  │  │                  │        │
│  │  [Foto producto  │  │  [Foto producto  │  │  [Foto producto  │        │
│  │   packaging]     │  │   packaging]     │  │   packaging]     │        │
│  │                  │  │                  │  │                  │        │
│  │  ETIOPÍA         │  │  COLOMBIA        │  │  BLEND VERMUT    │        │
│  │  Yirgacheffe     │  │  Huila           │  │  Casa            │        │
│  │                  │  │                  │  │                  │        │
│  │  Notas: frutos   │  │  Notas: caramelo │  │  Notas: chocolate│        │
│  │  rojos, jazmín   │  │  avellana, cítrico│  │  nuez, ahumado  │        │
│  │                  │  │                  │  │                  │        │
│  │  ●●●○○ Suave     │  │  ●●●●○ Medio    │  │  ●●●●● Intenso  │        │
│  │                  │  │                  │  │                  │        │
│  │  250g — 12,50€   │  │  250g — 13,00€   │  │  250g — 11,00€   │        │
│  │                  │  │                  │  │                  │        │
│  │ [Añadir carrito] │  │ [Añadir carrito] │  │ [Añadir carrito] │        │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘        │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Desktop: 3 tarjetas en fila. Tablet: 2 tarjetas. Mobile: carrusel deslizable (swipe).
- La foto de packaging debe ser de alta calidad, fondo neutro o contexto de cafetería.
- El indicador de intensidad (puntos rellenos/vacíos) hace la comparación visual inmediata.
- "Añadir al carrito" directo desde la tarjeta reduce pasos. Opcionalmente, clic en tarjeta va al PDP.
- "Ver todos los cafés →" es el enlace a `/tienda`.

---

### [SECCIÓN 4 — HISTORIA / MARCA]
**Objetivo:** Conectar emocionalmente con el cliente local. Humanizar. Reforzar el "comprar local".

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   [Fotografía — interior del local: barra de madera, luz de tarde,        │
│    tal vez una persona preparando café. Tono cálido, íntimo]              │
│                                       │                                    │
│   [TEXTO — lado izquierdo o abajo]    │  [FOTO — lado derecho o arriba]   │
│                                       │                                    │
│   Desde 2018 en el                    │                                    │
│   corazón de Gràcia.                  │                                    │
│                                       │                                    │
│   Empezamos como una pequeña          │                                    │
│   cafetería de barrio con una         │                                    │
│   obsesión: encontrar los mejores     │                                    │
│   cafés del mundo y traerlos a tu     │                                    │
│   taza.                               │                                    │
│                                       │                                    │
│   Hoy tostamos, molemos y             │                                    │
│   enviamos desde el mismo             │                                    │
│   local donde tomamos el              │                                    │
│   primer café del día.                │                                    │
│                                       │                                    │
│   [ Conoce nuestra historia → ]       │                                    │
│   (enlaza a /nosotros)                │                                    │
│                                       │                                    │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Layout: 50/50 en desktop (foto + texto), stack vertical en mobile (foto arriba, texto abajo).
- Texto en serif o sans-serif legible, tamaño cuerpo 16-18px, line-height amplio.
- El CTA secundario "Conoce nuestra historia →" es un link de texto o botón ghost.
- Fondo: color crema o terracoterizo muy claro, diferenciado de la sección anterior.

---

### [SECCIÓN 5 — PRUEBA SOCIAL / RESEÑAS]
**Objetivo:** Reducir la fricción de compra con validación externa. Especialmente importante para clientes que llegan por primera vez.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   Lo que dicen nuestros clientes                                           │
│                                                                            │
│   ★★★★★  4.9 de 5  (127 reseñas)                                         │
│                                                                            │
│  ┌─────────────────────────┐  ┌─────────────────────────┐                │
│  │  ★★★★★                 │  │  ★★★★★                 │                │
│  │                         │  │                         │                │
│  │  "El mejor café que he  │  │  "Llevo 6 meses         │                │
│  │  tomado en casa. El de  │  │  pidiendo el Etiopía.   │                │
│  │  Etiopía Yirgacheffe    │  │  El envío siempre en    │                │
│  │  es una maravilla."     │  │  48h y el café fresco   │                │
│  │                         │  │  de verdad."            │                │
│  │  — Marta G., Barcelona  │  │  — Jordi P., Madrid     │                │
│  └─────────────────────────┘  └─────────────────────────┘                │
│                                                                            │
│   [Opcional: logo Google con enlace a perfil de Google My Business]       │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Mostrar 2 reseñas en desktop, 1 en mobile (con indicador para deslizar).
- La puntuación global (4.9/5) debe ser prominente. Genera confianza inmediata.
- Incluir nombre y ciudad del cliente, no solo iniciales. Mayor credibilidad.
- Las reseñas deben ser reales. Importar desde Google My Business si es posible.

---

### [SECCIÓN 6 — BANNER ENVÍO / INCENTIVO]
**Objetivo:** Romper la última barrera de compra: el coste de envío.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   [Fondo color acento — terracota, verde oliva o negro carbón]            │
│                                                                            │
│              ENVÍO GRATIS EN PEDIDOS A PARTIR DE 25€                      │
│                                                                            │
│         Recibe tu café en casa en 48 horas laborables.                    │
│                                                                            │
│                      [ Comprar ahora →  ]                                 │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Sección estrecha (banner), no es una sección de contenido completa.
- Texto centrado, tamaño grande para el headline del umbral de envío gratuito.
- CTA con alto contraste sobre el color de fondo.
- Posicionada antes del footer, es el último empuje hacia la compra.

---

### [SECCIÓN 7 — VISITA EL LOCAL (componente local SEO)]
**Objetivo:** Capturar a clientes de Gràcia / Barcelona que prefieren el canal físico. Refuerza SEO local.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│   También estamos en Gràcia                                                │
│                                                                            │
│   [Mapa estático (imagen de Google Maps o Mapbox) con pin]                │
│                                                                            │
│   Carrer de [nombre], [número]     Lunes a viernes: 8:00 – 19:00          │
│   08012 Barcelona                  Sábado: 9:00 – 15:00                   │
│   (Barrio de Gràcia)               Domingo: cerrado                       │
│                                                                            │
│   [ Cómo llegar →  ]   [ Llamar: 93X XXX XXX ]                           │
│   (abre Google Maps)   (tel: href)                                         │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- El mapa puede ser estático (imagen) para no cargar el script de Google Maps y mejorar performance.
- El número de teléfono debe ser un enlace `tel:` para que mobile lo marque directamente.
- "Cómo llegar →" abre Google Maps en una nueva pestaña con la dirección preconfigurada.
- Contribuye al posicionamiento SEO local de "cafetería Gràcia Barcelona".

---

### [FOOTER]
**Objetivo:** Navegación secundaria, información legal, contacto rápido.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  [Logo Café Vermut]                                                        │
│  Café de especialidad. Tueste artesanal. Gràcia, Barcelona.               │
│                                                                            │
│  NAVEGAR              LEGAL                  SÍGUENOS                     │
│  ──────────           ──────────             ──────────                   │
│  Tienda               Política de privacidad  [Instagram]  [Facebook]     │
│  Nuestro café         Aviso legal                                          │
│  Nosotros             Política de cookies                                  │
│  Contacto             Términos y condiciones                               │
│                                                                            │
│  ─────────────────────────────────────────────────────────────────────    │
│  © 2026 Café Vermut · Todos los derechos reservados                       │
│  Diseño web por [Agencia]                                                  │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

**Notas de diseño:**
- Fondo oscuro (negro carbón o marrón oscuro), texto claro. Contrasta con el resto de la página.
- En mobile: layout en columna única, links apilados.
- Los iconos de redes sociales enlazan a los perfiles reales. Si no existen aún, omitir.
- El banner de cookies (RGPD) aparece sobre el footer en la primera visita.

---

## RESUMEN DE DECISIONES UX

| Decisión | Justificación |
|----------|---------------|
| 4 páginas principales + flujo de compra separado | Mantiene el proyecto dentro del presupuesto de €2.000 sin sacrificar funcionalidad crítica |
| Catálogo plano (sin subcategorías) | En fase inicial con 2-4 SKUs no se necesita taxonomía compleja |
| Checkout en una sola página | Reduce pasos y fricción; más conversiones que checkout multi-step |
| Envío gratis desde €25 | Incentiva pedidos de 2 unidades (2 × 250g o 1 × 500g). Mejora AOV |
| Hero con CTA doble (comprar + ver local) | Cubre los dos perfiles de usuario: comprador online y cliente local |
| Prueba social en homepage | Crítico para e-commerce nuevo sin historial. Importar reseñas de Google My Business desde el día 1 |
| Mapa estático (no embed) | Evita la penalización de performance de Google Maps. Mejora Core Web Vitals |
| Mobile-first | Estimado >60% de tráfico desde móvil para este perfil de negocio |

---

*Documento generado como baseline para Eval 1 — Café Vermut UX Architecture.*
*Este documento es el punto de partida para comparar con variantes generadas en iteraciones posteriores.*
