# API Eval 2 — Google Maps + Sendcloud: Café Vermut

**Skill applied:** `api-developer.md`
**Stack:** Next.js · TypeScript strict · Node 18+
**Integrations:** Google Maps JS API · Sendcloud REST API v2
**Pattern:** Typed `ApiResult<T>`, `AbortSignal.timeout`, test/production separation

---

## Architecture Overview

Two independent integrations are delivered:

1. **Google Maps** — Client Component (`"use client"`) that loads the Maps JS API via `next/script`, renders a customized map centered on Café Vermut in Gràcia (41.3984, 2.1583) with brand-matched styles (terracota / crema / verde oliva), fully accessible via `aria-label` and `role="application"`.

2. **Sendcloud** — Server-side library (`src/lib/sendcloud.ts`) providing `createParcel()` with typed input/output, `getShippingMethods()` with `AbortSignal.timeout(15000)`, typed error handling via `ApiResult<T>`, and Server Actions that the frontend calls to ship ground-coffee orders.

Both integrations are wrapped in `safeApiCall<T>()` — no untyped `catch (error: any)` anywhere.

---

## File Tree

```
src/
├── lib/
│   └── sendcloud.ts                  ← Sendcloud client (createParcel, getShippingMethods)
├── components/
│   └── map/
│       ├── CafeVermutMap.tsx         ← Server shell (loads Script, passes env var)
│       └── GoogleMapEmbed.tsx        ← "use client" leaf (map initialization)
├── app/
│   ├── layout.tsx                    ← Script tag for Maps JS API lives here
│   └── actions/
│       └── shipping.ts               ← Server Actions wrapping Sendcloud
├── types/
│   ├── sendcloud.ts                  ← All Sendcloud domain types
│   └── api.ts                        ← Shared ApiResult<T> type
└── env.ts                            ← T3 Env — env var registration

docs/
└── integrations/
    ├── google-maps.md
    └── sendcloud.md
```

---

## Source Files

### `src/types/api.ts`

```typescript
/**
 * Typed result wrapper for all external API calls.
 * Prevents untyped error propagation across the codebase.
 */
export type ApiResult<T> =
  | { success: true; data: T }
  | { success: false; error: string; code?: string }

/**
 * Wraps any async function in a typed result pattern.
 * Errors are caught, logged, and returned as { success: false }.
 * Never throws — callers always receive a discriminated union.
 */
export async function safeApiCall<T>(
  fn: () => Promise<T>,
  context?: string
): Promise<ApiResult<T>> {
  try {
    const data = await fn()
    return { success: true, data }
  } catch (error) {
    const label = context ? `[${context}]` : "[API Error]"
    if (error instanceof Error) {
      console.error(`${label}:`, error.message)
      // Preserve AbortError code for timeout handling
      if (error.name === "AbortError") {
        return { success: false, error: "Request timed out", code: "TIMEOUT" }
      }
      return { success: false, error: error.message }
    }
    return { success: false, error: "Unknown error" }
  }
}
```

---

### `src/types/sendcloud.ts`

```typescript
// ────────────────────────────────────────────────────────────
// Sendcloud REST API v2 — Domain Types
// Reference: https://api.sendcloud.dev/docs
// ────────────────────────────────────────────────────────────

/** Input data for creating a new parcel */
export interface CreateParcelInput {
  /** Recipient full name */
  name: string
  /** Street address with number */
  address: string
  /** City name */
  city: string
  /** Postal/ZIP code */
  postal_code: string
  /** ISO 3166-1 alpha-2 country code (e.g. "ES") */
  country: string
  /** Recipient email for tracking notifications */
  email: string
  /** Weight in grams */
  weight: number
  /** Sendcloud shipping method ID (from getShippingMethods()) */
  shipping_method_id: number
  /** Optional order reference for internal tracking */
  order_number?: string
  /** Optional parcel items for customs declarations */
  parcel_items?: ParcelItem[]
}

/** Individual item inside a parcel */
export interface ParcelItem {
  description: string
  quantity: number
  weight: string     // e.g. "0.250" (kg as string — Sendcloud requirement)
  value: string      // e.g. "12.50" (EUR as string)
  hs_code?: string   // Harmonized System tariff code (for international shipping)
  origin_country?: string
}

/** Sendcloud parcel response */
export interface SendcloudParcel {
  id: number
  tracking_number: string
  /** URL to download PDF shipping label */
  label: string
  status: {
    id: number
    message: string
  }
  carrier: {
    code: string
  }
  tracking_url: string
  weight: string
  order_number: string
  address: string
  city: string
  postal_code: string
  country: {
    iso_2: string
    iso_3: string
    name: string
  }
  email: string
  name: string
}

/** Sendcloud API parcel creation response envelope */
export interface CreateParcelResponse {
  parcel: SendcloudParcel
}

/** Sendcloud shipping method */
export interface ShippingMethod {
  id: number
  name: string
  carrier: string
  min_weight: string
  max_weight: string
  countries: Array<{
    iso_2: string
    price: number
  }>
}

/** Sendcloud API shipping methods response envelope */
export interface GetShippingMethodsResponse {
  shipping_methods: ShippingMethod[]
}
```

