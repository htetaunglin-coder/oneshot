# Case 03 — two-features-one-policy

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

Two features each have a discount helper. Product has confirmed the rule is a single
company-wide policy: "10% off orders over $100, capped at $50". The cart preview and
the checkout total must always show the same number for the same subtotal; a mismatch
is treated as a billing bug.

### `src/features/checkout/lib/discount.ts`

```ts
const THRESHOLD_CENTS = 10_000;
const RATE = 0.1;
const CAP_CENTS = 5_000;

export function discountForSubtotal(subtotalCents: number): number {
  if (subtotalCents <= THRESHOLD_CENTS) return 0;
  return Math.min(Math.round(subtotalCents * RATE), CAP_CENTS);
}
```

Used by `src/features/checkout/components/order-summary.tsx`.

### `src/features/cart/lib/discount.ts`

```ts
const THRESHOLD_CENTS = 10_000;
const RATE = 0.1;
const CAP_CENTS = 5_000;

export function discountForSubtotal(subtotalCents: number): number {
  if (subtotalCents <= THRESHOLD_CENTS) return 0;
  return Math.min(Math.round(subtotalCents * RATE), CAP_CENTS);
}
```

Used by `src/features/cart/components/cart-drawer.tsx`.

No other feature computes a discount.

## Task

Product raised the cap from $50 to $60. Make the change. Decide where the
new/changed code lives and explain in one or two sentences.
