# Software Recommendation — Café Vermut

---

## Phase 1 — Client Context

```
## Client Context — Software Selection

**Site type:** E-commerce (tienda online de café especialidad)
**Expected monthly orders:** 50–200 pedidos/mes
**Average order value:** €25
**Monthly GMV estimate:** ~€1,250–€5,000
**Budget for services:** < €50/month (propietario de cafetería; sensible al coste fijo)
**Technical maintenance:** Nil manages (cliente no técnico)
**Geographic market:** España principalmente (EU secondary)
**Timeline to launch:** Standard (no rush indicated)
**Client's technical literacy:** None — propietario de cafetería sin perfil técnico
**Product complexity:** 2–3 variantes de gramaje por café (variantes simples, no matriz compleja)
**Shipping:** Sí — producto físico, envío nacional España
```

**Key constraints that will drive every decision:**
1. Cliente no técnico — preferir herramientas que no requieran intervención manual frecuente
2. Volumen bajo-medio — free tiers y planes de entrada deben cubrir la operación
3. Mercado España — Bizum es relevante; carriers españoles son prioritarios
4. GMV mensual máximo ~€5,000 — coste de pasarela de pago debe ser proporcional

---

## Phase 2 — Evaluation by Category

---

### PAYMENTS

#### Dimension Scores

| Dimension | Stripe | PayPal | Stripe + Bizum (via Redsys) |
|---|---|---|---|
| Price (at target volume) | 4 | 2 | 3 |
| Implementation effort | 4 | 3 | 2 |
| SDK quality | 5 | 3 | 2 |
| Community & support | 5 | 3 | 2 |
| Production reliability | 5 | 4 | 3 |
| Vendor risk | 5 | 4 | 3 |
| **Total** | **28** | **19** | **15** |

**Analysis:**

- **Stripe:** A 100 pedidos/mes de €25 = €2,500 GMV → coste estimado: 100 × (€0.25 + 1.5% × €25) = 100 × €0.625 = **~€62.50/mes**. A 200 pedidos = **~€125/mes**. Fees proporcionales y sin sorpresas. SDK TypeScript excelente, test mode confiable. 3D Secure integrado — imprescindible para SCA en Europa.

- **PayPal:** Fees del 3.49% duplican el coste vs Stripe para mercado EU. UX de checkout inferior. Relevante para demografías que no tienen tarjeta, pero en tienda de café especialidad el perfil de cliente es adulto con tarjeta. Rechazado como opción principal.

- **Bizum via Redsys:** Muy usado en España; el cliente probablemente lo solicitará. Sin embargo: 8–16h de implementación, requiere partnership bancario o contrato con Redsys, sin SDK oficial TypeScript. Para el volumen actual, el esfuerzo no justifica el beneficio. Se puede añadir en una segunda fase si el cliente lo demanda explícitamente post-lanzamiento.

**Recommendation: Stripe (solo)**

**Reason:** Stripe cubre el 95% de los pagos del mercado español con tarjeta, incluye 3D Secure/SCA automático (requisito regulatorio EU), y tiene el mejor SDK para la implementación Next.js de la agencia. A 100 pedidos/mes la comisión es ~€62, que es el coste razonable para un negocio de €2,500 GMV mensual. PayPal se descarta por fees más altos y peor UX. Bizum se pospone a Fase 2 — el esfuerzo de integración (8–16h + Redsys) no es proporcional al volumen actual.

**Alternative considered:** PayPal — rechazado. Fees 3.49% vs 1.5% de Stripe, con peor experiencia de checkout para usuarios europeos con tarjeta.

**Alternative considered:** Bizum (via Redsys) — pospuesto a Fase 2. Válido para mercado español, pero requiere 8–16h de implementación adicional y contrato con Redsys. Revisar si el propietario lo solicita explícitamente tras el lanzamiento.

**Estimated cost at target volume:**
- 50 pedidos/mes (€25 ticket medio): ~€31/mes en comisiones
- 100 pedidos/mes: ~€63/mes en comisiones
- 200 pedidos/mes: ~€125/mes en comisiones

---

### EMAIL TRANSACCIONAL

#### Dimension Scores

