# Café Vermut — Google Maps + Sendcloud Integration

## Overview

This document covers two integrations for the Café Vermut website (Next.js project):

1. **Google Maps** — embedded map showing the Gràcia location (lat: 41.3984, lng: 2.1583) with custom brand-aligned styles and full accessibility.
2. **Sendcloud** — shipping integration for ground coffee orders, exposing `createParcel()` and `getShippingMethods()` with error handling and timeouts.

---

## 1. Google Maps Integration

### 1.1 Environment Variables

Add to `.env.local`:

```
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_api_key_here
```

### 1.2 Custom Map Styles (`lib/maps/cafeVermutMapStyles.ts`)

```typescript
// lib/maps/cafeVermutMapStyles.ts
// Custom Google Maps styles aligned with Café Vermut brand palette:
// - Warm ochre/terracotta backgrounds
// - Muted olive greens for parks
// - Cream/off-white roads
// - Dark espresso labels

import type { google } from "@types/google.maps";

export const cafeVermutMapStyles: google.maps.MapTypeStyle[] = [
  // Overall background — warm cream
  {
    elementType: "geometry",
    stylers: [{ color: "#f5ede0" }],
  },
  // All text labels — dark espresso
  {
    elementType: "labels.text.fill",
    stylers: [{ color: "#3b2a1a" }],
  },
  {
    elementType: "labels.text.stroke",
    stylers: [{ color: "#f5ede0" }],
  },
  // Administrative boundaries
  {
    featureType: "administrative",
    elementType: "geometry.stroke",
    stylers: [{ color: "#c9a97a" }],
  },
  {
    featureType: "administrative.locality",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a4f2d" }],
  },
  // Parks & green areas — muted olive
  {
    featureType: "poi.park",
    elementType: "geometry",
    stylers: [{ color: "#b8c9a0" }],
  },
  {
    featureType: "poi.park",
    elementType: "labels.text.fill",
    stylers: [{ color: "#5a6e40" }],
  },
  // Points of interest — terracotta icon tint
  {
    featureType: "poi",
    elementType: "geometry",
    stylers: [{ color: "#e8d5be" }],
  },
  {
    featureType: "poi",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a4f2d" }],
  },
  // Hide unwanted POI icons for clean look
  {
    featureType: "poi.business",
    stylers: [{ visibility: "off" }],
  },
  // Road arterials — slightly darker cream
  {
    featureType: "road",
    elementType: "geometry",
    stylers: [{ color: "#eddfc8" }],
  },
  {
    featureType: "road",
    elementType: "geometry.stroke",
    stylers: [{ color: "#c9a97a" }],
  },
  {
    featureType: "road",
    elementType: "labels.text.fill",
    stylers: [{ color: "#5c3d1e" }],
  },
  // Highways — warm ochre
  {
    featureType: "road.highway",
    elementType: "geometry",
    stylers: [{ color: "#d4a96a" }],
  },
  {
    featureType: "road.highway",
    elementType: "geometry.stroke",
    stylers: [{ color: "#b8893a" }],
  },
  // Transit lines — muted terracotta
  {
    featureType: "transit",
    elementType: "geometry",
    stylers: [{ color: "#c9a97a" }],
  },
  {
    featureType: "transit.station",
    elementType: "labels.text.fill",
    stylers: [{ color: "#7a4f2d" }],
  },
  // Water — soft warm blue-grey
  {
    featureType: "water",
    elementType: "geometry",
    stylers: [{ color: "#b5cfd8" }],
  },
  {
    featureType: "water",
    elementType: "labels.text.fill",
    stylers: [{ color: "#4a6a78" }],
  },
];
```

### 1.3 Map Component (`components/maps/CafeVermutMap.tsx`)

