# UX Design — Ana López, Fotógrafa Freelance
## Landing Page One-Pager: Portfolio, Bio & Contacto

**Proyecto:** Landing page personal de Ana López, fotógrafa freelance  
**Fecha:** 2026-05-13  
**Tipo:** One-pager (single scroll page)  
**Objetivos de conversión primarios:**
1. Que clientes potenciales contacten a Ana (acción directa)
2. Que visitantes reconozcan su estilo y quieran trabajar con ella (conversión de percepción)

---

## 1. SITEMAP — Arquitectura de Secciones

Aunque es una sola URL, documentar las secciones como "nodos de contenido" permite planificar la jerarquía informativa, las anclas de navegación y el flujo de atención.

```
/ (raíz — única URL)
│
├── [S1] HERO / ABOVE THE FOLD
│   ├── Logotipo o nombre tipográfico "Ana López"
│   ├── Tagline fotográfico (estilo + especialidad)
│   ├── CTA primario: "Ver portfolio" (ancla #portfolio)
│   └── CTA secundario: "Contactar" (ancla #contacto)
│
├── [S2] PORTFOLIO GALLERY
│   ├── Filtro de categorías (ej. Retratos / Bodas / Editorial / Comercial)
│   ├── Grid de imágenes (masonry o grid uniforme)
│   └── Lightbox por imagen (modal in-page, sin salir de la URL)
│
├── [S3] BIO / SOBRE ANA
│   ├── Fotografía de Ana (humaniza la marca personal)
│   ├── Texto biográfico: historia, visión, estilo
│   ├── Datos de credibilidad (años de experiencia, clientes notables, premios)
│   └── CTA embebido: "¿Hablamos?" → ancla #contacto
│
├── [S4] SERVICIOS (opcional pero recomendado)
│   ├── Listado de especialidades (Retratos, Eventos, Corporativo…)
│   ├── Formato visual: tarjetas o lista con icono
│   └── Sin precios (CTA al contacto para presupuesto)
│
├── [S5] TESTIMONIOS / PRUEBA SOCIAL
│   ├── 3–5 citas de clientes anteriores
│   ├── Nombre + tipo de proyecto del cliente
│   └── (Opcional) Foto del cliente o logo de empresa
│
├── [S6] FORMULARIO DE CONTACTO
│   ├── Campos: Nombre, Email, Tipo de proyecto, Mensaje
│   ├── CTA de envío: "Enviar mensaje"
│   ├── Estado de éxito inline (sin redirección)
│   └── Información de contacto alternativa (email directo, Instagram)
│
└── [S7] FOOTER
    ├── Copyright y nombre
    ├── Links a redes sociales (Instagram prioritario)
    └── (Opcional) Link a política de privacidad (RGPD)
```

**Notas de arquitectura:**
- Todas las secciones viven en la misma URL raíz `/`
- Cada sección tiene un `id` HTML para anclas de navegación (`#portfolio`, `#bio`, `#contacto`)
- El orden sigue la lógica de confianza progresiva: impactar → mostrar trabajo → humanizar → cerrar con contacto

---

## 2. FLUJO DE CONTACTO PRINCIPAL

### Flujo completo: Llegada → Envío del formulario

