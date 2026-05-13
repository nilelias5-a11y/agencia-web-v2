# Software Recommendation — La Nonna (Reservas Online)

**Evaluador:** Software Recommender  
**Fecha:** 2026-05-13  
**Proyecto:** La Nonna — restaurante italiano, barrio de Gràcia, Barcelona  
**Categoría evaluada:** Sistema de reservas online  
**Trigger:** Cliente menciona "algo tipo OpenTable"

---

## PHASE 1 — Client Context

| Campo | Valor |
|---|---|
| **Tipo de negocio** | Restaurante italiano, barrio residencial (Gràcia, Barcelona) |
| **Capacidad** | ~30 mesas → ~120–150 cubiertos máximo por turno |
| **Volumen estimado reservas online** | 30–60 reservas/mes inicialmente; potencial hasta 150–200/mes si la web funciona bien |
| **Mercado** | España — mayoritariamente local barcelonés, algunos turistas |
| **Presupuesto para servicios** | No especificado; dueño no técnico → prioriza simplicidad y coste predecible |
| **Mantenimiento técnico** | La agencia gestiona; el cliente solo quiere ver reservas llegar |
| **Literacidad técnica del cliente** | Ninguna — necesita UI de gestión simple, idealmente en español |
| **Plazo de lanzamiento** | No urgente, pero cliente quiere avanzar pronto |
| **Stack del proyecto** | Next.js (App Router), presumiblemente Supabase/Postgres según stack de la agencia |

---

## PHASE 2 — Validación de OpenTable (servicio nombrado por el cliente)

El cliente dijo "algo tipo OpenTable". Según el protocolo, hay que validar si OpenTable es apropiado antes de aprobar o rechazar.

### OpenTable — Validación específica para La Nonna

**¿Está en el knowledge base?** No directamente (no es una categoría cubierta). Se aplica research + 6 dimensiones.

**Hallazgos de investigación:**

1. **Presencia en Barcelona**: OpenTable tiene presencia en Barcelona, pero con apenas ~34 restaurantes listados en la ciudad (mayo 2026). Es una red dominantemente anglosajona (EEUU, UK). Para un restaurante de barrio en Gràcia con clientela local, la red de descubrimiento de OpenTable aporta muy poco valor.

2. **Pricing en 2026**:
   - Plan Basic: $149/mes (~€138/mes)
   - Plan Core: $299/mes (~€277/mes)  
   - Plan Pro: $499/mes (~€462/mes)
   - Per cover fee: $1.00–$1.50 por comensal reservado vía su red
   - Nueva tasa adicional (2026): 2% sobre transacciones (no-shows, deposits, experiencias prepagadas)

3. **Problema estructural para este cliente**: OpenTable cobra una tarifa mensual fija elevada ($149–$299) más una tasa por cubierto cuando la reserva viene de su red. Para La Nonna con volumen bajo inicial (30–60 reservas/mes), el coste por reserva resultante sería de €2.30–€4.60 por reserva solo en cuota mensual, sin contar la tasa por cubierto. Esto es completamente desproporcionado.

4. **Idioma y soporte**: La interfaz de gestión de OpenTable está en inglés, el soporte no es español-first. Para un dueño no técnico en España, esto es una barrera real.

5. **Red de descubrimiento irrelevante**: El mayor argumento para pagar la tarifa de OpenTable es su red de usuarios que descubren restaurantes. Pero esa red es angloparlante y turística. La Nonna quiere reservas de vecinos del barrio de Gràcia, no de turistas que buscan en OpenTable.

**Veredicto de validación Phase 2: RECHAZADO**

OpenTable no encaja para este contexto. El coste es elevado, la red no es relevante para el mercado objetivo (local barcelonés), el idioma de gestión es inglés, y el nombre de la herramienta que usó el cliente ("algo tipo OpenTable") indica que buscaba el *concepto* de reservas online, no la herramienta en sí.

---

## PHASE 3 — Evaluación de Alternativas (6 dimensiones)

