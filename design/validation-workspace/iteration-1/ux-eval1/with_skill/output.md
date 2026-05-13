# UX Architecture — Café Vermut
**Prepared by:** UX Designer  
**Date:** 2026-05-13  
**Brief:** Cafetería artesanal barcelonesa. Vende café molido online. Objetivos: atraer clientes locales + ventas online de café molido. Presupuesto: €2.000. Alcance: 4-5 páginas principales.

---

## PHASE 1 — Sitemap

### Sitemap — Café Vermut

---

#### Primary Navigation

| URL | Page Name | User Goal | Primary CTA | Priority |
|-----|-----------|-----------|-------------|----------|
| `/` | Home | Entender qué es Café Vermut, generar deseo de visitar y/o comprar | "Compra café online" | Critical |
| `/cafe` | Nuestra Carta / El Café | Explorar la oferta de cafés molidos disponibles, elegir variedad | "Añadir al carrito" | Critical |
| `/tienda` | Tienda Online | Comprar café molido (producto principal de conversión económica) | "Comprar ahora" | Critical |
| `/nosotros` | Nuestra Historia | Entender el origen, los valores y el por qué de Café Vermut; construir confianza | "Descubre nuestro café" → `/tienda` | Important |
| `/visita` | Visítanos | Localizar el café, ver horarios, planificar la visita física | "Cómo llegar" (Google Maps) | Important |

---

#### Secondary Navigation (sub-páginas de tienda)

| URL | Page Name | User Goal | Primary CTA | Priority |
|-----|-----------|-----------|-------------|----------|
| `/tienda/[slug]` | Página de Producto | Leer descripción detallada, tasting notes, origen y decidir compra | "Añadir al carrito" | Critical |
| `/tienda/carrito` | Carrito | Revisar selección antes de pagar | "Finalizar compra" | Critical |
| `/tienda/checkout` | Checkout | Completar datos de envío y pago | "Pagar" | Critical |

---

#### Footer Navigation

| URL | Page Name | Priority |
|-----|-----------|----------|
| `/nosotros` | Historia & Valores | Important |
| `/visita` | Visítanos / Horarios | Important |
| `/politica-privacidad` | Política de Privacidad | Supplementary |
| `/terminos` | Términos y Condiciones | Supplementary |
| `/contacto` | Contacto | Supplementary |

---

#### Mobile Navigation

- Hamburger menu (top-right) que abre un **drawer overlay full-screen** con links grandes (min. 48px tap target)
- Orden de prioridad móvil reordenado respecto a desktop:
  1. Tienda Online (conversión primaria)
  2. El Café (producto)
  3. Visítanos (local intent — usuarios móviles con alta intención de visita física)
  4. Nuestra Historia
- CTA sticky en móvil: botón "Comprar café" visible en la parte inferior de la pantalla en las páginas Home, Nosotros y Visítanos
- El carrito muestra badge con número de ítems en el header (visible siempre)

---

#### Hidden / Utility Pages

| URL | Page Name | Purpose |
|-----|-----------|---------|
| `/pedido-confirmado` | Confirmación de Pedido | Pantalla de éxito post-compra con resumen y upsell |
| `/404` | Not Found | Página de error con navegación de recuperación |
| `/contacto/gracias` | Formulario enviado | Confirmación de formulario de contacto |

---

### Justificación de arquitectura

- **5 páginas principales** (Home, Carta/Café, Tienda, Nosotros, Visítanos) caben dentro del presupuesto de €2.000 con desarrollo en Next.js App Router + Stripe para pagos.
- La tienda está separada de la carta porque la **carta** cumple función de inspiración/descubrimiento (contenido editorial), mientras que la **tienda** es 100% funcional/transaccional. Esto reduce la fricción en cada objetivo.
- `/visita` existe como página independiente porque la atracción de clientes locales es un objetivo de negocio declarado. Una página con mapa, horarios y fotos del local tiene peso SEO local independiente.
- No se incluye blog en MVP (presupuesto limitado), pero la URL `/blog` queda reservada para iteración futura.