```
ENTRADA AL SITIO
     │
     ▼
[Punto de entrada]
Visitante llega desde:
  · Instagram (enlace en bio)
  · Referido de boca a boca (URL directa)
  · Google (búsqueda local "fotógrafa [ciudad]")
  · Email o tarjeta de visita
     │
     ▼
[S1 — HERO]
Primer impacto visual: imagen fotográfica de fondo o encima del fold
Lectura: Nombre → Tagline → CTA

     ┌─────────────────────────────────────┐
     │  ¿La foto de portada genera interés? │
     └──────────┬──────────────────┬────────┘
                │ SÍ               │ NO
                ▼                  ▼
         Scroll natural       Bounce (abandono)
         hacia abajo          [fuera del alcance UX]
                │
                ▼
[S2 — PORTFOLIO]
Visualiza imágenes → filtra por categoría
Reconoce estilo → genera deseo de trabajar con Ana

     ┌─────────────────────────────────────────┐
     │  ¿El portfolio conecta con sus necesidades? │
     └──────────┬──────────────────────┬────────┘
                │ SÍ                   │ NO → Bounce
                ▼
[S3 — BIO]
Lee sobre Ana → construye confianza personal
Lee el CTA embebido "¿Hablamos?"
     │
     ├── [OPCIÓN A] Hace clic en CTA embebido → salta a #contacto
     │
     └── [OPCIÓN B] Sigue scrolleando
                │
                ▼
[S4 — SERVICIOS]
Confirma que Ana ofrece lo que necesita
                │
                ▼
[S5 — TESTIMONIOS]
Lee experiencias de otros clientes → confianza reforzada
                │
                ▼
[S6 — FORMULARIO DE CONTACTO]  ◄── También accesible desde nav sticky
     │
     ▼
INTERACCIÓN CON EL FORMULARIO:

  Paso 1: Usuario lee el título de la sección
          "¿Tienes un proyecto en mente? Cuéntame."
          
  Paso 2: Completa campos
          · Nombre* (text input)
          · Email* (email input — validación inline)
          · Tipo de proyecto (select: Retrato / Boda / Editorial / Corporativo / Otro)
          · Mensaje (textarea — placeholder con sugerencia de contenido)
          
  Paso 3: Clic en "Enviar mensaje"
  
     ┌────────────────────────────────────┐
     │  ¿Validación del lado del cliente?  │
     └──────────┬─────────────────────────┘
                │
         Campos válidos → Submit
         Campos inválidos → Error inline por campo (no toast global)
                │
                ▼
  Paso 4: Estado de carga (spinner o botón deshabilitado)
  
  Paso 5a: ÉXITO
           Mensaje inline en la misma sección:
           "¡Mensaje enviado! Ana te responderá en 24–48 horas."
           El formulario se oculta o limpia (no hay redirección)
           
  Paso 5b: ERROR (fallo de servidor)
           Mensaje inline:
           "Algo falló. Prueba escribiendo directamente a ana@analopez.com"
           [enlace mailto activo]

CONVERSIÓN COMPLETADA ✓
```

### Puntos de entrada al formulario (múltiples rutas):
1. **Nav sticky** → "Contactar" siempre visible → ancla directa a #contacto
2. **CTA del Hero** → "Contactar" en el fold inicial
3. **CTA embebido en Bio** → "¿Hablamos?" after the trust moment
4. **Tarjeta de servicios** → cada servicio puede tener un micro-CTA "Pedir presupuesto"
5. **Scroll natural** → llegar organicamente al formulario al final de la página

---

## 3. WIREFRAME TEXTUAL — SECCIÓN POR SECCIÓN

### Convenciones del wireframe:
- `[ ]` = elemento interactivo (botón, enlace, input)
- `{ }` = imagen o media
- `|···|` = contenedor/layout
- `---` = separador visual o espaciado
- `≡` = menú / hamburguesa
- `★` = elemento de énfasis o relevancia alta

---

### NAV — Barra de navegación sticky

```
┌─────────────────────────────────────────────────────────────────────┐
│  ANA LÓPEZ ★                    [Portfolio] [Bio] [Servicios] [Contactar ▶] │
│  (logotipo tipográfico)                               (CTA destacado)       │
└─────────────────────────────────────────────────────────────────────┘

Mobile:
┌─────────────────────────────────────────────────────────────────────┐
│  ANA LÓPEZ                                              [Contactar] [≡] │
└─────────────────────────────────────────────────────────────────────┘

Comportamiento:
- Fondo transparente sobre el hero, se vuelve opaco (blanco/oscuro) al hacer scroll
- "Contactar" siempre visible como botón CTA (no se oculta en mobile)
- Menú hamburguesa en mobile despliega anclas de sección en panel lateral
```

---