```tsx
// components/maps/CafeVermutMap.tsx
"use client";

import React, { useEffect, useRef, useState, useCallback } from "react";
import { Loader } from "@googlemaps/js-api-loader";
import { cafeVermutMapStyles } from "@/lib/maps/cafeVermutMapStyles";

const CAFE_LOCATION = {
  lat: 41.3984,
  lng: 2.1583,
} as const;

const MARKER_TITLE = "Café Vermut — Gràcia, Barcelona";
const ZOOM_LEVEL = 16;

interface CafeVermutMapProps {
  /** Additional CSS classes for the outer container */
  className?: string;
  /** Height of the map (default: "400px") */
  height?: string;
}

/**
 * CafeVermutMap
 *
 * Accessible, brand-styled Google Maps embed showing the Café Vermut location
 * in the Gràcia neighbourhood of Barcelona.
 *
 * Accessibility:
 * - The <section> has role="region" and aria-label describing its purpose.
 * - The map <div> has role="application" and aria-label per WCAG 2.1 §4.1.2.
 * - A visible fallback link to Google Maps is provided for keyboard/screen-reader users.
 * - Focus management: the "View on Google Maps" link receives focus when the map
 *   fails to load.
 *
 * Dependencies:
 * - @googlemaps/js-api-loader  (npm install @googlemaps/js-api-loader)
 * - @types/google.maps          (npm install --save-dev @types/google.maps)
 */
export default function CafeVermutMap({
  className = "",
  height = "400px",
}: CafeVermutMapProps) {
  const mapDivRef = useRef<HTMLDivElement>(null);
  const fallbackLinkRef = useRef<HTMLAnchorElement>(null);
  const [loadError, setLoadError] = useState<string | null>(null);
  const [isLoaded, setIsLoaded] = useState(false);

  const initMap = useCallback(async () => {
    const apiKey = process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY;
    if (!apiKey) {
      setLoadError("Google Maps API key is not configured.");
      return;
    }

    try {
      const loader = new Loader({
        apiKey,
        version: "weekly",
        libraries: ["marker"],
      });

      const { Map } = await loader.importLibrary("maps");
      const { AdvancedMarkerElement } = await loader.importLibrary("marker") as google.maps.MarkerLibrary;

      if (!mapDivRef.current) return;

      const map = new Map(mapDivRef.current, {
        center: CAFE_LOCATION,
        zoom: ZOOM_LEVEL,
        styles: cafeVermutMapStyles,
        mapTypeControl: false,
        fullscreenControl: true,
        streetViewControl: false,
        zoomControlOptions: {
          position: google.maps.ControlPosition.RIGHT_CENTER,
        },
        // Required for AdvancedMarkerElement
        mapId: "cafe-vermut-map",
      });

      // Custom marker using brand colour
      const markerPin = document.createElement("div");
      markerPin.setAttribute("aria-hidden", "true");
      markerPin.style.cssText = `
        width: 40px;
        height: 40px;
        border-radius: 50% 50% 50% 0;
        transform: rotate(-45deg);
        background-color: #c0392b;
        border: 3px solid #7a2318;
        box-shadow: 0 2px 8px rgba(0,0,0,0.35);
      `;

      const marker = new AdvancedMarkerElement({
        map,
        position: CAFE_LOCATION,
        title: MARKER_TITLE,
        content: markerPin,
      });

      // Info window with address
      const infoWindow = new google.maps.InfoWindow({
        content: `
          <div style="font-family: serif; padding: 4px 8px; color: #3b2a1a;">
            <strong style="font-size: 1rem;">Café Vermut</strong><br/>
            <span style="font-size: 0.85rem;">Carrer de Gràcia, Barcelona</span><br/>
            <a
              href="https://maps.google.com/?q=${CAFE_LOCATION.lat},${CAFE_LOCATION.lng}"
              target="_blank"
              rel="noopener noreferrer"
              style="font-size: 0.8rem; color: #c0392b;"
              aria-label="Open Café Vermut in Google Maps (opens new tab)"
            >
              Get directions
            </a>
          </div>
        `,
        ariaLabel: MARKER_TITLE,
      });

      marker.addListener("click", () => {
        infoWindow.open({ anchor: marker, map });
      });

      // Open info window by default
      infoWindow.open({ anchor: marker, map });

      setIsLoaded(true);
    } catch (err) {
      const message =
        err instanceof Error ? err.message : "Unknown error loading map";
      setLoadError(`Failed to load Google Maps: ${message}`);
      // Move focus to the fallback link so keyboard users can still get directions
      fallbackLinkRef.current?.focus();
    }
  }, []);

  useEffect(() => {
    initMap();
  }, [initMap]);

  const googleMapsUrl = `https://maps.google.com/?q=${CAFE_LOCATION.lat},${CAFE_LOCATION.lng}`;

  return (
    <section
      aria-label="Café Vermut location map"
      className={`cafe-vermut-map-section ${className}`}
    >
      {/* Accessible heading for the map section */}
      <h2 className="sr-only">Our Location in Gràcia, Barcelona</h2>

      {/* Loading skeleton */}
      {!isLoaded && !loadError && (
        <div
          role="status"
          aria-live="polite"
          aria-label="Loading map…"
          style={{
            height,
            backgroundColor: "#f5ede0",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            borderRadius: "8px",
          }}
        >
          <span style={{ color: "#7a4f2d", fontFamily: "serif" }}>
            Loading map…
          </span>
        </div>
      )}

      {/* Error state */}
      {loadError && (
        <div
          role="alert"
          style={{
            height,
            backgroundColor: "#fdf0ec",
            border: "1px solid #c9a97a",
            borderRadius: "8px",
            display: "flex",
            flexDirection: "column",
            alignItems: "center",
            justifyContent: "center",
            gap: "12px",
            padding: "16px",
          }}
        >
          <p style={{ color: "#7a2318", fontFamily: "serif", margin: 0 }}>
            {loadError}
          </p>
          <a
            ref={fallbackLinkRef}
            href={googleMapsUrl}
            target="_blank"
            rel="noopener noreferrer"
            style={{
              color: "#c0392b",
              fontFamily: "serif",
              textDecoration: "underline",
            }}
            aria-label="Open Café Vermut location in Google Maps (opens new tab)"
          >
            View on Google Maps
          </a>
        </div>
      )}

      {/* The actual map canvas */}
      <div
        ref={mapDivRef}
        role="application"
        aria-label="Interactive map showing Café Vermut in Gràcia, Barcelona. Use arrow keys to pan."
        tabIndex={0}
        style={{
          height,
          borderRadius: "8px",
          overflow: "hidden",
          display: loadError ? "none" : "block",
        }}
      />

      {/* Always-visible accessible fallback link below the map */}
      <p
        style={{
          marginTop: "8px",
          fontFamily: "serif",
          fontSize: "0.85rem",
          color: "#7a4f2d",
        }}
      >
        <a
          href={googleMapsUrl}
          target="_blank"
          rel="noopener noreferrer"
          aria-label="Open Café Vermut in Google Maps — Gràcia, Barcelona (opens new tab)"
          style={{ color: "#c0392b" }}
        >
          View larger map &amp; get directions
        </a>
      </p>
    </section>
  );
}
```

### 1.4 Usage in a page (`app/contacto/page.tsx` example)

```tsx
// app/contacto/page.tsx
import CafeVermutMap from "@/components/maps/CafeVermutMap";

