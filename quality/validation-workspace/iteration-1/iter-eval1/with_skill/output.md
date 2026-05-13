# Iteration Round 1 Summary

**Project:** La Nonna
**Feedback received:** 2026-05-13
**Source:** Client (primera revisión de la web)

---

## Processed Items: 6 total

- P1 Immediate: 0
- P2 This iteration: 4 — Assigned to: `ui-designer`, `frontend-developer`, `content-manager`
- P3 Next iteration: 0
- Scope flags: 2 — Escalated to `director`

---

## Item Log

| ID | Category | Priority | Description | Assigned to | Status |
|----|----------|----------|-------------|-------------|--------|
| R1-001 | visual | P2 | Hero background color demasiado oscuro — ajustar a tono más cálido y luminoso | `ui-designer` | ⏳ Open |
| R1-002 | visual | P2 | Logo más grande en header — se pierde visualmente | `ui-designer` | ⏳ Open |
| R1-003 | content | P2 | Añadir 12 fotos reales del restaurante (enviadas por WhatsApp) en sección galería / donde corresponda | `frontend-developer` + `content-manager` | ⏳ Open — pendiente recibir fotos |
| R1-004 | functional | P2 | Formulario de reservas: añadir validación máximo 8 personas por reserva online, con mensaje de error explícito si se supera | `frontend-developer` | ⏳ Open |
| R1-005 | scope-new | Escalate | Nueva sección "Últimas reseñas de Google Maps" — mostrar las 5 más recientes | → `director` | ⏳ Scope flag |
| R1-006 | scope-new | Escalate | Versión en inglés de la web | → `director` | ⏳ Scope flag — ver nota |

---

## Scope Flags Requiring Director Decision

### Scope Flag R1-005 — Sección de reseñas Google Maps

**Item:** Añadir sección con las últimas 5 reseñas de Google Maps.

**Why it's out of scope:** El brief original no incluía ninguna sección de reseñas externas. Requiere integración con la Google Places API (o web scraping no recomendable), gestión de API key, posible coste de uso, y lógica de refresco de datos. No fue mencionado en el briefing inicial.

**Estimated effort:** 4–6 horas (setup Places API, componente server-side, diseño de la sección, actualización de traducciones ES/EN).

**Recommendation:** Incluir en esta misma iteración si el cliente acepta un pequeño sobrecoste, o diferir a Fase 2 (post-lanzamiento) con una solución más económica (widget embebido de Google Reviews de terceros, ej. Elfsight, ~$5/mes).

---

### Scope Flag R1-006 — Versión en inglés

**NOTA IMPORTANTE:** Según el requirements document (requirements.md, sección 5), el soporte multiidioma castellano + inglés fue **confirmado explícitamente por el cliente el 2026-05-11** como parte del brief original. Adicionalmente, el proyecto ya tiene implementado `next-intl v4` con `es.json` + `en.json` completos y páginas EN funcionales en `/en/*` (estado Fase 3, 2026-05-11).

**Conclusión:** Este ítem NO es scope creep real. El cliente lo menciona como si fuera nuevo, pero ya está implementado. Se requiere una comunicación al cliente para informarle de que la versión inglesa ya existe y mostrarle cómo acceder a ella.

**Action:** `director` o `project-manager` debe confirmar con el cliente si la versión EN ya visible cubre su necesidad, o si hay algo concreto que falta (ej. textos que no quedaron traducidos, o la URL `/en/` no es visible en la navegación).

**Assigned to:** `director` / `project-manager` para comunicación al cliente. Si hay ajustes menores de visibilidad del selector de idioma, asignar a `frontend-developer`.

---

## Duplicates from Previous Rounds

Ninguno. Este es el Round 1, no existen rondas anteriores.

---

## Detail by Item

### R1-001 — Hero background color (visual / P2)

El cliente describe el fondo actual como "muy oscuro" y pide que "transmita calidez". El hero es la primera impresión del restaurante y tiene impacto directo en conversión.

**Action para `ui-designer`:** Proponer 2–3 alternativas de color/tratamiento para el hero que mantengan legibilidad del texto pero aporten calidez (tonos tierra, ámbar suave, crema). Confirmar con el cliente antes de implementar.

**Action para `frontend-developer`:** Implementar el color aprobado en `globals.css` o variable CSS correspondiente.

---

### R1-002 — Logo más grande (visual / P2)

El logo se pierde en el header actual. Ajuste de tamaño de CSS, posiblemente también revisar proporciones en mobile vs desktop.

**Action para `ui-designer`:** Definir tamaños recomendados (px o rem) para desktop y mobile. Verificar que el tamaño mayor no rompe el layout del header.

**Action para `frontend-developer`:** Aplicar los nuevos tamaños en el componente Header.

---

### R1-003 — Fotos reales del restaurante (content / P2)

El cliente ha enviado 12 fotos por WhatsApp. Los placeholders de Unsplash deben ser reemplazados. Este ítem estaba previsto en el brief (fotografías del local disponibles, pendiente recibirlas formalmente).

**Action para `content-manager` / `director`:** Solicitar las fotos en formato digital al cliente (Google Drive, WeTransfer, o email). Verificar calidad mínima (resolución, iluminación) antes de integrar.

**Action para `frontend-developer`:** Una vez recibidas las fotos, optimizarlas (WebP, compresión), subirlas a `/public/images/` y reemplazar los `<Image>` con src Unsplash.

**Blocker:** Ítem bloqueado hasta recibir las fotos. No se puede completar sin los assets.

---

### R1-004 — Validación máximo 8 personas en formulario de reservas (functional / P2)

El formulario de reservas ya tiene validación con Zod. Solo hay que añadir la regla `max(8)` al campo de número de personas, con mensaje de error claro en ES y EN.

**Action para `frontend-developer`:**
- En el schema Zod del formulario de reservas: `personas: z.number().min(1).max(8, 'Máximo 8 personas por reserva online. Para grupos más grandes, llámanos.')`
- Añadir la misma cadena traducida en `en.json`
- Verificar que el mensaje de error aparece inline bajo el campo (no como alert)

---

## Estimated Completion

- R1-001, R1-002, R1-004: 1–2 días de desarrollo una vez aprobados los cambios de diseño
- R1-003: Bloqueado hasta recibir fotos del cliente (ETA desconocido)
- R1-005: Pendiente decisión de director (scope / presupuesto)
- R1-006: Requiere comunicación al cliente — probablemente 0 horas de desarrollo adicional (ya implementado)

**Estimación global:** 1–3 días de trabajo, excluyendo el blocker de fotos y las decisiones de scope.

---

## Blocking Anything?

**Yes.**

- **R1-003** (fotos) bloquea la entrega final de la web. Sin fotografías reales, el sitio no puede considerarse listo para lanzamiento. Acción urgente: solicitar las fotos al cliente esta semana.
- **R1-005** (reseñas Google) requiere decisión de director antes de que el equipo pueda actuar.
- **R1-006** (inglés) requiere comunicación al cliente para aclarar que ya está implementado — evita trabajo innecesario.

---

## Communication Note for Director

Recomendación: al responder al cliente, aprovechar para:

1. Confirmarle que todos sus comentarios han sido recibidos y priorizados.
2. Pedirle las 12 fotos en alta resolución (Google Drive / WeTransfer).
3. Informarle de que la versión inglesa **ya está implementada** y mostrarle cómo acceder.
4. Presentarle la opción de las reseñas Google Maps como posible add-on con coste estimado.
5. Preguntar su preferencia sobre el tono cálido del hero antes de implementar (evitar un segundo ciclo de revisión en este punto).
