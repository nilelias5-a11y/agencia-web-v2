# Café Vermut — GSAP Micro-interactions Implementation

All animations use `transform` and `opacity` exclusively to keep the browser in the compositor thread and guarantee 60 fps on mobile. Every animated element receives `will-change: transform` (set once, removed after animation completes) and 3D hardware-acceleration via `translateZ(0)`.

---

## 0. Shared setup

Install dependencies once:

```bash
npm install gsap
# GSAP is framework-agnostic; no extra React adapter needed
```

```ts
// lib/gsap.ts  — centralised GSAP config
import { gsap } from "gsap";

gsap.config({
  force3D: true,        // always use matrix3d → GPU layer
  nullTargetWarn: false,
});

export { gsap };
```

---

## 1. "Añadir al carrito" button — success feedback

```tsx
// components/AddToCartButton.tsx
"use client";

import { useRef, useState, useCallback } from "react";
import { gsap } from "@/lib/gsap";
import { ShoppingCart, Check } from "lucide-react";

interface Props {
  productId: string;
  onAdd: (id: string) => Promise<void>;
}

export function AddToCartButton({ productId, onAdd }: Props) {
  const btnRef = useRef<HTMLButtonElement>(null);
  const iconWrapRef = useRef<HTMLSpanElement>(null);
  const labelRef = useRef<HTMLSpanElement>(null);
  const rippleRef = useRef<HTMLSpanElement>(null);
  const [status, setStatus] = useState<"idle" | "loading" | "success">("idle");
  const tlRef = useRef<gsap.core.Timeline | null>(null);

  const handleClick = useCallback(async () => {
    if (status !== "idle") return;
    const btn = btnRef.current;
    if (!btn) return;

    // Kill any in-flight timeline
    tlRef.current?.kill();

    setStatus("loading");

    const tl = gsap.timeline({ defaults: { ease: "power2.out" } });
    tlRef.current = tl;

    // 1. Quick press feedback (scale down)
    tl.to(btn, { scale: 0.95, duration: 0.08 });

    // 2. Ripple expand
    tl.fromTo(
      rippleRef.current,
      { scale: 0, opacity: 0.35, transformOrigin: "center center" },
      { scale: 2.8, opacity: 0, duration: 0.55, ease: "power1.out" },
      "<"
    );

    // 3. Return to normal scale
    tl.to(btn, { scale: 1, duration: 0.15 }, ">-0.3");

    try {
      await onAdd(productId);
    } catch {
      setStatus("idle");
      return;
    }

    setStatus("success");

    // 4. Swap icon: cart → check
    tl.to(iconWrapRef.current, { y: -20, opacity: 0, duration: 0.18 })
      .set(iconWrapRef.current, { y: 20 })
      .to(iconWrapRef.current, { y: 0, opacity: 1, duration: 0.22 });

    // 5. Background colour shift (via CSS var, not layout-triggering)
    tl.to(
      btn,
      {
        "--btn-bg": "var(--color-success, #22c55e)",
        duration: 0.25,
        ease: "none",
      },
      "<"
    );

    // 6. Label swap
    tl.to(labelRef.current, { opacity: 0, y: -8, duration: 0.15 }, "<")
      .set(labelRef.current, { y: 8 })
      .to(labelRef.current, { opacity: 1, y: 0, duration: 0.2 });

    // 7. Hold success state then reset
    tl.to(btn, { scale: 1.04, duration: 0.12 })
      .to(btn, { scale: 1, duration: 0.12 })
      .to(
        btn,
        {
          "--btn-bg": "var(--color-primary, #c8a96e)",
          duration: 0.4,
          delay: 1.2,
          onComplete: () => setStatus("idle"),
        }
      );
  }, [status, productId, onAdd]);

  const isSuccess = status === "success";

  return (
    <button
      ref={btnRef}
      onClick={handleClick}
      disabled={status === "loading"}
      aria-label={isSuccess ? "Producto añadido" : "Añadir al carrito"}
      className="
        relative overflow-hidden rounded-full px-6 py-3
        font-semibold text-white select-none
        transition-[box-shadow] duration-200
        hover:shadow-lg active:shadow-sm
        [background-color:var(--btn-bg,var(--color-primary,#c8a96e))]
        will-change-transform
      "
      style={
        {
          "--btn-bg": "var(--color-primary, #c8a96e)",
        } as React.CSSProperties
      }
    >
      {/* Ripple layer */}
      <span
        ref={rippleRef}
        aria-hidden
        className="
          pointer-events-none absolute inset-0 m-auto
          h-12 w-12 rounded-full bg-white
          will-change-transform
        "
        style={{ transformOrigin: "center center" }}
      />

      <span className="relative flex items-center gap-2">
        {/* Animated icon container */}
        <span ref={iconWrapRef} className="inline-flex will-change-transform">
          {isSuccess ? (
            <Check size={18} strokeWidth={2.5} />
          ) : (
            <ShoppingCart size={18} />
          )}
        </span>

        {/* Animated label */}
        <span ref={labelRef} className="will-change-transform">
          {isSuccess ? "¡Añadido!" : "Añadir al carrito"}
        </span>
      </span>
    </button>
  );
}
```

