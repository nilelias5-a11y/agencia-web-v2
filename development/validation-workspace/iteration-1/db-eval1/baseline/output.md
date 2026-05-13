# Café Vermut — Schema de Base de Datos para Tienda Online de Café

## Visión general

Schema completo diseñado para Supabase (PostgreSQL 15+) con Row Level Security (RLS), índices justificados y migraciones SQL listas para producción. El modelo cubre tres entidades principales:

- **products** — catálogo de cafés molidos con variantes de gramaje
- **orders** — pedidos con ciclo de vida completo y referencia a Stripe
- **order_items** — líneas de pedido que congelan precio y cantidad en el momento de compra

---

## 1. Tabla `products`

### Decisiones de diseño

| Decisión | Justificación |
|---|---|
| `variants` como `JSONB` | Los gramajes (250 g, 500 g, 1 kg) comparten todos los atributos del producto; modelarlos en una tabla separada añade joins sin beneficio real a esta escala. JSONB permite validación con `CHECK` y consultas con operadores `@>`. |
| `price_cents INTEGER` | Evita errores de coma flotante en dinero. Todas las operaciones aritméticas son exactas. Se expone en euros dividiendo entre 100. |
| `slug TEXT UNIQUE` | URLs canónicas del producto sin depender del `id`. Facilita SSG/ISR en Next.js. |
| `active BOOLEAN` | Soft-delete: desactivar un producto no rompe `order_items` históricos que apuntan a él. |
| `image_urls TEXT[]` | Array de URLs (Supabase Storage o CDN externo). Sencillo y sin tabla extra de media. |

### DDL

```sql
-- ============================================================
-- EXTENSION: asegura que uuid_generate_v4() está disponible
-- ============================================================
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ============================================================
-- ENUM: variante de gramaje
-- ============================================================
CREATE TYPE weight_option AS ENUM ('250g', '500g', '1kg');

-- ============================================================
-- TABLE: products
-- ============================================================
CREATE TABLE products (
    -- Identificación
    id              UUID            PRIMARY KEY DEFAULT uuid_generate_v4(),
    slug            TEXT            NOT NULL UNIQUE,

    -- Contenido
    name            TEXT            NOT NULL CHECK (char_length(name) BETWEEN 2 AND 120),
    description     TEXT            CHECK (char_length(description) <= 2000),
    origin          TEXT,                          -- ej. "Etiopía, Yirgacheffe"
    roast_level     TEXT            CHECK (roast_level IN ('light', 'medium', 'dark', 'espresso')),

    -- Precio base (en céntimos de euro, ej. 1250 = 12,50 €)
    -- Es el precio del gramaje más pequeño o el precio único si no hay variantes
    price_cents     INTEGER         NOT NULL CHECK (price_cents > 0),

    -- Variantes de gramaje: array de objetos {weight, price_cents, sku}
    -- Ejemplo: [{"weight":"250g","price_cents":1250,"sku":"CV-ESP-250"},
    --           {"weight":"500g","price_cents":2200,"sku":"CV-ESP-500"}]
    variants        JSONB           NOT NULL DEFAULT '[]'::jsonb
                    CHECK (jsonb_typeof(variants) = 'array'),

    -- Inventario / disponibilidad
    stock_quantity  INTEGER         NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    active          BOOLEAN         NOT NULL DEFAULT true,
    featured        BOOLEAN         NOT NULL DEFAULT false,

    -- Media
    image_urls      TEXT[]          NOT NULL DEFAULT ARRAY[]::TEXT[],

    -- SEO
    meta_title      TEXT            CHECK (char_length(meta_title) <= 70),
    meta_description TEXT           CHECK (char_length(meta_description) <= 160),

    -- Auditoría
    created_at      TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE  products                IS 'Catálogo de cafés molidos de Café Vermut';
COMMENT ON COLUMN products.price_cents    IS 'Precio base en céntimos de euro (precio ÷ 100 = EUR)';
COMMENT ON COLUMN products.variants       IS 'Array JSONB de variantes: [{weight, price_cents, sku}]';
COMMENT ON COLUMN products.stock_quantity IS 'Stock total; se descuenta al confirmar pedido (CONFIRMED)';
```

### Trigger `updated_at`

```sql
-- Función reutilizable para todos los updated_at
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$;

CREATE TRIGGER products_set_updated_at
    BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

### Índices — `products`

```sql
-- 1. Listado público: productos activos ordenados por destacados y nombre
--    Usado en PLP (Product Listing Page) sin filtros adicionales
CREATE INDEX idx_products_active_featured
    ON products (active, featured DESC, name ASC)
    WHERE active = true;

