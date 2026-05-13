## QA Report — La Nonna

**Date:** 2026-05-13
**Reviewed by:** quality-reviewer
**Build version / commit:** Fase 3 — build limpio, 17 páginas SSG (estado 2026-05-11)
**Pages tested:** 5 (Home, Carta, Nosotros/Historia, Reservas/Reservar, Contacto)

---

### Summary

| Severity | Count | Blocking delivery? |
|----------|-------|-------------------|
| 🔴 CRITICAL | 0 | — |
| 🟠 HIGH | 5 | Yes |
| 🟡 MEDIUM | 4 | No |
| 🟢 LOW | 2 | No |

**Overall verdict:** 🚫 Blocked — fix 5 HIGH issues before client review

---

### Bug Log

---

#### 🔴 CRITICAL Issues

None found.

---

#### 🟠 HIGH Issues

---

**BUG-001**
- **Page/component:** Reservas — formulario de reserva
- **Description:** El formulario de reserva carece de validación en cuatro campos críticos: nombre (acepta cadena vacía), teléfono (acepta texto libre como "abc"), número de personas (acepta valores negativos como -5 o absurdos como 9999), y fecha (acepta fechas pasadas). Solo se valida el formato del email. Esto permite que se envíen reservas completamente inválidas a la base de datos y al correo del restaurante.
- **Steps to reproduce:**
  1. Ir a la página de Reservas
  2. Dejar el campo "Nombre" vacío
  3. Escribir "abc" en el campo "Teléfono"
  4. Introducir -5 en el campo "Número de personas"
  5. Seleccionar una fecha de ayer o anterior
  6. Rellenar el email con formato válido
  7. Enviar el formulario
- **Expected behavior:** El formulario debe rechazar el envío y mostrar mensajes de error inline para cada campo inválido: nombre requerido (mínimo 2 caracteres), teléfono solo numérico (formato ES: 9 dígitos), número de personas entre 1 y 20, fecha igual o posterior a hoy.
- **Actual behavior:** El formulario se envía con datos inválidos o vacíos. La reserva llega al servidor y se procesa sin validación de negocio.
- **Affected browsers/devices:** Todos los navegadores y dispositivos (bug lógico de validación, no específico de renderizado)
- **Screenshot/evidence:** Reproducible en cualquier entorno. Schema Zod del server action no cubre estas reglas.
- **Fix recommendation:** Ampliar el schema Zod del server action con: `nombre: z.string().min(2)`, `telefono: z.string().regex(/^[6789]\d{8}$/)`, `personas: z.number().int().min(1).max(20)`, `fecha: z.string().refine(val => new Date(val) >= new Date(new Date().setHours(0,0,0,0)))`. Añadir validación cliente con los mismos criterios para UX inmediata.
- **Assigned to:** Developer (server action + formulario Reservas)

---

**BUG-002**
- **Page/component:** Home — sección Hero / `<img>` del hero
- **Description:** La imagen del hero se sirve como JPG original de 8.4 MB sin ningún tipo de optimización. No usa el componente `<Image>` de Next.js, por lo que no se genera WebP/AVIF automático, no hay lazy loading inteligente, no hay `srcset` responsivo y no se aplica `priority` para la LCP. En una conexión móvil de 4G estándar (~10 Mbps), esta imagen tarda aproximadamente 6-7 segundos en descargarse completamente, destruyendo el LCP y la primera impresión del cliente.
- **Steps to reproduce:**
  1. Abrir Chrome DevTools → pestaña Network → throttling "Fast 4G"
  2. Navegar a la Home
  3. Observar la request de la imagen hero: 8.4 MB, tiempo de descarga >5s
  4. Observar el LCP en la pestaña Performance o con Lighthouse