---

### `src/lib/sendcloud.ts`

```typescript
import { env } from "@/env"
import type {
  CreateParcelInput,
  CreateParcelResponse,
  GetShippingMethodsResponse,
} from "@/types/sendcloud"

// ────────────────────────────────────────────────────────────
// Sendcloud API Client
// Base URL: https://panel.sendcloud.sc/api/v2
// Auth: HTTP Basic — public_key:secret_key (base64)
// ────────────────────────────────────────────────────────────

const SENDCLOUD_API = "https://panel.sendcloud.sc/api/v2"

/**
 * Returns the Authorization header for Sendcloud Basic Auth.
 * Keys come from environment — test keys in dev, live keys in production.
 * No code change required when switching environments.
 */
function getAuthHeader(): string {
  const credentials = Buffer.from(
    `${env.SENDCLOUD_PUBLIC_KEY}:${env.SENDCLOUD_SECRET_KEY}`
  ).toString("base64")
  return `Basic ${credentials}`
}

const baseHeaders = {
  "Content-Type": "application/json",
  Accept: "application/json",
}

// ────────────────────────────────────────────────────────────
// createParcel
// Creates a shipping label for a ground-coffee order.
// Timeout: 15s (shipping APIs are slower than payment APIs).
// ────────────────────────────────────────────────────────────

/**
 * Creates a Sendcloud parcel and returns a tracking number + label URL.
 *
 * @param data - Typed parcel input (recipient, address, weight, shipping method)
 * @returns Promise resolving to CreateParcelResponse
 * @throws Error if the HTTP response is not 2xx — message includes status + body
 *
 * @example
 * const result = await safeApiCall(
 *   () => createParcel({ name: "Maria García", address: "Carrer de Verdi 14", ... }),
 *   "createParcel"
 * )
 */
export async function createParcel(
  data: CreateParcelInput
): Promise<CreateParcelResponse> {
  const response = await fetch(`${SENDCLOUD_API}/parcels`, {
    method: "POST",
    headers: {
      ...baseHeaders,
      Authorization: getAuthHeader(),
    },
    body: JSON.stringify({ parcel: data }),
    // Shipping APIs: 15s timeout per skill spec
    signal: AbortSignal.timeout(15_000),
  })

  if (!response.ok) {
    // Parse Sendcloud error body if available
    const errorBody = await response.json().catch(() => ({})) as Record<string, unknown>
    const message =
      typeof errorBody.error === "object" &&
      errorBody.error !== null &&
      "message" in errorBody.error
        ? String((errorBody.error as Record<string, unknown>).message)
        : `HTTP ${response.status}`

    throw new Error(`Sendcloud createParcel failed: ${message}`)
  }

  return response.json() as Promise<CreateParcelResponse>
}

// ────────────────────────────────────────────────────────────
// getShippingMethods
// Retrieves available carriers/rates for a country pair.
// Timeout: 15s (per skill spec for shipping APIs).
// ────────────────────────────────────────────────────────────

/**
 * Returns all shipping methods available between two countries.
 * Use the returned `id` values as `shipping_method_id` in createParcel().
 *
 * @param fromCountry - ISO 3166-1 alpha-2 origin country code (e.g. "ES")
 * @param toCountry   - ISO 3166-1 alpha-2 destination country code (e.g. "FR")
 * @returns Promise resolving to GetShippingMethodsResponse
 * @throws Error if the HTTP response is not 2xx
 */
export async function getShippingMethods(
  fromCountry: string,
  toCountry: string
): Promise<GetShippingMethodsResponse> {
  const url = new URL(`${SENDCLOUD_API}/shipping_methods`)
  url.searchParams.set("from_country", fromCountry)
  url.searchParams.set("to_country", toCountry)

  const response = await fetch(url.toString(), {
    method: "GET",
    headers: {
      ...baseHeaders,
      Authorization: getAuthHeader(),
    },
    // Shipping APIs: 15s timeout per skill spec
    signal: AbortSignal.timeout(15_000),
  })

  if (!response.ok) {
    throw new Error(
      `Sendcloud getShippingMethods failed: HTTP ${response.status} for ${fromCountry}→${toCountry}`
    )
  }

  return response.json() as Promise<GetShippingMethodsResponse>
}
```