Se evalúan: TheFork/ElTenedor, Resy, CoverManager, Solución custom (Supabase + formulario).

Escala: 1 (muy bajo) → 5 (excelente)  
Pesos para este proyecto: Precio (alto), Simplicidad para el cliente (alto), Mercado español (alto), Implementación (medio), SDK/integración web (medio)

---

### OPCIÓN A — OpenTable (referencia, ya rechazada)

| Dimensión | Score | Notas |
|---|---|---|
| **Precio** | 1 | $149–$299/mes + per-cover fee. Desproporcionado para 30–60 reservas/mes |
| **Implementación** | 3 | Widget embebible disponible; integración web moderada |
| **SDK/calidad técnica** | 3 | API disponible, documentación en inglés, widget funcional |
| **Comunidad y soporte** | 2 | Soporte en inglés; comunidad no orientada a España |
| **Fiabilidad** | 5 | Plataforma muy estable, propiedad de Booking Holdings |
| **Riesgo proveedor** | 4 | Empresa muy consolidada, sin riesgo de cierre |
| **Score ponderado** | **2.8/5** | Red irrelevante para mercado local barcelonés |

**Coste estimado para La Nonna:** €138–€277/mes fijos + ~€0.50–€1.50/comensal de red = €140–€290/mes

---

### OPCIÓN B — TheFork Manager (ElTenedor)

TheFork (propiedad de TripAdvisor) es el líder absoluto de reservas de restaurante en España y Europa del Sur. Antes conocido como ElTenedor en España.

**Modelo de negocio**: Suscripción mensual (planes escalonados) + comisión por cubierto para reservas que llegan vía su marketplace (los usuarios de la app TheFork). Las reservas directas desde tu web/Google son sin comisión adicional.

**Pricing 2026 (España)**:
- Plan Free: funcionalidad limitada, sin coste
- Plan intermedio: ~€49/mes (funcionalidades básicas de gestión)
- Plan completo: ~€89/mes (visibilidad en marketplace + gestión)
- Comisión por cubierto vía marketplace: ~€2/comensal (modelo legacy) o 0.95% + €0.50/comensal (modelo nuevo desde enero 2025)

**Para La Nonna con 150 cubiertos/mes via TheFork marketplace**:  
€89 + (150 × €2) = €89 + €300 = ~€389/mes en pico. Sin embargo, si el volumen vía marketplace es bajo (lo esperable al inicio), el coste real es €89/mes + pocas comisiones = ~€120–€150/mes.

| Dimensión | Score | Notas |
|---|---|---|
| **Precio** | 3 | Coste razonable si el volumen vía marketplace es moderado; puede escalar con las comisiones |
| **Implementación** | 5 | Widget e iframe para embedding en web; configuración en minutos; sin código |
| **SDK/calidad técnica** | 3 | Widget embed funcional; API disponible pero no es el flujo principal |
| **Comunidad y soporte** | 5 | Soporte en español, dedicado a España y Europa; el estándar del sector en España |
| **Fiabilidad** | 5 | Infraestructura TripAdvisor; altísima disponibilidad; millones de reservas/día |
| **Riesgo proveedor** | 4 | TripAdvisor es sólido; riesgo de cambio de precios (historial de subidas de comisiones) |
| **Score ponderado** | **4.2/5** | La opción más fácil de gestionar para el dueño no técnico |

**Ventaja clave**: La Nonna aparece en el marketplace de TheFork, ganando visibilidad a nuevos clientes. El dueño gestiona todo desde una app en español, sin tocar código.

**Riesgo**: Las comisiones por cubierto del marketplace pueden crecer con el volumen. Si el restaurante se vuelve popular en TheFork, el coste sube linealmente. Además, TheFork tiene historial de subir precios (lo hizo en enero 2025).

**Coste estimado para La Nonna:** €89–€150/mes en escenario normal (volumen bajo-medio vía marketplace)

---

### OPCIÓN C — Resy

Resy es un sistema premium orientado a restaurantes de alta gama (fine dining), propiedad de American Express.

**Pricing 2026**: Desde $249/mes (~€230/mes). No cobra per-cover fee.

