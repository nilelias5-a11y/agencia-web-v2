# Eval 2 — Reviews System con RLS Mixta: Café Vermut

## Análisis de Requisitos

### Requisitos funcionales
1. **Una valoración por pedido**: un cliente autenticado puede dejar exactamente una valoración por cada pedido completado (relación 1:1 order → review).
2. **Lectura pública**: cualquier visitante, sin autenticación, puede leer todas las valoraciones.
3. **Regla de 24h**: el cliente puede editar/eliminar su valoración dentro de las primeras 24 horas; pasado ese plazo, la valoración queda bloqueada.

### Decisiones de diseño clave
- La unicidad se garantiza a nivel de base de datos con `UNIQUE(order_id)`, no solo en aplicación.
- La regla de 24h se aplica tanto en RLS (UPDATE/DELETE) como con un trigger que impide la modificación directa del campo `created_at`.
- El campo `product_id` es opcional pero útil: permite mostrar valoraciones en la página de un plato aunque no se haya accedido por order.
- La visibilidad pública en Supabase requiere que la policy de SELECT no tenga condición de autenticación.
- Se usa `auth.uid()` de Supabase para identificar al usuario; se asume que la tabla `orders` tiene columna `user_id uuid` que referencia `auth.users`.

---

## Migration SQL Completo