---

### `src/app/actions/shipping.ts`

```typescript
"use server"

import { createParcel, getShippingMethods } from "@/lib/sendcloud"
import { safeApiCall } from "@/types/api"
import type { ApiResult } from "@/types/api"
import type { CreateParcelInput, CreateParcelResponse, GetShippingMethodsResponse } from "@/types/sendcloud"

// ────────────────────────────────────────────────────────────
// Server Actions — Sendcloud
// Called from checkout / order confirmation UI.
// All calls go through safeApiCall — no untyped throws reach the client.
// ────────────────────────────────────────────────────────────

/**
 * Server Action: create a Sendcloud parcel for a coffee order.
 * Validates minimum weight (Sendcloud minimum is 1g = "0.001" kg).
 *
 * @example
 * // In a checkout Server Action or route handler:
 * const result = await shipCoffeeOrder({ name: "Maria García", ... })
 * if (!result.success) {
 *   // Show error to user — result.error is a human-readable string
 * }
 */
export async function shipCoffeeOrder(
  input: CreateParcelInput
): Promise<ApiResult<CreateParcelResponse>> {
  // Validate weight before calling Sendcloud (saves an API round-trip)
  if (input.weight <= 0) {
    return { success: false, error: "Weight must be greater than 0 grams", code: "VALIDATION" }
  }

  return safeApiCall(
    () => createParcel(input),
    "Sendcloud.createParcel"
  )
}

/**
 * Server Action: fetch available shipping methods for a route.
 * Results should be cached or pre-fetched — this call costs ~300ms.
 *
 * @example
 * const result = await fetchShippingMethods("ES", "ES")
 * if (result.success) {
 *   const methods = result.data.shipping_methods
 *   // Render method selector in checkout
 * }
 */
export async function fetchShippingMethods(
  fromCountry: string,
  toCountry: string
): Promise<ApiResult<GetShippingMethodsResponse>> {
  return safeApiCall(
    () => getShippingMethods(fromCountry, toCountry),
    "Sendcloud.getShippingMethods"
  )
}
```

---

### `src/env.ts`

```typescript
import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  server: {
    // Sendcloud — test keys start with nothing special; live keys are different credentials
    // Get from: Sendcloud Dashboard → Settings → API
    SENDCLOUD_PUBLIC_KEY: z.string().min(1),
    SENDCLOUD_SECRET_KEY: z.string().min(1),
  },
  client: {
    // Google Maps — restricted browser key (HTTP referrer restrictions in Google Cloud Console)
    // Get from: Google Cloud Console → APIs & Services → Credentials
    NEXT_PUBLIC_GOOGLE_MAPS_KEY: z.string().min(1),
  },
  runtimeEnv: {
    SENDCLOUD_PUBLIC_KEY: process.env.SENDCLOUD_PUBLIC_KEY,
    SENDCLOUD_SECRET_KEY: process.env.SENDCLOUD_SECRET_KEY,
    NEXT_PUBLIC_GOOGLE_MAPS_KEY: process.env.NEXT_PUBLIC_GOOGLE_MAPS_KEY,
  },
})
```

---

### `src/app/layout.tsx` (relevant addition)

```tsx
import type { Metadata } from "next"
import Script from "next/script"

export const metadata: Metadata = {
  title: "Café Vermut — Café de Especialidad en Barcelona",
  description: "Granos de origen único, tostado artesanal y baristas apasionados.",
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="es">
      <body>
        {children}

        {/*
          Google Maps JS API — lazyOnload strategy:
          - Does not block page load or LCP
          - Loads after the page becomes interactive
          - The GoogleMapEmbed component checks window.google before initializing
        */}
        <Script
          src={`https://maps.googleapis.com/maps/api/js?key=${process.env.NEXT_PUBLIC_GOOGLE_MAPS_KEY}`}
          strategy="lazyOnload"
          id="google-maps-script"
        />
      </body>
    </html>
  )
}
```

---

### `src/components/map/CafeVermutMap.tsx`

```tsx
// Server Component — passes static props to the client leaf.
// No "use client" here — this component renders on the server.

