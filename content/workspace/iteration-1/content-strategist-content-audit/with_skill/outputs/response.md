# Auditoría de Contenido: E-commerce de Ropa Sostenible

**Contexto del cliente:** E-commerce de moda sostenible, 2 años de actividad, 30 artículos en el blog, tráfico orgánico casi nulo.

---

## 1. Diagnóstico

El problema no es la cantidad de contenido (30 artículos es un inventario respetable), sino la arquitectura y la intención detrás de cada pieza. Con tráfico orgánico cercano a cero a los 2 años, el síntoma más probable es uno de estos tres escenarios — o una combinación de los tres:

- **Sin autoridad temática:** artículos sueltos sin estructura de pilares ni interenlazado, lo que Google interpreta como un sitio sin especialización real.
- **Keyword mismatch:** se ha escrito sobre lo que la marca quiere decir, no sobre lo que el cliente busca en Google.
- **Canibalización o contenido zombie:** varios artículos atacan las mismas keywords sin saberlo, o cubren temas con volumen de búsqueda insignificante.

**Oportunidad principal:** El mercado de moda sostenible tiene demanda de búsqueda creciente y competencia SEO media-baja comparado con moda convencional. Un e-commerce que construya autoridad temática ahora puede posicionar en 6-9 meses con una estrategia bien ejecutada.

---

## 2. Clasificación del Contenido Existente

### Sistema de 4 categorías

Antes de crear nada nuevo, clasifica cada uno de los 30 artículos en estas 4 categorías usando datos de Google Search Console (GSC) y Google Analytics (GA4):

| Categoría | Criterio de clasificación | Qué hacer |
|---|---|---|
| **Estrella** | Tráfico orgánico mensual notable (>100 visitas) + posición media <20 en GSC | Actualizar, ampliar, añadir CTA al producto, linkear desde otros artículos |
| **Promesa** | Tráfico bajo (<100 visitas) pero posición media entre 4 y 20 en GSC (el artículo "casi" posiciona) | Optimizar de forma prioritaria: ampliar contenido, mejorar H2/H3, añadir FAQ, mejorar interlinking |
| **Duplicado** | Dos o más artículos cubren la misma keyword o intención de búsqueda | Hacer merge del mejor con el peor: redirigir (301) el más débil al más fuerte y fusionar el contenido útil |
| **Zombie** | Tráfico orgánico 0 + posición media >50 en GSC, o artículo sin datos en GSC tras 6+ meses publicado | Eliminar (con 301 a la home o a un artículo relacionado) o redirigir. No actualizar zombies: el esfuerzo no vale la pena |

### Proceso práctico para clasificar tus 30 artículos

1. Entra en **Google Search Console > Rendimiento > Páginas** y exporta a CSV.
2. Entra en **GA4 > Informes > Compromiso > Páginas** y exporta a CSV.
3. Cruza ambas exportaciones en una hoja de cálculo con estas columnas:

| URL | Título | Fecha publicación | Impresiones GSC | Clics GSC | Posición media | Sesiones GA4 | Categoría |
|---|---|---|---|---|---|---|---|
| /blog/moda-sostenible-que-es | ¿Qué es la moda sostenible? | 2023-03 | 1.200 | 8 | 38 | 12 | Promesa |
| /blog/marcas-ropa-ecologica | Las 10 mejores marcas ecológicas | 2023-05 | 0 | 0 | — | 0 | Zombie |
| /blog/tejidos-naturales | Tejidos naturales vs sintéticos | 2024-01 | 3.400 | 140 | 11 | 95 | Estrella |

4. Asigna la categoría manualmente a cada fila.

**Expectativa realista para un blog con casi cero tráfico:** en 30 artículos probablemente encontrarás 0-2 Estrellas, 5-8 Promesas, 3-5 Duplicados y 15-22 Zombies. Ese ratio es normal y no significa que el trabajo pasado fue inútil; significa que ahora sabes exactamente dónde actuar.

---

## 3. Plan de Acción Priorizado