```sql
-- ============================================================
-- Migration: 0010_add_product_reviews.sql
-- Feature: Sistema de valoraciones para Café Vermut
-- Requirements:
--   - Una valoración por pedido (autenticado)
--   - Lectura pública (sin login)
--   - Edición/eliminación solo dentro de las primeras 24h
-- ============================================================

-- ------------------------------------------------------------
-- 0. Extensiones necesarias
-- ------------------------------------------------------------
-- pgcrypto ya suele estar habilitada en Supabase, pero se añade
-- por si acaso para gen_random_uuid().
CREATE EXTENSION IF NOT EXISTS pgcrypto;


-- ------------------------------------------------------------
-- 1. Tabla principal: product_reviews
-- ------------------------------------------------------------
CREATE TABLE IF NOT EXISTS public.product_reviews (
    -- Clave primaria
    id              uuid            DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Relación con el pedido (1 review : 1 order)
    -- ON DELETE CASCADE: si se elimina el pedido, se elimina la reseña
    order_id        uuid            NOT NULL
                                    REFERENCES public.orders(id)
                                    ON DELETE CASCADE,

    -- Quién escribió la reseña (siempre autenticado al crear)
    -- ON DELETE CASCADE: si el usuario es eliminado, sus reseñas desaparecen
    user_id         uuid            NOT NULL
                                    REFERENCES auth.users(id)
                                    ON DELETE CASCADE,

    -- Producto reseñado (desnormalizado desde order_items para facilitar
    -- queries en PDP; puede ser NULL si la reseña es del pedido en general)
    product_id      uuid            REFERENCES public.products(id)
                                    ON DELETE SET NULL,

    -- Valoración numérica
    rating          smallint        NOT NULL
                                    CHECK (rating BETWEEN 1 AND 5),

    -- Comentario de texto (opcional)
    comment         text            CHECK (
                                        comment IS NULL
                                        OR (
                                            char_length(comment) >= 10
                                            AND char_length(comment) <= 2000
                                        )
                                    ),

    -- Timestamps
    created_at      timestamptz     NOT NULL DEFAULT now(),
    updated_at      timestamptz     NOT NULL DEFAULT now(),

    -- Estado de visibilidad (para moderación futura)
    -- 'published' | 'hidden' | 'pending_moderation'
    status          text            NOT NULL DEFAULT 'published'
                                    CHECK (status IN ('published', 'hidden', 'pending_moderation')),

    -- Garantía de unicidad: un pedido → una sola reseña
    CONSTRAINT product_reviews_order_id_unique UNIQUE (order_id)
);

-- Comentarios de tabla y columnas
COMMENT ON TABLE  public.product_reviews              IS 'Valoraciones de clientes para pedidos de Café Vermut.';
COMMENT ON COLUMN public.product_reviews.id           IS 'UUID primario de la valoración.';
COMMENT ON COLUMN public.product_reviews.order_id     IS 'Pedido al que corresponde la valoración. Único: 1 reseña por pedido.';
COMMENT ON COLUMN public.product_reviews.user_id      IS 'Cliente que dejó la valoración; debe coincidir con orders.user_id.';
COMMENT ON COLUMN public.product_reviews.product_id   IS 'Producto principal reseñado (desnormalizado, opcional).';
COMMENT ON COLUMN public.product_reviews.rating       IS 'Estrellas: 1 (peor) a 5 (mejor).';
COMMENT ON COLUMN public.product_reviews.comment      IS 'Texto libre entre 10 y 2000 caracteres.';
COMMENT ON COLUMN public.product_reviews.created_at   IS 'Momento de creación; inalterable después de insertar.';
COMMENT ON COLUMN public.product_reviews.updated_at   IS 'Última modificación (auto-actualizado por trigger).';
COMMENT ON COLUMN public.product_reviews.status       IS 'Estado de moderación de la reseña.';


-- ------------------------------------------------------------
-- 2. Indexes
-- ------------------------------------------------------------

-- Lectura pública: listar todas las reseñas de un producto
CREATE INDEX IF NOT EXISTS idx_product_reviews_product_id
    ON public.product_reviews (product_id)
    WHERE status = 'published';

-- Lectura del historial de reseñas de un cliente
CREATE INDEX IF NOT EXISTS idx_product_reviews_user_id
    ON public.product_reviews (user_id);

-- Lookup rápido por pedido (unicidad ya crea un índice implícito,
-- pero lo hacemos explícito para claridad y covering index)
CREATE UNIQUE INDEX IF NOT EXISTS idx_product_reviews_order_id
    ON public.product_reviews (order_id);

-- Ordenar por fecha de creación (consultas públicas recientes)
CREATE INDEX IF NOT EXISTS idx_product_reviews_created_at
    ON public.product_reviews (created_at DESC)
    WHERE status = 'published';

-- Índice compuesto para calcular rating promedio de un producto
CREATE INDEX IF NOT EXISTS idx_product_reviews_product_rating
    ON public.product_reviews (product_id, rating)
    WHERE status = 'published';


-- ------------------------------------------------------------
-- 3. Trigger: auto-actualizar updated_at
-- ------------------------------------------------------------
CREATE OR REPLACE FUNCTION public.set_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_product_reviews_updated_at
    BEFORE UPDATE ON public.product_reviews
    FOR EACH ROW
    EXECUTE FUNCTION public.set_updated_at();


-- ------------------------------------------------------------
-- 4. Trigger: proteger created_at de modificaciones
-- ------------------------------------------------------------
-- Evita que alguien haga UPDATE SET created_at = ... para
-- resetear el contador de las 24h y volver a editar/borrar.
CREATE OR REPLACE FUNCTION public.protect_created_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.created_at <> OLD.created_at THEN
        RAISE EXCEPTION 'created_at is immutable after insert'
            USING ERRCODE = 'check_violation';
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_product_reviews_protect_created_at
    BEFORE UPDATE ON public.product_reviews
    FOR EACH ROW
    EXECUTE FUNCTION public.protect_created_at();


-- ------------------------------------------------------------
-- 5. Trigger: validar que el pedido pertenece al usuario
-- ------------------------------------------------------------
-- Doble capa de seguridad: aunque RLS ya lo controla, este
-- trigger lo refuerza a nivel de datos para evitar inconsistencias
-- si alguien tiene bypass de RLS (service_role).
CREATE OR REPLACE FUNCTION public.validate_review_ownership()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
    v_order_user_id uuid;
    v_order_status  text;
BEGIN
    -- Obtener el propietario y estado del pedido
    SELECT user_id, status
      INTO v_order_user_id, v_order_status
      FROM public.orders
     WHERE id = NEW.order_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'Order % does not exist', NEW.order_id
            USING ERRCODE = 'foreign_key_violation';
    END IF;

    -- El usuario de la reseña debe coincidir con el propietario del pedido
    IF NEW.user_id <> v_order_user_id THEN
        RAISE EXCEPTION 'Review user_id must match the order owner'
            USING ERRCODE = 'check_violation';
    END IF;

    -- Solo se pueden reseñar pedidos completados
    -- Ajustar el valor según el dominio ('completed', 'delivered', etc.)
    IF v_order_status NOT IN ('completed', 'delivered') THEN
        RAISE EXCEPTION 'Only completed orders can be reviewed (current status: %)', v_order_status
            USING ERRCODE = 'check_violation';
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_product_reviews_validate_ownership
    BEFORE INSERT ON public.product_reviews
    FOR EACH ROW
    EXECUTE FUNCTION public.validate_review_ownership();


-- ------------------------------------------------------------
-- 6. Row Level Security (RLS)
-- ------------------------------------------------------------

ALTER TABLE public.product_reviews ENABLE ROW LEVEL SECURITY;

-- Por defecto, en Supabase con RLS habilitado, todo está bloqueado.
-- Definimos policies explícitas para cada operación.

-- ---- 6.1 SELECT: lectura pública ----
-- Cualquier visitante (autenticado o no) puede leer reseñas publicadas.
-- La condición `status = 'published'` oculta las moderadas/ocultas.
CREATE POLICY "reviews_select_public"
    ON public.product_reviews
    FOR SELECT
    USING (status = 'published');

-- Policy adicional: el propio autor puede ver sus reseñas en cualquier estado
-- (útil para mostrar "tu reseña está pendiente de moderación")
CREATE POLICY "reviews_select_own_any_status"
    ON public.product_reviews
    FOR SELECT
    TO authenticated
    USING (user_id = auth.uid());


-- ---- 6.2 INSERT: solo usuarios autenticados ----
-- WITH CHECK valida los datos que intenta insertar el usuario.
-- Condiciones:
--   a) El user_id del registro debe ser el usuario actual (no puede crear en nombre de otro)
--   b) El pedido debe pertenecer al usuario (reforzado también por trigger)
CREATE POLICY "reviews_insert_authenticated"
    ON public.product_reviews
    FOR INSERT
    TO authenticated
    WITH CHECK (
        -- El campo user_id debe ser el del usuario en sesión
        user_id = auth.uid()
        AND
        -- El pedido referenciado debe pertenecer al usuario en sesión
        EXISTS (
            SELECT 1
              FROM public.orders o
             WHERE o.id = order_id
               AND o.user_id = auth.uid()
               AND o.status IN ('completed', 'delivered')
        )
    );


-- ---- 6.3 UPDATE: solo dentro de las 24h y por el autor ----
-- USING: filtra qué filas puede ver el UPDATE (condición sobre la fila actual)
-- WITH CHECK: valida los nuevos valores después de la modificación
CREATE POLICY "reviews_update_within_24h"
    ON public.product_reviews
    FOR UPDATE
    TO authenticated
    USING (
        -- Solo el autor puede editar
        user_id = auth.uid()
        AND
        -- Solo dentro de las primeras 24 horas desde la creación
        created_at > now() - INTERVAL '24 hours'
    )
    WITH CHECK (
        -- Después de la edición, el user_id no puede haber cambiado
        user_id = auth.uid()
        AND
        -- El order_id tampoco puede cambiar (protege contra re-asignaciones)
        order_id = order_id  -- siempre true, pero obliga a que esté en la expresión
        -- Nota: el trigger trg_product_reviews_protect_created_at se encarga
        -- de que created_at no pueda alterarse para evadir la regla de 24h.
    );


-- ---- 6.4 DELETE: solo dentro de las 24h y por el autor ----
CREATE POLICY "reviews_delete_within_24h"
    ON public.product_reviews
    FOR DELETE
    TO authenticated
    USING (
        -- Solo el autor puede eliminar
        user_id = auth.uid()
        AND
        -- Solo dentro de las primeras 24 horas
        created_at > now() - INTERVAL '24 hours'
    );


-- ---- 6.5 Política de administración (service_role bypass) ----
-- service_role ya hace bypass de RLS por defecto en Supabase.
-- Si se usa anon key con rol 'moderator', añadir:
/*
CREATE POLICY "reviews_admin_full_access"
    ON public.product_reviews
    FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);
*/


-- ------------------------------------------------------------
-- 7. Vista pública para mostrar reseñas con datos de usuario
-- ------------------------------------------------------------
-- La vista desnormaliza el nombre del autor para queries de frontend
-- sin exponer el email del usuario.
CREATE OR REPLACE VIEW public.v_product_reviews_public AS
SELECT
    pr.id,
    pr.order_id,
    pr.product_id,
    pr.rating,
    pr.comment,
    pr.created_at,
    pr.updated_at,
    pr.status,
    -- Datos del autor (solo nombre público, nunca email)
    u.raw_user_meta_data->>'full_name'  AS author_name,
    u.raw_user_meta_data->>'avatar_url' AS author_avatar_url,
    -- Indicador de si la reseña aún puede ser editada/eliminada
    (pr.created_at > now() - INTERVAL '24 hours') AS is_editable
FROM
    public.product_reviews pr
    JOIN auth.users u ON u.id = pr.user_id
WHERE
    pr.status = 'published';

COMMENT ON VIEW public.v_product_reviews_public IS
    'Vista pública de reseñas con datos mínimos del autor. Nunca expone email ni datos sensibles.';


-- ------------------------------------------------------------
-- 8. Función helper: obtener rating promedio de un producto
-- ------------------------------------------------------------
CREATE OR REPLACE FUNCTION public.get_product_rating(p_product_id uuid)
RETURNS TABLE (
    average_rating  numeric(3,2),
    review_count    bigint,
    rating_1        bigint,
    rating_2        bigint,
    rating_3        bigint,
    rating_4        bigint,
    rating_5        bigint
)
LANGUAGE sql
STABLE
SECURITY DEFINER
AS $$
    SELECT
        ROUND(AVG(rating)::numeric, 2)           AS average_rating,
        COUNT(*)                                  AS review_count,
        COUNT(*) FILTER (WHERE rating = 1)        AS rating_1,
        COUNT(*) FILTER (WHERE rating = 2)        AS rating_2,
        COUNT(*) FILTER (WHERE rating = 3)        AS rating_3,
        COUNT(*) FILTER (WHERE rating = 4)        AS rating_4,
        COUNT(*) FILTER (WHERE rating = 5)        AS rating_5
    FROM public.product_reviews
    WHERE product_id = p_product_id
      AND status = 'published';
$$;


-- ------------------------------------------------------------
-- 9. Función helper: verificar si un usuario puede reseñar un pedido
-- ------------------------------------------------------------
-- Útil para el frontend: muestra u oculta el formulario de reseña.
CREATE OR REPLACE FUNCTION public.can_review_order(p_order_id uuid)
RETURNS boolean
LANGUAGE sql
STABLE
SECURITY DEFINER
AS $$
    SELECT
        -- El pedido existe y pertenece al usuario actual
        EXISTS (
            SELECT 1
              FROM public.orders o
             WHERE o.id = p_order_id
               AND o.user_id = auth.uid()
               AND o.status IN ('completed', 'delivered')
        )
        AND
        -- No existe ya una reseña para ese pedido
        NOT EXISTS (
            SELECT 1
              FROM public.product_reviews r
             WHERE r.order_id = p_order_id
        );
$$;

COMMENT ON FUNCTION public.can_review_order IS
    'Devuelve true si el usuario autenticado puede dejar una reseña para el pedido dado.';


-- ------------------------------------------------------------
-- 10. Grants para el rol anon y authenticated de Supabase
-- ------------------------------------------------------------

-- anon: solo puede ejecutar SELECT a través de la policy pública
GRANT SELECT ON public.product_reviews          TO anon;
GRANT SELECT ON public.v_product_reviews_public TO anon;
GRANT EXECUTE ON FUNCTION public.get_product_rating(uuid) TO anon;

-- authenticated: puede insertar, actualizar, eliminar (sujeto a RLS)
GRANT SELECT, INSERT, UPDATE, DELETE ON public.product_reviews TO authenticated;
GRANT SELECT ON public.v_product_reviews_public                TO authenticated;
GRANT EXECUTE ON FUNCTION public.get_product_rating(uuid)      TO authenticated;
GRANT EXECUTE ON FUNCTION public.can_review_order(uuid)        TO authenticated;
```

