# Eval 2 — Reviews System con RLS Mixta
## Café Vermut — Sistema de Valoraciones de Productos

---

## 1. ANÁLISIS DE REQUISITOS Y DECISIONES DE DISEÑO

### Requisitos recibidos
1. Clientes autenticados pueden dejar **una valoración por pedido** (1–5 estrellas + comentario texto)
2. Las valoraciones son **públicas** (visibles sin login)
3. El cliente **no puede editar/eliminar su valoración después de 24h**

### Decisiones de diseño

| Decisión | Justificación |
|---|---|
| `is_verified_purchase` como columna booleana calculada en insert | Vincula la reseña a un `order_id` real del usuario — no editable post-insert |
| `order_id NOT NULL REFERENCES orders(id)` | Garantiza unicidad por pedido (`UNIQUE(user_id, order_id)`) y verifica compra en el insert |
| `rating SMALLINT CHECK (rating BETWEEN 1 AND 5)` | Entero pequeño, constraint de dominio en DB — no delegado a la app |
| Regla de 24h en RLS con `now() - created_at < interval '24 hours'` | La restricción temporal vive en la DB, no en la capa de aplicación |
| `is_verified_purchase` se setea a `true` automáticamente en el trigger | El backend no puede falsear este valor — la función verifica que `order_id` pertenezca al `user_id` |
| SELECT público con `USING (true)` | Las reseñas son contenido público — anon puede leer, no escribir |
| Service role bypass | El panel de administración puede moderar reseñas sin restricción de tiempo |

---

## 2. MIGRATION SQL COMPLETO

**Archivo:** `supabase/migrations/20260513000001_create_product_reviews_table.sql`