### Primer paso: Optimizar las "Promesas" (semanas 1-4)
**Por qué primero:** Son el ROI más rápido. Estos artículos ya tienen señales positivas de Google (aparecen en posiciones 4-20), lo que significa que el algoritmo los considera relevantes. Un artículo en posición 8 que pasa a posición 3 puede multiplicar el tráfico x4 sin crear contenido nuevo.

**Qué hacer con cada Promesa:**
- Ampliar el artículo al menos un 30% con información que no tiene (secciones FAQ, ejemplos, datos actualizados).
- Añadir la keyword principal en el H1, primer párrafo y al menos un H2.
- Añadir enlaces internos desde otros artículos del blog hacia esta URL.
- Añadir un bloque FAQ al final con 3-5 preguntas que aparecen en "People Also Ask" de Google para esa keyword.
- Revisar que el meta title y meta description incluyan la keyword y tengan un CTA claro.

### Segundo paso: Eliminar o redirigir los "Zombies" (semanas 3-6)
**Por qué segundo:** El contenido zombie no solo no ayuda, activamente daña. Google distribuye el "presupuesto de rastreo" entre todas tus URLs. 20 artículos zombies consumen rastreo que podría dedicarse a las Promesas y a las páginas de producto. Eliminarlos libera autoridad de dominio que se redistribuye al contenido que sí funciona.

**Cómo ejecutarlo:**
- Para cada Zombie, configura un redirect 301 hacia el artículo temáticamente más cercano o hacia la home si no hay equivalente.
- No los dejes en 404: el 301 transfiere parte de la autoridad.
- Haz esto en un solo bloque de tiempo para no generar cambios constantes que confundan al rastreador.

### Tercer paso: Consolidar "Duplicados" (semanas 5-8)
**Por qué tercero:** La canibalización de keywords es invisible hasta que la mides. Dos artículos atacando "ropa de algodón orgánico" compiten entre sí y ninguno gana. La consolidación (merge) convierte dos artículos débiles en uno fuerte.

**Proceso de merge:**
1. Decide cuál de los dos artículos tiene mejor posición media o más backlinks externos.
2. Ese es el que conservas. El otro se redirige (301) al primero.
3. Fusiona el contenido útil del artículo eliminado dentro del artículo que conservas.
4. Actualiza todos los enlaces internos que apuntaban a la URL eliminada.

### Cuarto paso: Crear contenido nuevo para cubrir gaps (mes 2 en adelante)
**Por qué al final:** No tiene sentido crear contenido nuevo si el existente está mal estructurado. Primero saneas, luego construyes. Los gaps los identificarás en el análisis JTBD de la siguiente sección.

---

## 4. JTBD Framework: Identificar Gaps de Contenido

El cliente de ropa sostenible no busca "ropa sostenible" directamente. Busca soluciones a situaciones concretas de su vida. Usa este framework para identificar qué contenido te falta.

### Las 5 situaciones principales del comprador de moda sostenible

| Situación (trigger) | Trabajo que quiere completar | Nivel de conciencia | Contenido que necesitas |
|---|---|---|---|
| "Acabo de enterarme del impacto de la fast fashion" | Entender qué puedo hacer como consumidor individual sin renunciar a vestirme bien | TOFU — busca información, no marcas | "Fast fashion: qué es y cómo dejar de contribuir sin gastar más" |
| "Necesito renovar el armario pero quiero hacerlo bien" | Saber cómo construir un armario sostenible paso a paso, con presupuesto real | TOFU-MOFU — busca un método | "Cómo construir un armario cápsula sostenible desde cero (con presupuesto ajustado)" |
| "Quiero comprar una prenda concreta sostenible pero no sé qué marca es fiable" | Evaluar opciones y asegurarse de que no es greenwashing | MOFU — busca comparativas y validación | "Cómo saber si una marca es realmente sostenible (y no hace greenwashing)" |
| "Tengo [ocasión especial] y necesito algo concreto sostenible" | Encontrar una opción concreta para una necesidad específica | MOFU-BOFU — intención de compra cercana | "Ropa de boda sostenible", "ropa de trabajo sostenible mujer", "regalo sostenible moda" |
| "Ya soy cliente, quiero aprender a cuidar mejor mis prendas" | Maximizar la vida útil de lo que ya tiene y sentirse parte de una comunidad | Fidelización | "Cómo lavar ropa orgánica para que dure más", "guía de cuidado de tejidos naturales" |