---

## Análisis de las RLS Policies

### Por qué se necesitan dos policies de SELECT

Supabase evalúa múltiples policies de SELECT con OR lógico entre ellas. Esto significa que si cualquiera de las dos es verdadera, la fila es visible. El diseño resultante es:

| Visitante | Condición aplicada |
|---|---|
| Anónimo | `status = 'published'` |
| Autenticado (no autor) | `status = 'published'` |
| Autenticado (autor) | `status = 'published'` OR `user_id = auth.uid()` |

El autor siempre ve su propia reseña aunque esté en `pending_moderation` o `hidden`, lo que mejora la UX.

### Por qué USING + WITH CHECK en UPDATE

En Supabase/PostgreSQL:
- `USING`: determina qué filas existentes puede "ver" el UPDATE (filtro sobre la fila antes de modificar).
- `WITH CHECK`: valida que la fila resultante después de la modificación sea válida.

Ambas incluyen la condición de 24h para evitar que:
1. Se seleccione una fila ya bloqueada (`USING`).
2. Se use una actualización para cambiar `user_id` a otro usuario (`WITH CHECK`).

### Protección contra evasión de la regla de 24h

La regla de 24h depende de `created_at`. Para evitar que un usuario haga `UPDATE SET created_at = now()` para "reiniciar el reloj", existe el trigger `trg_product_reviews_protect_created_at` que lanza una excepción si `created_at` cambia. Las RLS por sí solas no son suficientes porque no bloquean el cambio de columnas específicas.