-- 2. Búsqueda full-text en nombre y descripción (español)
--    Permite búsquedas tipo "tostado intenso" o "etiopía"
CREATE INDEX idx_products_fts
    ON products
    USING gin(to_tsvector('spanish', coalesce(name,'') || ' ' || coalesce(description,'')));

-- 3. Consulta de variantes JSONB (ej: productos que tienen variante 500g)
CREATE INDEX idx_products_variants_gin
    ON products USING gin(variants);

-- 4. Lookup por slug (ya cubierto por el UNIQUE, pero se documenta explícitamente)
--    El índice UNIQUE en slug ya actúa como B-tree index.
```

### RLS — `products`

```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Política pública: cualquiera puede leer productos activos
CREATE POLICY "products_select_active"
    ON products
    FOR SELECT
    USING (active = true);

-- Política admin: el rol service_role (backend / admin) puede hacer todo
-- En Supabase, service_role bypasses RLS por defecto.
-- Esta política cubre operaciones desde el Dashboard o scripts de seed.
CREATE POLICY "products_all_service_role"
    ON products
    FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);
```

---

## 2. Tabla `orders`

### Decisiones de diseño

| Decisión | Justificación |
|---|---|
| `status` como ENUM | Los estados son finitos y conocidos; ENUM garantiza integridad y permite índices parciales eficientes. |
| `total_cents INTEGER` | Consistencia con `price_cents`. El total se calcula y persiste al crear el pedido; no se recalcula dinámicamente para preservar el histórico exacto. |
| `stripe_payment_intent_id` | Referencia al PaymentIntent de Stripe. Único (no nullable una vez confirmado). Permite reconciliar webhooks con pedidos. |
| `stripe_session_id` | Referencia a la Checkout Session, si se usa Stripe Hosted Checkout. Útil para recuperar sesiones abandonadas. |
| `shipping_address JSONB` | La dirección es un snapshot en el momento del pedido. Si el usuario cambia su dirección después, el pedido histórico conserva la original. |
| `user_id` nullable | Permite pedidos de invitados (guest checkout) además de usuarios registrados. |
| `guest_email` | Para enviar confirmación a compradores sin cuenta. |

### DDL

```sql
-- ============================================================
-- ENUM: estados del pedido
-- ============================================================
CREATE TYPE order_status AS ENUM (
    'PENDING',      -- creado, pago no completado
    'CONFIRMED',    -- pago confirmado por Stripe webhook
    'SHIPPED',      -- enviado (número de tracking disponible)
    'DELIVERED',    -- entregado al cliente
    'CANCELLED'     -- cancelado (por cliente o administrador)
);