| Dimension | Resend | SendGrid | Brevo (ex-Sendinblue) |
|---|---|---|---|
| Price (at target volume) | 5 | 4 | 4 |
| Implementation effort | 5 | 3 | 3 |
| SDK quality | 5 | 4 | 3 |
| Community & support | 4 | 5 | 3 |
| Production reliability | 4 | 5 | 4 |
| Vendor risk | 4 | 5 | 3 |
| **Total** | **27** | **26** | **20** |

**Analysis:**

**Emails esperados para Café Vermut:**
- Confirmación de pedido (1 por pedido): 50–200/mes
- Confirmación de envío (1 por pedido): 50–200/mes
- Recuperación de carrito abandonado (opcional): ~20–80/mes
- Total estimado: **120–500 emails/mes** — muy por debajo del free tier de Resend (3,000/mes)

- **Resend:** Free tier 3,000/mes cubre holgadamente el volumen. Integración nativa con React Email — la agencia ya usa Next.js, las plantillas de email se desarrollan como componentes React. TypeScript SDK de primera clase. Sin necesidad de configurar DKIM/SPF manualmente (gestionado en el dashboard). Tiempo de implementación: 1–2h.

- **SendGrid:** Más potente en analytics y deliverability a escala, pero overkill para 200–500 emails/mes. El free tier (100/día = 3,000/mes) es equivalente pero la DX es inferior a Resend. Sin ventaja real en este contexto.

- **Brevo:** Interesante por tener marketing email + transaccional en un plan, pero el cliente no tiene necesidad inmediata de email marketing. La DX es inferior y la marca está en transición de nombre (riesgo menor de estabilidad percibida).

**Recommendation: Resend**

**Reason:** El volumen transaccional de Café Vermut (máximo ~500 emails/mes en escenario optimista) cabe con margen en el free tier de Resend. La integración con React Email permite desarrollar plantillas de confirmación de pedido y envío como componentes Next.js — coherente con el stack de la agencia. Cero coste fijo garantizado durante al menos el primer año de operación.

**Alternative considered:** SendGrid — rechazado. DX inferior, configuración más compleja, sin ventaja en deliverability que justifique el cambio para este volumen.

**Estimated cost:** €0/mes (free tier: 3,000 emails/mes; el cliente usará < 500/mes)

---

### BASE DE DATOS

#### Dimension Scores

| Dimension | Supabase | PlanetScale | Firebase (Firestore) |
|---|---|---|---|
| Price (at target volume) | 5 | 3 | 3 |
| Implementation effort | 5 | 3 | 4 |
| SDK quality | 5 | 4 | 4 |
| Community & support | 5 | 4 | 5 |
| Production reliability | 4 | 5 | 5 |
| Vendor risk | 4 | 3 | 5 |
| **Total** | **28** | **22** | **26** |

**Analysis:**

- **Supabase:** PostgreSQL gestionado con free tier generoso (500MB, 2GB bandwidth). Para una tienda de café con 50–200 pedidos/mes el volumen de datos es mínimo — catálogo de ~10–20 SKUs, ~200 pedidos/mes máximo, clientes registrados. El free tier no se saturará en 1–2 años de operación normal. CLI de migraciones, generación de tipos TypeScript, RLS para seguridad de datos. Elección estándar de la agencia. **Nota:** el free tier pausa la base de datos tras 1 semana de inactividad — en producción con pedidos regulares esto no debería ocurrir; si hubiera periodos de inactividad, upgrade al plan Pro ($25/mes) se justifica.

- **PlanetScale:** MySQL escalable con branching de esquemas. Excelente para equipos con múltiples developers. Para un proyecto gestionado por la agencia con cliente no técnico, la complejidad adicional no aporta valor. Además PlanetScale cambió su pricing en 2024 eliminando el free tier — ahora mínimo $39/mes, lo que lo descalifica para este presupuesto.

- **Firebase (Firestore):** NoSQL — no es la mejor opción para un ecommerce con relaciones entre pedidos, productos, clientes, variantes. Consultas complejas (ej: "todos los pedidos de un cliente con sus productos") requieren desnormalización y múltiples reads. Sin equivalente a RLS de Postgres. El TypeScript SDK es menos elegante. Solo se recomendaría si hubiera necesidad de real-time sync, que no existe aquí.

**Recommendation: Supabase (Free tier → Pro si hay crecimiento)**