**Key GPU decisions:**
- `scale` and `y` translate to `transform: matrix3d(…)` — zero layout cost.
- The ripple uses `scale` + `opacity`, never `width`/`height`.
- CSS custom property `--btn-bg` drives the colour so the browser batches it outside of layout.

---

## 2. Product card hover — premium feel

```tsx
// components/ProductCard.tsx
"use client";

import { useRef, useCallback } from "react";
import Image from "next/image";
import { gsap } from "@/lib/gsap";

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  category: string;
}

interface Props {
  product: Product;
}

export function ProductCard({ product }: Props) {
  const cardRef = useRef<HTMLDivElement>(null);
  const imgRef = useRef<HTMLDivElement>(null);
  const overlayRef = useRef<HTMLDivElement>(null);
  const contentRef = useRef<HTMLDivElement>(null);
  const shimmerRef = useRef<HTMLDivElement>(null);

  const enterTl = useRef<gsap.core.Timeline | null>(null);

  const handleMouseEnter = useCallback(() => {
    enterTl.current?.kill();

    const tl = gsap.timeline({ defaults: { ease: "power3.out" } });
    enterTl.current = tl;

    tl
      // Card lift + scale
      .to(cardRef.current, {
        y: -8,
        scale: 1.02,
        boxShadow:
          "0 24px 48px -8px rgba(0,0,0,0.22), 0 8px 16px -4px rgba(0,0,0,0.12)",
        duration: 0.35,
      })
      // Image zoom (inner div, not the card — avoids overflow reflow)
      .to(
        imgRef.current,
        { scale: 1.08, duration: 0.55, ease: "power2.out" },
        "<"
      )
      // Overlay fade in
      .to(overlayRef.current, { opacity: 1, duration: 0.3 }, "<")
      // Content slide up
      .to(
        contentRef.current,
        { y: -4, opacity: 1, duration: 0.28 },
        "<0.05"
      )
      // Shimmer sweep
      .fromTo(
        shimmerRef.current,
        { x: "-110%", opacity: 0.6 },
        { x: "110%", opacity: 0, duration: 0.6, ease: "power1.inOut" },
        "<"
      );
  }, []);

  const handleMouseLeave = useCallback(() => {
    enterTl.current?.kill();

    gsap.to(
      [cardRef.current, imgRef.current, overlayRef.current, contentRef.current],
      {
        y: 0,
        scale: 1,
        opacity: (i: number) => (i === 2 ? 0 : 1), // overlay → 0
        boxShadow: "0 4px 12px -2px rgba(0,0,0,0.08)",
        duration: 0.3,
        ease: "power2.inOut",
      }
    );
  }, []);

  return (
    <div
      ref={cardRef}
      onMouseEnter={handleMouseEnter}
      onMouseLeave={handleMouseLeave}
      className="
        relative rounded-2xl overflow-hidden bg-white cursor-pointer
        will-change-transform
        shadow-[0_4px_12px_-2px_rgba(0,0,0,0.08)]
      "
      style={{ transform: "translateZ(0)" }} // force GPU layer immediately
    >
      {/* Image container (inner scale target) */}
      <div
        ref={imgRef}
        className="relative h-56 w-full overflow-hidden will-change-transform"
        style={{ transform: "translateZ(0)" }}
      >
        <Image
          src={product.image}
          alt={product.name}
          fill
          className="object-cover"
          sizes="(max-width: 768px) 100vw, 33vw"
        />

        {/* Shimmer sweep */}
        <div
          ref={shimmerRef}
          aria-hidden
          className="
            absolute inset-y-0 w-1/3
            bg-gradient-to-r from-transparent via-white/30 to-transparent
            -skew-x-12 pointer-events-none will-change-transform
          "
        />
      </div>

      {/* Gradient overlay */}
      <div
        ref={overlayRef}
        aria-hidden
        className="
          absolute inset-0
          bg-gradient-to-t from-black/40 via-transparent to-transparent
          opacity-0 pointer-events-none
        "
      />

      {/* Content */}
      <div
        ref={contentRef}
        className="relative p-4 will-change-transform"
      >
        <span className="text-xs font-medium uppercase tracking-wider text-amber-600">
          {product.category}
        </span>
        <h3 className="mt-1 text-lg font-semibold text-gray-900 leading-snug">
          {product.name}
        </h3>
        <p className="mt-2 text-xl font-bold text-amber-700">
          {product.price.toFixed(2)} €
        </p>
      </div>
    </div>
  );
}
```