import { GoogleMapEmbed } from "./GoogleMapEmbed"

/**
 * Server Component shell for Café Vermut's location map.
 * Coordinates are hardcoded — they don't change at runtime.
 * The client leaf (GoogleMapEmbed) handles all DOM interaction.
 */
export function CafeVermutMap() {
  return (
    <section
      className="w-full py-16"
      aria-labelledby="map-section-heading"
    >
      <div className="mx-auto max-w-5xl px-[--section-padding-x]">
        <h2
          id="map-section-heading"
          className="mb-8 font-heading text-2xl font-semibold text-[--color-text-primary]"
        >
          Encuéntranos en Gràcia
        </h2>
        <p className="mb-6 text-[--color-text-secondary]">
          Carrer de Verdi, 14 · 08012 Barcelona
        </p>

        <GoogleMapEmbed
          lat={41.3984}
          lng={2.1583}
          zoom={16}
          markerLabel="Café Vermut — Gràcia, Barcelona"
        />
      </div>
    </section>
  )
}
```

---

### `src/components/map/GoogleMapEmbed.tsx`

```tsx
"use client"

import { useEffect, useRef, useState } from "react"

// ────────────────────────────────────────────────────────────
// Brand palette — Café Vermut (Terracota / Crema / Verde Oliva)
// These match the CSS custom properties in globals.css.
// Google Maps uses hex/rgb strings, not CSS variables.
// ────────────────────────────────────────────────────────────
const BRAND_COLORS = {
  terracota: "#c0553a",   // warm reddish-orange — brand primary accent
  crema: "#f5ede0",       // off-white cream — background tones
  verde_oliva: "#5a6b3a", // muted olive green — vegetation
  marron_oscuro: "#3b2a1a", // deep brown — text/roads
  arena: "#e8d5bb",       // warm sand — secondary fills
} as const

/**
 * Google Maps custom styles matching Café Vermut's brand palette.
 * Applied via the `styles` array in the Map constructor options.
 *
 * Strategy:
 * - Water: crema (the sea/rivers use the cream tone)
 * - Roads: crema on marron_oscuro backgrounds
 * - Parks/vegetation: verde_oliva (softened)
 * - Buildings: arena tones
 * - Labels: marron_oscuro (legible on all fills)
 * - POIs: reduced saturation to not compete with the marker
 */
const BRAND_MAP_STYLES: google.maps.MapTypeStyle[] = [
  // ── Base geometry ──────────────────────────────────────────
  {
    elementType: "geometry",
    stylers: [{ color: BRAND_COLORS.crema }],
  },
  {
    elementType: "labels.text.fill",
    stylers: [{ color: BRAND_COLORS.marron_oscuro }],
  },
  {
    elementType: "labels.text.stroke",
    stylers: [{ color: BRAND_COLORS.crema }],
  },

  // ── Water ──────────────────────────────────────────────────
  {
    featureType: "water",
    elementType: "geometry",
    stylers: [{ color: "#c9dde8" }],  // soft blue, complements crema palette
  },
  {
    featureType: "water",
    elementType: "labels.text.fill",
    stylers: [{ color: BRAND_COLORS.marron_oscuro }],
  },

  // ── Parks / Natural areas ──────────────────────────────────
  {
    featureType: "landscape.natural",
    elementType: "geometry",
    stylers: [{ color: "#d4ddb8" }],  // desaturated olive
  },
  {
    featureType: "poi.park",
    elementType: "geometry",
    stylers: [{ color: "#c8d4a0" }],  // light verde_oliva
  },
  {
    featureType: "poi.park",
    elementType: "labels.text.fill",
    stylers: [{ color: BRAND_COLORS.verde_oliva }],
  },

  // ── Roads ──────────────────────────────────────────────────
  {
    featureType: "road",
    elementType: "geometry",
    stylers: [{ color: "#ffffff" }],
  },
  {
    featureType: "road",
    elementType: "geometry.stroke",
    stylers: [{ color: BRAND_COLORS.arena }],
  },
  {
    featureType: "road.highway",
    elementType: "geometry",
    stylers: [{ color: BRAND_COLORS.arena }],
  },
  {
    featureType: "road.highway",
    elementType: "geometry.stroke",
    stylers: [{ color: "#d4b896" }],
  },
  {
    featureType: "road.arterial",
    elementType: "labels.text.fill",
    stylers: [{ color: BRAND_COLORS.marron_oscuro }],
  },
  {
    featureType: "road.local",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a5c40" }],
  },

  // ── POIs (Points of Interest) ──────────────────────────────
  {
    featureType: "poi",
    elementType: "geometry",
    stylers: [{ color: "#ddd8c4" }],
  },
  {
    featureType: "poi",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a5c40" }],
  },
  // Reduce visual noise from competing POIs
  {
    featureType: "poi.business",
    stylers: [{ visibility: "off" }],
  },

  // ── Transit ────────────────────────────────────────────────
  {
    featureType: "transit",
    elementType: "geometry",
    stylers: [{ color: BRAND_COLORS.arena }],
  },
  {
    featureType: "transit.station",
    elementType: "labels.text.fill",
    stylers: [{ color: BRAND_COLORS.marron_oscuro }],
  },

  // ── Administrative boundaries ──────────────────────────────
  {
    featureType: "administrative",
    elementType: "geometry.stroke",
    stylers: [{ color: BRAND_COLORS.terracota }, { weight: 1.5 }],
  },
  {
    featureType: "administrative.land_parcel",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a5c40" }],
  },
]