---

## PHASE 2 — Conversion Flows

---

### Conversion Flow 1 — Compra Online de Café Molido (Primary)

**Trigger:** Usuario llega desde búsqueda orgánica ("café molido barcelona", "comprar café molido online"), Instagram, o referral directo  
**Goal:** Completar una compra en la tienda online (pedido confirmado)

---

**Step 1: Home (`/`)**  
→ El usuario ve el Hero con imagen del café molido + propuesta de valor clara ("Café de origen, molido a mano en Barcelona")  
→ CTA principal: "Compra café online" → redirige a `/tienda`  
→ Decisión: ¿Quiero saber más antes de comprar? → sección de featured products en Home actúa como shortcut directo a `/tienda/[slug]`  

**Step 2: Tienda (`/tienda`)**  
→ El usuario ve el catálogo de cafés molidos disponibles (tarjetas con foto, nombre, origen, precio, botón "Ver más")  
→ Filtra/explora por: tipo de tostado, origen, intensidad  
→ Decisión: ¿Quiero leer más sobre este café? → click en tarjeta → `/tienda/[slug]`  
→ Decisión rápida: Usuario con alta intención puede hacer "Añadir al carrito" directamente desde la tarjeta (sin entrar al producto)  

**Step 3: Página de Producto (`/tienda/[slug]`)**  
→ El usuario lee: nombre, origen, tasting notes, peso disponible (250g / 500g), proceso de tostado, historia del productor  
→ Selecciona: variante de peso + molido (grano entero / molido para espresso / molido para filtro)  
→ CTA: "Añadir al carrito" → mini-drawer de carrito aparece desde la derecha  
→ Decisión: ¿Continuar comprando? → botón "Seguir comprando" en el drawer → vuelve a `/tienda`  

**Step 4: Carrito (`/tienda/carrito`)**  
→ El usuario ve: lista de productos, cantidades editables, precio por producto, subtotal, coste de envío estimado, total  
→ Upsell suave: "Los clientes que compraron esto también compraron..." (1-2 productos relacionados)  
→ CTA: "Finalizar compra" → `/tienda/checkout`  
→ Decisión: ¿Cambio de opinión? → botón eliminar por ítem, campo de cupón descuento  

**Step 5: Checkout (`/tienda/checkout`)**  
→ Formulario en un solo paso (o dos pasos: datos de envío → pago):  
  - Paso 1: Nombre, email, dirección de envío  
  - Paso 2: Datos de tarjeta (Stripe Elements embebido), revisión final del pedido  
→ CTA: "Pagar [total]€" → procesa pago vía Stripe  
→ Decisión: ¿Tienes cuenta? → opción de login/registro opcional (no obligatorio — no aumenta fricción)  

**Step 6: Confirmación (`/pedido-confirmado`)**  
→ El usuario ve: número de pedido, resumen del pedido, fecha estimada de entrega, email de confirmación enviado  
→ CTA secundario: "Sigue explorando nuestros cafés" → `/tienda` (reinicia ciclo)  
→ CTA local: "Visítanos en el café" → `/visita` (cross-convert a cliente físico)  

---

**Drop-off risks:**  
- **Tienda → Producto:** Foto de baja calidad o precio sin contexto de valor → mitigar con tasting notes visibles en tarjeta y badge "Lo más vendido"  
- **Producto → Carrito:** Duda sobre variantes de molido (¿qué elijo?) → mitigar con tooltip/guía de molido visible junto al selector  
- **Carrito → Checkout:** Coste de envío descubierto en el último momento → mostrar envío estimado desde la ficha de producto y en el carrito  
- **Checkout → Pago:** Desconfianza en formulario de pago de marca desconocida → mitigar con logo de Stripe, sello SSL visible, política de devolución en 30 días visible en el checkout  

**Recovery mechanisms:**  
- Carrito persistente (cookies o localStorage) — el usuario que abandona y vuelve encuentra sus productos  
- Email de carrito abandonado (si se implementa Klaviyo/Mailchimp en iteración v2)  
- Sticky "Ver carrito (X)" badge en header en toda la tienda  