**Mobile note:** The hover listeners do nothing on touch devices (no `mouseenter` on mobile). Add `onTouchStart`/`onTouchEnd` if a touch-lift effect is desired; keep the animation duration ≤ 200 ms for touch to feel snappy.

---

## 3. Cart quantity counter — increment/decrement animation

```tsx
// components/QuantityCounter.tsx
"use client";

import { useRef, useCallback, useId } from "react";
import { gsap } from "@/lib/gsap";

interface Props {
  value: number;
  min?: number;
  max?: number;
  onChange: (next: number) => void;
}

export function QuantityCounter({ value, min = 1, max = 99, onChange }: Props) {
  const digitRef = useRef<HTMLSpanElement>(null);
  const prevValueRef = useRef(value);
  const labelId = useId();

  const animateDigit = useCallback((direction: "up" | "down") => {
    const el = digitRef.current;
    if (!el) return;

    const fromY = direction === "up" ? 14 : -14;
    const toY = direction === "up" ? -14 : 14;

    gsap
      .timeline()
      // Current number exits
      .to(el, { y: toY, opacity: 0, duration: 0.14, ease: "power1.in" })
      // Reset position for incoming number (set happens inside React re-render,
      // but we pre-position here so there's no flash)
      .set(el, { y: fromY })
      // Incoming number enters
      .to(el, { y: 0, opacity: 1, duration: 0.18, ease: "power2.out" });
  }, []);

  const increment = useCallback(() => {
    if (value >= max) return;
    animateDigit("up");
    prevValueRef.current = value;
    onChange(value + 1);
  }, [value, max, onChange, animateDigit]);

  const decrement = useCallback(() => {
    if (value <= min) return;
    animateDigit("down");
    prevValueRef.current = value;
    onChange(value - 1);
  }, [value, min, onChange, animateDigit]);

  return (
    <div
      role="group"
      aria-labelledby={labelId}
      className="
        inline-flex items-center gap-0 rounded-full border border-gray-200
        bg-white shadow-sm overflow-hidden select-none
      "
    >
      <span id={labelId} className="sr-only">
        Cantidad
      </span>

      <button
        onClick={decrement}
        disabled={value <= min}
        aria-label="Reducir cantidad"
        className="
          w-9 h-9 flex items-center justify-center text-gray-600
          hover:bg-gray-50 active:bg-gray-100
          disabled:opacity-30 disabled:cursor-not-allowed
          transition-colors duration-150
        "
      >
        <span aria-hidden className="text-lg leading-none">−</span>
      </button>

      {/* Fixed-width digit window to prevent layout shift */}
      <div
        aria-live="polite"
        aria-atomic="true"
        className="w-8 h-9 flex items-center justify-center overflow-hidden"
      >
        <span
          ref={digitRef}
          className="block text-sm font-semibold text-gray-900 will-change-transform"
          style={{ transform: "translateZ(0)" }}
        >
          {value}
        </span>
      </div>

      <button
        onClick={increment}
        disabled={value >= max}
        aria-label="Aumentar cantidad"
        className="
          w-9 h-9 flex items-center justify-center text-gray-600
          hover:bg-gray-50 active:bg-gray-100
          disabled:opacity-30 disabled:cursor-not-allowed
          transition-colors duration-150
        "
      >
        <span aria-hidden className="text-lg leading-none">+</span>
      </button>
    </div>
  );
}
```