### S1 — HERO / ABOVE THE FOLD

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   {FOTOGRAFÍA DE PORTADA — full-bleed, 100vh}                        │
│   (imagen de alto impacto: retrato, paisaje urbano, o escena de boda │
│    con bokeh suave — representa el estilo de Ana)                    │
│                                                                       │
│   ┌──────────────────────────────────────────┐                       │
│   │   Overlay semitransparente (gradient)     │                       │
│   │                                           │                       │
│   │   ★ ANA LÓPEZ                             │                       │
│   │     Fotografía de autor · Barcelona        │                       │
│   │                                           │                       │
│   │   "Capturo momentos que importan,          │                       │
│   │    personas que brillan."                  │                       │
│   │                                           │                       │
│   │   [Ver portfolio ↓]   [Contactar →]        │                       │
│   └──────────────────────────────────────────┘                       │
│                                                                       │
│   ↓ scroll indicator (chevron animado)                               │
└─────────────────────────────────────────────────────────────────────┘

Especificaciones:
- Altura: 100vh (viewport completo)
- Imagen: 1 foto de máximo impacto (no carrusel en el hero — distrae)
- Tipografía: nombre grande, tagline en cuerpo menor, quote en itálica
- CTA primario: "Ver portfolio" — ancla smooth scroll a #portfolio
- CTA secundario: "Contactar" — ancla smooth scroll a #contacto
- Scroll indicator: chevron pulsante para indicar contenido debajo
- Texto sobre imagen: fondo overlay para legibilidad (WCAG AA)
```

---

### S2 — PORTFOLIO GALLERY

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   Portfolio                               (título de sección h2)     │
│   ──────────────────────────────────────────────────────────         │
│                                                                       │
│   [Todos]  [Retratos]  [Bodas]  [Editorial]  [Corporativo]           │
│   (filtros — tab activo subrayado o con fondo)                       │
│                                                                       │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│   │          │  │          │  │          │                          │
│   │  {img1}  │  │  {img2}  │  │  {img3}  │                          │
│   │          │  │          │  │          │                          │
│   └──────────┘  └──────────┘  └──────────┘                          │
│   ┌───────┐  ┌──────────────┐  ┌───────┐                            │
│   │{img4} │  │    {img5}    │  │{img6} │                            │
│   │       │  │  (destacada) │  │       │                            │
│   └───────┘  └──────────────┘  └───────┘                            │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│   │          │  │          │  │          │                          │
│   │  {img7}  │  │  {img8}  │  │  {img9}  │                          │
│   └──────────┘  └──────────┘  └──────────┘                          │
│                                                                       │
│   [Cargar más fotos]  (lazy load — no paginación dura)               │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

ON CLICK EN IMAGEN → LIGHTBOX:
┌─────────────────────────────────────────────────────────────────────┐
│  [✕]                                                    [← →]       │
│                                                                       │
│              {IMAGEN AMPLIADA — max 90vw × 90vh}                     │
│                                                                       │
│   Nombre del proyecto · Categoría · Año                               │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Especificaciones:
- Layout: masonry responsive (columnas: 3 desktop / 2 tablet / 1 mobile)
- Hover en imagen: overlay sutil con nombre de proyecto (no oculta la foto)
- Filtros: animación de fade/reordenamiento al cambiar categoría
- Lightbox: modal accesible (foco atrapado, cerrable con Escape)
- Cantidad inicial: 9–12 fotos (3 filas × 3 cols)
- Lazy loading activado en todas las imágenes
```

---

