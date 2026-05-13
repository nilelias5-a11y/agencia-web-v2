# Stack de Servicios — Café Vermut

**Fecha:** 2026-05-13
**Perfil:** Tienda online de café especialidad | 50-200 pedidos/mes | Ticket medio €25 | Mercado España | Cliente no técnico

---

## Resumen Ejecutivo

Para un negocio de estas características el criterio principal es **coste bajo y complejidad operativa mínima**, no escalabilidad extrema. Con un GMV máximo de 5.000 €/mes (200 pedidos × €25), cada euro de infraestructura importa. La recomendación prioriza servicios con tier gratuito generoso, documentación en español o muy accesible, y panel de administración que pueda manejar el propietario sin ayuda técnica continuada.

---

## 1. Pagos — Stripe

**Elección: Stripe**

**Justificación:**
- Único proveedor con soporte completo de SCA/PSD2 (obligatorio en España/UE) sin configuración adicional.
- Payment Links y Checkout hosted eliminan la necesidad de programar una UI de pago — clave para cliente no técnico.
- Panel de Stripe permite al propietario gestionar reembolsos manualmente sin tocar código.
- Integración nativa con Stripe Tax para calcular IVA español automáticamente (21% general, aplicable a café).
- SDK oficial para Next.js (el stack más probable para este tipo de proyecto).

**Coste:**
- Tarifa estándar: **1,5% + €0,25** por transacción para tarjetas europeas (tarifa específica España/EEA).
- A 200 pedidos × €25 = €5.000/mes → comisión ≈ **€125/mes** (2,5% efectivo).
- A 50 pedidos × €25 = €1.250/mes → comisión ≈ **€31/mes**.
- Sin cuota fija mensual, sin coste de configuración.
- Stripe Tax: +0,5% adicional sobre transacciones donde se calcula el impuesto (~€25/mes en escenario máximo). Opcional — se puede gestionar IVA manualmente al inicio.

**Alternativas descartadas:**
- **Redsys (TPV Virtual BBVA/Caixabank):** Coste fijo mensual ~€25-40 + comisiones similares, integración técnicamente más compleja, UX de pago anticuada. Solo valdría si el banco del propietario ofrece condiciones especiales.
- **PayPal:** Comisiones más altas (3,4% + €0,35 para pequeños comercios), menos control sobre la experiencia.
- **Square:** Presencia limitada en España, soporte en español deficiente.

---

## 2. Email Transaccional — Resend

**Elección: Resend**

**Justificación:**
- Tier gratuito cubre **3.000 emails/mes** — más que suficiente para este volumen (confirmaciones de pedido, envío, recuperación de contraseña suman ~600 emails/mes en escenario máximo).
- API moderna diseñada para Next.js/React — permite crear templates de email con React Email, lo que facilita mantener el diseño de los correos.
- Dashboard simple para ver logs de entrega, sin curva de aprendizaje técnico avanzado.
- Dominios verificados con SPF/DKIM fácil de configurar.
- Precios predecibles si se supera el tier gratuito.

**Coste:**
- **Tier gratuito: €0/mes** (hasta 3.000 emails/mes, 100 emails/día).
- Si se crece: plan Pro $20/mes por 50.000 emails — no necesario en este rango de pedidos.
- **Coste efectivo: €0/mes** para el volumen proyectado.

**Alternativas descartadas:**
- **SendGrid:** Tier gratuito solo 100 emails/día (restrictivo), interfaz más compleja, no está pensado para integración Next.js moderna.
- **Mailgun:** Tier gratuito eliminado en 2022; mínimo $35/mes para uso real.
- **Postmark:** Excelente deliverability pero sin tier gratuito útil (~$15/mes desde el inicio). Válido si deliverability es crítico, pero para este caso Resend es suficiente.
- **Amazon SES:** Muy barato (€0,10/1.000 emails) pero configuración técnica compleja (no apto para cliente no técnico que eventualmente quiera gestionar plantillas).

---

## 3. Base de Datos — Neon (PostgreSQL serverless)

**Elección: Neon**

**Justificación:**
- PostgreSQL gestionado serverless con **tier gratuito permanente**: 0,5 GB almacenamiento, 190 horas de cómputo/mes.
- Para 50-200 pedidos/mes con catálogo de 10-30 cafés y 2-3 variantes cada uno, el tier gratuito es más que suficiente indefinidamente.
- Branching de base de datos integrado: permite tener rama de staging separada sin coste adicional — muy útil para el agente de desarrollo.
- Compatible con Prisma ORM out-of-the-box (stack estándar Next.js).
- Autoescala a cero cuando no hay actividad — no paga cómputo cuando la tienda no recibe visitas.
- Integración directa con Vercel (deploy automático de migraciones).

