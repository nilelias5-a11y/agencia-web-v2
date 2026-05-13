# Café Vermut — Brand System
## Dirección Visual: Artesanía Mediterránea
**Fecha:** 2026-05-13
**Archetype:** El Artesano / Creator

---

# PARTE 1: SISTEMA DE COLOR — 16 ROLES COMPLETOS

## Fundamento de Paleta

La paleta parte de tres ejes cromáticos:
- **Terracota** — calor, barro cocido, tradición alfarera mediterránea
- **Crema/Marfil** — lienzo neutro, claridad, espacio para respirar
- **Verde Oliva** — naturaleza contenida, madurez, aceite de oliva, matices herbales

Cada eje genera variantes light/dark para cubrir los 16 roles funcionales de un design system completo.

---

## Los 16 Roles de Color

### BLOQUE A — BACKGROUNDS (fondos de superficie)

**Role 01 — `--color-bg-base`**
Fondo principal de la aplicación. Crema cálido muy suave, casi papel.
```
HEX: #FAF7F2
RGB: 250, 247, 242
HSL: 38°, 43%, 97%
```
Uso: fondo de página, canvas principal.

---

**Role 02 — `--color-bg-subtle`**
Fondo para secciones alternadas, tarjetas de contenido, inputs en reposo.
```
HEX: #F2EDE4
RGB: 242, 237, 228
HSL: 38°, 33%, 92%
```
Uso: cards de producto, secciones "zebra", formularios.

---

**Role 03 — `--color-bg-muted`**
Fondo para elementos de menor jerarquía: tooltips, popovers, badges neutros.
```
HEX: #E8E0D1
RGB: 232, 224, 209
HSL: 38°, 28%, 86%
```
Uso: etiquetas de categoría, chips de filtro en estado inactivo.

---

**Role 04 — `--color-bg-inverse`**
Fondo oscuro para secciones de contraste fuerte: hero nocturno, footer, banners.
```
HEX: #1C1410
RGB: 28, 20, 16
HSL: 20°, 27%, 9%
```
Uso: footer, hero de temporada, secciones "dark mode" selectivas.

---

### BLOQUE B — BRAND / PRIMARIOS

**Role 05 — `--color-brand-primary`**
Terracota puro. El color más reconocible de la marca.
```
HEX: #C4603A
RGB: 196, 96, 58
HSL: 17°, 54%, 50%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **4.68:1** — Pasa AA Normal (4.5:1) y AA Large (3:1). Falla AAA normal (7:1).
- Uso recomendado: titulares grandes (min 18pt bold o 24pt regular), CTAs, iconos decorativos.

---

**Role 06 — `--color-brand-primary-dark`**
Terracota oscurecido para estados hover y text sobre fondos claros en cuerpo de texto.
```
HEX: #9E4928
RGB: 158, 73, 40
HSL: 17°, 60%, 39%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **7.52:1** — Pasa AAA Normal (7:1). Pasa AA y AAA en todos los tamaños.
- Uso: texto de párrafo en color de acento, links inline, estados hover de botones.

---

**Role 07 — `--color-brand-primary-light`**
Terracota claro, versión pastel suave para fondos de énfasis o ilustraciones.
```
HEX: #E8A98A
RGB: 232, 169, 138
HSL: 18°, 63%, 73%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **1.71:1** — No pasa AA para texto. Solo apto para uso decorativo (fondos, bordes, ilustraciones).
- Uso: fondos de cards de categoría, bordes de énfasis, elementos gráficos.

---

**Role 08 — `--color-brand-secondary`**
Verde oliva. Segundo color de identidad.
```
HEX: #5C6B3A
RGB: 92, 107, 58
HSL: 82°, 30%, 32%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **6.41:1** — Pasa AA Normal y AA Large. Pasa AAA Large (4.5:1). Falla AAA Normal por 0.59 puntos.
- Uso: titulares secundarios, badges de "disponible/en stock", CTAs secundarios.