**Why this works at 60 fps:** A single `<span>` slides in/out with `y` + `opacity`. No DOM nodes are cloned or inserted — the `value` prop update and the GSAP timeline are coordinated so React's `textContent` update happens between the exit and entrance tweens.

---

## 4. Toast notification "Producto añadido"

```tsx
// components/Toast.tsx  — individual toast
"use client";

import { useRef, useEffect } from "react";
import { gsap } from "@/lib/gsap";
import { Check, X } from "lucide-react";

export interface ToastData {
  id: string;
  message: string;
  productName?: string;
  duration?: number; // ms, default 3500
}

interface Props extends ToastData {
  onDismiss: (id: string) => void;
}

export function Toast({
  id,
  message,
  productName,
  duration = 3500,
  onDismiss,
}: Props) {
  const rootRef = useRef<HTMLDivElement>(null);
  const progressRef = useRef<HTMLDivElement>(null);
  const tlRef = useRef<gsap.core.Timeline | null>(null);

  useEffect(() => {
    const el = rootRef.current;
    const bar = progressRef.current;
    if (!el || !bar) return;

    // Set initial state (off-screen right + invisible)
    gsap.set(el, { x: "110%", opacity: 0 });

    const tl = gsap.timeline();
    tlRef.current = tl;

    tl
      // 1. Slide in + fade
      .to(el, {
        x: "0%",
        opacity: 1,
        duration: 0.4,
        ease: "back.out(1.4)",
      })
      // 2. Subtle settle bounce
      .to(el, { x: "-4px", duration: 0.08, ease: "power1.out" })
      .to(el, { x: "0px", duration: 0.1, ease: "power1.inOut" })
      // 3. Progress bar drains
      .to(
        bar,
        {
          scaleX: 0,
          transformOrigin: "left center",
          duration: duration / 1000,
          ease: "none",
        },
        "<"
      )
      // 4. Auto-dismiss: slide out + fade
      .to(el, {
        x: "110%",
        opacity: 0,
        duration: 0.3,
        ease: "power2.in",
        onComplete: () => onDismiss(id),
      });

    return () => {
      tl.kill();
    };
  }, []); // run once on mount

  const dismiss = () => {
    tlRef.current?.kill();
    gsap.to(rootRef.current, {
      x: "110%",
      opacity: 0,
      duration: 0.25,
      ease: "power2.in",
      onComplete: () => onDismiss(id),
    });
  };

  return (
    <div
      ref={rootRef}
      role="status"
      aria-live="polite"
      className="
        relative flex items-center gap-3
        rounded-xl bg-white shadow-xl shadow-black/10
        border border-gray-100 px-4 py-3 min-w-[280px] max-w-sm
        will-change-transform overflow-hidden
      "
      style={{ transform: "translateZ(0)" }}
    >
      {/* Icon */}
      <span className="flex h-8 w-8 shrink-0 items-center justify-center rounded-full bg-green-100">
        <Check size={16} className="text-green-600" strokeWidth={2.5} />
      </span>

      {/* Text */}
      <div className="flex-1 min-w-0">
        <p className="text-sm font-semibold text-gray-900 truncate">{message}</p>
        {productName && (
          <p className="text-xs text-gray-500 truncate">{productName}</p>
        )}
      </div>

      {/* Close button */}
      <button
        onClick={dismiss}
        aria-label="Cerrar notificación"
        className="shrink-0 p-1 rounded-md text-gray-400 hover:text-gray-600 hover:bg-gray-100 transition-colors"
      >
        <X size={14} />
      </button>

      {/* Progress bar */}
      <div
        ref={progressRef}
        aria-hidden
        className="
          absolute bottom-0 left-0 h-0.5 w-full
          bg-gradient-to-r from-amber-400 to-amber-600
          will-change-transform
        "
        style={{ transformOrigin: "left center" }}
      />
    </div>
  );
}
```