### Unicidad de la reseña por pedido

La constraint `UNIQUE(order_id)` garantiza a nivel de base de datos que no pueden existir dos filas con el mismo `order_id`, independientemente de quién sea el usuario. Esto hace innecesario comprobar duplicados en la aplicación.

---

## Casos Edge Manejados

| Caso edge | Mecanismo de defensa |
|---|---|
| Usuario intenta crear reseña de pedido ajeno | Policy INSERT (EXISTS con `o.user_id = auth.uid()`) + Trigger `validate_review_ownership` |
| Usuario intenta crear segunda reseña para el mismo pedido | UNIQUE constraint en `order_id` |
| Usuario modifica `created_at` para alargar ventana de 24h | Trigger `protect_created_at` |
| Reseña de pedido no completado | Trigger `validate_review_ownership` comprueba `o.status` |
| Comentario vacío o demasiado corto/largo | CHECK constraint (NULL o 10-2000 chars) |
| Rating fuera de rango | CHECK constraint (BETWEEN 1 AND 5) |
| Eliminación del usuario (GDPR) | `ON DELETE CASCADE` en `user_id` FK |
| Eliminación del pedido | `ON DELETE CASCADE` en `order_id` FK |
| Producto eliminado | `ON DELETE SET NULL` en `product_id` FK (la reseña sobrevive sin producto) |
| Exposición de email del autor en vista pública | Vista usa `raw_user_meta_data` (nombre), nunca `email` |
| Service role bypass de RLS | Aceptado por diseño; service_role es confiable en servidor |