### Buyer Persona: Los 3 perfiles de tu cliente

**Perfil 1 — La Consciente Primeriza (TOFU)**
- Mujer, 25-38 años, vive en ciudad, ha visto un documental o viral sobre fast fashion.
- Dolor: quiere hacer el cambio pero le parece caro y complicado.
- Busca en Google: "cómo dejar de comprar fast fashion", "ropa sostenible barata", "qué tejidos son ecológicos".
- Contenido que le convierte: guías prácticas con presupuesto realista, posts sobre mitos ("la ropa sostenible no tiene por qué ser cara").

**Perfil 2 — La Compradora Informada (MOFU)**
- Mujer, 30-45 años, ya lleva meses investigando, conoce términos como GOTS, OEKO-TEX, algodón orgánico.
- Dolor: desconfía del greenwashing, necesita validación antes de comprar.
- Busca en Google: "certificados ropa sostenible", "marcas moda sostenible España", "diferencia algodón orgánico vs convencional".
- Contenido que le convierte: comparativas de certificados, análisis de marcas, explicaciones técnicas de materiales.

**Perfil 3 — El Regalo / Ocasión Especial (BOFU)**
- Hombre o mujer, 28-50 años, busca un regalo o tiene una necesidad concreta y urgente.
- Dolor: no sabe qué comprar y el tiempo apremia.
- Busca en Google: "regalo ropa sostenible", "ropa de lino mujer verano", "camiseta algodón orgánico hombre".
- Contenido que le convierte: guías de regalo, páginas de categoría bien optimizadas, artículos de producto.

### Gaps detectados (contenido que probablemente no tienes)

Basándome en el perfil típico de un blog de moda sostenible con tráfico cero, estos son los gaps más frecuentes:

1. No hay ninguna Pillar Page sobre el tema central (moda sostenible, armario cápsula, materiales ecológicos).
2. Falta contenido BOFU: artículos que llevan directamente al producto o a la compra.
3. Los artículos existentes no están interenlazados: cada post es una isla.
4. Las páginas de categoría del e-commerce no tienen texto optimizado (son solo rejilla de productos).
5. No hay contenido de comparativas de materiales o certificados, que es lo que busca el Perfil 2.

---

## 5. Calendario Editorial (Post-Auditoría)

Este calendario asume 1 persona a tiempo parcial produciendo contenido, empezando en el mes 2 (después de completar la fase de auditoría y limpieza del mes 1).