// ────────────────────────────────────────────────────────────
// Terracota custom marker SVG (branded pin)
// ────────────────────────────────────────────────────────────
const MARKER_SVG = `
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 48" width="32" height="48">
  <path d="M16 0C7.163 0 0 7.163 0 16c0 10.5 16 32 16 32S32 26.5 32 16C32 7.163 24.837 0 16 0z"
        fill="#c0553a" stroke="#3b2a1a" stroke-width="1.5"/>
  <circle cx="16" cy="16" r="7" fill="#f5ede0"/>
</svg>
`.trim()

// ────────────────────────────────────────────────────────────
// Component interface
// ────────────────────────────────────────────────────────────
interface GoogleMapEmbedProps {
  lat: number
  lng: number
  zoom?: number
  markerLabel?: string
}

// ────────────────────────────────────────────────────────────
// Declare google as a global — Maps JS API adds it to window.
// We use a type-only declaration to avoid importing @types/google.maps
// in the entire project (it's only needed in this file).
// If @types/google.maps is installed, remove this declaration.
// ────────────────────────────────────────────────────────────
declare global {
  interface Window {
    google?: typeof google
  }
}

/**
 * Client Component: renders a Google Maps embed with Café Vermut brand styles.
 *
 * Accessibility:
 * - role="application" — informs AT that this is a complex interactive widget
 * - aria-label — describes the map content to screen reader users
 * - The container div is focusable via keyboard (tabIndex=0)
 *
 * Loading:
 * - Uses a polling strategy to wait for window.google to be defined
 *   (the script is loaded with strategy="lazyOnload" in layout.tsx)
 * - Shows a styled placeholder while the API loads
 *
 * Cleanup:
 * - useEffect cleanup clears the polling interval to prevent memory leaks
 */