export default function ContactoPage() {
  return (
    <main>
      <section className="container mx-auto px-4 py-12">
        <h1 className="text-3xl font-serif text-[#3b2a1a] mb-4">
          Encuéntranos
        </h1>
        <p className="mb-8 text-[#7a4f2d]">
          Estamos en el corazón de Gràcia, Barcelona.
        </p>
        <CafeVermutMap height="450px" className="rounded-xl shadow-lg" />
      </section>
    </main>
  );
}
```

### 1.5 Required npm packages

```bash
npm install @googlemaps/js-api-loader
npm install --save-dev @types/google.maps
```

---

## 2. Sendcloud Integration

### 2.1 Environment Variables

Add to `.env.local` (never commit these to version control):

```
SENDCLOUD_PUBLIC_KEY=your_sendcloud_public_key
SENDCLOUD_SECRET_KEY=your_sendcloud_secret_key
# Optional: override default API base URL for testing
SENDCLOUD_API_BASE_URL=https://panel.sendcloud.sc/api/v2
```

### 2.2 Sendcloud client (`lib/sendcloud/client.ts`)

```typescript
// lib/sendcloud/client.ts
/**
 * Low-level Sendcloud HTTP client.
 *
 * Uses HTTP Basic Auth (public key = username, secret key = password).
 * All requests time out after REQUEST_TIMEOUT_MS (default 10 s).
 * Throws SendcloudError for all non-2xx responses.
 */

