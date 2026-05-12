# Copy Home — Agencia de Diseño Digital para Startups Tech

---

## Fase 1: Consultoría (marco estratégico)

**A quién habla:** Startups tech en fase seed o Serie A. Fundadores y co-fundadores técnicos que ya tienen producto pero saben que su presencia digital no está a la altura. Su dolor principal: tienen un producto sólido pero su web no lo transmite — pierden credibilidad en cada demo y cada pitch.

**Tono:** Directo, sin palabrería corporativa. Cercano pero profesional. Habla de tú a tú, como lo haría un CTO que también sabe de negocio. Nada de "soluciones integrales" ni "ecosistemas digitales".

**Propuesta de valor única:** La agencia entiende el contexto startup: velocidad, iteración, y que cada euro cuenta. No vendes "diseño bonito" — vendes credibilidad que convierte inversores y usuarios desde el primer scroll.

**Asumo que:** la agencia opera en España o Latam en español, usa stack Next.js, y el objetivo principal de la home es conseguir llamadas de discovery con potenciales clientes.

---

## Fase 2: Copy narrativo completo

---

### HERO — Sección principal

> **Fórmula PAS aplicada:**
> - **Problema:** Tu startup tiene un gran producto, pero tu web no lo demuestra
> - **Agitación:** Cada visita que no convierte es un inversor, cliente o socio que se va a la competencia
> - **Solución:** Diseñamos y construimos la presencia digital que tu startup merece

---

**Título (H1):** Tu startup merece una web que cierre tratos

**Subtítulo (H2):** Diseñamos, desarrollamos y activamos experiencias digitales para startups tech que necesitan credibilidad desde el primer scroll.

**Body (opcional, 1-2 frases):** Nada de plantillas genéricas. Trabajamos contigo para que tu producto hable por sí solo — antes de que abras la boca en el pitch.

**CTA primario:** Agenda tu llamada gratuita

**CTA secundario:** Ver proyectos

---

### SERVICIOS — Sección de servicios (3 tarjetas)

**Headline de sección (H2):** Lo que hacemos para que crezcas más rápido

**Subtítulo de sección:** Tres disciplinas, un solo objetivo: que tu negocio digital genere resultados reales.

---

#### Servicio 1 — Diseño

**Título:** Diseño que comunica antes de que nadie lea

**Subtítulo:** UI/UX diseñado para startups con prisa y con criterio.

**Body:**
- Diseñamos interfaces que tus usuarios entienden en segundos
- Sistema de diseño escalable: crece contigo sin empezar de cero
- Prototipado rápido para validar antes de desarrollar

**CTA:** Ver nuestro proceso

---

#### Servicio 2 — Desarrollo Web

**Título:** Código que aguanta el tráfico del lanzamiento

**Subtítulo:** Webs y apps en Next.js: rápidas, seguras y listas para escalar.

**Body:**
- Desarrollo con el stack que ya usas o con el que deberías usar
- Performance optimizada desde el día uno — sin deuda técnica innecesaria
- Entrega en semanas, no en meses

**CTA:** Hablar de tu proyecto

---

#### Servicio 3 — Estrategia Digital

**Título:** Estrategia para que cada clic tenga un propósito

**Subtítulo:** Definimos qué construir, para quién y en qué orden.

**Body:**
- Auditoría de tu presencia digital actual (lo que funciona y lo que no)
- Arquitectura de contenido y flujos de conversión
- Hoja de ruta priorizada por impacto en negocio

**CTA:** Pedir auditoría

---

### CTA FINAL — Sección de cierre

**Headline (H2):** ¿Tienes un producto que vale la pena mostrar?

**Subtítulo:** Cuéntanos dónde estás y adónde quieres llegar. La primera llamada es gratuita y sin compromiso.

**Body:** Trabajamos con startups en todas las fases — desde el MVP hasta la Serie A. Si tienes tracción pero tu digital no la refleja, es el momento de cambiar eso.

**CTA primario:** Agenda tu llamada — es gratis

**CTA secundario:** Escríbenos por email

---

## Bloque JSON — messages/es.json