**Coste:**
- **Tier gratuito: €0/mes** para este volumen.
- Si se crece significativamente: plan Launch $19/mes (10 GB, compute ilimitado).

**Alternativas descartadas:**
- **PlanetScale (MySQL):** Eliminó su tier gratuito en 2024. Mínimo $39/mes.
- **Supabase (PostgreSQL):** Buena alternativa con tier gratuito, pero incluye Auth, Storage y más servicios que generan complejidad innecesaria si se usan otros proveedores. El tier gratuito pausa proyectos inactivos 1 semana — problemático para una tienda.
- **Railway PostgreSQL:** Tier gratuito muy limitado ($5 crédito/mes), precio en producción no predecible.
- **MongoDB Atlas:** Sin beneficio real para datos relacionales de un ecommerce (pedidos, variantes, inventario tienen relaciones claras).

---

## 4. Autenticación — Clerk

**Elección: Clerk**

**Justificación:**
- Solución completa de autenticación como servicio con componentes UI preconstruidos en React/Next.js.
- El propietario puede gestionar usuarios directamente desde el dashboard de Clerk sin tocar código (ver clientes registrados, resetear contraseñas, ver sesiones activas).
- Soporta login con email/contraseña, magic links, y Google OAuth — todos relevantes para compradores de café online en España.
- Componentes `<SignIn>`, `<SignUp>`, `<UserProfile>` se integran en minutos y respetan el diseño de la tienda.
- Middleware de Next.js para proteger rutas de forma declarativa.
- Gestión de sesiones, JWT, refresh tokens — todo manejado automáticamente.

**Coste:**
- **Tier gratuito: €0/mes** hasta 10.000 usuarios activos mensuales (MAU).
- Para este negocio, llegar a 10.000 MAU significaría un éxito masivo que justificaría el upgrade.
- Plan Pro: $25/mes si se supera el límite.
- **Coste efectivo: €0/mes** en el horizonte proyectado.

**Alternativas descartadas:**
- **NextAuth.js (Auth.js):** Gratuito y open-source, pero requiere configurar y mantener la lógica de sesiones, adaptadores de base de datos, y no tiene dashboard de administración. Añade carga de mantenimiento para cliente no técnico.
- **Supabase Auth:** Válido si se usa Supabase como BD (economía de servicios), pero si se elige Neon como BD, combinar Supabase solo para Auth añade complejidad innecesaria.
- **Auth0:** Tier gratuito limitado a 7.500 MAU, precio del plan paid elevado ($23/mes+). No ofrece ventajas claras sobre Clerk para Next.js.

---

## 5. Shipping — Sendcloud

**Elección: Sendcloud**

**Justificación:**
- Plataforma de gestión de envíos con sede en Europa, presencia fuerte en España, y tarifas negociadas con Correos, MRW, DHL, SEUR, GLS.
- **Tier gratuito "Essential" hasta 400 envíos/mes** — cubre el escenario máximo proyectado (200 pedidos/mes) con margen.
- Panel web completo para imprimir etiquetas, gestionar devoluciones, y hacer seguimiento — operación diaria del propietario sin código.
- Tracking page branded (página de seguimiento con logo del negocio) incluida gratis.
- Integración con Shopify, WooCommerce, y API REST para tiendas custom Next.js.
- Permite configurar reglas de envío (gratis desde X €, tarifa plana por zona, etc.) desde el panel.
- Soporte en español.

**Coste:**
- **Plan Essential: €0/mes** (hasta 400 envíos/mes).
- Se paga el coste real del envío por paquete (tarifas negociadas, típicamente €3,50-6,00 por envío nacional dependiendo del peso del café).
- Estos costes de envío son del negocio, no de infraestructura tech.
- **Coste de la plataforma: €0/mes** en el rango proyectado.

**Alternativas descartadas:**
- **Shippo:** Buen producto pero orientado a mercado USA; tarifas en España menos competitivas, soporte en inglés.
- **EasyPost:** Similar a Shippo — muy enfocado en USA, integración con carriers españoles limitada.
- **Integración directa con Correos/SEUR:** Cada carrier tiene su propia API, contratos individuales, y sin panel unificado. Coste de desarrollo e-commerce muy superior.
- **Packlink PRO:** Alternativa española válida, tier gratuito hasta 200 envíos/mes. Segunda opción si Sendcloud presenta problemas. Interfaz algo menos pulida.