---

### Conversion Flow 2 — Visita al Local (Secondary / Local Intent)

**Trigger:** Usuario local en Barcelona busca "café artesanal Gràcia", "mejores cafeterías Barcelona", o llega por recomendación de alguien  
**Goal:** Planificar y realizar una visita física al café

---

**Step 1: Home (`/`)**  
→ Sección "Encuéntranos" en la parte baja del Home: foto del local, dirección, horarios resumidos  
→ CTA: "Ver cómo llegar" → `/visita`  

**Step 2: Visítanos (`/visita`)**  
→ El usuario ve: mapa de Google Maps embebido, dirección exacta, horarios completos por día, fotos del espacio interior/exterior, transporte público cercano  
→ CTA: "Cómo llegar" (abre Google Maps en nueva pestaña con ruta)  
→ Elemento de confianza: 2-3 reviews de Google destacadas  

**Drop-off risks:** Muy pocos — usuario con intención local ya tiene alta motivación. El único riesgo es información desactualizada (horarios incorrectos).  
**Recovery:** Número de teléfono/WhatsApp visible en `/visita` para consultas directas.  

---

### Conversion Flow 3 — Re-entry (Visitante recurrente que no compró)

**Trigger:** Usuario que visitó la Home o la Tienda pero no compró. Vuelve desde bookmark, Instagram, o búsqueda directa de marca.  
**Goal:** Convertir en la segunda o tercera visita

---

**Step 1: Home (`/`) o `/tienda`**  
→ El carrito persistente muestra badge con ítems previos → reduce fricción de re-inicio  
→ Featured products en Home rotan (si hay CMS) para mostrar algo nuevo  

**Step 2: `/tienda`**  
→ Si el usuario no añadió al carrito antes, posible ruta: scroll hacia abajo en Home → ver sección "Nuestros cafés destacados" → click → `/tienda/[slug]` directo  

**Step 3:** Igual que Flow 1 desde Step 3 en adelante  

**Recovery mechanisms:**  
- Newsletter signup en el footer (email + botón) — ofrece "10% de descuento en tu primer pedido" como incentivo  
- Stories/Reels de Instagram con link directo a producto específico  

---

## PHASE 3 — Wireframe Textual — Homepage (`/`)

---

## Homepage — `/`

**Page purpose:** Capturar la atención del usuario en los primeros 3 segundos, comunicar la propuesta de valor de Café Vermut (café artesanal barcelonés), y dirigir a la tienda online o generar intención de visita.  
**Primary CTA:** "Compra café online" → `/tienda`  
**Entry sources:** Búsqueda orgánica, Instagram/redes sociales, Google Maps (tráfico local), referral directo  

---

### Section 1 — Hero

**Layout:** Full-width (100vw), imagen o video de fondo, contenido centrado o alineado a la izquierda en la mitad inferior del frame

**Content:**
- **Headline:** ~40-55 caracteres, tono evocador + local. Ejemplo: *"Café de origen. Molido en Gràcia desde el primer día."* — debe transmitir artesanía, localismo y autenticidad en una línea
- **Subheadline:** 1 frase (máx. 80 caracteres). Ejemplo: *"Compra nuestros cafés molidos o visítanos en Barcelona."* — captura ambos objetivos de negocio
- **CTA primario:** Botón filled, tamaño L → "Compra café online" → `/tienda`
- **CTA secundario:** Botón ghost/outline → "Visítanos" → `/visita`
- **Visual de fondo:** Fotografía de alta calidad: taza de café en una mesa de madera con luz natural de cafetería, o close-up de granos siendo molidos. Sin stock photos genéricos. Overlay oscuro sutil (opacity ~40%) para contraste del texto.
- **Indicador de scroll:** Flecha animada hacia abajo o "scroll" en la parte inferior del hero, desaparece al primer scroll.

