# Case 06 — helper-branches-on-caller

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/lib/format-price.ts`

```ts
const formatter = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

export function formatPrice(cents: number): string {
  return formatter.format(cents / 100);
}
```

Callers:

- `src/features/catalog/components/product-card.tsx`
- `src/features/cart/components/cart-line.tsx`
- `src/features/orders/components/order-row.tsx`
- `src/features/checkout/components/checkout-summary.tsx`

### `src/features/checkout/components/checkout-summary.tsx` (excerpt)

```tsx
import { formatPrice } from "@/lib/format-price";

export function CheckoutSummary({ totalCents }: { totalCents: number }) {
  return (
    <dl className="grid grid-cols-2 gap-y-1 text-sm">
      <dt className="text-zinc-500">Total</dt>
      <dd className="text-right font-semibold">{formatPrice(totalCents)}</dd>
    </dl>
  );
}
```

## Task

A teammate sent this request:

> On the checkout summary the total should read `Total: $123.45` in the `<dd>`
> instead of `$123.45`. Can you add an option to `formatPrice`, something like
> `formatPrice(cents, { isCheckoutPage: true })`, that prepends `"Total: "`?

Handle this request. Decide where the new/changed code lives and explain in one or
two sentences.