-- ============================================================
-- TABLE: orders
-- ============================================================
CREATE TABLE orders (
    -- Identificación
    id                          UUID            PRIMARY KEY DEFAULT uuid_generate_v4(),
    order_number                TEXT            NOT NULL UNIQUE,
                                -- Formato legible: CV-2026-00001
                                -- Se genera mediante secuencia (ver abajo)

    -- Usuario (nullable para guest checkout)
    user_id                     UUID            REFERENCES auth.users(id) ON DELETE SET NULL,
    guest_email                 TEXT            CHECK (guest_email ~* '^[^@]+@[^@]+\.[^@]+$'),

    -- Estado del ciclo de vida
    status                      order_status    NOT NULL DEFAULT 'PENDING',

    -- Importes (en céntimos)
    subtotal_cents              INTEGER         NOT NULL CHECK (subtotal_cents >= 0),
    shipping_cents              INTEGER         NOT NULL DEFAULT 0 CHECK (shipping_cents >= 0),
    discount_cents              INTEGER         NOT NULL DEFAULT 0 CHECK (discount_cents >= 0),
    total_cents                 INTEGER         NOT NULL CHECK (total_cents >= 0),
                                -- Constraint: total = subtotal + shipping - discount
    CONSTRAINT orders_total_check
        CHECK (total_cents = subtotal_cents + shipping_cents - discount_cents),

    -- Stripe
    stripe_payment_intent_id    TEXT            UNIQUE,  -- pi_xxx
    stripe_session_id           TEXT            UNIQUE,  -- cs_xxx
    stripe_charge_id            TEXT,                    -- ch_xxx (para reembolsos)

    -- Dirección de envío (snapshot al crear pedido)
    -- {name, line1, line2, city, postal_code, province, country}
    shipping_address            JSONB           NOT NULL DEFAULT '{}'::jsonb,

    -- Tracking de envío
    tracking_number             TEXT,
    carrier                     TEXT,           -- ej. "correos", "seur", "glovo"

    -- Notas
    customer_notes              TEXT            CHECK (char_length(customer_notes) <= 500),
    internal_notes              TEXT,

    -- Auditoría de estados
    confirmed_at                TIMESTAMPTZ,
    shipped_at                  TIMESTAMPTZ,
    delivered_at                TIMESTAMPTZ,
    cancelled_at                TIMESTAMPTZ,
    cancellation_reason         TEXT,

    -- Auditoría general
    created_at                  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at                  TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE  orders                           IS 'Pedidos de la tienda online de Café Vermut';
COMMENT ON COLUMN orders.order_number              IS 'Número legible para el cliente: CV-YYYY-NNNNN';
COMMENT ON COLUMN orders.total_cents               IS 'subtotal + shipping - discount, en céntimos de euro';
COMMENT ON COLUMN orders.stripe_payment_intent_id  IS 'ID del PaymentIntent de Stripe (pi_xxx)';
COMMENT ON COLUMN orders.shipping_address          IS 'Snapshot de la dirección en el momento del pedido';
```

### Secuencia para `order_number`

```sql
-- Secuencia para el número correlativo del pedido
CREATE SEQUENCE orders_number_seq START 1;

-- Función para generar el número legible
CREATE OR REPLACE FUNCTION generate_order_number()
RETURNS TEXT LANGUAGE plpgsql AS $$
BEGIN
    RETURN 'CV-' || TO_CHAR(NOW(), 'YYYY') || '-' ||
           LPAD(nextval('orders_number_seq')::TEXT, 5, '0');
END;
$$;

-- Trigger que asigna order_number en INSERT si no se provee
CREATE OR REPLACE FUNCTION set_order_number()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    IF NEW.order_number IS NULL OR NEW.order_number = '' THEN
        NEW.order_number := generate_order_number();
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER orders_set_order_number
    BEFORE INSERT ON orders
    FOR EACH ROW EXECUTE FUNCTION set_order_number();
```

### Trigger `updated_at` y auditoría de estados

```sql
CREATE TRIGGER orders_set_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Trigger: registra timestamps de transición de estado automáticamente
CREATE OR REPLACE FUNCTION orders_audit_status()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    IF OLD.status IS DISTINCT FROM NEW.status THEN
        CASE NEW.status
            WHEN 'CONFIRMED'  THEN NEW.confirmed_at  := COALESCE(NEW.confirmed_at,  NOW());
            WHEN 'SHIPPED'    THEN NEW.shipped_at    := COALESCE(NEW.shipped_at,    NOW());
            WHEN 'DELIVERED'  THEN NEW.delivered_at  := COALESCE(NEW.delivered_at,  NOW());
            WHEN 'CANCELLED'  THEN NEW.cancelled_at  := COALESCE(NEW.cancelled_at,  NOW());
            ELSE NULL;
        END CASE;
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER orders_status_audit
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION orders_audit_status();
```

### Índices — `orders`

```sql
-- 1. Panel de admin: listar todos los pedidos ordenados por fecha (más común)
CREATE INDEX idx_orders_created_at
    ON orders (created_at DESC);

-- 2. Historial de pedidos del usuario autenticado
CREATE INDEX idx_orders_user_id
    ON orders (user_id, created_at DESC)
    WHERE user_id IS NOT NULL;

-- 3. Webhook de Stripe: buscar pedido por payment_intent (hot path crítico)
--    El UNIQUE ya crea un índice; se documenta por su relevancia operacional.
--    CREATE UNIQUE INDEX ya existe sobre stripe_payment_intent_id.

-- 4. Filtro por estado: cola de pedidos pendientes de procesar
CREATE INDEX idx_orders_status
    ON orders (status, created_at DESC);

-- 5. Índice parcial: pedidos PENDING (para detectar pagos huérfanos / expirados)
CREATE INDEX idx_orders_pending
    ON orders (created_at ASC)
    WHERE status = 'PENDING';

-- 6. Búsqueda por número de pedido (lookup desde panel de admin o email)
--    El UNIQUE sobre order_number ya crea el índice B-tree necesario.
```

### RLS — `orders`

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Usuarios autenticados: solo ven sus propios pedidos
CREATE POLICY "orders_select_own"
    ON orders
    FOR SELECT
    TO authenticated
    USING (auth.uid() = user_id);

-- Usuarios autenticados: solo pueden crear pedidos para sí mismos
CREATE POLICY "orders_insert_own"
    ON orders
    FOR INSERT
    TO authenticated
    WITH CHECK (auth.uid() = user_id);

-- Usuarios autenticados: no pueden modificar pedidos directamente
-- (las actualizaciones de estado las hace el backend con service_role)

-- Admin / backend (service_role): acceso total
CREATE POLICY "orders_all_service_role"
    ON orders
    FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);

-- Invitados (anon): pueden crear pedidos con user_id = NULL
CREATE POLICY "orders_insert_guest"
    ON orders
    FOR INSERT
    TO anon
    WITH CHECK (user_id IS NULL AND guest_email IS NOT NULL);

-- Invitados: NO pueden leer pedidos (no tienen sesión para identificarse)
-- El backend envía confirmación por email y gestiona el acceso.
```

---

## 3. Tabla `order_items`

### Decisiones de diseño

| Decisión | Justificación |
|---|---|
| `unit_price_cents` | Congela el precio en el momento de compra. Si el precio del producto cambia mañana, el histórico del pedido sigue siendo correcto. |
| `weight_variant` | Congela también la variante elegida (250 g, 500 g, 1 kg) para que el fulfillment sepa qué empaquetar. |
| `product_snapshot JSONB` | Snapshot opcional del nombre y descripción del producto al comprar. Útil para mostrar el detalle del pedido aunque el producto sea eliminado/renombrado. |
| FK `ON DELETE RESTRICT` (products) | Un producto con pedidos no puede borrarse accidentalmente. Fuerza el uso de `active = false`. |
| FK `ON DELETE CASCADE` (orders) | Si se elimina un pedido (raro, solo en limpieza de datos), sus ítems desaparecen también. |

### DDL

```sql
-- ============================================================
-- TABLE: order_items
-- ============================================================
CREATE TABLE order_items (
    -- Identificación
    id                  UUID        PRIMARY KEY DEFAULT uuid_generate_v4(),
    order_id            UUID        NOT NULL REFERENCES orders(id)   ON DELETE CASCADE,
    product_id          UUID        NOT NULL REFERENCES products(id) ON DELETE RESTRICT,

    -- Variante elegida (snapshot)
    weight_variant      TEXT        NOT NULL DEFAULT '250g'
                        CHECK (weight_variant IN ('250g', '500g', '1kg')),
    product_sku         TEXT,       -- SKU de la variante en el momento de compra

    -- Cantidad y precio congelados al momento de la compra
    quantity            INTEGER     NOT NULL CHECK (quantity > 0),
    unit_price_cents    INTEGER     NOT NULL CHECK (unit_price_cents > 0),
    line_total_cents    INTEGER     NOT NULL CHECK (line_total_cents > 0),
                        -- Constraint: line_total = quantity * unit_price
    CONSTRAINT order_items_line_total_check
        CHECK (line_total_cents = quantity * unit_price_cents),

    -- Snapshot del producto (nombre + descripción al momento de compra)
    product_snapshot    JSONB       NOT NULL DEFAULT '{}'::jsonb,
                        -- {name, description, image_url}

    -- Auditoría
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
    -- No updated_at: los ítems son inmutables una vez creados
);

COMMENT ON TABLE  order_items                   IS 'Líneas de pedido; precio y variante congelados al momento de compra';
COMMENT ON COLUMN order_items.unit_price_cents  IS 'Precio unitario en céntimos en el momento de la compra';
COMMENT ON COLUMN order_items.line_total_cents  IS 'quantity × unit_price_cents; siempre consistente por CHECK constraint';
COMMENT ON COLUMN order_items.product_snapshot  IS 'Copia de {name, description, image_url} del producto al comprar';
```

### Índices — `order_items`

```sql
-- 1. JOIN más frecuente: obtener ítems de un pedido (hot path en detalle de pedido)
CREATE INDEX idx_order_items_order_id
    ON order_items (order_id);

-- 2. Análisis de ventas por producto: cuántas unidades se han vendido de cada café
CREATE INDEX idx_order_items_product_id
    ON order_items (product_id);

-- 3. Índice compuesto para reportes de ventas: (product_id, created_at)
--    Permite consultas tipo "ventas del producto X en los últimos 30 días"
CREATE INDEX idx_order_items_product_date
    ON order_items (product_id, created_at DESC);
```

### RLS — `order_items`

```sql
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;

-- Usuarios autenticados: solo ven los ítems de sus propios pedidos
CREATE POLICY "order_items_select_own"
    ON order_items
    FOR SELECT
    TO authenticated
    USING (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id = auth.uid()
        )
    );

-- Usuarios autenticados: pueden insertar ítems en pedidos propios PENDING
CREATE POLICY "order_items_insert_own"
    ON order_items
    FOR INSERT
    TO authenticated
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id = auth.uid()
              AND orders.status = 'PENDING'
        )
    );

-- Invitados: pueden insertar ítems en pedidos sin usuario (guest checkout)
CREATE POLICY "order_items_insert_guest"
    ON order_items
    FOR INSERT
    TO anon
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id IS NULL
              AND orders.status = 'PENDING'
        )
    );

-- No se permiten UPDATE ni DELETE desde el cliente (solo service_role)

-- Admin / backend (service_role): acceso total
CREATE POLICY "order_items_all_service_role"
    ON order_items
    FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);
```

---

## 4. Migration SQL completa

Archivo único listo para ejecutar en Supabase SQL Editor o como migración en `supabase/migrations/`.

```sql
-- ==============================================================
-- Migration: 20260513000000_cafe_vermut_ecommerce_schema
-- Proyecto:  Café Vermut — Tienda Online
-- Base de datos: Supabase (PostgreSQL 15+)
-- ==============================================================

BEGIN;

-- ──────────────────────────────────────────────────────────────
-- 0. EXTENSIONES
-- ──────────────────────────────────────────────────────────────
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";


-- ──────────────────────────────────────────────────────────────
-- 1. TIPOS ENUMERADOS
-- ──────────────────────────────────────────────────────────────
DO $$ BEGIN
    CREATE TYPE weight_option AS ENUM ('250g', '500g', '1kg');
EXCEPTION WHEN duplicate_object THEN NULL;
END $$;

DO $$ BEGIN
    CREATE TYPE order_status AS ENUM (
        'PENDING',
        'CONFIRMED',
        'SHIPPED',
        'DELIVERED',
        'CANCELLED'
    );
EXCEPTION WHEN duplicate_object THEN NULL;
END $$;


-- ──────────────────────────────────────────────────────────────
-- 2. FUNCIÓN AUXILIAR: updated_at
-- ──────────────────────────────────────────────────────────────
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$;


-- ──────────────────────────────────────────────────────────────
-- 3. TABLA: products
-- ──────────────────────────────────────────────────────────────
CREATE TABLE IF NOT EXISTS products (
    id               UUID        PRIMARY KEY DEFAULT uuid_generate_v4(),
    slug             TEXT        NOT NULL UNIQUE,
    name             TEXT        NOT NULL CHECK (char_length(name) BETWEEN 2 AND 120),
    description      TEXT        CHECK (char_length(description) <= 2000),
    origin           TEXT,
    roast_level      TEXT        CHECK (roast_level IN ('light', 'medium', 'dark', 'espresso')),
    price_cents      INTEGER     NOT NULL CHECK (price_cents > 0),
    variants         JSONB       NOT NULL DEFAULT '[]'::jsonb
                     CHECK (jsonb_typeof(variants) = 'array'),
    stock_quantity   INTEGER     NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    active           BOOLEAN     NOT NULL DEFAULT true,
    featured         BOOLEAN     NOT NULL DEFAULT false,
    image_urls       TEXT[]      NOT NULL DEFAULT ARRAY[]::TEXT[],
    meta_title       TEXT        CHECK (char_length(meta_title) <= 70),
    meta_description TEXT        CHECK (char_length(meta_description) <= 160),
    created_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at       TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE  products                IS 'Catálogo de cafés molidos de Café Vermut';
COMMENT ON COLUMN products.price_cents    IS 'Precio base en céntimos de euro';
COMMENT ON COLUMN products.variants       IS 'Array JSONB [{weight, price_cents, sku}]';
COMMENT ON COLUMN products.stock_quantity IS 'Stock total; se descuenta al confirmar pedido';

-- Trigger updated_at
DROP TRIGGER IF EXISTS products_set_updated_at ON products;
CREATE TRIGGER products_set_updated_at
    BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Índices
CREATE INDEX IF NOT EXISTS idx_products_active_featured
    ON products (active, featured DESC, name ASC)
    WHERE active = true;

CREATE INDEX IF NOT EXISTS idx_products_fts
    ON products
    USING gin(to_tsvector('spanish', coalesce(name,'') || ' ' || coalesce(description,'')));

CREATE INDEX IF NOT EXISTS idx_products_variants_gin
    ON products USING gin(variants);

-- RLS
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "products_select_active"    ON products;
DROP POLICY IF EXISTS "products_all_service_role" ON products;

CREATE POLICY "products_select_active"
    ON products FOR SELECT
    USING (active = true);

CREATE POLICY "products_all_service_role"
    ON products FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);


-- ──────────────────────────────────────────────────────────────
-- 4. TABLA: orders
-- ──────────────────────────────────────────────────────────────
CREATE SEQUENCE IF NOT EXISTS orders_number_seq START 1;

CREATE OR REPLACE FUNCTION generate_order_number()
RETURNS TEXT LANGUAGE plpgsql AS $$
BEGIN
    RETURN 'CV-' || TO_CHAR(NOW(), 'YYYY') || '-' ||
           LPAD(nextval('orders_number_seq')::TEXT, 5, '0');
END;
$$;

CREATE OR REPLACE FUNCTION set_order_number()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    IF NEW.order_number IS NULL OR NEW.order_number = '' THEN
        NEW.order_number := generate_order_number();
    END IF;
    RETURN NEW;
END;
$$;

CREATE OR REPLACE FUNCTION orders_audit_status()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    IF OLD.status IS DISTINCT FROM NEW.status THEN
        CASE NEW.status
            WHEN 'CONFIRMED'  THEN NEW.confirmed_at  := COALESCE(NEW.confirmed_at,  NOW());
            WHEN 'SHIPPED'    THEN NEW.shipped_at    := COALESCE(NEW.shipped_at,    NOW());
            WHEN 'DELIVERED'  THEN NEW.delivered_at  := COALESCE(NEW.delivered_at,  NOW());
            WHEN 'CANCELLED'  THEN NEW.cancelled_at  := COALESCE(NEW.cancelled_at,  NOW());
            ELSE NULL;
        END CASE;
    END IF;
    RETURN NEW;
END;
$$;

CREATE TABLE IF NOT EXISTS orders (
    id                         UUID         PRIMARY KEY DEFAULT uuid_generate_v4(),
    order_number               TEXT         NOT NULL UNIQUE,
    user_id                    UUID         REFERENCES auth.users(id) ON DELETE SET NULL,
    guest_email                TEXT         CHECK (guest_email ~* '^[^@]+@[^@]+\.[^@]+$'),
    status                     order_status NOT NULL DEFAULT 'PENDING',
    subtotal_cents             INTEGER      NOT NULL CHECK (subtotal_cents >= 0),
    shipping_cents             INTEGER      NOT NULL DEFAULT 0 CHECK (shipping_cents >= 0),
    discount_cents             INTEGER      NOT NULL DEFAULT 0 CHECK (discount_cents >= 0),
    total_cents                INTEGER      NOT NULL CHECK (total_cents >= 0),
    CONSTRAINT orders_total_check
        CHECK (total_cents = subtotal_cents + shipping_cents - discount_cents),
    stripe_payment_intent_id   TEXT         UNIQUE,
    stripe_session_id          TEXT         UNIQUE,
    stripe_charge_id           TEXT,
    shipping_address           JSONB        NOT NULL DEFAULT '{}'::jsonb,
    tracking_number            TEXT,
    carrier                    TEXT,
    customer_notes             TEXT         CHECK (char_length(customer_notes) <= 500),
    internal_notes             TEXT,
    confirmed_at               TIMESTAMPTZ,
    shipped_at                 TIMESTAMPTZ,
    delivered_at               TIMESTAMPTZ,
    cancelled_at               TIMESTAMPTZ,
    cancellation_reason        TEXT,
    created_at                 TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at                 TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE  orders                          IS 'Pedidos de la tienda online de Café Vermut';
COMMENT ON COLUMN orders.order_number             IS 'Número legible para el cliente: CV-YYYY-NNNNN';
COMMENT ON COLUMN orders.total_cents              IS 'subtotal + shipping - discount en céntimos';
COMMENT ON COLUMN orders.stripe_payment_intent_id IS 'ID del PaymentIntent de Stripe (pi_xxx)';
COMMENT ON COLUMN orders.shipping_address         IS 'Snapshot de la dirección al momento del pedido';

-- Triggers
DROP TRIGGER IF EXISTS orders_set_order_number ON orders;
CREATE TRIGGER orders_set_order_number
    BEFORE INSERT ON orders
    FOR EACH ROW EXECUTE FUNCTION set_order_number();

DROP TRIGGER IF EXISTS orders_set_updated_at ON orders;
CREATE TRIGGER orders_set_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();

DROP TRIGGER IF EXISTS orders_status_audit ON orders;
CREATE TRIGGER orders_status_audit
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION orders_audit_status();

-- Índices
CREATE INDEX IF NOT EXISTS idx_orders_created_at
    ON orders (created_at DESC);

CREATE INDEX IF NOT EXISTS idx_orders_user_id
    ON orders (user_id, created_at DESC)
    WHERE user_id IS NOT NULL;

CREATE INDEX IF NOT EXISTS idx_orders_status
    ON orders (status, created_at DESC);

CREATE INDEX IF NOT EXISTS idx_orders_pending
    ON orders (created_at ASC)
    WHERE status = 'PENDING';

-- RLS
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "orders_select_own"        ON orders;
DROP POLICY IF EXISTS "orders_insert_own"        ON orders;
DROP POLICY IF EXISTS "orders_insert_guest"      ON orders;
DROP POLICY IF EXISTS "orders_all_service_role"  ON orders;

CREATE POLICY "orders_select_own"
    ON orders FOR SELECT
    TO authenticated
    USING (auth.uid() = user_id);

CREATE POLICY "orders_insert_own"
    ON orders FOR INSERT
    TO authenticated
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "orders_insert_guest"
    ON orders FOR INSERT
    TO anon
    WITH CHECK (user_id IS NULL AND guest_email IS NOT NULL);

CREATE POLICY "orders_all_service_role"
    ON orders FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);


-- ──────────────────────────────────────────────────────────────
-- 5. TABLA: order_items
-- ──────────────────────────────────────────────────────────────
CREATE TABLE IF NOT EXISTS order_items (
    id                UUID        PRIMARY KEY DEFAULT uuid_generate_v4(),
    order_id          UUID        NOT NULL REFERENCES orders(id)   ON DELETE CASCADE,
    product_id        UUID        NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    weight_variant    TEXT        NOT NULL DEFAULT '250g'
                      CHECK (weight_variant IN ('250g', '500g', '1kg')),
    product_sku       TEXT,
    quantity          INTEGER     NOT NULL CHECK (quantity > 0),
    unit_price_cents  INTEGER     NOT NULL CHECK (unit_price_cents > 0),
    line_total_cents  INTEGER     NOT NULL CHECK (line_total_cents > 0),
    CONSTRAINT order_items_line_total_check
        CHECK (line_total_cents = quantity * unit_price_cents),
    product_snapshot  JSONB       NOT NULL DEFAULT '{}'::jsonb,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE  order_items                  IS 'Líneas de pedido con precio congelado al momento de compra';
COMMENT ON COLUMN order_items.unit_price_cents IS 'Precio unitario en céntimos al momento de compra';
COMMENT ON COLUMN order_items.line_total_cents IS 'quantity × unit_price_cents';
COMMENT ON COLUMN order_items.product_snapshot IS 'Copia {name, description, image_url} del producto al comprar';

-- Índices
CREATE INDEX IF NOT EXISTS idx_order_items_order_id
    ON order_items (order_id);

CREATE INDEX IF NOT EXISTS idx_order_items_product_id
    ON order_items (product_id);

CREATE INDEX IF NOT EXISTS idx_order_items_product_date
    ON order_items (product_id, created_at DESC);

-- RLS
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "order_items_select_own"       ON order_items;
DROP POLICY IF EXISTS "order_items_insert_own"       ON order_items;
DROP POLICY IF EXISTS "order_items_insert_guest"     ON order_items;
DROP POLICY IF EXISTS "order_items_all_service_role" ON order_items;

CREATE POLICY "order_items_select_own"
    ON order_items FOR SELECT
    TO authenticated
    USING (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id = auth.uid()
        )
    );

CREATE POLICY "order_items_insert_own"
    ON order_items FOR INSERT
    TO authenticated
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id = auth.uid()
              AND orders.status = 'PENDING'
        )
    );

CREATE POLICY "order_items_insert_guest"
    ON order_items FOR INSERT
    TO anon
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM orders
            WHERE orders.id = order_items.order_id
              AND orders.user_id IS NULL
              AND orders.status = 'PENDING'
        )
    );

CREATE POLICY "order_items_all_service_role"
    ON order_items FOR ALL
    TO service_role
    USING (true)
    WITH CHECK (true);


-- ──────────────────────────────────────────────────────────────
-- 6. DATOS DE EJEMPLO (seed opcional)
-- ──────────────────────────────────────────────────────────────
-- Descomentar para poblar el entorno de desarrollo

/*
INSERT INTO products (slug, name, description, origin, roast_level, price_cents, variants, stock_quantity, active, featured, image_urls)
VALUES
(
    'espresso-barcelona',
    'Espresso Barcelona',
    'Un blend potente con notas de chocolate amargo y avellana. Ideal para espresso y cappuccino.',
    'Brasil / Colombia blend',
    'espresso',
    1250,
    '[
        {"weight":"250g","price_cents":1250,"sku":"CV-ESP-250"},
        {"weight":"500g","price_cents":2200,"sku":"CV-ESP-500"},
        {"weight":"1kg", "price_cents":3900,"sku":"CV-ESP-1KG"}
    ]'::jsonb,
    150,
    true,
    true,
    ARRAY['https://cdn.cafevermut.com/products/espresso-barcelona.jpg']
),
(
    'etiopie-natural',
    'Etiopía Natural',
    'Café de origen único con notas de frutos rojos, jazmín y miel. Perfecto para V60 o AeroPress.',
    'Etiopía, Yirgacheffe',
    'light',
    1650,
    '[
        {"weight":"250g","price_cents":1650,"sku":"CV-ETI-250"},
        {"weight":"500g","price_cents":3000,"sku":"CV-ETI-500"}
    ]'::jsonb,
    75,
    true,
    false,
    ARRAY['https://cdn.cafevermut.com/products/etiopia-natural.jpg']
),
(
    'colombia-washed',
    'Colombia Washed',
    'Suave y equilibrado con acidez brillante de manzana verde y dulzor de caramelo.',
    'Colombia, Huila',
    'medium',
    1450,
    '[
        {"weight":"250g","price_cents":1450,"sku":"CV-COL-250"},
        {"weight":"500g","price_cents":2600,"sku":"CV-COL-500"},
        {"weight":"1kg", "price_cents":4800,"sku":"CV-COL-1KG"}
    ]'::jsonb,
    120,
    true,
    true,
    ARRAY['https://cdn.cafevermut.com/products/colombia-washed.jpg']
);
*/

COMMIT;
```

---

## 5. Resumen de índices justificados

| Tabla | Índice | Tipo | Justificación |
|---|---|---|---|
| products | `idx_products_active_featured` | B-tree parcial | PLP principal: filtra activos, ordena destacados primero |
| products | `idx_products_fts` | GIN | Búsqueda full-text en nombre y descripción |
| products | `idx_products_variants_gin` | GIN | Consultas JSONB sobre variantes de gramaje |
| orders | `idx_orders_created_at` | B-tree | Listado de pedidos en panel admin ordenado por fecha |
| orders | `idx_orders_user_id` | B-tree parcial | Historial del cliente autenticado |
| orders | `idx_orders_status` | B-tree | Cola de trabajo por estado (fulfillment) |
| orders | `idx_orders_pending` | B-tree parcial | Detección de pagos huérfanos / expirados |
| order_items | `idx_order_items_order_id` | B-tree | JOIN frecuente: ítems de un pedido |
| order_items | `idx_order_items_product_id` | B-tree | Análisis de ventas por producto |
| order_items | `idx_order_items_product_date` | B-tree | Reportes de ventas: producto × período |

## 6. Resumen de RLS policies

| Tabla | Rol | Operación | Condición |
|---|---|---|---|
| products | `anon` / `authenticated` | SELECT | `active = true` |
| products | `service_role` | ALL | sin restricción |
| orders | `authenticated` | SELECT | `user_id = auth.uid()` |
| orders | `authenticated` | INSERT | `user_id = auth.uid()` |
| orders | `anon` | INSERT | `user_id IS NULL AND guest_email NOT NULL` |
| orders | `service_role` | ALL | sin restricción |
| order_items | `authenticated` | SELECT | pedido pertenece al usuario |
| order_items | `authenticated` | INSERT | pedido PENDING del usuario |
| order_items | `anon` | INSERT | pedido PENDING sin usuario |
| order_items | `service_role` | ALL | sin restricción |

## 7. Notas de implementación Next.js

- **Server Actions** que crean pedidos deben usar el cliente Supabase con `service_role` para saltarse RLS y poder transicionar estados.
- **Webhook de Stripe**: buscar por `stripe_payment_intent_id`, actualizar `status = 'CONFIRMED'` con `service_role`. El trigger `orders_audit_status` registra `confirmed_at` automáticamente.
- **Stock**: decrementar `stock_quantity` en products dentro de la misma transacción que confirma el pedido para evitar overselling. Usar `FOR UPDATE` o `UPDATE ... WHERE stock_quantity >= quantity`.
- **Carrito**: implementar como cookie/localStorage en el cliente hasta que el usuario inicia checkout, momento en que se crea el registro en `orders` con status `PENDING`.
- **Tipos TypeScript** — generar con `supabase gen types typescript --local > src/types/supabase.ts`.