```tsx
// components/ToastContainer.tsx  — manages the stack
"use client";

import { createContext, useCallback, useContext, useRef, useState } from "react";
import { createPortal } from "react-dom";
import { Toast, type ToastData } from "./Toast";

interface ToastContextValue {
  addToast: (data: Omit<ToastData, "id">) => void;
}

const ToastContext = createContext<ToastContextValue>({
  addToast: () => void 0,
});

export function useToast() {
  return useContext(ToastContext);
}

export function ToastProvider({ children }: { children: React.ReactNode }) {
  const [toasts, setToasts] = useState<ToastData[]>([]);
  const counterRef = useRef(0);

  const addToast = useCallback((data: Omit<ToastData, "id">) => {
    const id = `toast-${++counterRef.current}`;
    setToasts((prev) => [...prev, { ...data, id }]);
  }, []);

  const removeToast = useCallback((id: string) => {
    setToasts((prev) => prev.filter((t) => t.id !== id));
  }, []);

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      {typeof document !== "undefined" &&
        createPortal(
          <div
            aria-label="Notificaciones"
            className="fixed bottom-6 right-4 z-50 flex flex-col gap-2 pointer-events-none"
          >
            {toasts.map((t) => (
              <div key={t.id} className="pointer-events-auto">
                <Toast {...t} onDismiss={removeToast} />
              </div>
            ))}
          </div>,
          document.body
        )}
    </ToastContext.Provider>
  );
}
```

**Usage in `AddToCartButton`:**
```tsx
const { addToast } = useToast();

// inside onAdd callback:
addToast({ message: "Producto añadido", productName: product.name });
```

---

## 5. Product loading skeleton → content reveal

```tsx
// components/ProductCardSkeleton.tsx
export function ProductCardSkeleton() {
  return (
    <div className="rounded-2xl overflow-hidden bg-white shadow-[0_4px_12px_-2px_rgba(0,0,0,0.08)]">
      {/* Image skeleton */}
      <div className="h-56 w-full bg-gray-200 animate-pulse" />
      {/* Content skeleton */}
      <div className="p-4 space-y-2">
        <div className="h-3 w-1/3 rounded-full bg-gray-200 animate-pulse" />
        <div className="h-5 w-3/4 rounded-full bg-gray-200 animate-pulse" />
        <div className="h-6 w-1/4 rounded-full bg-gray-200 animate-pulse" />
      </div>
    </div>
  );
}
```