---

## 6. Analytics — Plausible Analytics

**Elección: Plausible Analytics**

**Justificación:**
- Analytics de privacidad primero: **no requiere banner de cookies** bajo RGPD/ePrivacy español — elimina fricción legal y de UX para el propietario.
- Dashboard extremadamente simple: visitas, conversiones, fuentes de tráfico, páginas más visitadas. Sin la complejidad de GA4.
- Script ligero (~1KB vs ~45KB de Google Analytics) — no impacta el rendimiento de la tienda.
- Funnels de conversión básicos incluidos: permite ver cuántos usuarios llegan a checkout vs. completan la compra.
- Datos propios, alojados en UE (cumplimiento RGPD automático).
- El propietario puede ver métricas clave en 30 segundos sin formación.

**Coste:**
- Plan Starter: **$9/mes** (hasta 10.000 pageviews/mes).
- Para una tienda pequeña de café, 10.000 pageviews/mes es razonable al inicio.
- Si crece: $19/mes hasta 100.000 pageviews.
- **Coste efectivo: ~€8/mes** (≈$9 con cambio actual).

**Alternativas descartadas:**
- **Google Analytics 4 (GA4):** Gratuito pero requiere banner de cookies (coste legal y de conversión), interfaz compleja poco adecuada para cliente no técnico, datos enviados a USA (complicaciones RGPD).
- **Posthog:** Tier gratuito generoso (1M eventos/mes) y más features, pero panel más técnico. Segunda opción válida si se quiere tracking de eventos más granular (add to cart, etc.) a coste cero.
- **Mixpanel:** Orientado a producto SaaS, excesivo para este caso de uso.
- **Umami (self-hosted):** Gratuito si se auto-hospeda, pero requiere servidor y mantenimiento — no apto para cliente no técnico.

---

## Resumen de Costes Mensuales

| Servicio | Proveedor | Coste Fijo/mes | Coste Variable |
|---|---|---|---|
| Pagos | Stripe | €0 | 1,5%+€0,25/transacción |
| Email transaccional | Resend | €0 | €0 (dentro del tier gratuito) |
| Base de datos | Neon | €0 | €0 (dentro del tier gratuito) |
| Autenticación | Clerk | €0 | €0 (dentro del tier gratuito) |
| Shipping | Sendcloud | €0 | €0 (coste del envío es del negocio) |
| Analytics | Plausible | ~€8 | €0 |
| **Total infraestructura** | | **~€8/mes** | **+comisión Stripe** |

**Coste total real por escenario:**

- **Escenario conservador (50 pedidos/mes, €1.250 GMV):**
  - Stripe: ~€31
  - Plausible: ~€8
  - **Total: ~€39/mes** (3,1% del GMV)

- **Escenario objetivo (100 pedidos/mes, €2.500 GMV):**
  - Stripe: ~€63
  - Plausible: ~€8
  - **Total: ~€71/mes** (2,8% del GMV)

- **Escenario máximo (200 pedidos/mes, €5.000 GMV):**
  - Stripe: ~€125
  - Plausible: ~€8
  - **Total: ~€133/mes** (2,7% del GMV)

*Nota: Estos costes no incluyen hosting (Vercel hobby/pro: €0-20/mes) ni dominio (~€10-15/año).*

---

## Consideraciones Adicionales

**Por qué NO Shopify:**
Shopify all-in-one podría parecer atractivo para cliente no técnico, pero a 200 pedidos/mes el coste sería $29-79/mes + 2% comisión adicional sobre ventas con Stripe externo (o 0% con Shopify Payments, que en España tiene comisión del 1,9%+€0,25). Una tienda Next.js custom con este stack es más económica y ofrece diferenciación de marca.

**Escalabilidad del stack:**
Si el negocio crece a 500+ pedidos/mes, la transición es fluida:
- Neon: upgrade a $19/mes
- Sendcloud: upgrade a €45/mes (plan Growth)
- Plausible: upgrade a $19/mes
- Sin cambios de proveedor ni migraciones complejas

**Recomendación de implementación:**
Para minimizar la curva de aprendizaje del propietario, priorizar la formación en tres paneles clave: Stripe (reembolsos y disputas), Sendcloud (generación de etiquetas), y Plausible (métricas semanales). Los demás servicios (Neon, Clerk, Resend) son transparentes para operación diaria.