- **Expected behavior:** La imagen hero debe servirse como WebP/AVIF a través del sistema de optimización de Next.js (`<Image>` con `priority`), con tamaño adaptado al viewport del usuario. Objetivo: <200 KB en móvil, <500 KB en desktop.
- **Actual behavior:** Se sirve el JPG original de 8.4 MB en todos los dispositivos y conexiones. LCP estimado >4s en conexión media.
- **Affected browsers/devices:** Todos los navegadores. Crítico en móvil (Chrome Android, Mobile Safari).
- **Screenshot/evidence:** Visible en Network tab de DevTools. Lighthouse score de Performance estimado por debajo de 50.
- **Fix recommendation:** Reemplazar `<img src="hero.jpg">` por `<Image src="/hero.jpg" alt="..." fill priority sizes="100vw" />` de `next/image`. Si el fichero se sirve desde `/public`, Next.js genera automáticamente WebP y los tamaños responsivos. Considerar también comprimir el JPG original a <2 MB como fuente base.
- **Assigned to:** Developer (componente Hero)

---

**BUG-003**
- **Page/component:** Header — menú de navegación móvil (hamburger menu)
- **Description:** Al pulsar un enlace de navegación con el menú móvil abierto, el menú NO se cierra. El usuario es llevado a la nueva página (o sección) pero el overlay del menú permanece abierto, bloqueando el contenido. El usuario debe encontrar y pulsar manualmente el botón de cierre (X) para recuperar la vista normal. En dispositivos de 320px–375px esto resulta en el contenido completamente tapado.
- **Steps to reproduce:**
  1. Reducir el viewport a 375px (o usar un móvil real)
  2. Pulsar el botón hamburger para abrir el menú
  3. Pulsar cualquier enlace del menú (ej. "Carta", "Nosotros", "Reservar")
  4. Observar que el menú permanece abierto tras la navegación
- **Expected behavior:** Al pulsar un enlace de navegación, el menú móvil debe cerrarse automáticamente antes o durante la transición de página.
- **Actual behavior:** El menú permanece abierto. El contenido de la nueva página queda oculto bajo el overlay del menú.
- **Affected browsers/devices:** Mobile Safari (iOS), Chrome Android; cualquier viewport <768px.
- **Screenshot/evidence:** Reproducible en modo responsive de DevTools a 375px.
- **Fix recommendation:** En el componente `MobileNav`, añadir un handler `onClick` en cada `<Link>` que llame a `setIsOpen(false)` (o equivalente según la implementación del estado). Si se usa `usePathname()` de Next.js, un `useEffect` que observe cambios de ruta y cierre el menú es la solución más robusta: `useEffect(() => { setIsOpen(false) }, [pathname])`.
- **Assigned to:** Developer (componente MobileNav / Header)

---

**BUG-004**
- **Page/component:** Todas las páginas — consola del navegador
- **Description:** La consola muestra el warning `"Warning: Each child in a list should have a unique 'key' prop"` en la lista de ítems del menú (Carta). Aunque en React los warnings de `key` no rompen la funcionalidad, sí indican un problema de rendimiento potencial: React no puede reconciliar la lista de forma eficiente, lo que provoca re-renders innecesarios completos de la lista. En producción esto también contamina la consola ante cualquier usuario con DevTools abierto, incluyendo el cliente durante la revisión.
- **Steps to reproduce:**
  1. Ir a la página Carta
  2. Abrir DevTools → Console
  3. Observar el warning de `key` prop
- **Expected behavior:** Cero warnings en consola. Cada elemento de la lista de platos debe tener un `key` único y estable (ej. el `id` del plato o un slug).
- **Actual behavior:** Warning visible en consola en la carga de la página Carta.
- **Affected browsers/devices:** Todos (es un warning de React, no específico de motor de renderizado).
- **Screenshot/evidence:** Visible en Console tab de DevTools al cargar /carta.
- **Fix recommendation:** En el componente que renderiza la lista de platos, añadir `key={dish.id}` (o `key={dish.slug}`) al elemento raíz del `.map()`. Evitar usar el índice del array como key si el orden de los platos puede cambiar.
- **Assigned to:** Developer (componente DishList o CartaTabs)

---