**Mobile behavior:**
- La imagen de fondo hace crop al 60% superior (foco en el café, no en el fondo)
- Headline se reduce a 1 línea si es posible (evitar saltos raros)
- Los dos CTAs se apilan verticalmente (CTA primario arriba, ghost abajo), ambos de ancho completo
- El hero ocupa 90vh en mobile para que el usuario perciba que hay contenido debajo
- Sticky CTA "Comprar café" aparece en el footer fijo del móvil desde esta sección en adelante

**Animation hint:** Fade-in suave del texto (0.3s delay tras carga de imagen). La imagen de fondo puede tener un muy ligero zoom-in (Ken Burns, 8s) para dar vida. Nada que demore la carga (no autoplay video en móvil).

**Conversion notes:** El Hero define si el usuario se queda o se va. La doble CTA captura los dos perfiles: comprador online y visitante local. El fondo fotográfico (no ilustración) genera credibilidad artesanal inmediata. La subheadline elimina ambigüedad sobre qué es el negocio.

---

### Section 2 — Propuesta de Valor / Trust Strip

**Layout:** Centrado, 3 columnas en desktop (iconos + texto corto), fondo neutro claro (diferencia visual del Hero)

**Content:**
- **3 micro-propuestas con icono:**
  1. Icono grano de café → "Café de origen único" — de productores seleccionados en Ethiopia, Colombia y Guatemala
  2. Icono reloj/calendar → "Tostado a la semana" — molido y enviado en menos de 7 días tras el tostado
  3. Icono caja/envío → "Envío en 48h a toda España" — pedidos antes de las 12h, salen el mismo día
- Máximo 15 palabras por columna (headline + 1 frase explicativa)
- Sin botones en esta sección — es puramente informativa/trust

**Mobile behavior:**
- Las 3 columnas se apilan en una sola columna (icon + texto en fila horizontal por ítem)
- Padding generoso entre ítems para respiración visual
- Si el espacio es muy reducido, esta sección puede colapsarse a 2 iconos + texto en 2 columnas (quitar el menos diferencial)

**Animation hint:** Fade-in con stagger de 0.15s entre cada columna al entrar en viewport.

**Conversion notes:** Esta sección reduce la fricción de compra respondiendo las 3 preguntas que un comprador online se hace: ¿es café bueno? ¿es fresco? ¿cómo llega?. Idealmente aparece en el primer scroll, antes de que el usuario necesite buscar esta información.

---

### Section 3 — Featured Products (Cafés Destacados)

**Layout:** Grid 3 columnas en desktop, con encabezado de sección. Fondo blanco o muy ligero.

**Content:**
- **Headline de sección:** "Nuestros cafés" (H2, tipografía editorial, no bold forzado)
- **Subheadline:** "Tres orígenes, tres historias. Todos tostados esta semana." (~60 chars)
- **3 tarjetas de producto (Featured):** Una por cada café destacado
  - Cada tarjeta: foto del producto (bolsa de café), nombre del café, origen (país + región), perfil de sabor (2-3 descriptores: "notas de chocolate, frambuesa, caramelo"), precio base (ej. "desde 12€ / 250g"), CTA "Ver más" → `/tienda/[slug]`
  - Tarjeta puede tener badge: "Lo más vendido" o "Nuevo" para jerarquía visual
- **CTA de sección:** Botón centrado debajo del grid → "Ver toda la tienda" → `/tienda`

**Mobile behavior:**
- Grid cambia a 1 columna (scroll vertical de tarjetas)
- Alternativa: carrusel horizontal con snap scroll (recomendado para mobile UX — permite descubrir sin scroll infinito)
- La foto ocupa la mitad superior de la tarjeta en mobile, descripción debajo
- Badge de precio siempre visible en la tarjeta
- CTA "Ver toda la tienda" se mantiene, ancho completo

**Animation hint:** Las 3 tarjetas entran con slide-up + fade al hacer scroll a la sección. Stagger de 0.1s entre tarjetas.