**Reason:** PostgreSQL es el modelo de datos correcto para un ecommerce (relaciones entre productos, variantes, pedidos, clientes). El free tier cubre holgadamente el volumen de Café Vermut. La agencia tiene experiencia con Supabase y sus migraciones CLI — reduce tiempo de implementación. PlanetScale queda descartado por pricing (sin free tier desde 2024). Firebase queda descartado por modelo NoSQL inadecuado para ecommerce relacional.

**Alternative considered:** PlanetScale — rechazado. Eliminó el free tier en 2024; mínimo $39/mes incompatible con el presupuesto del cliente.

**Alternative considered:** Firebase — rechazado. Modelo NoSQL inadecuado para relaciones de ecommerce; RLS equivalente más complejo; mayor riesgo de costes imprevistos por modelo pay-per-read.

**Estimated cost:** €0/mes (free tier). Si el cliente supera 500MB de datos o 2GB de bandwidth (escenario de ~2,000+ pedidos/mes acumulados), upgrade a Pro: $25/mes.

---

### AUTENTICACIÓN

#### Dimension Scores

| Dimension | NextAuth v5 (Auth.js) | Clerk | Supabase Auth |
|---|---|---|---|
| Price | 5 | 4 | 5 |
| Implementation effort | 3 | 5 | 4 |
| SDK quality | 4 | 5 | 4 |
| Community & support | 4 | 5 | 4 |
| Production reliability | 4 | 5 | 5 |
| Vendor risk | 5 | 3 | 4 |
| **Total** | **25** | **27** | **26** |

**Analysis:**

**Perfil de auth para Café Vermut:**
- Clientes registrados: cuenta para ver historial de pedidos
- Admin: panel para gestionar pedidos (1–2 usuarios: el propietario)
- No hay organización multi-tenant, no hay roles complejos
- El propietario no tiene perfil técnico — el panel admin debe ser simple

- **NextAuth v5:** Free, open source, elección estándar de la agencia. La UI de login debe construirse manualmente — para un cliente no técnico esto significa que Nil/la agencia controla el 100% del diseño. Compatible con Supabase Adapter. Para este proyecto, la complejidad de construir el UI de login es baja (solo email/password o magic link).

- **Clerk:** Pre-built UI components — si el propietario necesita un área "Mi cuenta" con historial de pedidos y la agencia quiere ahorrar 2–3h de UI development, Clerk es una opción válida. Free hasta 10,000 MAU — Café Vermut no llegará a esa cifra en mucho tiempo. El riesgo es vendor lock-in en la UI de auth y posible cambio de pricing (Clerk ha ajustado su modelo en el pasado).

- **Supabase Auth:** Dado que ya usamos Supabase como base de datos, Supabase Auth es una opción natural — evita una dependencia adicional. Buena integración con RLS. Sin embargo, la DX para Next.js App Router requiere más configuración que NextAuth con su adapter.

**Recommendation: NextAuth v5 (Auth.js)**

**Reason:** Elección estándar de la agencia. Free, sin vendor lock-in, funciona perfectamente con el Supabase Adapter. Para el caso de uso de Café Vermut (clientes con cuenta básica + propietario como admin), NextAuth cubre todos los requisitos sin coste adicional. El UI de login/registro es simple de construir en Next.js App Router. Clerk sería válido si el cliente valorase la velocidad de implementación del UI por encima del control — pero dado que la agencia gestiona el proyecto, ese trade-off no aplica aquí.

**Alternative considered:** Clerk — válido pero introduce dependencia de vendor con historial de cambios de pricing. Se recomienda solo si el cliente paga por velocidad de entrega del UI.

**Alternative considered:** Supabase Auth — posible, pero añade complejidad de configuración en Next.js App Router vs el Supabase Adapter de NextAuth que la agencia ya tiene documentado.

**Estimated cost:** €0/mes (open source)

---

### SHIPPING (ENVÍOS)

#### Dimension Scores

| Dimension | Sendcloud | Packlink PRO | Integración directa MRW/SEUR |
|---|---|---|---|
| Price | 3 | 5 | 4 |
| Implementation effort | 3 | 3 | 2 |
| SDK quality | 3 | 3 | 1 |
| Community & support | 4 | 3 | 2 |
| Production reliability | 5 | 4 | 3 |
| Vendor risk | 5 | 4 | 3 |
| **Total** | **23** | **22** | **15** |