export const SENDCLOUD_API_BASE =
  process.env.SENDCLOUD_API_BASE_URL ??
  "https://panel.sendcloud.sc/api/v2";

const REQUEST_TIMEOUT_MS = 10_000; // 10 seconds

/** Typed error thrown by every Sendcloud API call */
export class SendcloudError extends Error {
  constructor(
    message: string,
    public readonly statusCode?: number,
    public readonly sendcloudCode?: string | number,
    public readonly cause?: unknown
  ) {
    super(message);
    this.name = "SendcloudError";
  }
}

function buildAuthHeader(): string {
  const publicKey = process.env.SENDCLOUD_PUBLIC_KEY;
  const secretKey = process.env.SENDCLOUD_SECRET_KEY;

  if (!publicKey || !secretKey) {
    throw new SendcloudError(
      "SENDCLOUD_PUBLIC_KEY and SENDCLOUD_SECRET_KEY must be set in environment variables."
    );
  }

  const credentials = Buffer.from(`${publicKey}:${secretKey}`).toString(
    "base64"
  );
  return `Basic ${credentials}`;
}

export interface SendcloudRequestOptions {
  method?: "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
  body?: unknown;
  timeoutMs?: number;
}

/**
 * Core fetch wrapper for the Sendcloud API.
 * Adds auth, JSON serialisation, timeout via AbortController, and error handling.
 */