**Conversion notes:** Esta sección actúa como shortcut para usuarios con alta intención de compra que no quieren explorar toda la tienda. Los descriptores de sabor (tasting notes) en la tarjeta dan contexto de valor y diferencian el producto de café genérico de supermercado. El botón "Ver toda la tienda" captura a los exploradores.

---

### Section 4 — Historia / About Teaser

**Layout:** Split 50-50 en desktop (imagen izquierda, texto derecha). Fondo ligeramente cálido (crema/off-white).

**Content:**
- **Imagen izquierda:** Foto del/la barista o fundador/a detrás del mostrador, o manos moliendo café. Tono íntimo, no publicitario.
- **Contenido derecho:**
  - **Eyebrow label:** "Nuestra historia" (pequeño, uppercase, tracking amplio)
  - **Headline:** ~50 chars. Ejemplo: *"Un café que vale la pena levantarse temprano."*
  - **Párrafo:** 3-4 líneas máximo. La historia en una sola idea: quiénes son, por qué empezaron, qué les obsesiona del café. No biografía completa — gancho emocional.
  - **CTA:** Texto-link o botón ghost → "Conoce nuestra historia" → `/nosotros`

**Mobile behavior:**
- Split se colapsa a stack: imagen arriba (crop cuadrado, full-width), texto abajo
- El párrafo puede recortarse a 2 líneas con "Leer más" si el texto es largo
- El CTA se mantiene visible siempre

**Animation hint:** La imagen entra con ligero scale (1.05 → 1.0) al hacer scroll. El texto hace fade-in desde la derecha. Suave, no dramático.

**Conversion notes:** Esta sección sirve a los usuarios que necesitan confianza antes de comprar a una marca desconocida. El tono es cercano e informal (coherente con "café de barrio barcelonés"). No es obligatoria para la conversión directa, pero reduce el abandono de usuarios en fase de consideración.

---

### Section 5 — Localizador / Visítanos en el Café

**Layout:** Full-width dividido en 2 partes: mapa estático (o iframe de Google Maps embebido) en la mitad, y panel de información en la otra mitad. Fondo oscuro (contraste con sección anterior).

**Content:**
- **Panel de información:**
  - **Eyebrow:** "Encuéntranos"
  - **Headline:** "En el corazón de Gràcia" (o la dirección específica si está definida)
  - **Dirección:** Calle, número, Barcelona — con enlace "Ver en Google Maps" (abre en nueva pestaña)
  - **Horarios:** Tabla o lista simple por días. Ej: Lun-Vie 8:00-18:00 / Sáb 9:00-14:00 / Dom cerrado
  - **Texto breve:** 1 frase de invitación. Ej: *"Pásate a probar el café antes de comprarlo."*
  - **CTA:** Botón filled → "Cómo llegar" → abre Google Maps con ruta
- **Mapa:** Google Maps embebido (iframe) o imagen estática del mapa con pin. Si se usa iframe: cargado con lazy loading para no penalizar performance.

**Mobile behavior:**
- El mapa va arriba (ancho completo, altura fija de ~200px)
- El panel de información va debajo
- Horarios en formato compacto (lista, no tabla)
- El botón "Cómo llegar" es ancho completo y muy visible (usuarios móviles con alta intención de visita usan esta función activamente)

**Animation hint:** Fade-in del panel de texto al entrar en viewport. Mapa sin animación (carga funcional).

**Conversion notes:** Usuarios locales en móvil son el perfil más probable de usar esta sección. El botón "Cómo llegar" debe abrir la app de Google Maps nativa en móvil (intent URL: `https://maps.google.com/?q=...` o `maps://` deeplink). Esta sección también tiene valor SEO local si los horarios y la dirección están en HTML semántico (no solo en el iframe de Google Maps).

---

### Section 6 — Social Proof / Reviews

**Layout:** Centrado, fondo blanco. 3 reviews en fila (desktop) o carrusel (mobile).