**Analysis:**

**Contexto de envíos de Café Vermut:**
- 50–200 paquetes/mes (café en bolsas de 250g–1kg)
- Mercado España principalmente
- Cliente no técnico — necesita proceso simple: pedido entra → label se genera → paquete sale
- Sin necesidad de gestión multi-warehouse o devoluciones complejas

- **Sendcloud:** Free tier incluye 400 paquetes/mes con carriers limitados. Para 50–200 paquetes/mes, el free tier puede cubrir la operación. Incluye portal de tracking de marca para el cliente final — importante para la experiencia post-compra. Fuerte presencia en España (MRW, SEUR, Correos integrados). REST API integrable con Next.js. **Importante:** en el free tier los carriers disponibles son limitados — verificar que cubren la tarifa de envío esperada por el cliente. Si el propietario quiere acceso a tarifas de MRW o SEUR negociadas, puede requerir el plan Lite (€40/mes).

- **Packlink PRO:** Pay-per-shipment, sin cuota mensual. Ideal si el cliente quiere coste variable puro. A 50 paquetes/mes puede ser más económico que Sendcloud Lite. Menos automatización — el propietario tendría que generar cada label manualmente o via API (que requiere implementación similar). Para un cliente no técnico, la automatización de Sendcloud es más valiosa que el ahorro en cuota fija.

- **Integración directa MRW/SEUR:** Cada carrier tiene su propia API. Sin SDK oficial TypeScript. Requeriría integrar múltiples APIs para dar opciones al cliente. Impractical para este proyecto.

**Recommendation: Sendcloud (Free tier)**

**Reason:** El free tier de Sendcloud (400 paquetes/mes) cubre la operación completa de Café Vermut a su volumen esperado (50–200 paquetes/mes). La integración con carriers españoles (MRW, SEUR, Correos) es robusta. El portal de tracking de marca mejora la experiencia post-compra sin coste adicional. Para un propietario no técnico, la automatización de generación de etiquetas es clave — Sendcloud lo facilita mejor que Packlink PRO. Si el crecimiento lleva a más de 400 paquetes/mes o el cliente requiere tarifas negociadas, el upgrade a Lite (€40/mes) se evalúa en ese momento.

**Alternative considered:** Packlink PRO — válido para clientes que prefieren coste variable puro. Descartado porque Sendcloud free tier ya tiene coste €0 y ofrece mayor automatización, que es prioritaria para un cliente no técnico.

**Alternative considered:** Integración directa con carrier (MRW/SEUR) — rechazado. Sin SDK TypeScript, requiere múltiples integraciones, exceso de complejidad para el volumen del proyecto.

**Estimated cost:** €0/mes (free tier, hasta 400 paquetes/mes). Upgrade a Lite si el cliente supera 400 paquetes/mes o necesita tarifas negociadas: €40/mes.

---

### ANALYTICS

#### Dimension Scores

| Dimension | Google Analytics 4 | Plausible | Posthog |
|---|---|---|---|
| Price | 5 | 3 | 4 |
| Implementation effort | 4 | 5 | 3 |
| SDK quality | 4 | 4 | 4 |
| Community & support | 5 | 4 | 4 |
| Production reliability | 5 | 5 | 4 |
| Vendor risk | 5 | 4 | 3 |
| **Total** | **28** | **25** | **22** |

**Analysis:**

**Necesidades de analytics de Café Vermut:**
- Tráfico básico: pageviews, fuentes de tráfico
- Conversiones: añadir al carrito → checkout → pedido completado
- El propietario necesita una vista simple — no es analista de datos
- Mercado España → regulación GDPR aplica

- **Google Analytics 4:** Gratis, estándar de industria. Integración con `@next/third-parties/google` en 30 min. Potente para tracking de conversiones (GA4 ecommerce events). **Problema:** GDPR requiere cookie consent banner — añade fricción UX y complejidad de implementación (necesario Consent Mode v2 para retener datos con usuarios que no consientan). El propietario no tendrá capacidad de interpretar la interfaz de GA4 sin formación.