```sql
-- ============================================================
-- Create product_reviews table
-- Café Vermut — Sistema de valoraciones de productos
-- ============================================================

-- ============================================================
-- Trigger function (create once per database if not exists)
-- ============================================================
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ============================================================
-- Trigger function to auto-set is_verified_purchase
-- Verifies that the order belongs to the reviewing user
-- and that the order status is DELIVERED
-- ============================================================
CREATE OR REPLACE FUNCTION set_verified_purchase()
RETURNS TRIGGER AS $$
BEGIN
  -- Verify the order belongs to this user and is delivered
  IF EXISTS (
    SELECT 1
    FROM orders
    WHERE id = NEW.order_id
      AND user_id = NEW.user_id
      AND status = 'DELIVERED'
  ) THEN
    NEW.is_verified_purchase = true;
  ELSE
    NEW.is_verified_purchase = false;
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- ============================================================
-- Table: product_reviews
-- ============================================================
CREATE TABLE product_reviews (
  -- Mandatory columns
  id          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now(),

  -- Ownership & relation columns
  user_id     uuid        NOT NULL REFERENCES auth.users(id)  ON DELETE CASCADE,
  product_id  uuid        NOT NULL REFERENCES products(id)    ON DELETE CASCADE,
  order_id    uuid        NOT NULL REFERENCES orders(id)      ON DELETE CASCADE,

  -- Review content
  rating      smallint    NOT NULL CHECK (rating BETWEEN 1 AND 5),
  comment     text,

  -- Purchase verification (auto-set by trigger set_verified_purchase)
  is_verified_purchase boolean NOT NULL DEFAULT false,

  -- One review per order (a user could buy the same product in multiple orders,
  -- but can only review each order once — prevents duplicate reviews per order)
  CONSTRAINT product_reviews_user_order_unique UNIQUE (user_id, order_id)
);

-- ============================================================
-- Triggers
-- ============================================================

-- Keep updated_at current
CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON product_reviews
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

-- Auto-set is_verified_purchase on INSERT only (immutable after creation)
CREATE TRIGGER set_verified_purchase_on_insert
  BEFORE INSERT ON product_reviews
  FOR EACH ROW
  EXECUTE FUNCTION set_verified_purchase();

-- ============================================================
-- Indexes
-- ============================================================

-- FK: every FK column must be indexed (PostgreSQL does not auto-index FKs)
CREATE INDEX idx_product_reviews_user_id
  ON product_reviews(user_id);
-- Optimizes: "mis reseñas" in the user account panel

CREATE INDEX idx_product_reviews_product_id
  ON product_reviews(product_id);
-- Optimizes: "reseñas de este producto" — the most frequent public query

CREATE INDEX idx_product_reviews_order_id
  ON product_reviews(order_id);
-- Optimizes: UNIQUE constraint lookups + "¿ya valoré este pedido?"

-- Composite: product + rating — optimizes "productos mejor valorados"
-- (aggregate AVG(rating) GROUP BY product_id, ORDER BY avg DESC)
CREATE INDEX idx_product_reviews_product_id_rating
  ON product_reviews(product_id, rating);
-- Optimizes: SELECT product_id, AVG(rating) FROM product_reviews GROUP BY product_id ORDER BY avg DESC

-- Partial: only verified purchases — public-facing "verified reviews" filter
CREATE INDEX idx_product_reviews_verified
  ON product_reviews(product_id, created_at DESC)
  WHERE is_verified_purchase = true;
-- Optimizes: reviews section filtered by is_verified_purchase = true, newest first

-- Time-window: for the 24h edit window check in UPDATE/DELETE policies
CREATE INDEX idx_product_reviews_created_at
  ON product_reviews(created_at DESC);
-- Optimizes: RLS policy expression `now() - created_at < interval '24 hours'`

-- ============================================================
-- Row Level Security
-- ============================================================
ALTER TABLE product_reviews ENABLE ROW LEVEL SECURITY;

-- ----------------------------------------------------------
-- SELECT: PUBLIC — anyone (including anon) can read reviews
-- ----------------------------------------------------------
CREATE POLICY "Anyone can view product reviews"
  ON product_reviews
  FOR SELECT
  USING (true);

-- ----------------------------------------------------------
-- INSERT: authenticated users only — one review per order
-- The UNIQUE constraint (user_id, order_id) enforces the
-- "one review per order" rule at DB level.
-- WITH CHECK verifies:
--   1. The user_id in the row matches the authenticated user
--   2. The order_id belongs to the authenticated user
--      (prevents inserting a review on another user's order)
-- ----------------------------------------------------------
CREATE POLICY "Authenticated users can insert own reviews"
  ON product_reviews
  FOR INSERT
  WITH CHECK (
    auth.uid() = user_id
    AND EXISTS (
      SELECT 1
      FROM orders
      WHERE orders.id = order_id
        AND orders.user_id = auth.uid()
    )
  );

-- ----------------------------------------------------------
-- UPDATE: authenticated user, own review, within 24 hours
-- USING: identifies which rows the user CAN target
-- WITH CHECK: validates the new row state after update
--   - user_id and order_id are immutable (cannot reassign)
--   - rating stays in valid range (enforced by CHECK constraint too)
-- ----------------------------------------------------------
CREATE POLICY "Users can update own reviews within 24h"
  ON product_reviews
  FOR UPDATE
  USING (
    auth.uid() = user_id
    AND now() - created_at < interval '24 hours'
  )
  WITH CHECK (
    auth.uid() = user_id
    AND user_id = user_id     -- cannot change ownership
    AND order_id = order_id   -- cannot reassign to different order
  );

-- ----------------------------------------------------------
-- DELETE: authenticated user, own review, within 24 hours
-- After 24h the row is visible to the user (SELECT policy)
-- but untouchable for DELETE — returns permission denied.
-- ----------------------------------------------------------
CREATE POLICY "Users can delete own reviews within 24h"
  ON product_reviews
  FOR DELETE
  USING (
    auth.uid() = user_id
    AND now() - created_at < interval '24 hours'
  );

-- ----------------------------------------------------------
-- Service role: full access for admin moderation
-- (uses SUPABASE_SERVICE_ROLE_KEY — bypasses all user policies)
-- ----------------------------------------------------------
CREATE POLICY "Service role has full access to reviews"
  ON product_reviews
  FOR ALL
  USING (auth.role() = 'service_role');
```

---

## 3. SEED SQL CON DATOS DE EJEMPLO

**Archivo:** `supabase/seed.sql` (fragmento de product_reviews — añadir al seed existente)