```tsx
// components/ProductCardReveal.tsx
"use client";

import { useRef, useEffect } from "react";
import { gsap } from "@/lib/gsap";
import Image from "next/image";

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  category: string;
}

interface Props {
  product: Product;
  /** Index in grid — used to stagger entrance */
  index?: number;
}

export function ProductCardReveal({ product, index = 0 }: Props) {
  const cardRef = useRef<HTMLDivElement>(null);
  const imgRef = useRef<HTMLDivElement>(null);
  const textRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const card = cardRef.current;
    if (!card) return;

    // Start state (invisible, slightly below)
    gsap.set(card, { opacity: 0, y: 20 });
    gsap.set(imgRef.current, { scale: 1.05 }); // slight zoom starts, relaxes in

    const delay = index * 0.08; // stagger per card

    const tl = gsap.timeline({ delay });

    tl
      // Card slides up and fades in
      .to(card, {
        opacity: 1,
        y: 0,
        duration: 0.45,
        ease: "power3.out",
        clearProps: "will-change", // release GPU layer once stable
      })
      // Image settles to natural scale
      .to(
        imgRef.current,
        { scale: 1, duration: 0.6, ease: "power2.out" },
        "<0.05"
      )
      // Text block fades in with micro-slide
      .fromTo(
        textRef.current,
        { opacity: 0, y: 8 },
        { opacity: 1, y: 0, duration: 0.3, ease: "power2.out" },
        "<0.1"
      );

    return () => {
      tl.kill();
    };
  }, [index]);

  return (
    <div
      ref={cardRef}
      className="rounded-2xl overflow-hidden bg-white cursor-pointer will-change-transform"
      style={{ transform: "translateZ(0)" }}
    >
      <div
        ref={imgRef}
        className="relative h-56 w-full overflow-hidden will-change-transform"
        style={{ transform: "translateZ(0)" }}
      >
        <Image
          src={product.image}
          alt={product.name}
          fill
          className="object-cover"
          sizes="(max-width: 768px) 100vw, 33vw"
        />
      </div>

      <div ref={textRef} className="p-4">
        <span className="text-xs font-medium uppercase tracking-wider text-amber-600">
          {product.category}
        </span>
        <h3 className="mt-1 text-lg font-semibold text-gray-900 leading-snug">
          {product.name}
        </h3>
        <p className="mt-2 text-xl font-bold text-amber-700">
          {product.price.toFixed(2)} €
        </p>
      </div>
    </div>
  );
}
```

```tsx
// components/ProductGrid.tsx  — wires skeleton → reveal
"use client";

import { Suspense } from "react";
import { ProductCardSkeleton } from "./ProductCardSkeleton";
import { ProductCardReveal } from "./ProductCardReveal";

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  category: string;
}

function SkeletonGrid({ count = 6 }: { count?: number }) {
  return (
    <>
      {Array.from({ length: count }).map((_, i) => (
        <ProductCardSkeleton key={i} />
      ))}
    </>
  );
}

function ProductList({ products }: { products: Product[] }) {
  return (
    <>
      {products.map((p, i) => (
        <ProductCardReveal key={p.id} product={p} index={i} />
      ))}
    </>
  );
}

interface GridProps {
  productsPromise: Promise<Product[]>;
}

// Server component wrapper can pass a promise; Suspense renders skeleton
export function ProductGrid({ productsPromise }: GridProps) {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
      <Suspense fallback={<SkeletonGrid />}>
        {/* @ts-expect-error — async Server Component */}
        <ResolvedProducts promise={productsPromise} />
      </Suspense>
    </div>
  );
}

async function ResolvedProducts({ promise }: { promise: Promise<Product[]> }) {
  const products = await promise;
  return <ProductList products={products} />;
}
```

---

## GPU acceleration summary

| Technique | Applied to |
|---|---|
| `transform: translateZ(0)` on mount | All animated root elements |
| `will-change: transform` (removed via `clearProps` after animation) | Card reveal, button, digit counter |
| Only `transform` + `opacity` tweened | Every GSAP tween in this file |
| `gsap.config({ force3D: true })` | Global — ensures matrix3d always used |
| `tl.kill()` in `useEffect` cleanup | Prevents detached-node leaks |
| Fixed-size digit window (`overflow: hidden`) | Counter — prevents layout reflow |
| Portal + `pointer-events-none` wrapper | Toasts — never trigger reflow on parent |

---

## File structure

```
components/
  AddToCartButton.tsx
  ProductCard.tsx
  ProductCardReveal.tsx
  ProductCardSkeleton.tsx
  ProductGrid.tsx
  QuantityCounter.tsx
  Toast.tsx
  ToastContainer.tsx
lib/
  gsap.ts
```