### S3 — BIO / SOBRE ANA

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   Sobre mí                                (título de sección h2)     │
│   ──────────────────────────────────────────────────────────         │
│                                                                       │
│   |── COLUMNA IZQUIERDA (40%) ──|── COLUMNA DERECHA (60%) ──|       │
│   │                             │                             │       │
│   │   {FOTO DE ANA}             │  Hola, soy Ana López        │       │
│   │   (retrato editorial,       │                             │       │
│   │    cálido, cercano)         │  Fotografío personas y      │       │
│   │                             │  momentos desde hace más    │       │
│   │   ───────────────           │  de 10 años en Barcelona.  │       │
│   │   12 años de experiencia    │  Me especializo en          │       │
│   │   +200 bodas fotografiadas  │  [descripción de estilo:    │       │
│   │   Publicada en Vogue ES     │  luz natural, composición   │       │
│   │                             │  minimalista, mirada        │       │
│   │   (credenciales en iconos   │  honesta]                   │       │
│   │    o pequeñas pastillas)    │                             │       │
│   │                             │  Creo que una buena foto    │       │
│   │                             │  no es solo técnica,        │       │
│   │                             │  es conexión.               │       │
│   │                             │                             │       │
│   │                             │  [¿Hablamos de tu proyecto?]│       │
│   │                             │  (CTA texto secundario →   │       │
│   │                             │   ancla #contacto)          │       │
│   └─────────────────────────────┴─────────────────────────────┘     │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Mobile: columna única, foto arriba, texto debajo

Especificaciones:
- Foto de Ana: horizontal o cuadrada, no un selfie casual (branding visual)
- Texto: tono personal y directo, primera persona, sin jerga técnica
- Credenciales: máximo 3 datos impactantes (cantidad > calidad de descripción)
- CTA embebido: enlace de texto (no botón) para no competir con el CTA principal del hero
```

---

### S4 — SERVICIOS

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   Servicios                               (título de sección h2)     │
│   ──────────────────────────────────────────────────────────         │
│                                                                       │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │
│   │                 │  │                 │  │                 │    │
│   │  {icono cámara} │  │  {icono flores} │  │  {icono corp.}  │    │
│   │                 │  │                 │  │                 │    │
│   │  RETRATOS       │  │  BODAS &        │  │  FOTOGRAFÍA     │    │
│   │  PERSONALES     │  │  EVENTOS        │  │  CORPORATIVA    │    │
│   │                 │  │                 │  │                 │    │
│   │  Sesiones       │  │  Desde el       │  │  Equipo,        │    │
│   │  individuales,  │  │  preboda hasta  │  │  producto,      │    │
│   │  parejas y      │  │  la celebración │  │  eventos de     │    │
│   │  familias       │  │  completa       │  │  empresa        │    │
│   │                 │  │                 │  │                 │    │
│   │ [Pedir info →]  │  │ [Pedir info →]  │  │ [Pedir info →]  │    │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘    │
│                                                                       │
│   Nota: Los CTAs "Pedir info" rellenan el campo "Tipo de proyecto"   │
│   del formulario automáticamente (parámetro en ancla o query param)  │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Especificaciones:
- 3 tarjetas es el número óptimo para portfolio personal (no abrumar)
- Sin precios visibles — la conversión es hacia el contacto, no la compra directa
- Los CTAs de tarjeta son secundarios; el CTA principal de toda la sección
  es implícito (scroll hacia testimonios y luego formulario)
- Mobile: tarjetas en columna única con scroll vertical
```

---

### S5 — TESTIMONIOS

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   Lo que dicen mis clientes          (título de sección h2)         │
│   ──────────────────────────────────────────────────────────         │
│                                                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                                                               │   │
│   │   "Ana captó exactamente lo que queríamos en nuestra boda.   │   │
│   │    Las fotos son atemporales. Las miro y lloro de emoción."  │   │
│   │                                                               │   │
│   │   {foto cliente}  María & Jorge · Boda en Sitges · 2024      │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                       │
│   [←]  ●○○  [→]   (carrusel de 3–5 testimonios)                    │
│                                                                       │
│   — O —                                                              │
│                                                                       │
│   ┌───────────────────┐  ┌───────────────────┐  ┌───────────────┐  │
│   │ "Testimonial 1"   │  │ "Testimonial 2"   │  │ "Testim. 3"   │  │
│   │ — Cliente A       │  │ — Cliente B       │  │ — Cliente C   │  │
│   └───────────────────┘  └───────────────────┘  └───────────────┘  │
│   (layout grid estático en desktop, carrusel en mobile)             │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Especificaciones:
- Mínimo 3, máximo 5 testimonios (más no incrementa confianza linealmente)
- El testimonio debe incluir: cita, nombre, tipo de sesión — la foto es opcional
- Priorizar testimonios que mencionan el estilo o la experiencia emocional
  (no solo "fue muy profesional" — eso es genérico)
- Desktop: 3 tarjetas visibles / Mobile: carrusel de 1 en 1
```

---

### S6 — FORMULARIO DE CONTACTO

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   ¿Tienes un proyecto en mente?          (título de sección h2 ★)   │
│   Cuéntame. Respondo en 24–48 horas.                                 │
│   ──────────────────────────────────────────────────────────         │
│                                                                       │
│   |── FORMULARIO (60%) ──────────────────|── DATOS (40%) ──|        │
│   │                                       │                  │        │
│   │  Nombre *                             │  O escríbeme     │        │
│   │  [________________________]           │  directamente:   │        │
│   │                                       │                  │        │
│   │  Email *                              │  ✉ ana@          │        │
│   │  [________________________]           │  analopez.com    │        │
│   │                                       │                  │        │
│   │  ¿Qué tipo de proyecto?               │  📷 @analopez    │        │
│   │  [Selecciona una opción     ▼]        │  (Instagram)     │        │
│   │                                       │                  │        │
│   │  Cuéntame más                         │  📍 Barcelona    │        │
│   │  [                                ]   │  y alrededores   │        │
│   │  [                                ]   │                  │        │
│   │  [                                ]   │                  │        │
│   │  [                      250 chars  ]  │                  │        │
│   │                                       │                  │        │
│   │  [  Enviar mensaje  ▶  ]              │                  │        │
│   │  (CTA primario, full-width en mobile) │                  │        │
│   │                                       │                  │        │
│   │  — Estado de éxito (tras submit) —   │                  │        │
│   │  ┌─────────────────────────────────┐ │                  │        │
│   │  │ ✓ ¡Mensaje enviado! Ana te      │ │                  │        │
│   │  │   responderá en 24–48 horas.    │ │                  │        │
│   │  └─────────────────────────────────┘ │                  │        │
│   └───────────────────────────────────────┴──────────────────┘       │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Campos detallados:
  · Nombre: required, text, placeholder "Tu nombre"
  · Email: required, email type, validación inline al blur
  · Tipo de proyecto: select required
      opciones: "Retrato personal", "Pareja / Familia", "Boda o evento",
                "Fotografía corporativa", "Otro"
  · Mensaje: textarea, no required, placeholder
      "Cuéntame la fecha aproximada, el lugar, o lo que tengas en mente.
       No hace falta que esté todo definido :)"
  · Botón: "Enviar mensaje" — disabled durante submit, spinner activo

Estados de error inline:
  · Email inválido: "Introduce un email válido" (texto rojo bajo el campo)
  · Campo vacío al submit: "Este campo es obligatorio"
  · Error de servidor: mensaje prominente + enlace mailto alternativo

Mobile: columna única, datos de contacto alternativos debajo del form
```

---

### S7 — FOOTER

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│   ANA LÓPEZ                                                          │
│   Fotografía · Barcelona                                             │
│                                                                       │
│   [Instagram]  [LinkedIn]  [Pinterest]                               │
│   (iconos de redes, Instagram primero — canal principal)            │
│                                                                       │
│   © 2026 Ana López · [Política de privacidad]                       │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

Especificaciones:
- Footer minimalista: no duplicar nav ni contenido de secciones
- RGPD: enlace a política de privacidad (obligatorio si hay formulario)
- Instagram: abrir en nueva pestaña (no perder la página)
- Sin mapa de sitio en footer (es un one-pager, sería redundante)
```

---

## 4. GLOBAL INTERACTION PRINCIPLES — ONE-PAGER PORTFOLIO

Principios específicos para el contexto de una landing page de portfolio fotográfico de persona individual, donde la experiencia visual y la confianza son los motores de conversión.

---

### 4.1 La fotografía manda, la UI sirve

**Principio:** La interfaz es el marco del cuadro, no el cuadro mismo.

- Los controles de UI (botones, filtros, nav) deben tener un peso visual menor que las fotografías
- Usar paleta de color neutral (blancos, grises, negros) para no competir con las imágenes de Ana
- Los textos sobre imágenes requieren overlay, no bloquear o competir con la foto
- Evitar elementos decorativos UI (gradientes de marca, ilustraciones) que distraigan del trabajo fotográfico
- Regla práctica: si un elemento UI llama más la atención que la foto más cercana, reducir su peso visual

---

### 4.2 Scroll como narrativa, no como búsqueda

**Principio:** En un one-pager, el usuario no busca información: la descubre. El scroll debe sentirse como una historia con inicio, nudo y desenlace.

- El orden de secciones sigue la curva de confianza: impactar → interesar → creer → actuar
- No interrumpir el scroll con pop-ups, banners de cookies intrusivos, ni chat widgets al inicio
- El ritmo de secciones debe alternar entre "inmersión" (portfolio, hero) y "lectura" (bio, testimonios)
- Las transiciones entre secciones deben ser suaves (fade-in al entrar en viewport, no animaciones abruptas)
- No usar efectos parallax agresivos que marean o ralentizan el scroll

---

### 4.3 Navegación sticky: acceso sin perder contexto

**Principio:** El usuario puede estar en cualquier sección y querer actuar. La nav sticky debe ser siempre accesible sin interrumpir la experiencia.

- La barra de navegación permanece visible en todo momento al hacer scroll
- En el hero: nav transparente (no compite con la foto de portada)
- Post-hero: nav con fondo sólido (legibilidad garantizada sobre cualquier sección)
- El CTA "Contactar" en la nav NUNCA se oculta, ni en mobile
- La indicación de sección activa (underline, color) ayuda al usuario a saber dónde está en la narrativa

---

### 4.4 Confianza progresiva: no pedir antes de dar

**Principio:** El formulario de contacto es el último paso, no el primero. El usuario debe haber recibido valor antes de ser invitado a actuar.

- No mostrar el formulario above the fold ni en posición temprana
- El formulario de contacto aparece después de portfolio, bio y testimonios
- Los CTAs intermedios (bio, servicios) son invitaciones suaves, no obligaciones
- El formulario pide poca información: nombre, email, tipo de proyecto, mensaje (no teléfono, no fecha exacta, no presupuesto)
- Tone of voice del formulario: conversacional, no transaccional ("Cuéntame" vs "Rellena el formulario")

---

### 4.5 Carga percibida: la primera impresión debe ser instantánea

**Principio:** En un portfolio fotográfico, las imágenes son el contenido, pero también el mayor riesgo de rendimiento percibido.

- La imagen del hero debe estar optimizada y en formato moderno (WebP/AVIF) para carga instantánea
- Las imágenes del portfolio se cargan con lazy loading (solo las visibles en viewport)
- Usar placeholders de color o blur mientras carga la imagen real (evitar layout shift)
- El hero no debe bloquearse esperando que carguen las imágenes del portfolio
- Objetivo de rendimiento: LCP (Largest Contentful Paint) < 2.5 segundos

---

### 4.6 Lightbox: inmersión sin abandono de página

**Principio:** Ver una foto en detalle es una micro-conversión de interés. No debe interrumpir el flujo ni llevar al usuario fuera de la landing.

- El lightbox es un modal in-page (no nueva pestaña, no nueva URL)
- Navegación entre fotos con flechas y con teclado (← →)
- Cierre con: botón ✕, clic fuera del modal, y tecla Escape
- Foco de teclado atrapado dentro del modal mientras está abierto (accesibilidad)
- Swipe lateral en mobile para navegar entre fotos
- El lightbox no muestra botones de redes sociales que distraigan del flujo principal

---

### 4.7 Formulario: fricción mínima, confianza máxima

**Principio:** Cada campo extra reduce la tasa de envío. Cada signal de confianza la aumenta.

- Máximo 4 campos obligatorios: Nombre, Email, Tipo de proyecto, (Mensaje como opcional)
- Validación inline al abandonar cada campo (blur), no al intentar hacer submit
- El botón de submit refleja el estado: activo → cargando → éxito/error
- Mensaje de éxito inline (no redirección, no toast que desaparece, no nueva página)
- El mensaje de éxito incluye expectativa de respuesta: "en 24–48 horas"
- Incluir RGPD: checkbox o nota de privacidad bajo el botón ("Tus datos solo se usan para responderte")
- Si hay error de servidor: dar alternativa inmediata (email directo), no dejar al usuario bloqueado

---

### 4.8 Mobile-first: el portfolio se ve desde el móvil

**Principio:** La mayoría de visitas vendrán de Instagram (mobile). El diseño debe ser impecable en pantalla pequeña antes de adaptarse a desktop.

- El hero ocupa 100svh en mobile (usar `svh` no `vh` para evitar problemas con barras del navegador móvil)
- Los filtros de portfolio son scrollables horizontalmente en mobile (no se colapsan en menú)
- Las tarjetas de servicios son un stack vertical en mobile
- El formulario en mobile: columna única, campos full-width, teclado no oculta el botón de submit
- Los touch targets tienen mínimo 44×44px (WCAG 2.1 AA)
- Los textos no son menores de 16px en mobile (evitar zoom automático del navegador)

---

### 4.9 Accesibilidad como calidad, no como requisito

**Principio:** Las imágenes de Ana deben ser accesibles, y la estructura de la página debe ser navegable por teclado y lectores de pantalla.

- Todas las imágenes del portfolio tienen `alt` descriptivo (no vacío, no genérico "foto")
- La jerarquía de headings es coherente: h1 (nombre/hero) → h2 (secciones) → h3 (subsecciones)
- El contraste de texto sobre fondo cumple WCAG AA (ratio ≥ 4.5:1 para texto normal)
- Los filtros de portfolio son botones reales (no divs con onclick) con estado aria-pressed
- El lightbox tiene role="dialog", aria-modal="true" y foco gestionado
- Los errores de formulario son anunciados por lectores de pantalla (aria-live o aria-describedby)

---

### 4.10 Coherencia visual: la marca personal es la fotografía

**Principio:** Ana no es una marca corporativa. Su identidad visual debe ser discreta pero consistente, y nunca eclipsar su trabajo.

- Fuente tipográfica: máximo 2 familias (1 serif elegante para nombre/headings + 1 sans-serif legible para cuerpo)
- Paleta de color: neutros base (blanco, gris claro, negro) + 1 color de acento (usado solo en CTAs y elementos interactivos)
- No usar efectos visuales de moda (glassmorphism excesivo, gradientes vibrantes) que envejezcan rápido
- La página debe verse tan bien en 2 años como hoy (evergreen design)
- Los iconos de servicios y footer deben ser de la misma familia iconográfica (consistencia)
- El spacing entre secciones debe ser generoso (respiro visual) — en un portfolio, el espacio vacío también comunica

---

## 5. RESUMEN EJECUTIVO

| Elemento | Decisión de diseño | Justificación |
|---|---|---|
| Estructura | 7 secciones en 1 URL | Máxima simplicidad, sin fricción de navegación |
| CTA primario | "Contactar" siempre accesible (nav sticky) | Conversión disponible en todo momento |
| Portfolio | Grid masonry + lightbox in-page | Experiencia inmersiva sin abandonar la página |
| Bio | Foto + texto + credenciales + CTA suave | Confianza personal antes de pedir acción |
| Formulario | 4 campos, éxito inline, tono conversacional | Fricción mínima, máxima tasa de completado |
| Mobile | Mobile-first, touch targets ≥44px | La mayoría de visitas vendrán de Instagram |
| Paleta | Neutros + 1 acento | Las fotos son el contenido visual, no la UI |
| Rendimiento | LCP <2.5s, lazy loading, WebP | La espera mata la conversión en portfolios |

**Flujo de conversión optimizado:**
> Hero (impacto) → Portfolio (interés) → Bio (confianza) → Testimonios (validación social) → Formulario (acción)

Este orden sigue la curva natural de decisión de un cliente potencial que no conoce a Ana: primero necesita ver si le gusta el trabajo, luego quiere saber con quién trabaja, luego busca prueba social, y solo entonces está listo para contactar.