**Content:**
- **Headline:** "Lo que dicen nuestros clientes" (H2, discreto)
- **3 reviews:** Texto de review real (extraído de Google o Tripadvisor), nombre del reviewer, estrella rating (5/5). Máximo 3 líneas de texto por review.
  - Ejemplo de tonos: "El mejor flat white de Gràcia, sin duda.", "Compré el café molido online y llegó en 2 días perfecto.", "Sitio acogedor, gente amable, café de verdad."
  - Nota: incluir reviews que cubran ambos tipos de cliente (online + presencial)
- **CTA:** Texto-link → "Ver más opiniones en Google" → Google Business profile (nueva pestaña)

**Mobile behavior:**
- Carrusel horizontal con snap (1 review visible a la vez)
- Indicadores de posición (dots) debajo del carrusel
- Swipe gesture habilitado

**Animation hint:** Fade-in del bloque completo al entrar en viewport.

**Conversion notes:** Las reviews son el cierre de confianza antes del CTA final de compra. Fundamental incluir al menos 1 review que mencione el proceso de compra online (envío, frescura del café) porque el usuario que está considerando comprar online necesita prueba social de ese canal específico.

---

### Section 7 — CTA Final / Newsletter

**Layout:** Full-width, fondo de color de marca (café oscuro o verde botella, contraste alto). Contenido centrado.

**Content:**
- **Headline:** ~40 chars. Ejemplo: *"¿Aún no has probado nuestro café?"*
- **Subheadline:** 1 línea. Ejemplo: *"Consigue un 10% de descuento en tu primer pedido."*
- **CTA primario:** Botón grande, color contrastante → "Compra ahora" → `/tienda`
- **CTA secundario / Newsletter:**
  - Campo de email (placeholder: "tu@email.com") + botón "Quiero el descuento"
  - Nota legal mini: "Sin spam. Solo cuando tostamos algo especial." (reduce fricción de signup)
  - On submit: mensaje inline de éxito + código de descuento en pantalla (y por email)

**Mobile behavior:**
- Headline y subheadline centrados, una línea cada uno
- Los dos CTAs se apilan: botón "Compra ahora" arriba (full-width), formulario email debajo
- El campo de email debe ser de tipo `email` para abrir teclado correcto en iOS/Android

**Animation hint:** Slide-up del contenido al entrar en viewport.

**Conversion notes:** Esta sección cierra la visita con dos opciones: conversión directa (compra) o conversión blanda (email). El incentivo del 10% descuento activa al usuario que estuvo considerando pero no decidió. Es la última oportunidad antes del footer.

---

### Section 8 — Footer

**Layout:** Fondo oscuro (más oscuro que la sección anterior, o igual si ya es dark). Grid de 3-4 columnas en desktop.

**Content:**
- **Columna 1 — Marca:**
  - Logo de Café Vermut
  - Tagline corto: "Café artesanal desde Gràcia, Barcelona."
  - Iconos de redes sociales: Instagram, posiblemente TikTok
- **Columna 2 — Navegación:**
  - Links a todas las páginas principales (Home, Café, Tienda, Nosotros, Visítanos)
- **Columna 3 — Info práctica:**
  - Dirección
  - Horarios (versión muy compacta)
  - Email de contacto o link a `/contacto`
- **Columna 4 — Legal:**
  - Política de Privacidad
  - Términos y Condiciones
  - Aviso de Cookies (si aplica RGPD)
- **Línea inferior:** Copyright © 2026 Café Vermut. Todos los derechos reservados.

**Mobile behavior:**
- Las 4 columnas se colapsan en acordeones expandibles (tap para expandir cada sección)
- Alternativa más simple: stack vertical sin acordeón, con separadores visuales
- Los iconos de redes sociales son siempre visibles sin colapsar
- El footer sticky CTA ("Comprar café") desaparece en el footer para no duplicar

**Animation hint:** Sin animación — el footer es funcional, no debe competir visualmente.

**Conversion notes:** El footer es navegación de rescate y trust signal. La dirección física en el footer refuerza la credibilidad de la marca y contribuye al SEO local. Mantenerlo limpio y legible.

---