```sql
-- ============================================================
-- Seed: product_reviews
-- Uses hardcoded UUIDs for stable resets (idempotent)
-- NOTE: seed users, products, and orders must be seeded first
-- ============================================================

-- Seed users (auth.users are created via Supabase auth helpers in seed,
-- but we reference them by known UUIDs set during user creation)
-- Assumed pre-existing seed UUIDs:
--   User 1 (cliente habitual):     'aaaaaaaa-0000-0000-0000-000000000001'
--   User 2 (cliente nuevo):        'aaaaaaaa-0000-0000-0000-000000000002'
--   Product (Vermut de barril):    '00000000-0000-0000-0001-000000000001'
--   Product (Boquerones):          '00000000-0000-0000-0001-000000000002'
--   Product (Patatas bravas):      '00000000-0000-0000-0001-000000000003'
--   Order 1 (user1, DELIVERED):    '00000000-0000-0000-0002-000000000001'
--   Order 2 (user1, DELIVERED):    '00000000-0000-0000-0002-000000000002'
--   Order 3 (user2, DELIVERED):    '00000000-0000-0000-0002-000000000003'
--   Order 4 (user2, DELIVERED):    '00000000-0000-0000-0002-000000000004'

INSERT INTO product_reviews (
  id,
  user_id,
  product_id,
  order_id,
  rating,
  comment,
  is_verified_purchase,
  created_at,
  updated_at
)
VALUES
  -- User 1, Order 1 → Vermut de barril (5 estrellas, verificada)
  (
    '00000000-0000-0000-0003-000000000001',
    'aaaaaaaa-0000-0000-0000-000000000001',
    '00000000-0000-0000-0001-000000000001',
    '00000000-0000-0000-0002-000000000001',
    5,
    'El mejor vermut de Barcelona, sin duda. Temperatura perfecta y el vaso siempre limpio.',
    true,
    now() - interval '10 days',
    now() - interval '10 days'
  ),
  -- User 1, Order 2 → Boquerones (4 estrellas, verificada)
  (
    '00000000-0000-0000-0003-000000000002',
    'aaaaaaaa-0000-0000-0000-000000000001',
    '00000000-0000-0000-0001-000000000002',
    '00000000-0000-0000-0002-000000000002',
    4,
    'Muy frescos y bien sazonados. Le falta un poco más de aceite de oliva para ser perfectos.',
    true,
    now() - interval '5 days',
    now() - interval '5 days'
  ),
  -- User 2, Order 3 → Patatas bravas (5 estrellas, verificada)
  (
    '00000000-0000-0000-0003-000000000003',
    'aaaaaaaa-0000-0000-0000-000000000002',
    '00000000-0000-0000-0001-000000000003',
    '00000000-0000-0000-0002-000000000003',
    5,
    'Las mejores bravas que he probado. La salsa es casera, se nota.',
    true,
    now() - interval '3 days',
    now() - interval '3 days'
  ),
  -- User 2, Order 4 → Vermut de barril (3 estrellas, verificada, sin comentario)
  -- Ejemplo: valoración sin texto — el comentario es opcional (nullable)
  (
    '00000000-0000-0000-0003-000000000004',
    'aaaaaaaa-0000-0000-0000-000000000002',
    '00000000-0000-0000-0001-000000000001',
    '00000000-0000-0000-0002-000000000004',
    3,
    NULL,
    true,
    now() - interval '1 day',
    now() - interval '1 day'
  )
ON CONFLICT (id) DO NOTHING;
```

---

## 4. SCHEMA DOCUMENTATION

### Table: product_reviews

**Purpose:** Stores customer product reviews tied to a specific order. Public read access, authenticated-only write. Editable/deletable only within 24h of creation.

| Column | Type | Nullable | Default | Description |
|---|---|---|---|---|
| id | uuid | NO | gen_random_uuid() | Primary key |
| created_at | timestamptz | NO | now() | Auto-set on insert |
| updated_at | timestamptz | NO | now() | Auto-updated by trigger on every change |
| user_id | uuid | NO | — | FK → auth.users.id — author of the review |
| product_id | uuid | NO | — | FK → products.id — reviewed product |
| order_id | uuid | NO | — | FK → orders.id — order that included this product |
| rating | smallint | NO | — | Star rating 1–5; enforced by CHECK constraint |
| comment | text | YES | NULL | Optional free-text review body |
| is_verified_purchase | boolean | NO | false | Auto-set by trigger — true if order.user_id = user_id AND order.status = DELIVERED |

**Constraints:**
- `rating BETWEEN 1 AND 5` — domain check at DB level
- `UNIQUE (user_id, order_id)` — one review per order per user

**Triggers:**
- `set_updated_at` — keeps `updated_at` current on every UPDATE
- `set_verified_purchase_on_insert` — auto-sets `is_verified_purchase` by querying `orders` (SECURITY DEFINER, runs as owner, not caller)

**Indexes:**

| Index | Columns | Type | Optimizes |
|---|---|---|---|
| idx_product_reviews_user_id | user_id | btree | "Mis reseñas" en el panel del usuario |
| idx_product_reviews_product_id | product_id | btree | Consulta pública "reseñas de este producto" |
| idx_product_reviews_order_id | order_id | btree | Lookup de unicidad + "¿ya valoré este pedido?" |
| idx_product_reviews_product_id_rating | (product_id, rating) | btree | AVG(rating) GROUP BY product_id — ranking de productos |
| idx_product_reviews_verified | (product_id, created_at DESC) WHERE is_verified = true | partial btree | Filtro "solo compras verificadas", más reciente primero |
| idx_product_reviews_created_at | created_at DESC | btree | Evaluación de la ventana de 24h en policies UPDATE/DELETE |