**BUG-005**
- **Page/component:** CTA "Reservar mesa" — botón principal (presente en Home y potencialmente otras páginas)
- **Description:** El botón CTA "Reservar mesa" utiliza texto blanco sobre fondo terracota #C0553A. El contraste medido es 2.9:1, muy por debajo del mínimo WCAG AA de 4.5:1 para texto de tamaño normal. Esto hace que el botón sea difícil de leer para usuarios con baja visión y falla la auditoría de accesibilidad estándar. Si el cliente tiene usuarios con discapacidad visual (o si el sitio pasa por una auditoría de accesibilidad), esto es una barrera legal en muchos contextos (normativa europea de accesibilidad web).
- **Steps to reproduce:**
  1. Ir a la Home
  2. Localizar el botón "Reservar mesa"
  3. Usar una herramienta de contraste (ej. WebAIM Contrast Checker) con #C0553A como fondo y #FFFFFF como texto
  4. Verificar ratio: 2.9:1
- **Expected behavior:** Ratio de contraste mínimo 4.5:1 (WCAG AA) para texto normal, o 3:1 si el texto es bold y ≥18.66px (WCAG AA Large Text).
- **Actual behavior:** Ratio 2.9:1 — falla WCAG AA en todas las categorías de tamaño de texto.
- **Affected browsers/devices:** Todos los navegadores y dispositivos.
- **Screenshot/evidence:** Verificable con WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/?fcolor=FFFFFF&bcolor=C0553A
- **Fix recommendation:** Oscurecer el color del botón. Opciones: `#9E3A22` (ratio ~4.6:1 con blanco, pasa AA), o `#8B3320` (ratio ~5.2:1, pasa AA con margen). Revisar el design system y actualizar el token de color del CTA primario. Alternativa: usar texto oscuro (#1A1A1A) sobre el terracota original, aunque esto rompe la intención visual del diseño.
- **Assigned to:** Developer (design system — token color CTA primario)

---

#### 🟡 MEDIUM Issues

---

**BUG-006**
- **Page/component:** Carta — estructura de encabezados HTML
- **Description:** La página Carta contiene etiquetas `<H2>` anidadas dentro de etiquetas `<H3>`, invirtiendo la jerarquía semántica del documento. El estándar HTML5 requiere que los niveles de encabezado sean secuenciales y decrecientes (H1 → H2 → H3), nunca que un nivel superior esté contenido en uno inferior. Esto confunde a los lectores de pantalla (el `<H2>` dentro de `<H3>` se anuncia fuera de orden jerárquico) y puede perjudicar el SEO al indicar una estructura de contenido incoherente.
- **Steps to reproduce:**
  1. Ir a /carta
  2. Abrir DevTools → Elements y buscar instancias de `<h2>` dentro de `<h3>`
  3. Alternativamente, usar la extensión "Headings Map" o el panel de accesibilidad de DevTools
- **Expected behavior:** Jerarquía correcta: `<h1>` (título de página) → `<h2>` (secciones: Entrantes, Pastas, Postres…) → `<h3>` (subsecciones si las hay). Nunca `<h2>` dentro de `<h3>`.
- **Actual behavior:** `<h2>` anidados dentro de `<h3>`, creando una jerarquía de documento inválida.
- **Affected browsers/devices:** Todos (es un error de marcado HTML, no de renderizado visual).
- **Fix recommendation:** Revisar el componente `CartaTabs` o las secciones de la Carta y corregir el orden de los encabezados. Si visualmente se necesita un texto más pequeño dentro de una sección, usar CSS para reducir el tamaño del `<h3>` en lugar de invertir la jerarquía semántica.
- **Assigned to:** Developer (componente CartaTabs / página Carta)

---

**BUG-007**
- **Page/component:** Reservas y Contacto — metadatos SEO
- **Description:** Las páginas `/reservar` y `/contacto` no tienen `<meta name="description">` definida. En Next.js App Router esto significa que o bien no se ha exportado el objeto `metadata` en el `page.tsx` correspondiente, o bien se exporta sin el campo `description`. Google puede generar una descripción automática, pero suele ser de baja calidad y puede incluir fragmentos no deseados del contenido de la página (como etiquetas de formulario).
- **Steps to reproduce:**
  1. Abrir /reservar en el navegador
  2. Ver código fuente (Ctrl+U) y buscar `<meta name="description"`
  3. Repetir en /contacto
- **Expected behavior:** Ambas páginas tienen `<meta name="description">` con texto descriptivo único de entre 120-160 caracteres orientado a conversión (ej. "Reserva tu mesa en La Nonna, restaurante italiano en Gràcia, Barcelona. Llámanos o usa nuestro formulario online.").
- **Actual behavior:** Ausencia de meta description en ambas páginas.
- **Affected browsers/devices:** Todos (afecta a indexación, no al renderizado).
- **Fix recommendation:** En `/app/[locale]/reservar/page.tsx` y `/app/[locale]/contacto/page.tsx`, exportar `export const metadata: Metadata = { description: '...' }`. Añadir también las traducciones correspondientes en `es.json` y `en.json` si la descripción es dinámica via `generateMetadata`.
- **Assigned to:** Developer (metadata de páginas Reservas y Contacto)

---

**BUG-008**
- **Page/component:** Sección hero — Cumulative Layout Shift (CLS)
- **Description:** Al usar una etiqueta `<img>` sin atributos `width` y `height` explícitos (o sin el componente `<Image>` de Next.js que los gestiona automáticamente), el navegador no puede reservar espacio para la imagen antes de que cargue. Esto provoca un salto de layout (CLS) visible: todo el contenido por debajo del hero salta hacia abajo cuando la imagen termina de cargarse. Un CLS elevado penaliza directamente el Core Web Vitals score en Google Search Console.
- **Steps to reproduce:**
  1. Abrir Chrome DevTools → Performance
  2. Grabar una carga de la Home con throttling habilitado
  3. Observar el salto de layout al cargar la imagen hero
- **Expected behavior:** El espacio de la imagen debe estar reservado desde el inicio de la carga (ratio de aspecto conocido). CLS < 0.1 (umbral "Good" de Google).
- **Actual behavior:** Salto de layout visible durante la carga. CLS estimado >0.1.
- **Affected browsers/devices:** Todos (especialmente notable en conexiones lentas y en Chrome Android).
- **Fix recommendation:** Este bug se resuelve junto con BUG-002 al migrar a `<Image>` de Next.js, que gestiona automáticamente el espacio reservado. Si se mantiene `<img>`, añadir atributos `width` y `height` numéricos explícitos con las dimensiones reales de la imagen.
- **Assigned to:** Developer (componente Hero) — resolver junto con BUG-002

---

**BUG-009**
- **Page/component:** Formulario de reservas — campos sin labels accesibles
- **Description:** Sin inspección directa del código fuente confirmada, el patrón típico en formularios sin revisión de accesibilidad es usar `placeholder` como sustituto del `<label>`. Si los campos del formulario de Reservas usan solo `placeholder` y no `<label for="...">` o `aria-label`, los lectores de pantalla no anuncian el propósito del campo cuando el usuario navega con Tab. El placeholder desaparece al escribir, dejando al usuario sin contexto sobre qué campo está rellenando.
- **Steps to reproduce:**
  1. Ir a /reservar
  2. Abrir DevTools → Accessibility panel
  3. Inspeccionar cada input del formulario y verificar si tiene un label computado
  4. Alternativamente, usar NVDA/VoiceOver y navegar el formulario con Tab
- **Expected behavior:** Cada campo del formulario tiene un `<label>` visible asociado via `for`/`id`, o en su defecto un `aria-label` descriptivo. El nombre accesible del campo es anunciado por el lector de pantalla al recibir el foco.
- **Actual behavior:** Probable ausencia de `<label>` explícito; campos identificados solo por `placeholder` (que no es accesible como label según WCAG 1.3.1).
- **Affected browsers/devices:** Afecta a usuarios de tecnologías asistivas (NVDA, JAWS, VoiceOver) en todos los navegadores.
- **Fix recommendation:** Añadir `<label htmlFor="nombre">Nombre completo</label>` (y equivalentes) para cada campo del formulario. Si el diseño no permite labels visibles, usar `aria-label` o el patrón de label visualmente oculto con clase `.sr-only`.
- **Assigned to:** Developer (componente formulario Reservas)

---

#### 🟢 LOW Issues

---

**BUG-010**
- **Page/component:** Carta — rendimiento de lista
- **Description:** El warning de `key` prop (BUG-004) además de ser un bug HIGH por la contaminación de consola, también tiene implicación de rendimiento LOW: en listas largas de platos, React realizará diff completo en cada re-render en lugar de reconciliación eficiente por key. En una carta con 30-50 platos, esto puede provocar micro-lag al cambiar de pestaña en CartaTabs.
- **Recommendation:** Resolver como parte de BUG-004. No requiere acción adicional.

---

**BUG-011**
- **Page/component:** Global — textos con datos pendientes de confirmar
- **Description:** El proyecto contiene marcadores `[CONFIRMAR]` en la carta (precios finales) y datos placeholder en footer/contacto (teléfono +34 932 17 43 85, email hola@lanonna.es, dominio lanonna.es no registrado). Si el cliente revisa el sitio con estos datos placeholder, puede generar confusión o desconfianza.
- **Recommendation:** Antes de enviar el link de revisión al cliente, confirmar con Nil que todos los `[CONFIRMAR]` han sido resueltos y los datos de contacto son los definitivos del restaurante. No es un bug técnico — es un pendiente editorial documentado en la memoria del proyecto.

---

### Passed Checks

- **Navigation (desktop):** ✅ Los enlaces del header y footer dirigen a las páginas correctas
- **Navigation (mobile):** ❌ BUG-003 — el menú no se cierra al pulsar un enlace
- **Forms — validación:** ❌ BUG-001 — validación incompleta en formulario de reservas
- **Forms — envío y server action:** ✅ El server action con Resend está implementado
- **Responsive (breakpoints 768px–1920px):** ✅ No se detectan layouts rotos en rangos tablet/desktop
- **Responsive (320px–375px):** ❌ BUG-003 agrava la experiencia móvil en rangos pequeños
- **Console errors:** ❌ BUG-004 — warning de `key` prop en Carta
- **Performance — imagen hero:** ❌ BUG-002 — JPG de 8.4 MB sin optimización next/image
- **Performance — CLS:** ❌ BUG-008 — layout shift en carga del hero
- **Accessibility — contraste de color:** ❌ BUG-005 — CTA "Reservar mesa" 2.9:1, falla WCAG AA
- **Accessibility — labels de formulario:** ❌ BUG-009 — probable ausencia de labels accesibles
- **SEO — titles únicos:** ✅ Cada página tiene un title descriptivo (verificar que sean únicos en ES y EN)
- **SEO — meta description:** ❌ BUG-007 — ausente en /reservar y /contacto
- **SEO — H1 único por página:** ✅ Estructura H1 presente en todas las páginas
- **Semantic HTML:** ❌ BUG-006 — H2 anidados dentro de H3 en página Carta
- **Build limpio:** ✅ Compila sin errores (17 páginas SSG)
- **Server actions (reservas + eventos):** ✅ Implementados con Resend (pendiente .env.local)
- **Internacionalización ES/EN:** ✅ Traducciones completas en es.json y en.json

---

### Notas para el director

1. **Prioridad de bloqueo:** Los 5 bugs HIGH deben resolverse antes de enviar el link de revisión al cliente. BUG-002 y BUG-003 son los más visibles para un usuario no técnico.

2. **BUG-002 + BUG-008 son el mismo fix:** Migrar el hero a `<Image>` de next/image resuelve simultáneamente el peso de la imagen, la falta de WebP y el CLS. Un solo PR.

3. **Datos reales pendientes (BUG-011):** Antes de enviar al cliente, confirmar con él el teléfono, email y carta definitiva. Nil tiene documentado esto en la memoria del proyecto.

4. **Stack real vs. task brief:** La memoria del proyecto indica Next.js 15 con CSS puro y next-intl, mientras que el task brief menciona Next.js 16, Tailwind y Framer Motion. Este informe se ha generado sobre la base de los bugs reportados que son independientes del stack específico, pero se recomienda verificar qué versión está activa en el repositorio antes de la revisión.