**Page Notes — Homepage:**
- El sticky CTA móvil ("Comprar café") debe desaparecer cuando el usuario está sobre la sección Hero (donde ya hay un CTA) para evitar redundancia visual, y reaparecer después.
- El carrito en el header debe mostrar un badge numérico en tiempo real con los ítems añadidos. Este elemento es siempre visible en el header (no solo en la tienda) para recordar al usuario que tiene productos seleccionados.
- Cookie consent banner (RGPD): aparece al primer visit, fijo en la parte inferior de la pantalla, no ocupa más del 15% de la pantalla en mobile. No bloquea el contenido.
- Performance note: las imágenes del Hero y de Featured Products deben estar en formato WebP con `srcset` responsivo. El iframe de Google Maps debe cargarse con `loading="lazy"`.
- Accessibility: la imagen de fondo del Hero debe tener un `alt` vacío (decorativa) y el texto del overlay debe cumplir WCAG AA (ratio 4.5:1 mínimo). El `skip to main content` link debe ser el primer elemento tabulable de la página.

---

## PHASE 4 — Global Interaction Principles

---

### Navigation Behavior

- **Desktop:** Header sticky — sólido al hacer scroll (si se empieza transparente sobre el Hero, se vuelve opaco y con sombra ligera a los 80px de scroll)
- **Mobile:** Hamburger icon (top-right) que abre un drawer full-screen overlay con animación slide-from-right. El overlay incluye todos los links de navegación principal + CTA "Comprar café" en la parte inferior del drawer.
- **Active state:** Link de la sección actual con subrayado de color de marca o punto debajo del label. En mobile drawer, el link activo aparece en negrita + color de marca.
- **Scroll progress:** No — no se usa barra de progreso de scroll (interfiere con el estilo artesanal/editorial del sitio).
- **Cart badge:** Siempre visible en el header (icono carrito + número de ítems). Visible tanto en desktop como en mobile. Al añadir un ítem, el badge hace una micro-animación de scale (bounce) para confirmar la acción.

---

### Scroll Behavior

- **Smooth scroll:** Sí — para anchor links internos (ej. "Ver cómo llegar" en Home hace scroll suave a la sección del localizador)
- **Section snapping:** No — el scroll libre es más natural para un site de contenido editorial/e-commerce. El snapping puede frustrar a usuarios que quieren leer a su propio ritmo.
- **Lazy loading:** Imágenes y componentes de mapa. Los componentes above-the-fold (Hero) nunca son lazy. Todo lo que esté por debajo del fold inicial se carga con `loading="lazy"` o Intersection Observer.
- **Back-to-top:** Sí — aparece después de 3 scroll heights (cuando el usuario ha scrolleado significativamente). Icono flotante en la esquina inferior derecha, con z-index por debajo del sticky CTA móvil para no superponerse.

---

### Forms

- **Validación:** On blur (valida cada campo cuando el usuario lo abandona) + on submit para validación final. No real-time mientras escribe (reduce ansiedad de usuario).
- **Mensajes de error:** Inline debajo del campo con icono de alerta y texto en rojo/naranja. Ejemplo: "Este campo es obligatorio." o "Por favor, introduce un email válido."
- **Error de resumen:** En el checkout (formulario largo), adicionalmente un summary de errores al top del formulario al hacer submit con campos incompletos.
- **Success state:**
  - Newsletter signup: mensaje inline de éxito en la misma sección, el formulario desaparece y muestra "¡Código enviado a tu email!"
  - Checkout: redirect a `/pedido-confirmado`
  - Formulario de contacto: redirect a `/contacto/gracias`
- **Required field indicator:** Asterisco (*) al lado del label del campo. Nota al inicio del formulario: "Los campos marcados con * son obligatorios."

---

### Links & CTAs