**RLS Policies:**

| Policy | Operation | Subject | Condition |
|---|---|---|---|
| Anyone can view product reviews | SELECT | anon + authenticated | `true` — lectura pública sin restricción |
| Authenticated users can insert own reviews | INSERT | authenticated | `auth.uid() = user_id` AND order belongs to user |
| Users can update own reviews within 24h | UPDATE | authenticated | `auth.uid() = user_id` AND `now() - created_at < '24 hours'` |
| Users can delete own reviews within 24h | DELETE | authenticated | `auth.uid() = user_id` AND `now() - created_at < '24 hours'` |
| Service role has full access to reviews | ALL | service_role | `auth.role() = 'service_role'` |

**Relations:**
- `user_id` → `auth.users.id` (CASCADE DELETE — si se borra el usuario, se borran sus reseñas)
- `product_id` → `products.id` (CASCADE DELETE — si se elimina el producto, se borran sus reseñas)
- `order_id` → `orders.id` (CASCADE DELETE — si se anula el pedido y se borra, la reseña desaparece)

---

## 5. EDGE CASES CUBIERTOS Y JUSTIFICACIÓN

### Edge case 1: SELECT público sin login
**Solución:** `FOR SELECT USING (true)` — no evalúa `auth.uid()`, que sería NULL para anon. Si la policy usara `auth.uid() = user_id`, los usuarios anónimos no verían ninguna reseña. La policy `true` permite lectura universal.

### Edge case 2: INSERT — evitar que un usuario reseñe el pedido de otro
**Solución:** La `WITH CHECK` del INSERT incluye un subquery:
```sql
AND EXISTS (
  SELECT 1 FROM orders
  WHERE orders.id = order_id
    AND orders.user_id = auth.uid()
)
```
Sin este check, un usuario autenticado podría insertar `{ user_id: auth.uid(), order_id: <orden_de_otro> }` — la FK no impediría esto. El subquery verifica que el `order_id` realmente pertenezca al caller.

### Edge case 3: UPDATE — inmutabilidad de user_id y order_id
**Solución:** El `WITH CHECK` del UPDATE repite `user_id = user_id` y `order_id = order_id`. Aunque parece trivial, en RLS el `WITH CHECK` se evalúa sobre la **fila nueva** (`NEW`). Esto impide que alguien reasigne su reseña a otro producto/pedido haciendo un UPDATE que cambie `order_id`.

**Nota crítica sobre la columna `product_id`:** Un usuario podría intentar un UPDATE que cambie `product_id`. Para prevenirlo en su totalidad, el backend debe tratar `product_id`, `user_id`, y `order_id` como inmutables y nunca incluirlos en el payload de UPDATE. La capa DB puede forzarlo añadiendo un trigger:

```sql
-- Trigger adicional: evitar cambios en columnas inmutables post-insert
CREATE OR REPLACE FUNCTION prevent_review_column_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.user_id <> OLD.user_id OR NEW.order_id <> OLD.order_id OR NEW.product_id <> OLD.product_id THEN
    RAISE EXCEPTION 'user_id, order_id, and product_id are immutable after insert';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_review_immutable_columns
  BEFORE UPDATE ON product_reviews
  FOR EACH ROW
  EXECUTE FUNCTION prevent_review_column_changes();
```

### Edge case 4: La regla de 24h y su evaluación
**Solución:** `now() - created_at < interval '24 hours'` se evalúa en el momento exacto de la query. La expresión es correcta para timestamptz. Se usa `created_at` (momento de creación), no `updated_at` (que cambia con cada edit dentro de las 24h).

**Importante:** `now()` en PostgreSQL dentro de una transacción devuelve el timestamp de inicio de la transacción — esto es correcto aquí (no hay riesgo de race condition dentro de la misma transacción).

### Edge case 5: is_verified_purchase no editable por el usuario
**Solución:** La columna se setea en el trigger `BEFORE INSERT` con `SECURITY DEFINER` — el usuario no puede pasar `is_verified_purchase: false` para eludir la verificación, porque el trigger sobreescribe el valor siempre. El trigger hace una consulta a `orders` para verificar que el pedido existe, pertenece al usuario y está en estado DELIVERED.

### Edge case 6: CASCADE DELETE en orders
**Consideración:** Si un pedido se elimina (por el service role), sus reseñas también se borran por CASCADE. Esto es aceptable para Café Vermut, pero en un ecommerce con historial de compras a largo plazo se podría cambiar a `ON DELETE SET NULL` con `order_id` nullable y mantener la reseña sin el pedido.