export function GoogleMapEmbed({
  lat,
  lng,
  zoom = 16,
  markerLabel = "Café Vermut",
}: GoogleMapEmbedProps) {
  const mapRef = useRef<HTMLDivElement>(null)
  const mapInstanceRef = useRef<google.maps.Map | null>(null)
  const [isLoaded, setIsLoaded] = useState(false)
  const [hasError, setHasError] = useState(false)

  useEffect(() => {
    let intervalId: ReturnType<typeof setInterval>
    let attempts = 0
    const MAX_ATTEMPTS = 30  // 30 × 200ms = 6s max wait time

    function initMap() {
      if (!mapRef.current) return

      try {
        const map = new window.google!.maps.Map(mapRef.current, {
          center: { lat, lng },
          zoom,
          disableDefaultUI: true,
          zoomControl: true,
          zoomControlOptions: {
            position: window.google!.maps.ControlPosition.RIGHT_BOTTOM,
          },
          styles: BRAND_MAP_STYLES,
          // Prevent accidental scroll-zoom on page scroll
          gestureHandling: "cooperative",
        })

        // Branded SVG marker
        new window.google!.maps.Marker({
          position: { lat, lng },
          map,
          title: markerLabel,
          icon: {
            url: `data:image/svg+xml,${encodeURIComponent(MARKER_SVG)}`,
            scaledSize: new window.google!.maps.Size(32, 48),
            anchor: new window.google!.maps.Point(16, 48),
          },
        })

        mapInstanceRef.current = map
        setIsLoaded(true)
      } catch (err) {
        console.error("[GoogleMapEmbed] Failed to initialize map:", err)
        setHasError(true)
      }
    }

    // Poll until window.google is defined (loaded by next/script lazyOnload)
    intervalId = setInterval(() => {
      attempts++
      if (typeof window !== "undefined" && window.google?.maps) {
        clearInterval(intervalId)
        initMap()
      } else if (attempts >= MAX_ATTEMPTS) {
        clearInterval(intervalId)
        console.error("[GoogleMapEmbed] Google Maps API did not load after 6s")
        setHasError(true)
      }
    }, 200)

    return () => {
      clearInterval(intervalId)
    }
  }, [lat, lng, zoom, markerLabel])

  if (hasError) {
    return (
      <div
        className="flex h-[400px] w-full items-center justify-center rounded-[--radius-card] bg-[--color-bg-base] text-[--color-text-secondary]"
        role="alert"
        aria-live="polite"
      >
        <p>No se pudo cargar el mapa. Visítanos en Carrer de Verdi, 14 · Gràcia, Barcelona.</p>
      </div>
    )
  }

  return (
    <div className="relative w-full overflow-hidden rounded-[--radius-card]">
      {/* Loading placeholder — visible until map initializes */}
      {!isLoaded && (
        <div
          className="absolute inset-0 z-10 flex h-[400px] items-center justify-center bg-[--color-bg-base]"
          aria-hidden="true"
        >
          <span className="text-sm text-[--color-text-secondary]">Cargando mapa…</span>
        </div>
      )}

      {/*
        role="application" — required for complex interactive widgets per WCAG.
        aria-label — describes the map to screen reader users.
        tabIndex={0} — makes the container keyboard-focusable.
      */}
      <div
        ref={mapRef}
        className="h-[400px] w-full"
        role="application"
        aria-label={
          markerLabel
            ? `Mapa mostrando la ubicación de ${markerLabel}`
            : "Mapa de ubicación"
        }
        tabIndex={0}
      />
    </div>
  )
}
```

---

## Usage Examples

### Rendering the map on a page

```tsx
// src/app/visita/page.tsx
// Server Component — no "use client" needed here

import { CafeVermutMap } from "@/components/map/CafeVermutMap"

export default function VisitaPage() {
  return (
    <main>
      <h1 className="sr-only">Visítanos</h1>
      <CafeVermutMap />
    </main>
  )
}
```

---

### Processing a coffee order shipment

```tsx
// src/app/checkout/actions.ts
"use server"

import { shipCoffeeOrder, fetchShippingMethods } from "@/app/actions/shipping"