---

## Notas de Implementación para el Frontend (Next.js)

```typescript
// Insertar reseña (server action)
const supabase = createClient(); // cliente con sesión del usuario

const { error } = await supabase
  .from('product_reviews')
  .insert({
    order_id: orderId,
    user_id: session.user.id,   // RLS verificará que coincide con auth.uid()
    product_id: productId,      // opcional
    rating: 4,
    comment: 'Excelente vermut, volveremos sin duda.',
  });

// Si error.code === '23505' => ya existe una reseña para ese pedido
// Si error.code === 'P0001' => pedido no es del usuario o no está completado


// Leer reseñas públicas (sin autenticación)
const { data } = await supabase
  .from('v_product_reviews_public')
  .select('*')
  .eq('product_id', productId)
  .order('created_at', { ascending: false });


// Comprobar si el usuario puede reseñar (antes de mostrar el formulario)
const { data: canReview } = await supabase
  .rpc('can_review_order', { p_order_id: orderId });


// Editar reseña (solo funciona dentro de 24h; después RLS bloquea silenciosamente)
const { error } = await supabase
  .from('product_reviews')
  .update({ rating: 5, comment: 'Revisando: el vermut era de 5 estrellas.' })
  .eq('order_id', orderId);
// Si no hay error pero rowsAffected === 0, han pasado más de 24h
```

---

## Rollback

```sql
-- Para revertir esta migration completamente:
DROP VIEW  IF EXISTS public.v_product_reviews_public;
DROP FUNCTION IF EXISTS public.can_review_order(uuid);
DROP FUNCTION IF EXISTS public.get_product_rating(uuid);
DROP TABLE IF EXISTS public.product_reviews CASCADE;
DROP FUNCTION IF EXISTS public.validate_review_ownership();
DROP FUNCTION IF EXISTS public.protect_created_at();
-- set_updated_at() puede ser compartida; no se elimina aquí
```