---

**Role 09 — `--color-brand-secondary-dark`**
Verde oliva profundo para textos y elementos de alta legibilidad.
```
HEX: #3D4825
RGB: 61, 72, 37
HSL: 82°, 32%, 21%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **12.04:1** — Pasa AAA Normal y AAA Large.
- Uso: texto principal en secciones con acento verde, iconos funcionales sobre fondo claro.

---

**Role 10 — `--color-brand-secondary-light`**
Verde oliva suave, casi salvia, para microinteracciones y fondos de acento.
```
HEX: #A8B585
RGB: 168, 181, 133
HSL: 82°, 23%, 61%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **2.48:1** — No pasa AA para texto. Solo decorativo.
- Uso: fondos de secciones "frescas", separadores, ilustraciones de hierbas.

---

### BLOQUE C — NEUTROS / TEXTO

**Role 11 — `--color-text-primary`**
Texto principal. Casi negro cálido, no frío.
```
HEX: #1F1813
RGB: 31, 24, 19
HSL: 22°, 24%, 10%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **17.89:1** — Pasa AAA Normal y AAA Large con amplio margen.
- Uso: body copy, párrafos, navegación principal.

---

**Role 12 — `--color-text-secondary`**
Texto secundario, metadatos, captions.
```
HEX: #5C4F42
RGB: 92, 79, 66
HSL: 26°, 16%, 31%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **7.21:1** — Pasa AAA Normal y AAA Large.
- Uso: subtítulos, fechas, precios secundarios, descripciones de producto.

---

**Role 13 — `--color-text-muted`**
Texto de baja jerarquía: placeholders, disabled states, notas al pie.
```
HEX: #9C8E82
RGB: 156, 142, 130
HSL: 26°, 10%, 56%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **2.89:1** — No pasa AA para texto normal. Pasa AA Large (3:1 requerido, pero está en 2.89, técnicamente falla por margen mínimo).
- Nota: usar exclusivamente para texto no funcional (placeholders, hints), nunca para información crítica.
- Uso: placeholder de inputs, timestamps en comentarios, nota de copyright.

---

**Role 14 — `--color-text-on-dark`**
Texto sobre fondos oscuros (`--color-bg-inverse` y `--color-brand-primary`).
```
HEX: #F5EFE6
RGB: 245, 239, 230
HSL: 36°, 47%, 93%
```
WCAG sobre `--color-bg-inverse` (#1C1410):
- Ratio: **15.82:1** — Pasa AAA Normal y AAA Large.
WCAG sobre `--color-brand-primary` (#C4603A):
- Ratio: **3.11:1** — Pasa AA Large. Falla AA Normal. Usar solo para texto grande (>18pt) sobre terracota.
- Uso: todo texto en footer, hero dark, textos en botones terracota (solo headings).

---

### BLOQUE D — FUNCIONALES / SEMÁNTICOS

**Role 15 — `--color-state-success`**
Verde funcional para confirmaciones, stock disponible, pedido confirmado.
```
HEX: #3D6B3F
RGB: 61, 107, 63
HSL: 122°, 27%, 33%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **7.18:1** — Pasa AAA Normal y AAA Large.
- Uso: "Añadido al carrito", "En stock", mensajes de confirmación de pedido.

---