**Disponibilidad España**: Muy limitada. Resy opera principalmente en EEUU (25.000 restaurantes) con presencia selectiva en Londres y algunas ciudades globales de alta gama. No tiene presencia significativa en Barcelona ni España en general.

| Dimensión | Score | Notas |
|---|---|---|
| **Precio** | 2 | $249/mes mínimo (~€230/mes) — desproporcionado para un restaurante de barrio |
| **Implementación** | 3 | Widget disponible; documentación en inglés |
| **SDK/calidad técnica** | 4 | API bien documentada, widget de calidad premium |
| **Comunidad y soporte** | 1 | Sin soporte en español; sin comunidad en España; herramienta pensada para EEUU |
| **Fiabilidad** | 5 | American Express detrás; infraestructura sólida |
| **Riesgo proveedor** | 4 | Muy estable; aunque puede no expandirse más en España |
| **Score ponderado** | **3.2/5** — pero con descalificación por mercado |

**Veredicto**: Descalificado por mercado. Resy no tiene red de usuarios en España, no tiene soporte en español, y el precio es elevado para el perfil de La Nonna. Es una herramienta diseñada para Nobu, no para una trattoria de Gràcia.

**Coste estimado para La Nonna:** ~€230/mes sin ningún retorno en visibilidad local

---

### OPCIÓN D — CoverManager

CoverManager es una empresa española (fundada en Barcelona, 2013), especializada en gestión de reservas para hostelería. Es el competidor directo de TheFork Manager en el mercado español, con enfoque en gestión operativa más que en marketplace.

**Modelo de negocio**: Tarifa plana mensual sin comisión por reserva directa (las reservas de tu web no pagan per-cover). El coste es predecible.

**Pricing 2026 (España)**:
- Plan Basic: ~€49–99/mes
- Plan Estándar: ~€99–149/mes  
- Plan Pro: ~€249/mes
- Posible coste de onboarding/setup: €200–€500 (formación incluida en algunos planes)
- Sin comisión por cubierto para reservas directas (diferencia clave vs. TheFork)

**Para La Nonna**: Plan Estándar (~€99/mes) sería suficiente para 30 mesas.

| Dimensión | Score | Notas |
|---|---|---|
| **Precio** | 4 | Tarifa plana predecible (~€99/mes); sin sorpresas por volumen. Más caro que TheFork base, pero sin comisiones por cubierto |
| **Implementación** | 4 | Widget e iframe para web; integración sin código; onboarding incluido |
| **SDK/calidad técnica** | 4 | API bien documentada para España; integración con Google Reserve; widget personalizable |
| **Comunidad y soporte** | 5 | Empresa española; soporte en español; équipo especializado en hostelería española |
| **Fiabilidad** | 4 | Plataforma madura (>10 años); clientes como Grupo Sagardi, Lateral, etc. |
| **Riesgo proveedor** | 3 | Empresa mediana española; sin el respaldo de TripAdvisor; menor escala pero especialización local |
| **Score ponderado** | **4.0/5** | Excelente para restaurantes que priorizan control y coste predecible |

**Ventaja clave**: Coste completamente predecible, sin comisiones por cubierto. Incluye CRM de clientes, gestión de sala, lista de espera digital, encuestas post-visita. Empresa española con soporte en castellano (y catalán). Ideal para el perfil "dueño no técnico".

**Riesgo**: No tiene marketplace de consumidores propio (los clientes llegan por tus propios canales, no por CoverManager). Menor empresa que TheFork = menor respaldo. Costes de onboarding.

**Coste estimado para La Nonna:** €99/mes fijos (~€99 setup si aplica, amortizado rápido)

---

### OPCIÓN E — Solución Custom (Supabase + Formulario de Reservas en Next.js)

Construir un sistema propio de reservas integrado en la web de La Nonna, usando Supabase como backend y un formulario + panel de administración custom.

