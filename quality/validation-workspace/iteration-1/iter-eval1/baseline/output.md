# Feedback Round 1 — La Nonna
**Fecha:** 2026-05-13
**Proyecto:** Web La Nonna (Next.js 15, App Router, next-intl)
**Estado del proyecto:** Fase 3 en curso — build limpio, 17 páginas SSG, traducciones ES+EN completas

---

## Análisis del feedback del cliente

El cliente ha enviado 6 items en su mensaje. A continuación se clasifican, priorizan y asignan.

---

## Clasificación de cada item

### ITEM 1 — Color de fondo del hero "muy oscuro"
- **Tipo:** Ajuste visual / diseño — dentro del scope original
- **Prioridad:** Alta (afecta primera impresión, percepción de marca)
- **Scope:** Dentro del scope — cambio de CSS, ninguna funcionalidad nueva
- **Complejidad:** Baja (1–2h)
- **Responsable:** Frontend / diseño
- **Iteración:** Esta iteración (Iter 2)
- **Acción:** Aclarar con el cliente qué tono de calidez busca (mostrar 2–3 opciones de color: crema, terracota suave, marfil) antes de ejecutar. Modificar variable CSS en `globals.css`.

---

### ITEM 2 — Logo más grande en el header
- **Tipo:** Ajuste visual / diseño — dentro del scope original
- **Prioridad:** Media-Alta
- **Scope:** Dentro del scope — ajuste de CSS/tamaño de imagen
- **Complejidad:** Muy baja (30 min)
- **Responsable:** Frontend
- **Iteración:** Esta iteración (Iter 2)
- **Acción:** Aumentar tamaño del logo en el Header component. Verificar que no rompe responsive en móvil.

---

### ITEM 3 — Fotos del restaurante (12 fotos por WhatsApp)
- **Tipo:** Contenido — pendiente previsto, dentro del scope original
- **Prioridad:** Alta (los placeholders Unsplash no son apropiados para entrega final)
- **Scope:** Dentro del scope — estaba explícitamente en los pendientes de Fase 3 ("Fotografías reales del cliente (placeholders Unsplash por ahora)")
- **Complejidad:** Media (recibir archivos, optimizar, sustituir placeholders, verificar alt texts)
- **Responsable:** Frontend (integración) + Cliente (entrega de fotos)
- **Iteración:** Esta iteración (Iter 2) — bloqueado hasta recibir las fotos
- **Acción:** Solicitar las fotos por un canal adecuado (no WhatsApp para archivos de trabajo — pedir Drive, WeTransfer o similar). Optimizar con next/image. Definir qué foto va en cada sección.

---

### ITEM 4 — Sección de reseñas de Google Maps (últimas 5)
- **Tipo:** Nueva funcionalidad — SCOPE CREEP
- **Prioridad:** Media (valor de negocio para conversión, pero no estaba en el briefing)
- **Scope:** FUERA DEL SCOPE ORIGINAL — el cliente lo reconoce explícitamente ("se me olvidó deciros cuando hicimos el briefing")
- **Complejidad:** Media-Alta
  - Opción A (estática): Copiar las 5 reseñas manualmente como contenido hardcoded o en JSON → rápido pero requiere actualización manual
  - Opción B (dinámica): Google Places API / Maps JavaScript API → requiere cuenta Google Cloud, API key, costes de API, lógica de fetching, caché
  - El cliente dice "últimas 5 reseñas" lo que implica dinamismo (Opción B), pero necesita confirmación
- **Responsable:** A definir tras conversación de scope
- **Iteración:** REQUIERE CONVERSACIÓN DE SCOPE antes de planificar
- **Acción:** Hablar con el cliente. Presentar dos opciones con sus implicaciones de tiempo y coste. Si acepta scope adicional, planificar para Iter 3.

---

### ITEM 5 — Web en inglés ("se pueda ver en inglés")
- **Tipo:** Ya implementado — aclaración necesaria
- **Prioridad:** Alta (pero puede ser un malentendido)
- **Scope:** YA ESTÁ IMPLEMENTADO — el proyecto tiene next-intl v4 con ES (default) + EN en `/en/*`, `es.json` + `en.json` completas, y LanguageSwitcher en el Header
- **Complejidad:** Ninguna (si el cliente no lo ha visto) — o baja (si hay algo que no funciona correctamente)
- **Responsable:** Frontend (demo/comunicación al cliente)
- **Iteración:** Esta iteración — aclaración inmediata
- **Acción:** URGENTE: Informar al cliente de que la web ya está en inglés. Mostrarle cómo funciona el LanguageSwitcher. Si no lo ha visto es posiblemente porque está revisando un entorno local sin el switcher visible. Verificar que el switcher es suficientemente prominente en la UI.