- **External links:** Nueva pestaña (`target="_blank"` con `rel="noopener noreferrer"`) — aplica a Google Maps, Google Reviews, redes sociales.
- **CTA hierarchy:**
  - **Primary (filled):** Acción principal de conversión — comprar, finalizar compra, pagar. Un único botón primary por sección.
  - **Secondary (outline/ghost):** Acción alternativa valiosa — "Visítanos", "Ver más", "Conoce nuestra historia". Puede coexistir con un primary.
  - **Ghost/text-link:** Navegación no crítica, acciones de baja prioridad — "Leer más" en cards, links del footer, "Ver en Google Maps". Nunca compite visualmente con primary.
- **Hover states:** Todos los elementos interactivos tienen cursor `pointer`. Los botones primary/secondary hacen transition de background/border color en 150ms. Los text-links tienen underline en hover. Las tarjetas de producto tienen ligero box-shadow y scale(1.02) en hover.
- **Focus states:** Todos los elementos interactivos tienen un focus ring visible (2px offset, color de marca). Esto cubre tanto navegación por teclado como accesibilidad.

---

### Accessibility Defaults

- **Focus ring:** Siempre visible — nunca `outline: none` sin reemplazo. El focus ring usa el color de marca con suficiente contraste sobre el fondo.
- **Skip navigation:** Sí — requerido. Primer elemento tabulable de cada página: `<a href="#main-content" class="skip-link">Saltar al contenido principal</a>`. Visible solo al recibir focus, oculto visualmente de otro modo (no `display:none` — debe ser accesible por teclado).
- **Keyboard tab order:** 
  - Header: logo → nav links (izquierda a derecha) → carrito → hamburger (mobile)
  - Formularios: orden DOM natural (de arriba a abajo, izquierda a derecha)
  - Modal/drawer de carrito: el focus se trapa dentro del modal mientras está abierto; al cerrarlo, el focus vuelve al elemento que lo abrió (botón "Añadir al carrito")
- **ARIA:**
  - El hamburger button tiene `aria-label="Abrir menú de navegación"` y `aria-expanded` que cambia en estado abierto/cerrado
  - Los carruseles tienen `role="region"` y `aria-label` descriptivo
  - Las imágenes de producto tienen `alt` descriptivo (nombre del café + descripción breve). Las imágenes decorativas tienen `alt=""`
- **Contraste:** Mínimo WCAG AA (4.5:1 para texto normal, 3:1 para texto grande y elementos UI). El overlay del Hero y los fondos oscuros de secciones deben verificarse con la paleta final de colores.
- **Reducción de movimiento:** Todas las animaciones respetan `prefers-reduced-motion: reduce` — si el usuario ha activado esta preferencia en su sistema, las animaciones se desactivan o reducen a simples fades (sin slides, zooms ni parallax).

---

## Notas Finales de Arquitectura

1. **Alcance MVP confirmado:** 5 páginas principales + 3 utility pages (confirmación pedido, 404, gracias-contacto) + sub-páginas dinámicas de producto (`/tienda/[slug]`) y checkout. Todo dentro del rango presupuestario de €2.000 con Next.js App Router.

2. **Decisiones que requieren validación con el cliente antes de UI:**
   - ¿La sección "El Café" (`/cafe`) es editorial (inspiración) o directamente la tienda? Si el cliente quiere simplificar, se puede fusionar en una sola página `/tienda` con buena introducción editorial arriba. Recomendación: mantener separadas para claridad de propósito y SEO.
   - ¿Los horarios son estables o cambian frecuentemente? Si cambian, necesitamos un mini-CMS solo para esa información (Sanity/Notion sync) — añadiría coste.
   - ¿Se implementa carrito en v1 o se redirige a un Stripe Checkout hosted? Para el presupuesto de €2.000, Stripe Checkout hosted es más rápido de implementar y suficientemente profesional. El carrito personalizado puede ser v2.

3. **Flujo de handoff:**
   - Este documento va a `ui-designer` para definir paleta, tipografía y componentes visuales sobre esta estructura.
   - Va a `prototype-designer` para crear el prototipo clickable (Figma) para validación con el cliente.
   - El `coordinator` debe agendar revisión del sitemap con el cliente antes de avanzar a UI.