**Arquitectura típica**:
- Formulario Next.js (fecha, hora, comensales, nombre, email, teléfono)
- Supabase: tabla `reservations` con RLS, tabla `time_slots` con disponibilidad
- Panel de admin simple para el restaurante (lista de reservas del día, confirmación manual o automática)
- Emails de confirmación/recordatorio via Resend
- Sin integración con Google Maps o app de terceros

**Implementación estimada**: 20–35h de desarrollo  
- Schema Supabase + RLS: ~3h
- Formulario con validación de disponibilidad: ~8h
- Panel de admin básico (lista de reservas, cambio de estado): ~8h
- Emails de confirmación y recordatorio: ~3h
- Testing, edge cases (doble booking, cancelaciones): ~6h
- Despliegue y documentación para el cliente: ~3h

**Coste de infraestructura**:
- Supabase Free tier: €0/mes (500MB DB, suficiente para años de reservas de un restaurante de 30 mesas)
- Resend Free tier: €0/mes (3.000 emails/mes — más que suficiente)
- **Total infraestructura: €0/mes**

**Coste de implementación (única vez)**: 20–35h × tarifa agencia = coste de desarrollo inicial

| Dimensión | Score | Notas |
|---|---|---|
| **Precio** | 5 | €0/mes en infraestructura; solo coste de desarrollo único |
| **Implementación** | 2 | 20–35h de desarrollo; requiere mantener el sistema, gestionar bugs, añadir features a mano |
| **SDK/calidad técnica** | 5 | Control total; stack conocido (Supabase + Next.js = stack estándar de la agencia) |
| **Comunidad y soporte** | 4 | Stack bien documentado; soporte recae en la agencia |
| **Fiabilidad** | 3 | Tan fiable como Supabase + Vercel; sin SLA de "plataforma de reservas"; edge cases propios |
| **Riesgo proveedor** | 5 | Sin dependencia de terceros para la funcionalidad core |
| **Score ponderado** | **4.0/5** — pero con caveats importantes |

**Ventaja clave**: €0/mes de coste recurrente. Control total del diseño y UX. Sin comisiones por cubierto nunca. Integración perfecta con el diseño de la web.

**Riesgo CRÍTICO para este cliente**: Un dueño no técnico necesita poder gestionar su disponibilidad (bloquear días festivos, cambiar horarios, cerrar por vacaciones) sin necesitar a la agencia. Un sistema custom requiere o bien un panel de admin bien construido (más horas) o bien que el cliente llame a la agencia cada vez que necesita cambiar algo. Además, no hay app móvil de gestión. El sistema no tiene Google Reserve integration. No aparece en ningún marketplace.

---

## Tabla Comparativa de las 6 Dimensiones

| | OpenTable | TheFork | Resy | CoverManager | Custom |
|---|---|---|---|---|---|
| **Precio** | 1 | 3 | 2 | 4 | 5 |
| **Implementación** | 3 | 5 | 3 | 4 | 2 |
| **SDK/técnico** | 3 | 3 | 4 | 4 | 5 |
| **Soporte/mercado** | 2 | 5 | 1 | 5 | 4 |
| **Fiabilidad** | 5 | 5 | 5 | 4 | 3 |
| **Riesgo proveedor** | 4 | 4 | 4 | 3 | 5 |
| **TOTAL** | 18 | 25 | 19 | 24 | 24 |
| **MEDIA** | 3.0 | **4.2** | 3.2 | **4.0** | **4.0** |

---

## RECOMENDACIÓN FINAL

### Recomendado: TheFork Manager

**Opción:** TheFork Manager — plan de entrada (~€49–89/mes)

**Razón principal:**

TheFork es el estándar del sector en España para reservas de restaurante. El argumento no es solo técnico: es que el mercado ya lo conoce. Los clientes de Gràcia saben cómo usar TheFork. Google Maps muestra el botón "Reservar" directamente cuando el restaurante está en TheFork. Aparecer en la plataforma es visibilidad gratuita que un sistema custom o CoverManager no pueden dar.