---

### ITEM 6 — Validación en formulario de reservas: máximo 8 personas
- **Tipo:** Ajuste funcional / UX — dentro del scope (el formulario ya existe, Zod ya está implementado)
- **Prioridad:** Media-Alta (regla de negocio real del restaurante)
- **Scope:** Dentro del scope — es una regla de negocio que debería haberse recogido en el briefing, pero la implementación es trivial dado que Zod ya valida el formulario
- **Complejidad:** Baja (30–60 min — añadir `.max(8)` en el schema Zod + mensaje de error en UI)
- **Responsable:** Frontend
- **Iteración:** Esta iteración (Iter 2)
- **Acción:** Añadir validación `personas: z.number().int().min(1).max(8, { message: 'Máximo 8 personas por reserva online. Para grupos más grandes, llámanos.' })` en el schema de reservas. Mostrar error inline en el campo. Aplicar también en EN.

---

## Tabla resumen

| # | Item | Tipo | Scope | Prioridad | Complejidad | Iteración | Responsable |
|---|------|------|-------|-----------|-------------|-----------|-------------|
| 1 | Color fondo hero | Ajuste visual | Dentro | Alta | Baja | Iter 2 | Frontend |
| 2 | Logo más grande | Ajuste visual | Dentro | Media-Alta | Muy baja | Iter 2 | Frontend |
| 3 | 12 fotos restaurante | Contenido pendiente | Dentro | Alta | Media | Iter 2 (bloqueado) | Frontend + Cliente |
| 4 | Reseñas Google Maps | Nueva funcionalidad | **FUERA** | Media | Media-Alta | Requiere conversación | A definir |
| 5 | Web en inglés | Ya implementado | N/A | Alta | Ninguna | Inmediato (aclaración) | Frontend (demo) |
| 6 | Máx. 8 personas reservas | Ajuste funcional | Dentro | Media-Alta | Baja | Iter 2 | Frontend |

---

## Items que requieren conversación de scope

### ITEM 4 — Reseñas de Google Maps
Este es el único item que representa scope creep real. El cliente lo reconoce ("se me olvidó deciros"). Puntos a aclarar en la conversación:

1. **¿Estáticas o dinámicas?**
   - Estáticas (hardcoded/JSON): se copian las 5 reseñas ahora, se actualizan manualmente cuando quiera cambiarlas. Coste: ~2–3h. Sin coste recurrente.
   - Dinámicas (Google Places API): siempre muestra las últimas 5. Requiere configurar Google Cloud, API key, lógica de caché (para no llamar la API en cada visita). Coste: ~6–8h + posible coste de API según volumen.

2. **¿Es para esta iteración o puede esperar?** — Recomendación: planificar para Iter 3 si se aprueba el scope adicional.

3. **¿Tiene impacto económico?** — Si el contrato es cerrado, este item debe presupuestarse aparte o intercambiarse por otro.

---

## Plan para esta iteración (Iter 2)

**Acciones inmediatas (esta semana):**

1. **Comunicar al cliente** que la web ya está en inglés — mostrar demo con LanguageSwitcher. Aclarar si hay algo que no ve correctamente.
2. **Iniciar conversación de scope** sobre las reseñas de Google Maps — presentar opciones A y B con tiempos y costes.
3. **Solicitar las fotos** por canal adecuado (Drive/WeTransfer), con indicación de qué sección corresponde a cada foto.

**Desarrollo Iter 2 (en paralelo o tras aclaraciones):**

4. Ajustar color fondo hero — proponer 2–3 opciones al cliente antes de ejecutar, o ejecutar la más obvia (crema/marfil cálido) y confirmar.
5. Aumentar tamaño del logo en Header.
6. Añadir validación máx. 8 personas en schema Zod del formulario de reservas (ES + EN).
7. Integrar las 12 fotos cuando el cliente las entregue.

**Bloqueado:**
- Fotos (esperando entrega del cliente)
- Reseñas Google Maps (esperando decisión de scope)

---

## Notas adicionales

- El cliente tiene una percepción positiva general ("me gusta bastante", "el menú ha quedado muy bien") — buen punto de partida para la conversación sobre scope creep.
- El hecho de que no haya visto el inglés sugiere que puede estar revisando la web en un entorno sin acceso completo, o que el LanguageSwitcher no es suficientemente visible. Vale la pena revisar su prominencia en la UI antes de la demo.
- Las fotos son críticas para la calidad final — los placeholders Unsplash no son apropiados para entregar al cliente. Priorizar su recepción.