| Semana | Título propuesto | Nivel embudo | Keyword principal | Formato | Canal | Estado |
|---|---|---|---|---|---|---|
| S5 | Guía completa de moda sostenible: qué es, por qué importa y cómo empezar | TOFU | moda sostenible guía | Pillar Page 3.000p | Blog | Por hacer |
| S6 | Qué es el algodón orgánico y por qué es mejor que el convencional | TOFU | algodón orgánico qué es | Blog post 1.200p | Blog + Instagram | Por hacer |
| S7 | Certificados de ropa sostenible: GOTS, OEKO-TEX, Fair Trade explicados | MOFU | certificados ropa sostenible | Blog post 1.500p | Blog + Pinterest | Por hacer |
| S8 | Cómo construir un armario cápsula sostenible (con lista de prendas) | TOFU | armario cápsula sostenible | Blog post 1.800p + descargable | Blog + Email | Por hacer |
| S9 | Greenwashing en moda: cómo detectarlo y marcas que lo hacen | MOFU | greenwashing moda | Blog post 1.200p | Blog + Instagram | Por hacer |
| S10 | Los mejores tejidos sostenibles para ropa: lino, tencel, cáñamo, bambú | TOFU | tejidos sostenibles ropa | Blog comparativa 1.500p | Blog + Pinterest | Por hacer |
| S11 | Ropa sostenible para trabajo: cómo vestir bien sin fast fashion en la oficina | MOFU | ropa sostenible trabajo mujer | Blog post 1.000p | Blog + LinkedIn | Por hacer |
| S12 | Guía de regalo de ropa sostenible: ideas para cada ocasión y presupuesto | BOFU | regalo ropa sostenible | Blog post 1.000p | Blog + Email | Por hacer |
| S13 | Cómo lavar ropa de algodón orgánico para que dure más años | Fidelización | lavar ropa algodón orgánico | Blog post 900p | Blog + Email | Por hacer |
| S14 | Fast fashion vs moda sostenible: impacto real y alternativas concretas | TOFU | fast fashion vs moda sostenible | Blog post 1.500p | Blog + Instagram | Por hacer |
| S15 | [ACTUALIZACIÓN] Pillar Page: añadir FAQ, datos 2025, nuevos interenlaces | TOFU | moda sostenible | Actualización | Blog | Por hacer |
| S16 | Camisetas de algodón orgánico para hombre: guía de compra | BOFU | camiseta algodón orgánico hombre | Blog post 900p + CTA producto | Blog | Por hacer |

**Distribución del calendario:** 5 TOFU (42%) / 4 MOFU (33%) / 2 BOFU (17%) / 1 Fidelización (8%). Ligeramente más TOFU al inicio para construir base de tráfico; los BOFU aumentan en el trimestre 2 cuando el blog ya tiene autoridad.

---

## 6. Quick Wins (resultados en menos de 30 días)

1. **Optimiza el artículo Promesa con mejor posición actual.** Si tienes un artículo en posición 8-15, añádele 400 palabras de contenido complementario, una sección FAQ con 3 preguntas de "People Also Ask" y un enlace interno desde otros 2 artículos. Esto puede subirte a posición 4-7 en 3-4 semanas.

2. **Añade texto a tus páginas de categoría del e-commerce.** Las páginas de categoría (tipo `/ropa-mujer/`, `/camisetas-organicas/`) probablemente tienen 0 texto optimizado. Añadir 200-300 palabras con la keyword de categoría en H1 y primer párrafo es la acción SEO más directa para un e-commerce y Google la indexa rápido.

3. **Interenlaza los artículos existentes entre sí.** Revisa cada artículo del blog y añade 2-3 enlaces internos a otros artículos relacionados. Esta tarea no requiere crear contenido nuevo, se hace en pocas horas y mejora la distribución de autoridad interna de forma inmediata.

---

## 7. KPIs de Seguimiento

| Métrica | Herramienta | Cuándo revisar | Objetivo a 6 meses |
|---|---|---|---|
| Impresiones totales en GSC | Google Search Console | Mensual | x3 respecto al inicio |
| Clics orgánicos totales | Google Search Console | Mensual | De casi 0 a 500+/mes |
| Posición media de las Promesas | Google Search Console por página | Cada 3 semanas | Pasar de posición 8-20 a 3-8 |
| Tráfico orgánico a páginas de categoría | GA4 | Mensual | De 0 a 200+/mes por categoría principal |
| Tasa de conversión blog → producto | GA4 (embudo) | Mensual | 0,5-2% de sesiones blog con clic a producto |
| Páginas indexadas en Google | Google Search Console > Cobertura | Tras cada eliminación de zombies | Reducción limpia sin páginas con errores |

**Revisión estratégica:** después de completar la auditoría y los primeros 2 meses de producción nueva, programa una revisión completa del inventario para reclasificar. Los artículos que eran Promesa habrán evolucionado (algunos a Estrella, otros necesitarán más trabajo).

---

*Estrategia elaborada por el agente Content Strategist — Agencia Web. Fecha de referencia: mayo 2026.*