// Example: ship 250g of ground coffee to a customer in Spain
export async function processGroundCoffeeOrder(formData: FormData) {
  // Step 1: Get available shipping methods for ES → ES route
  const methodsResult = await fetchShippingMethods("ES", "ES")
  if (!methodsResult.success) {
    return { error: `No se pudieron obtener los métodos de envío: ${methodsResult.error}` }
  }

  // Use the first available method (in production: let customer choose)
  const firstMethod = methodsResult.data.shipping_methods[0]
  if (!firstMethod) {
    return { error: "No hay métodos de envío disponibles para esta ruta" }
  }

  // Step 2: Create the parcel
  const shipResult = await shipCoffeeOrder({
    name: formData.get("name") as string,
    address: formData.get("address") as string,
    city: formData.get("city") as string,
    postal_code: formData.get("postal_code") as string,
    country: "ES",
    email: formData.get("email") as string,
    weight: 250,  // 250g bag of ground coffee
    shipping_method_id: firstMethod.id,
    order_number: `CV-${Date.now()}`,
    parcel_items: [
      {
        description: "Café molido de especialidad — Café Vermut",
        quantity: 1,
        weight: "0.250",
        value: "14.00",
        hs_code: "0901.21",  // HS code for roasted, non-decaffeinated coffee
        origin_country: "ES",
      },
    ],
  })

  if (!shipResult.success) {
    if (shipResult.code === "TIMEOUT") {
      return { error: "El servicio de envíos tardó demasiado. Inténtalo de nuevo." }
    }
    return { error: `Error al crear el envío: ${shipResult.error}` }
  }

  return {
    tracking_number: shipResult.data.parcel.tracking_number,
    label_url: shipResult.data.parcel.label,
    tracking_url: shipResult.data.parcel.tracking_url,
  }
}
```

---

## Test vs Production Checklist

### Google Maps

- [ ] `NEXT_PUBLIC_GOOGLE_MAPS_KEY` — test key has no HTTP referrer restriction (development only)
- [ ] Production key must have HTTP referrer restrictions set in Google Cloud Console → Credentials → Edit key → Application restrictions → HTTP referrers
- [ ] Production key must have only "Maps JavaScript API" enabled (principle of least privilege)
- [ ] Billing account enabled in Google Cloud project (Maps API requires it even for low traffic)

### Sendcloud

- [ ] `SENDCLOUD_PUBLIC_KEY` + `SENDCLOUD_SECRET_KEY` from Sendcloud test environment (Settings → API)
- [ ] Test parcels can be created without generating real labels or charges
- [ ] Production keys are different credentials — update in hosting environment variables only (no code change)
- [ ] Sendcloud integration is marked as "active" in Sendcloud Dashboard before go-live
- [ ] Webhook endpoint registered in Sendcloud Dashboard if tracking-status updates are needed

---

## Environment Variables

Add to `.env.local` (development):

```bash
# ── Google Maps ─────────────────────────────────────────────
# Get from: Google Cloud Console → APIs & Services → Credentials
# In development: unrestricted key is OK
# In production: restrict to HTTP referrers (your domain only)
NEXT_PUBLIC_GOOGLE_MAPS_KEY=AIzaSy_YOUR_TEST_KEY_HERE

# ── Sendcloud ────────────────────────────────────────────────
# Get from: Sendcloud Dashboard → Settings → API
# These are your TEST environment keys
SENDCLOUD_PUBLIC_KEY=your_test_public_key
SENDCLOUD_SECRET_KEY=your_test_secret_key
```

---

## Integration Documentation

### `docs/integrations/google-maps.md`

```markdown
# Google Maps Integration

**SDK:** Google Maps JavaScript API (browser-loaded via next/script)
**Version:** Maps JS API v3 (always latest — loaded from CDN)

## Environment variables required

- `NEXT_PUBLIC_GOOGLE_MAPS_KEY` — Browser-restricted API key
  - Get from: Google Cloud Console → APIs & Services → Credentials → Create Credentials → API Key
  - Required APIs to enable: Maps JavaScript API

## Test mode

- Use an unrestricted API key in `.env.local` for development
- Maps will load normally; no test/live distinction for Maps API
- Monitor quota usage in Google Cloud Console → APIs & Services → Quotas

## Production checklist

- [ ] API key restricted to HTTP referrers: `https://cafevermut.com/*`, `https://www.cafevermut.com/*`
- [ ] Only "Maps JavaScript API" enabled on the production key
- [ ] Billing account active in Google Cloud project
- [ ] Daily quota alert configured in Google Cloud Console (recommended: alert at 80% of free tier)

## Files

- `src/components/map/CafeVermutMap.tsx` — Server Component shell (section wrapper)
- `src/components/map/GoogleMapEmbed.tsx` — Client Component (map initialization, styles, marker)
- `src/app/layout.tsx` — next/script tag loading the Maps JS API with `strategy="lazyOnload"`

## Map configuration