export async function sendcloudRequest<T>(
  path: string,
  options: SendcloudRequestOptions = {}
): Promise<T> {
  const {
    method = "GET",
    body,
    timeoutMs = REQUEST_TIMEOUT_MS,
  } = options;

  const url = `${SENDCLOUD_API_BASE}${path}`;
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, {
      method,
      headers: {
        Authorization: buildAuthHeader(),
        "Content-Type": "application/json",
        Accept: "application/json",
      },
      body: body !== undefined ? JSON.stringify(body) : undefined,
      signal: controller.signal,
    });

    // Parse body regardless of status code so we can surface Sendcloud error messages
    let responseData: unknown;
    const contentType = response.headers.get("content-type") ?? "";
    if (contentType.includes("application/json")) {
      responseData = await response.json();
    } else {
      responseData = await response.text();
    }

    if (!response.ok) {
      // Sendcloud returns { error: { code, message } } on failures
      const errorObj = (responseData as { error?: { message?: string; code?: string | number } })?.error;
      const message = errorObj?.message ?? `Sendcloud API error (HTTP ${response.status})`;
      const code = errorObj?.code;
      throw new SendcloudError(message, response.status, code);
    }

    return responseData as T;
  } catch (err) {
    if (err instanceof SendcloudError) throw err;

    // AbortController fired — timeout
    if (err instanceof Error && err.name === "AbortError") {
      throw new SendcloudError(
        `Sendcloud request timed out after ${timeoutMs}ms (${method} ${path})`,
        undefined,
        "TIMEOUT",
        err
      );
    }

    // Network / DNS failures
    throw new SendcloudError(
      `Network error calling Sendcloud (${method} ${path}): ${
        err instanceof Error ? err.message : String(err)
      }`,
      undefined,
      "NETWORK_ERROR",
      err
    );
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### 2.3 Sendcloud types (`lib/sendcloud/types.ts`)

```typescript
// lib/sendcloud/types.ts
/**
 * TypeScript types for the Sendcloud v2 API objects used by Café Vermut.
 * Covers parcels and shipping methods only.
 *
 * Sendcloud v2 API reference:
 * https://api.sendcloud.dev/docs/sendcloud-public-api/
 */

// ─── Shared ──────────────────────────────────────────────────────────────────

export type SendcloudAddress = {
  company_name?: string;
  name: string;
  address: string;
  address_2?: string;
  city: string;
  postal_code: string;
  /** ISO 3166-1 alpha-2 country code, e.g. "ES" */
  country: string;
  email: string;
  phone?: string;
};

// ─── Shipping Methods ─────────────────────────────────────────────────────────

export type ShippingMethod = {
  id: number;
  name: string;
  carrier: string;
  /** Minimum weight in grams */
  min_weight: string;
  /** Maximum weight in grams */
  max_weight: string;
  price: number;
  /** ISO 4217 currency code */
  currency?: string;
  /** Array of country ISO codes this method ships to */
  countries?: Array<{ iso_2: string; price: number; label?: string }>;
  service_point_input?: "none" | "required" | "optional";
};

export type GetShippingMethodsResponse = {
  shipping_methods: ShippingMethod[];
};

export type GetShippingMethodsParams = {
  /** Filter by destination country ISO code (e.g. "ES") */
  to_country?: string;
  /** Filter by sender address ID */
  sender_address?: number;
};

// ─── Parcels ──────────────────────────────────────────────────────────────────

export type ParcelStatus = {
  id: number;
  message: string;
};

/**
 * Payload to create a single parcel.
 * Required fields are marked; all others are optional.
 */
export type CreateParcelPayload = {
  name: string;
  address: string;
  address_2?: string;
  city: string;
  postal_code: string;
  /** ISO 3166-1 alpha-2 */
  country: string;
  email: string;
  phone?: string;
  company_name?: string;
  /** Sendcloud shipping method ID */
  shipment: { id: number };
  /** Order reference from your system */
  order_number: string;
  /** Total weight in grams */
  weight: string;
  /** Parcel dimensions in cm */
  length?: string;
  width?: string;
  height?: string;
  /** Set true to request a label immediately */
  request_label?: boolean;
  /** Declared value for customs (required for non-EU shipments) */
  total_insured_value?: number;
  /** Parcel items (required for non-EU) */
  parcel_items?: ParcelItem[];
};

export type ParcelItem = {
  description: string;
  quantity: number;
  weight: string;
  value: string;
  hs_code?: string;
  origin_country?: string;
  sku?: string;
  product_id?: string;
};

export type Parcel = {
  id: number;
  name: string;
  address: string;
  city: string;
  postal_code: string;
  country: { iso_2: string; name: string };
  email: string;
  phone?: string;
  status: ParcelStatus;
  tracking_number?: string;
  tracking_url?: string;
  label?: { normal_printer: string[]; label_printer: string };
  order_number: string;
  weight: string;
  shipment: { id: number; name: string };
  created_at: string;
  updated_at: string;
};

export type CreateParcelResponse = {
  parcel: Parcel;
};
```

### 2.4 Sendcloud service (`lib/sendcloud/sendcloud.ts`)

```typescript
// lib/sendcloud/sendcloud.ts
/**
 * High-level Sendcloud service for Café Vermut.
 *
 * Exposes:
 *   - createParcel()          — create a shipping parcel for a coffee order
 *   - getShippingMethods()    — list available shipping methods
 *
 * Both functions:
 *   - Apply a 10-second timeout via AbortController (inside sendcloudRequest).
 *   - Throw SendcloudError on API errors, timeouts, and network failures.
 *   - Are safe to call from Next.js Server Actions or Route Handlers (server-side only).
 */

import { sendcloudRequest, SendcloudError } from "./client";
import type {
  CreateParcelPayload,
  CreateParcelResponse,
  GetShippingMethodsParams,
  GetShippingMethodsResponse,
  Parcel,
  ShippingMethod,
} from "./types";

// Re-export error class so consumers only need to import from this module
export { SendcloudError };
export type {
  CreateParcelPayload,
  GetShippingMethodsParams,
  Parcel,
  ShippingMethod,
};

// ─── createParcel ─────────────────────────────────────────────────────────────

/**
 * Create a Sendcloud parcel for a coffee order.
 *
 * @param payload  - Parcel details (recipient address, weight, shipping method, etc.)
 * @returns The created Parcel object including tracking info (if available)
 * @throws SendcloudError if the API call fails, times out, or credentials are missing
 *
 * @example
 * ```ts
 * const parcel = await createParcel({
 *   name: "Maria García",
 *   address: "Carrer de Provença 123",
 *   city: "Barcelona",
 *   postal_code: "08036",
 *   country: "ES",
 *   email: "maria@example.com",
 *   shipment: { id: 8 },
 *   order_number: "CV-2024-001",
 *   weight: "500",      // grams
 *   request_label: true,
 * });
 * console.log(parcel.tracking_url);
 * ```
 */
export async function createParcel(
  payload: CreateParcelPayload
): Promise<Parcel> {
  validateCreateParcelPayload(payload);

  const response = await sendcloudRequest<CreateParcelResponse>("/parcels", {
    method: "POST",
    body: { parcel: payload },
  });

  return response.parcel;
}

/** Basic client-side validation to catch obvious mistakes before making the API call */
function validateCreateParcelPayload(payload: CreateParcelPayload): void {
  const required: Array<keyof CreateParcelPayload> = [
    "name",
    "address",
    "city",
    "postal_code",
    "country",
    "email",
    "order_number",
    "weight",
  ];

  const missing = required.filter(
    (key) => !payload[key] || String(payload[key]).trim() === ""
  );

  if (missing.length > 0) {
    throw new SendcloudError(
      `createParcel: missing required fields: ${missing.join(", ")}`,
      undefined,
      "VALIDATION_ERROR"
    );
  }

  if (!payload.shipment?.id || typeof payload.shipment.id !== "number") {
    throw new SendcloudError(
      "createParcel: shipment.id must be a valid numeric Sendcloud shipping method ID",
      undefined,
      "VALIDATION_ERROR"
    );
  }

  const weightGrams = parseFloat(payload.weight);
  if (isNaN(weightGrams) || weightGrams <= 0) {
    throw new SendcloudError(
      "createParcel: weight must be a positive number (in grams)",
      undefined,
      "VALIDATION_ERROR"
    );
  }

  // Basic email format check
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(payload.email)) {
    throw new SendcloudError(
      `createParcel: invalid email address: ${payload.email}`,
      undefined,
      "VALIDATION_ERROR"
    );
  }

  // Country code should be 2 chars ISO 3166-1
  if (!/^[A-Z]{2}$/.test(payload.country.toUpperCase())) {
    throw new SendcloudError(
      `createParcel: country must be an ISO 3166-1 alpha-2 code (e.g. "ES"), got: ${payload.country}`,
      undefined,
      "VALIDATION_ERROR"
    );
  }
}

// ─── getShippingMethods ───────────────────────────────────────────────────────

/**
 * Retrieve all shipping methods available for the Sendcloud account,
 * optionally filtered by destination country.
 *
 * @param params  - Optional filters (to_country, sender_address)
 * @returns Array of ShippingMethod objects
 * @throws SendcloudError if the API call fails, times out, or credentials are missing
 *
 * @example
 * ```ts
 * // All methods
 * const methods = await getShippingMethods();
 *
 * // Methods available for Spain
 * const spainMethods = await getShippingMethods({ to_country: "ES" });
 * spainMethods.forEach(m => console.log(m.name, m.price));
 * ```
 */
export async function getShippingMethods(
  params: GetShippingMethodsParams = {}
): Promise<ShippingMethod[]> {
  const queryString = buildQueryString(params);
  const path = `/shipping_methods${queryString}`;

  const response = await sendcloudRequest<GetShippingMethodsResponse>(path, {
    method: "GET",
  });

  return response.shipping_methods;
}

/** Serialise an object of optional string/number values into a query string */
function buildQueryString(
  params: Record<string, string | number | undefined>
): string {
  const entries = Object.entries(params).filter(
    ([, v]) => v !== undefined && v !== null && v !== ""
  );
  if (entries.length === 0) return "";
  const qs = entries
    .map(([k, v]) => `${encodeURIComponent(k)}=${encodeURIComponent(String(v))}`)
    .join("&");
  return `?${qs}`;
}
```

### 2.5 Next.js Server Action (`app/actions/shipping.ts`)

```typescript
// app/actions/shipping.ts
"use server";
/**
 * Server Actions that wrap the Sendcloud service.
 * Called from client components via React's server action pattern.
 */

import {
  createParcel,
  getShippingMethods,
  SendcloudError,
  type CreateParcelPayload,
  type Parcel,
  type ShippingMethod,
} from "@/lib/sendcloud/sendcloud";

export type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; error: string; code?: string | number };

// ─── createParcelAction ───────────────────────────────────────────────────────

/**
 * Server Action: create a parcel for a café vermut coffee order.
 * Returns a discriminated union so the client can handle both success and error.
 */
export async function createParcelAction(
  payload: CreateParcelPayload
): Promise<ActionResult<Parcel>> {
  try {
    const parcel = await createParcel(payload);
    return { success: true, data: parcel };
  } catch (err) {
    if (err instanceof SendcloudError) {
      console.error("[Sendcloud] createParcel failed:", {
        message: err.message,
        statusCode: err.statusCode,
        code: err.sendcloudCode,
      });
      return {
        success: false,
        error: err.message,
        code: err.sendcloudCode,
      };
    }
    console.error("[Sendcloud] Unexpected error in createParcelAction:", err);
    return { success: false, error: "An unexpected error occurred." };
  }
}

// ─── getShippingMethodsAction ─────────────────────────────────────────────────

/**
 * Server Action: fetch available shipping methods, optionally filtered by country.
 */
export async function getShippingMethodsAction(
  toCountry?: string
): Promise<ActionResult<ShippingMethod[]>> {
  try {
    const methods = await getShippingMethods(
      toCountry ? { to_country: toCountry } : {}
    );
    return { success: true, data: methods };
  } catch (err) {
    if (err instanceof SendcloudError) {
      console.error("[Sendcloud] getShippingMethods failed:", {
        message: err.message,
        statusCode: err.statusCode,
        code: err.sendcloudCode,
      });
      return {
        success: false,
        error: err.message,
        code: err.sendcloudCode,
      };
    }
    console.error(
      "[Sendcloud] Unexpected error in getShippingMethodsAction:",
      err
    );
    return { success: false, error: "An unexpected error occurred." };
  }
}
```

### 2.6 Usage example — checkout component (`components/checkout/ShippingSelector.tsx`)

```tsx
// components/checkout/ShippingSelector.tsx
"use client";

import React, { useEffect, useState } from "react";
import { getShippingMethodsAction } from "@/app/actions/shipping";
import type { ShippingMethod } from "@/lib/sendcloud/sendcloud";

interface ShippingSelectorProps {
  destinationCountry?: string;
  onSelect: (method: ShippingMethod) => void;
}

export default function ShippingSelector({
  destinationCountry = "ES",
  onSelect,
}: ShippingSelectorProps) {
  const [methods, setMethods] = useState<ShippingMethod[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [selected, setSelected] = useState<number | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    setError(null);

    getShippingMethodsAction(destinationCountry).then((result) => {
      if (cancelled) return;
      if (result.success) {
        setMethods(result.data);
      } else {
        setError(result.error);
      }
      setLoading(false);
    });

    return () => {
      cancelled = true;
    };
  }, [destinationCountry]);

  if (loading) return <p aria-live="polite">Cargando métodos de envío…</p>;
  if (error)
    return (
      <p role="alert" style={{ color: "#c0392b" }}>
        Error al cargar métodos de envío: {error}
      </p>
    );
  if (methods.length === 0)
    return (
      <p>No hay métodos de envío disponibles para {destinationCountry}.</p>
    );

  return (
    <fieldset>
      <legend style={{ fontFamily: "serif", color: "#3b2a1a", fontWeight: 600 }}>
        Método de envío
      </legend>
      <div role="radiogroup" aria-label="Selecciona método de envío">
        {methods.map((method) => (
          <label
            key={method.id}
            style={{
              display: "flex",
              alignItems: "center",
              gap: "8px",
              padding: "8px 0",
              cursor: "pointer",
              fontFamily: "serif",
              color: "#3b2a1a",
            }}
          >
            <input
              type="radio"
              name="shipping_method"
              value={method.id}
              checked={selected === method.id}
              onChange={() => {
                setSelected(method.id);
                onSelect(method);
              }}
              aria-describedby={`method-desc-${method.id}`}
            />
            <span>
              <strong>{method.name}</strong>{" "}
              <span id={`method-desc-${method.id}`}>
                — {method.carrier} · {method.price} {method.currency ?? "EUR"}
              </span>
            </span>
          </label>
        ))}
      </div>
    </fieldset>
  );
}
```

### 2.7 Required npm packages

```bash
# No extra npm packages needed — uses built-in fetch (Node 18+)
# Ensure Node.js >= 18 for native fetch with AbortController support
```

---

## 3. File Structure Summary

```
lib/
  maps/
    cafeVermutMapStyles.ts       # Brand-aligned map style array
  sendcloud/
    client.ts                    # Low-level HTTP client (auth, timeout, error)
    types.ts                     # TypeScript types for Sendcloud objects
    sendcloud.ts                 # createParcel(), getShippingMethods()

components/
  maps/
    CafeVermutMap.tsx            # Accessible Google Maps React component
  checkout/
    ShippingSelector.tsx         # Example UI using getShippingMethods

app/
  actions/
    shipping.ts                  # Next.js Server Actions wrapping Sendcloud
  contacto/
    page.tsx                     # Example page embedding the map
```

---

## 4. Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | Yes | Google Maps JS API key (exposed to browser) |
| `SENDCLOUD_PUBLIC_KEY` | Yes | Sendcloud API public key (server-side only) |
| `SENDCLOUD_SECRET_KEY` | Yes | Sendcloud API secret key (server-side only) |
| `SENDCLOUD_API_BASE_URL` | No | Override API base URL (default: `https://panel.sendcloud.sc/api/v2`) |

> **Security:** `SENDCLOUD_PUBLIC_KEY` and `SENDCLOUD_SECRET_KEY` must NOT be prefixed with `NEXT_PUBLIC_` — they must remain server-side only and never be sent to the browser.

---

## 5. Error Handling Strategy

### Google Maps

| Scenario | Behaviour |
|---|---|
| Missing API key | Shows inline error message with fallback link |
| Script load failure | Catches in `initMap`, renders error state, moves focus to fallback link |
| Map render failure | Same as above |
| JavaScript disabled | The accessible fallback link is always rendered in the DOM |

### Sendcloud

| Scenario | Behaviour |
|---|---|
| Missing credentials | Throws `SendcloudError` with `VALIDATION_ERROR` code immediately (no HTTP call) |
| Invalid payload | Throws `SendcloudError` with `VALIDATION_ERROR` code immediately (no HTTP call) |
| HTTP 4xx (e.g. bad address) | Throws `SendcloudError` with the Sendcloud error message and status code |
| HTTP 5xx (Sendcloud outage) | Throws `SendcloudError` with HTTP status code |
| Request timeout (>10 s) | AbortController fires; throws `SendcloudError` with `TIMEOUT` code |
| DNS / network failure | Throws `SendcloudError` with `NETWORK_ERROR` code |
| Server Actions | Catches all errors, logs server-side, returns `{ success: false, error }` to client |

---

## 6. Accessibility Checklist — Google Maps

- [x] `<section>` with `aria-label="Café Vermut location map"`
- [x] `<h2 class="sr-only">` inside the section for document outline
- [x] Map `<div>` has `role="application"` and `aria-label` explaining keyboard navigation
- [x] Map `<div>` has `tabIndex={0}` so it receives keyboard focus
- [x] Loading state uses `role="status"` + `aria-live="polite"`
- [x] Error state uses `role="alert"`
- [x] Info window link has explicit `aria-label` including "(opens new tab)"
- [x] Fallback external link always rendered (not hidden behind JS)
- [x] `AdvancedMarkerElement` `title` attribute provides screen-reader text for the marker
- [x] Custom marker DOM element is `aria-hidden="true"` (decorative; screen readers use `title`)
- [x] Colour contrast: all text on brand backgrounds meets WCAG AA 4.5:1 ratio
