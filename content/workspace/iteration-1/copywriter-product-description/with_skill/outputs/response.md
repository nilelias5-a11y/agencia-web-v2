# Copywriter — Descripción de Producto
## Blend de Temporada Etiopía-Brasil · 250g · 18€

---

## Fase 1: Marco estratégico

**A quién habla:** Aficionados al café de especialidad en Barcelona, entre 28-50 años, que ya conocen la diferencia entre un café de supermercado y uno de origen. Pagan con gusto por calidad cuando entienden el porqué.

**Tono:** Cercano, sensorial y con criterio. Sin tecnicismos innecesarios, pero tampoco simplón. Habla como un barista experto que también es buen amigo.

**Propuesta de valor única:** No es un blend cualquiera — es una combinación de temporada pensada para equilibrar la fruta viva de Etiopía con el cuerpo achocolatado de Brasil. Edición limitada que justifica el precio con su historia.

---

## Fase 2: Copy narrativo

### Descripción de producto — Página PDP

**Título (H1):** Etiopía y Brasil en una taza

**Descripción principal:**

Dos orígenes. Un equilibrio perfecto.

Este blend de temporada une la **viveza frutal de Etiopía** con el **cuerpo redondo y achocolatado de Brasil** en un tostado medio que lo hace versátil para cualquier método de preparación — ya sea tu espresso matutino o un cold brew de tarde.

Cada sorbo abre con notas de **fruta roja madura** — frambuesa, cereza — y cierra con un largo postgusto de **chocolate negro suave**. Sin amargor agresivo. Sin acidez que canse.

**Por qué este blend, por qué ahora:**

- Cosecha actual de Etiopía Yirgacheffe (proceso natural, alta altitud)
- Café de Brasil Cerrado Mineiro como base que da cuerpo y dulzor
- Tostado medio: equilibrado para espresso, pour over y prensa francesa
- **Edición de temporada** — cuando se acabe, se acaba

**Cantidad:** 250g · Precio: 18€

**CTA primario:** Añadir al carrito
**CTA secundario:** Ver cómo lo tostamos

---

## Bloque JSON — messages/es.json

```json
{
  "ProductPage": {
    "blendEtiopiaBrasil": {
      "title": "Etiopía y Brasil en una taza",
      "tagline": "Dos orígenes. Un equilibrio perfecto.",
      "description": "Este blend de temporada une la viveza frutal de Etiopía con el cuerpo redondo y achocolatado de Brasil en un tostado medio que lo hace versátil para cualquier método de preparación.",
      "sensoryIntro": "Cada sorbo abre con notas de fruta roja madura — frambuesa, cereza — y cierra con un largo postgusto de chocolate negro suave. Sin amargor agresivo. Sin acidez que canse.",
      "reasonsTitle": "Por qué este blend, por qué ahora",
      "reasons": {
        "origin1": "Cosecha actual de Etiopía Yirgacheffe — proceso natural, alta altitud",
        "origin2": "Brasil Cerrado Mineiro como base que aporta cuerpo y dulzor",
        "roast": "Tostado medio: equilibrado para espresso, pour over y prensa francesa",
        "limited": "Edición de temporada — cuando se acabe, se acaba"
      },
      "weight": "250g",
      "price": "18€",
      "ctaPrimary": "Añadir al carrito",
      "ctaSecondary": "Ver cómo lo tostamos",
      "badge": "Edición de temporada",
      "tasteNotes": {
        "label": "Notas de cata",
        "notes": "Fruta roja · Chocolate negro · Tostado medio"
      }
    }
  }
}
```

---

## Notas de implementación

1. **Badge de temporada:** La clave `badge` debe mostrarse visualmente destacada (etiqueta, ribbon o chip de color) sobre la imagen del producto para reforzar la urgencia de edición limitada. Retírala del JSON cuando el producto se agote o cambie la temporada.

2. **Notas de cata (`tasteNotes`):** Renderizar como chips/pills visuales (iconografía de sabor), no como texto plano. Es el elemento que más impacta al usuario que escanea en diagonal.

3. **Razones (`reasons`):** Usar una lista `<ul>` con bullets estilizados — no párrafo corrido. Cada ítem debe verse completo en móvil sin scroll adicional.

4. **CTA primario (`ctaPrimary`):** Vincular directamente a la acción `addToCart` con el SKU del producto como parámetro. No navegar a otra página antes de añadir.

5. **CTA secundario (`ctaSecondary`):** Puede apuntar a una sección de la página (anchor `#proceso-tostado`) o a una página de contenido tipo `/proceso`. Bajo compromiso — no abre modal.