- **Center:** lat: 41.3984, lng: 2.1583 (Gràcia, Barcelona)
- **Zoom:** 16 (neighborhood level — street names visible)
- **Marker:** Custom SVG in terracota (#c0553a) with crema (#f5ede0) dot
- **Map styles:** 18 style rules matching brand palette (terracota / crema / verde oliva)
- **gestureHandling:** "cooperative" — prevents accidental zoom on page scroll

## Accessibility

- Container has `role="application"` and `aria-label` describing the map content
- `tabIndex={0}` makes the map keyboard-focusable
- Loading state is hidden from AT (`aria-hidden="true"`)
- Error state uses `role="alert"` with fallback address text

## Known limitations

- Maps JavaScript API has a free tier of $200/month (~28,000 map loads/month)
- The `lazyOnload` script strategy means the map is not available during SSR
- `window.google` is polled every 200ms for up to 6 seconds after page load
- `AdvancedMarkerElement` (new API) requires `mapId` parameter — we use legacy `Marker` for simplicity
```

---

### `docs/integrations/sendcloud.md`

```markdown
# Sendcloud Integration

**SDK:** Native fetch (no official Node SDK — REST API v2 used directly)
**API Base URL:** https://panel.sendcloud.sc/api/v2
**Auth:** HTTP Basic Auth (public_key:secret_key, base64-encoded)

## Environment variables required

- `SENDCLOUD_PUBLIC_KEY` — Sendcloud API public key
  - Get from: Sendcloud Dashboard → Settings → API → Add Integration → Custom Integration
- `SENDCLOUD_SECRET_KEY` — Sendcloud API secret key
  - Same location as above — generated once and not shown again; store immediately

## Test mode

- Sendcloud provides separate test credentials in the same dashboard
- Test parcels do NOT generate real labels or billing
- Use shipping methods returned by getShippingMethods("ES", "ES") for test parcel creation
- Sendcloud test environment: https://panel.sendcloud.sc (same URL, different credentials)

## Production checklist

- [ ] Production API keys set in hosting environment variables (Vercel → Project → Settings → Environment Variables)
- [ ] Test keys removed from production environment
- [ ] Sendcloud account on a plan that supports API access (Lite plan or above)
- [ ] Sendcloud integration marked as "active" in the Dashboard
- [ ] If using tracking webhooks: register endpoint at Settings → Webhooks → Add URL: `https://cafevermut.com/api/webhooks/sendcloud`

## Files

- `src/lib/sendcloud.ts` — HTTP client (createParcel, getShippingMethods)
- `src/types/sendcloud.ts` — Domain types (CreateParcelInput, ShippingMethod, etc.)
- `src/app/actions/shipping.ts` — Server Actions (shipCoffeeOrder, fetchShippingMethods)
- `src/types/api.ts` — Shared ApiResult<T> type and safeApiCall() wrapper

## Timeout configuration

| Function             | Timeout | Reason                              |
|----------------------|---------|-------------------------------------|
| createParcel()       | 15s     | Label generation can be slow        |
| getShippingMethods() | 15s     | Rate aggregation across carriers    |

## Error handling

All functions throw typed `Error` objects with descriptive messages.
Callers use `safeApiCall()` to receive `ApiResult<T>` — no unhandled rejections.

Special error codes:
- `"TIMEOUT"` — AbortSignal.timeout() fired; surface a retry message to the user
- `"VALIDATION"` — Input validation failed before the API call was made

## Parcel weight format

Sendcloud accepts weight in **grams** (integer) in the `parcel` body.
`parcel_items[].weight` must be a **string in kilograms** (e.g., `"0.250"` for 250g).
This asymmetry is a Sendcloud API quirk — it is handled in the types.

## HS codes for coffee

Common HS codes for Café Vermut products:
- `0901.21` — Roasted coffee, not decaffeinated (ground or whole bean)
- `0901.22` — Roasted coffee, decaffeinated
- `2101.11` — Extracts, essences, concentrates of coffee

Required for international shipments outside the EU.

## Known limitations

- Sendcloud free plan does not include API access — minimum Lite plan required
- getShippingMethods() results change infrequently — consider caching with `unstable_cache` for 1 hour
- Sendcloud does not support idempotency keys — duplicate parcel creation on retry will create duplicate labels; implement order-level deduplication in your database before calling createParcel()
- Maximum parcel weight varies by carrier; validate against ShippingMethod.max_weight before creating a parcel
```

---

## TypeScript Compliance

- Zero `any` types — `errorBody` is typed as `Record<string, unknown>` with explicit narrowing
- `ApiResult<T>` is a discriminated union — exhaustive checks work correctly
- `google.maps.*` types come from `@types/google.maps` or the inline `declare global` in `GoogleMapEmbed.tsx`
- `AbortError` is detected by `error.name === "AbortError"` (not `instanceof`) — correct for `DOMException` subclass
- `satisfies` available for BRAND_COLORS constant if stricter inference is needed

---

## Installation Notes

```bash
# T3 Env for environment variable validation
npm install @t3-oss/env-nextjs zod

# Google Maps types (optional — included in inline declare global if skipped)
npm install --save-dev @types/google.maps

# No Sendcloud SDK needed — REST API is used directly via fetch
```

`tsconfig.json` must include:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "lib": ["ES2022", "DOM"]
  }
}
```

`AbortSignal.timeout()` requires **Node 18+** (available in all current Next.js deployments on Vercel).