- **Plausible:** GDPR compliant sin cookie banner (datos agregados, sin tracking individual). Script < 1KB — mejor para Core Web Vitals. UI simple — el propietario puede ver tráfico, páginas más visitadas y conversiones sin training. $9/mes para hasta 10k pageviews. **Para Café Vermut** con tráfico estimado <10k pageviews/mes: $9/mes cubre la operación completa. Elimina la necesidad de implementar cookie consent para analytics.

- **Posthog:** Potente para product analytics (session recordings, funnels, feature flags). Overkill para este cliente. El free tier es generoso pero la complejidad de la herramienta excede las necesidades del propietario.

**Decisión de trade-off:**

GA4 es gratuito pero introduce complejidad GDPR y una UI que el propietario no podrá usar de forma autónoma. Plausible cuesta $9/mes pero elimina el cookie consent banner (mejor UX para los visitantes) y da al propietario una interfaz usable. **Para un cliente en el mercado español (GDPR), la simplificación del compliance y la UI usable para el propietario justifican los $9/mes.**

**Recommendation: Plausible**

**Reason:** Café Vermut opera en España (GDPR). Plausible es GDPR compliant por defecto sin necesidad de implementar cookie consent banner para analytics — simplifica el desarrollo y mejora la UX para los visitantes. La UI de Plausible es suficientemente simple para que el propietario (cliente no técnico) pueda revisar el tráfico de su tienda de forma autónoma. A <10k pageviews/mes, el coste es $9/mes (~€8). Esta es la primera instancia donde se recomienda Plausible sobre GA4 — el mercado español + cliente no técnico + necesidad de GDPR compliance sin friction es el caso de uso exacto para el que Plausible existe.

**Alternative considered:** Google Analytics 4 — rechazado como opción principal. GDPR requiere cookie consent (fricción UX + complejidad de implementación con Consent Mode v2). La UI de GA4 no es usable para un propietario no técnico sin formación específica. Se puede implementar GA4 adicionalmente si el cliente lo solicita explícitamente para integración con Google Ads en el futuro.

**Alternative considered:** Posthog — rechazado. Overkill en funcionalidades para un ecommerce de este tamaño; el propietario no explotará las capacidades avanzadas.

**Estimated cost:** $9/mes (~€8/mes, hasta 10k pageviews/mes)

---

## Phase 3 — Approved Stack Summary

| Category | Service | Plan | Monthly cost estimate |
|---|---|---|---|
| Payments | Stripe | Pay-per-transaction | ~€31–€125 (variable, en comisiones) |
| Email transaccional | Resend | Free tier | €0 |
| Base de datos | Supabase | Free tier | €0 |
| Autenticación | NextAuth v5 | Open source | €0 |
| Shipping | Sendcloud | Free tier | €0 |
| Analytics | Plausible | Starter ($9/mes) | ~€8/mes |
| **Total fijo** | | | **~€8/mes** |
| **Total variable** | | | **+€31–€125/mes en fees de Stripe** |

**Coste total mensual estimado:**
- Escenario conservador (50 pedidos/mes): ~€39/mes (€8 fijo + €31 Stripe)
- Escenario objetivo (100 pedidos/mes): ~€71/mes (€8 fijo + €63 Stripe)
- Escenario optimista (200 pedidos/mes): ~€133/mes (€8 fijo + €125 Stripe)

**Upgrades previstos (no inmediatos):**
- Supabase Pro ($25/mes): si se supera 500MB de datos o el free tier pausa en producción
- Sendcloud Lite (€40/mes): si se supera 400 paquetes/mes o se necesitan tarifas negociadas con carrier
- Resend Pro (€20/mes): no previsto — el free tier (3,000/mes) cubre >10x el volumen esperado

---

## Implementation Order for api-developer

1. **Supabase** — base de datos y schema de ecommerce (productos, variantes, pedidos, clientes)
2. **NextAuth v5** — autenticación con Supabase Adapter (cuentas de cliente + admin)
3. **Stripe** — Checkout Session + webhooks para confirmación de pedido
4. **Resend + React Email** — plantillas de confirmación de pedido y envío
5. **Sendcloud** — generación de etiquetas post-pago (trigger desde webhook de Stripe)
6. **Plausible** — script de tracking + eventos de conversión del funnel de compra

Ready for `api-developer` to begin integrations.