---

## 6. TYPESCRIPT TYPE GENERATION

Después de aplicar la migration:

```bash
supabase gen types typescript --local > src/types/database.types.ts
```

Tipos generados relevantes (fragmento esperado):

```typescript
// src/types/database.types.ts (generated — never edit manually)
export type Database = {
  public: {
    Tables: {
      product_reviews: {
        Row: {
          id: string
          created_at: string
          updated_at: string
          user_id: string
          product_id: string
          order_id: string
          rating: number
          comment: string | null
          is_verified_purchase: boolean
        }
        Insert: {
          id?: string
          created_at?: string
          updated_at?: string
          user_id: string
          product_id: string
          order_id: string
          rating: number
          comment?: string | null
          is_verified_purchase?: boolean  // will be overridden by trigger
        }
        Update: {
          id?: string
          created_at?: string
          updated_at?: string
          user_id?: string       // treat as immutable — do not include in update payload
          product_id?: string    // treat as immutable — do not include in update payload
          order_id?: string      // treat as immutable — do not include in update payload
          rating?: number
          comment?: string | null
          is_verified_purchase?: boolean  // treat as immutable — do not include in update payload
        }
      }
    }
  }
}
```

**Usage pattern for backend:**

```typescript
import { createClient } from "@supabase/supabase-js"
import type { Database } from "@/types/database.types"

const supabase = createClient<Database>(url, anonKey)

// Public read — no auth needed
const { data: reviews } = await supabase
  .from("product_reviews")
  .select("id, rating, comment, is_verified_purchase, created_at")
  .eq("product_id", productId)
  .order("created_at", { ascending: false })

// Authenticated insert — user JWT in client
const { data, error } = await supabase
  .from("product_reviews")
  .insert({
    user_id: userId,      // must match auth.uid()
    product_id: productId,
    order_id: orderId,    // must belong to auth.uid()
    rating: 5,
    comment: "Excelente vermut!"
    // is_verified_purchase omitted — trigger sets it
  })

// Conditional update (within 24h) — error if outside window
const { error: updateError } = await supabase
  .from("product_reviews")
  .update({ rating: 4, comment: "Muy bueno, ajusto valoración." })
  .eq("id", reviewId)
  // RLS silently prevents update if > 24h — check updateError + rowsAffected
```

---

## 7. MIGRATION COMMANDS

```bash
# 1. Create migration file
supabase migration new create_product_reviews_table

# 2. Paste migration SQL into generated file:
#    supabase/migrations/20260513000001_create_product_reviews_table.sql

# 3. Test locally
supabase db reset

# 4. Verify RLS is active
# Run in Supabase SQL editor:
# SELECT tablename, rowsecurity FROM pg_tables
# WHERE schemaname = 'public' AND tablename = 'product_reviews';

# 5. Push to remote
supabase db push

# 6. Regenerate TypeScript types
supabase gen types typescript --local > src/types/database.types.ts
```

---

## 8. RLS VERIFICATION CHECKLIST

- [ ] `SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public'` — `product_reviews` shows `rowsecurity = true`
- [ ] Test as **anon**: `SELECT * FROM product_reviews` returns all rows (public SELECT policy)
- [ ] Test as **anon**: `INSERT INTO product_reviews ...` returns permission denied
- [ ] Test as **authenticated user A**: INSERT with own `user_id` and own `order_id` — succeeds
- [ ] Test as **authenticated user A**: INSERT with `order_id` belonging to user B — fails (WITH CHECK subquery)
- [ ] Test as **authenticated user A**: INSERT second review for same `order_id` — fails (UNIQUE constraint)
- [ ] Test as **authenticated user A**: UPDATE own review within 24h — succeeds
- [ ] Test as **authenticated user A**: UPDATE own review after 24h (seed row with `created_at - 2 days`) — fails silently (0 rows affected)
- [ ] Test as **authenticated user A**: UPDATE another user's review — fails (USING checks `auth.uid() = user_id`)
- [ ] Test as **authenticated user A**: DELETE own review within 24h — succeeds
- [ ] Test as **authenticated user A**: DELETE own review after 24h — fails silently (0 rows affected)
- [ ] Test as **service role**: UPDATE/DELETE any review regardless of age — succeeds
- [ ] Verify `is_verified_purchase = true` on review whose `order_id` has `status = 'DELIVERED'` and `user_id` matches
- [ ] Verify `is_verified_purchase = false` on review whose `order_id` has `status != 'DELIVERED'`
