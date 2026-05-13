# Validation Results — Capa Orchestration v2
**Date:** 2026-05-13 | **Runs:** 20 (10 with-skill, 10 baseline)

---

## AGENTE 1: director.md

### Eval 1 — Café Vermut nuevo proyecto

**WITH SKILL:**
- Activa Phase 0 (web-researcher) ANTES de hablar al cliente
- Hace preguntas estratégicas: tono, audiencia, valores, qué debe hacer sentir la web
- Identifica competidores de referencia (Nomad, Bonanza, Satan's Coffee Corner)
- North star explícito: "¿Podría ganar un Awwwards Site of the Day?"
- Termina con client confirmation pattern: "¿Listo para responder las preguntas del briefing?"

**BASELINE:**
- Salta directamente a stack técnico (Next.js + Stripe + Prisma + Vercel)
- Da desglose de presupuesto (€600 + €900 + €50 + €300 + €150)
- Pide items prácticos: fotos, textos, dominio, referencias
- Propone llamada de 30 minutos esta semana

**Diferencia:** Alta. El skill previene "jump to solutions" antes de entender el cliente. La baseline responde como un freelancer competente; el skill responde como un director de agencia.

---

### Eval 2 — La Nonna feedback Fase 6

**WITH SKILL:**
- Categoriza: MINOR (color hero, logo pixelado) vs MAJOR (nueva sección = Phase 4 loop)
- Delega a agentes específicos: UI/Visual Designer, Asset/Tech Specialist, Content Strategist + UX Designer
- Declara que items 1-2 no requieren phase rollback, item 3 sí
- Draft de comunicación al cliente estructurada

**BASELINE:**
- También categoriza (bajo/medio-alto esfuerzo) con buena precisión
- Añade consejo práctico: git branch `fix/fase6-client-feedback`
- Pregunta si item 3 está en el contrato (scope check)
- Mención de change request documentado

**Diferencia:** Media. Ambos categorizan bien. El skill usa lenguaje de delegación de agentes y Phase Report. La baseline tiene insights prácticos útiles (scope check, git branch) que el skill no produce.

---

## AGENTE 2: project-manager.md

### Eval 1 — Wireframe blocker

**WITH SKILL:**
- Progress Report estructurado (tabla de estado)
- BLOCKER-001 log con todos los campos (phase, description, impact, responsible, date)
- Aplica la regla del 20% threshold para justificar escalada
- Escalada explícita al director (sin comunicar al cliente directamente)
- Sugiere que el dev use tiempo para Phase 3 prep

**BASELINE:**
- 5 pasos de acción prácticos
- Template de email de seguimiento sugerido
- Menciona cláusula contractual (client review period)
- También sugiere mantener al developer ocupado con prep work

**Diferencia:** Media-alta. El skill produce output estructurado y machine-readable. La baseline da más consejo práctico (template de email, protección contractual). El skill respeta el Harmony Protocol (escalada al director, no al cliente directamente).

---

### Eval 2 — Fast-track timeline Ana López

**WITH SKILL:**
- Milestone table con Owner/Duration/Start/End/Status columns
- Aplica Fast-Track compression explícitamente (fases 0-2.5 en paralelo)
- Buffer strategy: 25 May interno vs 27 May hard deadline
- Sección de blockers críticos (assets del cliente, rondas de feedback)

**BASELINE:**
- Timeline detallado día a día por semana
- Menciona team composition (1 diseñador, 1 developer, 1 PM)
- Tabla de riesgos (riesgo + mitigación)
- Supuestos explícitos

**Diferencia:** Media. Ambos producen buenos timelines. El skill usa el formato de milestone table estandarizado. La baseline tiene la risk table y team composition que el skill no incluye.

---

## AGENTE 3: coordinator.md

### Eval 1 — Brand → UX handoff

**WITH SKILL:**
- Handoff Package Structure completo (7 campos del template)
- Añade constraints: wireframes en escala de grises, sin decisiones de color, sin UI final
- Open questions identificadas: ¿reservas o solo contacto? ¿idiomas? ¿assets reales?
- 4-point check pasado explícitamente (completeness, format, clarity, compression)

**BASELINE:**
- Mensaje de handoff bien estructurado pero informal
- Incluye páginas a wireframear (Home, Carta, Nosotros, Contacto)
- Puntos a tener en cuenta tipografía/fotografía
- Sin constraints explícitas, sin open questions, sin checklist

**Diferencia:** Alta. Las constraints y open questions del skill previenen errores caros downstream. El UX designer sabiendo "no decisions sobre color" y "¿hay reservas online?" puede trabajar sin hacer asunciones que luego requieran rework.

---

### Eval 2 — Frontend vs SEO conflict

**WITH SKILL:**
- Conflict Resolution Protocol paso a paso
- Identifica el punto exacto: `loading="lazy"` en above-the-fold = antipatrón de LCP
- Resolución técnica con código Next.js (`priority={true}`)
- Declara "no requiere escalada al director"
- Documenta como precedente reutilizable

**BASELINE:**
- También resuelve correctamente con el mismo código (`priority={true}`, `fetchpriority="high"`)
- Explica por qué ambos tienen razón parcialmente
- Corrige al frontend-developer (confunde TTL con LCP)
- Aclara que el riesgo de crawling con `loading="lazy"` nativo es menor de lo que teme el SEO

**Diferencia:** Baja. Ambos dan la resolución técnica correcta. El skill añade protocolo y precedente; la baseline da más contexto educativo. En este eval específico el skill no añade valor técnico diferenciado.

---

## AGENTE 4: task-router.md

### Eval 1 — Model selection para 4 tareas

**WITH SKILL:**
- (a) Opus, (b) Haiku, (c) Sonnet, (d) **Haiku** ← corrección vs baseline
- Token Report con est. tokens por agente: ~6.100 total para la fase
- Justificación del ahorro: "70% vs usar Sonnet para todo"

**BASELINE:**
- (a) Opus, (b) Haiku, (c) Sonnet, (d) **Sonnet** ← diferencia clave
- Buena justificación pero sin Token Report
- (d) asignado a Sonnet porque "requiere coherent narrative" — razonable pero incorrecto según el framework

**Diferencia:** Media. La asignación correcta de Haiku para (d) es la diferencia clave — el progress report tiene formato predefinido y datos estructurados, exactamente el caso de uso de Haiku. El Token Report añade accountability de costes que el baseline no produce.

---

### Eval 2 — Context compression Fase 4→4.5

**WITH SKILL:**
- Compressed Context Package template con "Decisions locked (do not revisit)" section
- Elimina competidores completamente (solo análisis terminado, no accionable para QA)
- ~99% de reducción (~52.000 → ~500 tokens)
- Tabla "What was dropped and why" explicando cada decisión

**BASELINE:**
- También muy comprimido, claims similar reducción (~51.600 → ~300 palabras)
- Mantiene resumen del análisis competitivo (top takeaways)
- Añade "What is NOT in scope for this phase" — útil para el QA reviewer
- Sin template formal pero estructura clara

**Diferencia:** Baja. Ambos logran comprensión similar. La baseline añade una sección útil ("NOT in scope") que el skill no incluye. La diferencia es de formato, no de calidad.

---

## AGENTE 5: security-auditor.md

### Eval 1 — npm package audit

**WITH SKILL:**
- Audita los 4 paquetes con el checklist de 8 items
- react-hot-toast, framer-motion, nodemailer: ✅ SAFE
- next-stripe-helper: 🚫 DANGER con razonamiento (supply chain attack vector, credential harvesting)
- No hace web search → no detecta CVE de nodemailer@6.9.8
- Tiempo: 15.5s, 0 tool uses

**BASELINE:**
- Hace web search activo (8 tool uses, 42.9s)
- Detecta CVE específico para nodemailer@6.9.8 (ReDoS vulnerability)
- Cita fuentes: GitHub advisory, GitLab advisory, Snyk
- react-hot-toast marcado como CAUTION por versión desactualizada
- Más información pero sin el formato de checklist estructurado

**DIFERENCIA CRÍTICA:** La baseline encontró una vulnerabilidad real (o plausible) en nodemailer que el skill se perdió por no instruir al agente a usar web search. Para una auditoría de seguridad, esto es un gap serio.

---

### Eval 2 — Malicious agent file audit

**WITH SKILL:**
- Checklist item por item con Pass/Fail
- Cita la línea exacta maliciosa del frontmatter
- Identifica el threat type: "data exfiltration via embedded instruction in YAML frontmatter"
- 🚫 DANGER con instrucciones específicas (delete, verify colleague, log incident)
- Director override required — logged permanently in memory

**BASELINE:**
- También identifica el threat inmediatamente
- Explica las señales de alerta (vector de entrega, instrucción amplia, dominio .xyz, urgencia)
- Regla general para el futuro: "cualquier agente que diga 'lee archivos y envíalos a X' = malicioso por defecto"
- Más educativo, menos formal

**Diferencia:** Media-alta. El skill produce el Security Audit Report formal con checklist y threat identification. Ambos bloquean correctamente pero el skill tiene formato auditable.

---

## REPORTE RESUMEN

| Agente | Eval 1 | Eval 2 | Veredicto | Razón |
|---|---|---|---|---|
| director | Alta diferenciación | Media | **VALIDADO** | Research-first, Awwwards quality bar, Phase Report template |
| project-manager | Alta diferenciación | Media | **VALIDADO** | Structured templates (Progress Report, Milestone Table), escalation protocol |
| coordinator | Alta diferenciación | Baja | **VALIDADO** | Handoff Package con constraints + open questions previene rework downstream |
| task-router | Media | Baja | **VALIDADO*** | Model selection framework correcto (Haiku para reports). Compression menos diferenciada |
| security-auditor | Gap crítico (CVE) | Alta | **NECESITA ITERAR** | No instruye a usar web search para CVEs → falsos negativos en auditorías npm |

*task-router: validado pero necesita refuerzo en la compression protocol

---

## DECISIONES PARA SIGUIENTE CAPA

**Iterar antes de avanzar:** security-auditor.md → añadir instrucción explícita de web search para CVE lookup y requerir citar fuentes

**Validados tal cual:** director, project-manager, coordinator

**Observación transversal:** Los 3 baselines más fuertes (director eval 2, coordinator eval 2, task-router eval 2) tienen en común que son preguntas con solución técnica conocida donde Claude ya tiene el conocimiento necesario. Los skills añaden más valor cuando la tarea requiere un protocolo específico, un formato de output estandarizado, o una decisión de proceso (cuándo escalar, qué descartar).