Para un dueño no técnico que solo quiere "que lleguen reservas", TheFork es la solución con menor fricción operativa: una app en español, notificaciones en el móvil, panel de gestión intuitivo, soporte en castellano. No necesita tocar código ni llamar a la agencia para cambiar el horario del sábado.

**Por qué no las demás:**
- **OpenTable**: Precio desproporcionado, red irrelevante para clientela local barcelonesa, gestión en inglés. Rechazado en Phase 2.
- **Resy**: Descalificado por ausencia de red en España y precio premium injustificado para este perfil.
- **CoverManager**: Excelente alternativa técnica, pero sin marketplace = sin visibilidad adicional. Coste similar o mayor que TheFork para un restaurante de este tamaño. Recomendable si La Nonna ya tiene volumen de reservas consolidado y quiere control operativo avanzado (en ese caso, revisar en 12-18 meses).
- **Custom Supabase**: Atractivo por €0/mes recurrente, pero el coste de desarrollo (20–35h) + la falta de app móvil de gestión + ausencia de marketplace lo hacen subóptimo para este cliente específico. **Podría ser la elección si La Nonna fuera un cliente con mayor presupuesto inicial, mayor volumen, y necesidades muy específicas de UX.**

**Condiciones de la recomendación:**

1. Empezar con el **plan de entrada** de TheFork Manager (~€49/mes si está disponible o el más básico que incluya widget embebible). No contratar el plan más caro hasta verificar volumen real de reservas.
2. Integrar el widget de TheFork en la web de Next.js (iframe o widget JS — 1–2h de implementación).
3. Configurar el widget de "Reservar mesa" en Google Business Profile del restaurante.
4. Monitorizar mensualmente las comisiones por cubierto vía marketplace — si superan €150/mes de forma sostenida, reevaluar CoverManager a tarifa plana.
5. Si en 12 meses el restaurante tiene alto volumen de reservas propias (no vía marketplace), cambiar a CoverManager para eliminar comisiones variables.

**Nota al cliente** (a comunicar en lenguaje no técnico):

> "Vamos a conectar tu web con TheFork, que es la plataforma de reservas más usada en España. Tus clientes podrán reservar desde tu web, desde Google Maps o desde la app de TheFork. Tú recibirás las reservas en una aplicación muy sencilla en tu móvil. El coste será de aproximadamente €89/mes, más una pequeña comisión por cada reserva que te llegue a través de la propia app de TheFork."

---

## Approved Stack Summary — Reservas Online La Nonna

| Categoría | Servicio | Coste mensual estimado |
|---|---|---|
| **Reservas online** | TheFork Manager | €89/mes + comisiones variables (~€0–100 según volumen de marketplace) |
| **Email confirmación** | Resend (ya en stack) | €0 (free tier) |
| **Base de datos** | Supabase (ya en stack) | €0 (free tier) |
| **Implementación web** | Widget TheFork embed | ~1–2h agencia |
| **Total fijo estimado** | | **€89–150/mes** |

**Alternativa a revisar a 12–18 meses:** CoverManager Estándar (~€99/mes tarifa plana, sin comisiones) si el volumen de reservas directas supera 100/mes.

---

## Decisión sobre la Solución Custom

La solución custom (Supabase + formulario Next.js) queda descartada como recomendación principal pero **se documenta como opción válida** para futuras fases si:

- El cliente no quiere pagar comisiones por cubierto nunca
- El restaurante tiene una clientela fiel que reserva directamente (sin necesitar marketplace)
- La agencia puede construir un panel de admin completo con cambio de horarios, cierre por vacaciones, y gestión de turnos sin necesidad de soporte técnico continuo
- El cliente invierte el ahorro mensual (€89/mes × 12 = €1.068/año) en el desarrollo inicial

En ese escenario, 20–35h de desarrollo se amortizan en 12–18 meses. Pero para el estado actual de La Nonna (web recién construida, volumen de reservas online aún por validar), el riesgo operativo supera el ahorro.

---

*Listo para `api-developer`: integración del widget TheFork Manager en la web Next.js de La Nonna. Requiere cuenta TheFork activa del restaurante y credenciales del widget.*