```json
{
  "Home": {
    "hero": {
      "title": "Tu startup merece una web que cierre tratos",
      "subtitle": "Diseñamos, desarrollamos y activamos experiencias digitales para startups tech que necesitan credibilidad desde el primer scroll.",
      "body": "Nada de plantillas genéricas. Trabajamos contigo para que tu producto hable por sí solo — antes de que abras la boca en el pitch.",
      "ctaPrimary": "Agenda tu llamada gratuita",
      "ctaSecondary": "Ver proyectos"
    },
    "services": {
      "sectionTitle": "Lo que hacemos para que crezcas más rápido",
      "sectionSubtitle": "Tres disciplinas, un solo objetivo: que tu negocio digital genere resultados reales.",
      "design": {
        "title": "Diseño que comunica antes de que nadie lea",
        "subtitle": "UI/UX diseñado para startups con prisa y con criterio.",
        "bullet1": "Diseñamos interfaces que tus usuarios entienden en segundos",
        "bullet2": "Sistema de diseño escalable: crece contigo sin empezar de cero",
        "bullet3": "Prototipado rápido para validar antes de desarrollar",
        "cta": "Ver nuestro proceso"
      },
      "development": {
        "title": "Código que aguanta el tráfico del lanzamiento",
        "subtitle": "Webs y apps en Next.js: rápidas, seguras y listas para escalar.",
        "bullet1": "Desarrollo con el stack que ya usas o con el que deberías usar",
        "bullet2": "Performance optimizada desde el día uno — sin deuda técnica innecesaria",
        "bullet3": "Entrega en semanas, no en meses",
        "cta": "Hablar de tu proyecto"
      },
      "strategy": {
        "title": "Estrategia para que cada clic tenga un propósito",
        "subtitle": "Definimos qué construir, para quién y en qué orden.",
        "bullet1": "Auditoría de tu presencia digital actual (lo que funciona y lo que no)",
        "bullet2": "Arquitectura de contenido y flujos de conversión",
        "bullet3": "Hoja de ruta priorizada por impacto en negocio",
        "cta": "Pedir auditoría"
      }
    },
    "finalCta": {
      "title": "¿Tienes un producto que vale la pena mostrar?",
      "subtitle": "Cuéntanos dónde estás y adónde quieres llegar. La primera llamada es gratuita y sin compromiso.",
      "body": "Trabajamos con startups en todas las fases — desde el MVP hasta la Serie A. Si tienes tracción pero tu digital no la refleja, es el momento de cambiar eso.",
      "ctaPrimary": "Agenda tu llamada — es gratis",
      "ctaSecondary": "Escríbenos por email"
    }
  }
}
```

---

## Notas de implementación

1. **Variables a pasar en el hero:** ninguna variable dinámica por ahora. Si en el futuro se personaliza por segmento (ej. "para startups SaaS" vs "para apps móviles"), el campo `subtitle` puede convertirse en `"subtitle": "Diseñamos experiencias digitales para {segment} que necesitan credibilidad desde el primer scroll."` — solo hay que añadir la clave `segment` al componente.

2. **Bullets en servicios:** los campos `bullet1`, `bullet2`, `bullet3` se deben mapear con `useTranslations` en un array o renderizarse individualmente. Si usas `next-intl` con rich text, puedes agruparlos en un solo campo con separador y dividirlos en el componente, pero la estructura actual (claves individuales) es la más robusta y fácil de mantener.

3. **Imágenes necesarias:**
   - Hero: imagen o ilustración de fondo que transmita velocidad/tecnología (no stock genérico — preferiblemente un mockup real de un proyecto de la agencia)
   - Servicios: 3 iconos o ilustraciones ligeras (SVG) para diseño, desarrollo y estrategia
   - CTA final: foto de equipo o ilustración de "llamada / conversación" — evita el teléfono clásico; considera un emoji grande o una ilustración abstracta

4. **Componentes que consumen este copy:**
   - `<HeroSection>` → `Home.hero.*`
   - `<ServicesGrid>` o `<ServicesCards>` → `Home.services.*`
   - `<FinalCta>` o `<CtaBanner>` → `Home.finalCta.*`