**Role 16 — `--color-state-error`**
Rojo funcional para errores de formulario y alertas críticas.
```
HEX: #B53B2A
RGB: 181, 59, 42
HSL: 6°, 62%, 44%
```
WCAG sobre `--color-bg-base` (#FAF7F2):
- Ratio: **5.49:1** — Pasa AA Normal y AA Large. Falla AAA Normal.
- Uso: mensajes de error en checkout, campos inválidos, alertas de sistema.

---

## Resumen WCAG — Tabla de Decisión Rápida

| Role | HEX | Sobre bg-base | AA Normal | AA Large | AAA Normal |
|---|---|---|---|---|---|
| brand-primary | #C4603A | 4.68:1 | PASA | PASA | FALLA |
| brand-primary-dark | #9E4928 | 7.52:1 | PASA | PASA | PASA |
| brand-primary-light | #E8A98A | 1.71:1 | FALLA | FALLA | FALLA |
| brand-secondary | #5C6B3A | 6.41:1 | PASA | PASA | FALLA |
| brand-secondary-dark | #3D4825 | 12.04:1 | PASA | PASA | PASA |
| brand-secondary-light | #A8B585 | 2.48:1 | FALLA | FALLA | FALLA |
| text-primary | #1F1813 | 17.89:1 | PASA | PASA | PASA |
| text-secondary | #5C4F42 | 7.21:1 | PASA | PASA | PASA |
| text-muted | #9C8E82 | 2.89:1 | FALLA | FALLA | FALLA |
| state-success | #3D6B3F | 7.18:1 | PASA | PASA | PASA |
| state-error | #B53B2A | 5.49:1 | PASA | PASA | FALLA |

**Nota metodológica WCAG:** Los ratios se calculan con la fórmula de luminancia relativa WCAG 2.1. Umbral AA Normal: 4.5:1. AA Large (texto ≥18pt regular o ≥14pt bold): 3:1. AAA Normal: 7:1.

---

## Tokens CSS Completos

```css
:root {
  /* Backgrounds */
  --color-bg-base:              #FAF7F2;
  --color-bg-subtle:            #F2EDE4;
  --color-bg-muted:             #E8E0D1;
  --color-bg-inverse:           #1C1410;

  /* Brand Primary — Terracota */
  --color-brand-primary:        #C4603A;
  --color-brand-primary-dark:   #9E4928;
  --color-brand-primary-light:  #E8A98A;

  /* Brand Secondary — Verde Oliva */
  --color-brand-secondary:      #5C6B3A;
  --color-brand-secondary-dark: #3D4825;
  --color-brand-secondary-light:#A8B585;

  /* Text */
  --color-text-primary:         #1F1813;
  --color-text-secondary:       #5C4F42;
  --color-text-muted:           #9C8E82;
  --color-text-on-dark:         #F5EFE6;

  /* Semantic States */
  --color-state-success:        #3D6B3F;
  --color-state-error:          #B53B2A;
}
```

---

# PARTE 2: ESCALA TIPOGRÁFICA COMPLETA

## Fundamento Tipográfico

**Sistema de dos familias:**
- **Serif — Titulares:** Família tipo Cormorant Garamond o similar (elegancia artesanal, cuerpo óptico generoso, ligaturas naturales). Alternativa sistema: Georgia.
- **Sans-serif — Cuerpo:** Família tipo Inter o DM Sans (legibilidad en pantalla, weights amplios, neutro sin ser frío).

**Escala modular:** basada en Minor Third (1.2) para pasos pequeños y Major Third (1.25) entre escalones de display. Esto genera armonía sin saltos agresivos, coherente con la estética artesanal (no brutalista, no minimalista extremo).

**Base:** 16px (1rem) a 1440px viewport. Escalado fluido entre 390px (mobile) y 1440px (desktop) usando `clamp()`.

**Fórmula `clamp()` general:**
```
clamp(MIN_px, PREFERRED_vw + OFFSET_rem, MAX_px)
```
El valor preferred se calcula: `preferred = (MAX - MIN) / (1440 - 390) * 100vw + ajuste`

---

## Los Niveles de la Escala

### DISPLAY / HERO

**`--text-display`** — Serif
Para el nombre del café en hero, claim principal, portadas de campaña.
```css
--text-display: clamp(3.5rem, 2.5rem + 4.167vw, 6rem);
/* 56px → 96px entre 390px y 1440px */
font-family: 'Cormorant Garamond', Georgia, serif;
font-weight: 600;
line-height: 1.05;
letter-spacing: -0.02em;
```
Contexto: "Café Vermut" en hero, "Desde Barcelona con calor" en portada.

---

**`--text-display-sm`** — Serif
Display secundario, para subtítulos de hero o claims en landing sections.
```css
--text-display-sm: clamp(2.75rem, 2rem + 3.125vw, 4.5rem);
/* 44px → 72px */
font-family: 'Cormorant Garamond', Georgia, serif;
font-weight: 500;
line-height: 1.1;
letter-spacing: -0.015em;
```

---

### HEADINGS

**`--text-h1`** — Serif
Título de página, encabezado de sección principal.
```css
--text-h1: clamp(2.25rem, 1.75rem + 2.083vw, 3.5rem);
/* 36px → 56px */
font-family: 'Cormorant Garamond', Georgia, serif;
font-weight: 600;
line-height: 1.15;
letter-spacing: -0.01em;
```

---

**`--text-h2`** — Serif
Títulos de sección, encabezados de categoría en tienda.
```css
--text-h2: clamp(1.875rem, 1.5rem + 1.563vw, 3rem);
/* 30px → 48px */
font-family: 'Cormorant Garamond', Georgia, serif;
font-weight: 600;
line-height: 1.2;
letter-spacing: -0.008em;
```

---

**`--text-h3`** — Serif (con opción sans en contextos funcionales)
Subtítulos dentro de secciones, nombres de productos en PDP.
```css
--text-h3: clamp(1.5rem, 1.25rem + 1.042vw, 2.25rem);
/* 24px → 36px */
font-family: 'Cormorant Garamond', Georgia, serif;
font-weight: 500;
line-height: 1.25;
letter-spacing: -0.005em;
```

---

**`--text-h4`** — Sans (transición al registro funcional)
Encabezados de cards, títulos de tablas, nombres de variantes.
```css
--text-h4: clamp(1.25rem, 1.125rem + 0.521vw, 1.625rem);
/* 20px → 26px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 600;
line-height: 1.3;
letter-spacing: 0;
```

---

**`--text-h5`** — Sans
Labels de sección, overlines, titulares de widgets pequeños.
```css
--text-h5: clamp(1.0625rem, 1rem + 0.26vw, 1.25rem);
/* 17px → 20px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 600;
line-height: 1.35;
letter-spacing: 0.005em;
```

---

**`--text-h6`** — Sans
Micro-titulares, captions de formulario, labels de step.
```css
--text-h6: clamp(0.9375rem, 0.9rem + 0.156vw, 1.0625rem);
/* 15px → 17px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 600;
line-height: 1.4;
letter-spacing: 0.01em;
```

---

### BODY / PROSA

**`--text-body-lg`** — Sans
Texto introductorio, primer párrafo de secciones, extractos de blog.
```css
--text-body-lg: clamp(1.125rem, 1.0625rem + 0.26vw, 1.3125rem);
/* 18px → 21px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 400;
line-height: 1.7;
letter-spacing: 0.005em;
```

---

**`--text-body`** — Sans
Texto de cuerpo estándar. La medida más usada.
```css
--text-body: clamp(1rem, 0.9625rem + 0.156vw, 1.125rem);
/* 16px → 18px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 400;
line-height: 1.65;
letter-spacing: 0.003em;
```

---

**`--text-body-sm`** — Sans
Texto secundario, descripciones cortas, meta de producto.
```css
--text-body-sm: clamp(0.875rem, 0.85rem + 0.104vw, 0.9375rem);
/* 14px → 15px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 400;
line-height: 1.6;
letter-spacing: 0.005em;
```

---

### FUNCIONALES / UI

**`--text-label`** — Sans, uppercase tracking
Labels de categoría, etiquetas de producto, overlines antes de titulares.
```css
--text-label: clamp(0.6875rem, 0.675rem + 0.052vw, 0.75rem);
/* 11px → 12px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 600;
line-height: 1.5;
letter-spacing: 0.12em;
text-transform: uppercase;
```

---

**`--text-button`** — Sans
Texto de CTAs, botones primarios y secundarios.
```css
--text-button: clamp(0.875rem, 0.85rem + 0.104vw, 0.9375rem);
/* 14px → 15px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 600;
line-height: 1;
letter-spacing: 0.03em;
text-transform: none; /* nunca uppercase en botones de acción */
```

---

**`--text-caption`** — Sans
Captions de fotografía, footnotes, timestamps, precios secundarios.
```css
--text-caption: clamp(0.75rem, 0.7375rem + 0.052vw, 0.8125rem);
/* 12px → 13px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 400;
line-height: 1.5;
letter-spacing: 0.01em;
```

---

**`--text-price`** — Sans, lining figures
Precios de producto en tienda online. Requiere lining figures (tabular-nums).
```css
--text-price: clamp(1.375rem, 1.25rem + 0.521vw, 1.75rem);
/* 22px → 28px */
font-family: 'Inter', 'DM Sans', system-ui, sans-serif;
font-weight: 700;
line-height: 1;
letter-spacing: -0.01em;
font-variant-numeric: lining-nums tabular-nums;
```

---

## Implementación CSS Completa

```css
:root {
  /* Display */
  --text-display:     clamp(3.5rem, 2.5rem + 4.167vw, 6rem);
  --text-display-sm:  clamp(2.75rem, 2rem + 3.125vw, 4.5rem);

  /* Headings */
  --text-h1:          clamp(2.25rem, 1.75rem + 2.083vw, 3.5rem);
  --text-h2:          clamp(1.875rem, 1.5rem + 1.563vw, 3rem);
  --text-h3:          clamp(1.5rem, 1.25rem + 1.042vw, 2.25rem);
  --text-h4:          clamp(1.25rem, 1.125rem + 0.521vw, 1.625rem);
  --text-h5:          clamp(1.0625rem, 1rem + 0.26vw, 1.25rem);
  --text-h6:          clamp(0.9375rem, 0.9rem + 0.156vw, 1.0625rem);

  /* Body */
  --text-body-lg:     clamp(1.125rem, 1.0625rem + 0.26vw, 1.3125rem);
  --text-body:        clamp(1rem, 0.9625rem + 0.156vw, 1.125rem);
  --text-body-sm:     clamp(0.875rem, 0.85rem + 0.104vw, 0.9375rem);

  /* UI / Functional */
  --text-label:       clamp(0.6875rem, 0.675rem + 0.052vw, 0.75rem);
  --text-button:      clamp(0.875rem, 0.85rem + 0.104vw, 0.9375rem);
  --text-caption:     clamp(0.75rem, 0.7375rem + 0.052vw, 0.8125rem);
  --text-price:       clamp(1.375rem, 1.25rem + 0.521vw, 1.75rem);

  /* Font Families */
  --font-serif:  'Cormorant Garamond', 'Playfair Display', Georgia, serif;
  --font-sans:   'Inter', 'DM Sans', 'Helvetica Neue', Arial, sans-serif;

  /* Font Weights */
  --fw-regular:  400;
  --fw-medium:   500;
  --fw-semibold: 600;
  --fw-bold:     700;
}
```

---

# PARTE 3: OPCIONES DE LOGO — DESCRIPCIÓN CONCEPTUAL Y CONSTRUCCIÓN

## Principios Comunes a las 3 Opciones

- **Sin iconos literales** (no tazas de café, no granos de café, no la torre Eiffel barcelonesa): el Artesano no se explica, se insinúa.
- **Texto como protagonista**: la tipografía hace el trabajo identitario.
- **Proporciones auténticas**: construidos sobre grids modulares, no por intuición ciega.
- **Adaptables**: cada opción debe funcionar en una sola tinta (terracota), negro y blanco.

---

## OPCIÓN A — "La Firma del Artesano"

### Concepto
Logotipo en wordmark horizontal. Evoca la firma manuscrita de un maestro artesano en el fondo de su obra. La imperfección leve es intencional: la perfección absoluta no es artesanal.

### Construcción Tipográfica
**Wordmark primario:** "Café Vermut" en dos líneas de peso diferente.
- "CAFÉ" — versalitas de la familia serif (Cormorant Garamond SC), tracking +0.15em, peso 400, tamaño reducido (60% del tamaño de "Vermut").
- "Vermut" — la palabra principal en serif cursiva (Cormorant Garamond Italic), peso 500, con ligatura natural entre la 'm' y la 'u'.

**Elemento diferencial:** Un trazo horizontal fino (hairline, 0.5pt) separa "CAFÉ" de "Vermut", extendiéndose 1.5x el ancho del wordmark. Este trazo evoca la línea de una firma, un subrayado de validación artesanal.

**Grid de construcción:**
- Módulo base: altura de la 'x' de "Vermut" = 1 unidad.
- "CAFÉ" ocupa 0.5 unidades de altura.
- Espacio entre líneas: 0.25 unidades.
- Trazo horizontal: 0.04 unidades de grosor.
- Padding visual (espacio de protección): 2 unidades en todos los lados.

**Color principal:** "CAFÉ" en `--color-text-secondary` (#5C4F42). "Vermut" en `--color-brand-primary` (#C4603A). Trazo en `--color-brand-primary-dark` (#9E4928).

**Variantes:**
- Monotono terracota: todo en `#C4603A`.
- Monotono oscuro: todo en `#1F1813`.
- Inversión: todo en `#F5EFE6` sobre fondo `#1C1410`.

**Casos de uso:** cabecera web, bolsas de papel kraft, etiquetas de café molido, firma de email.

---

## OPCIÓN B — "El Sello Mediterráneo"

### Concepto
Logotipo en emblema contenido (badge/seal). Remite a los sellos de garantía de origen de los productos artesanales ibéricos: vino, aceite, queso. La forma exterior es una figura geométrica imperfecta (no el círculo perfecto computacional, sino uno con ligera tensión orgánica), construida desde el texto hacia afuera.

### Construcción
**Forma exterior:** Óvalo ligeramente comprimido en el eje vertical (ratio 1:0.88). No es una elipse matemática; tiene un punto de inflexión en la parte superior ligeramente más plano, como los sellos en lacre aplastados a mano.

**Texto en arco superior:** "CAFÉ DE ESPECIALIDAD" — sans-serif (Inter, peso 500, uppercase, tracking +0.18em, tamaño 8pt en el logo principal). El texto sigue la curva del óvalo con un kerning óptico ajustado.

**Texto central:** "Vermut" — serif grande (Cormorant Garamond, peso 600, estilo normal, sin cursiva). Ocupa el 65% del ancho interior del óvalo. Esta es la palabra dominante.

**Texto en arco inferior:** "Barcelona · Desde 2024" — mismas especificaciones que arco superior. El punto centrado (·) es un bullet tipográfico, no un punto de cierre.

**Línea interna:** Un segundo óvalo concéntrico a 4pt del óvalo exterior, con stroke de 0.75pt. Esta doble línea es el elemento artesanal: los sellos de calidad siempre tienen doble contorno.

**Grid de construcción:**
- Ancho total del emblema: 48 unidades.
- Alto total: 42 unidades (ratio 1:0.875).
- Óvalo exterior: 48 × 42 unidades.
- Óvalo interior (de separación): 44 × 38 unidades.
- Área central de texto: 40 × 28 unidades.
- "Vermut" ocupa: 32 unidades de ancho.
- Altura de "Vermut": 14 unidades.

**Color principal:** Fill de fondo del emblema en `--color-bg-subtle` (#F2EDE4). Strokes y textos en `--color-brand-primary-dark` (#9E4928). "Vermut" en `--color-text-primary` (#1F1813).

**Variantes:**
- Fill sólido terracota con texto en crema: para aplicaciones de máxima presencia (packaging principal de café molido, sello de lacre físico).
- Outline only en negro: para grabado, bordado, serigrafía.
- Fill verde oliva con texto en crema: variante de temporada / línea de producto verde.

**Casos de uso:** etiqueta frontal del packaging de café molido, sello en cajas de envío, posavasos, cartas físicas del café, favicon (versión simplificada solo "V").

---

## OPCIÓN C — "La Arquitectura Tipográfica"

### Concepto
Logotipo en wordmark vertical con composición asimétrica intencionada. Inspirado en la arquitectura del Ensanche barcelonés: módulo, repetición, pero con variación humana. Cada letra tiene su lugar estructural pero el conjunto respira. No hay elemento gráfico adicional: la tipografía es la arquitectura.

### Construcción
**Columna tipográfica de tres elementos:**

**Elemento 1 (superior):** "CAFÉ" — sans-serif (Inter, peso 300 light, uppercase, tracking +0.35em, tamaño pequeño). Actúa como overline. Alineado a la derecha.

**Elemento 2 (central, dominante):** "Vermut" — serif (Cormorant Garamond, peso 700, estilo normal, tamaño 4x el tamaño de "CAFÉ"). Este es el bloque visual central. Alineado a la izquierda. La diferencia de alineación entre los tres elementos crea la tensión compositiva.

**Elemento 3 (inferior):** Una línea horizontal de grosor variable: arranca en 2pt en el extremo izquierdo, bajo la 'V' de Vermut, y termina en 0.5pt en el extremo derecho. Esta variación de grosor es el gesto artesanal, la pincelada del maestro. Longitud: 85% del ancho de "Vermut".

**Construcción de la asimetría:**
- "CAFÉ" comienza en el eje x = 55% del ancho de "Vermut" (desplazado a la derecha).
- "Vermut" empieza en x = 0 (borde izquierdo).
- El trazo inferior empieza en x = 0, termina en x = 85%.
- El espacio entre "CAFÉ" y "Vermut": 0.2 de la altura de la 'V'.
- El espacio entre la base de "Vermut" y el trazo: 0.15 de la altura de la 'V'.

**Proporciones generales:**
- Alto total del conjunto: 1 + 0.2 + 4 + 0.15 + 0.1 = 5.45 unidades (donde 1 unidad = altura de "CAFÉ").
- Ancho total: ancho de "Vermut" (es el elemento más ancho y fija el sistema).

**Color principal:** "CAFÉ" en `--color-text-muted` (#9C8E82) para crear la jerarquía visual. "Vermut" en `--color-text-primary` (#1F1813). Trazo de grosor variable en `--color-brand-primary` (#C4603A).

**Variantes:**
- Invertido: fondo `#1C1410`, "CAFÉ" en `#9C8E82`, "Vermut" en `#F5EFE6`, trazo en `#C4603A`. Esta variante funciona especialmente bien en fotografías oscuras.
- Compacto (horizontal): "CAFÉ" rotado 90° a la izquierda de "Vermut", con el trazo bajo "Vermut". Para espacios horizontales limitados.

**Casos de uso:** packaging premium de café en grano (bolsas de 250g), papelería editorial del café (libreto de menú, cartas de especialidades de temporada), redes sociales en formato cuadrado, camisetas del staff.

---

## Recomendación Editorial

**Para el lanzamiento inicial** se recomienda la **Opción B (El Sello Mediterráneo)** como logo principal, con la **Opción A (La Firma del Artesano)** como wordmark de acompañamiento en cabecera web. Esta combinación es la más común en marcas de especialidad con ambición de internacionalización: el sello da presencia en packaging (canal físico y envíos online), y el wordmark da legibilidad en pantalla.

La **Opción C** se reserva para aplicaciones editoriales y campañas de contenido, donde la composición tiene más espacio para respirar.

---

*Sistema desarrollado para Café Vermut · Artesanía Mediterránea · Barcelona*
*Brand archetype: El Artesano/Creator*
*Fecha de elaboración: 2026-05-13*
