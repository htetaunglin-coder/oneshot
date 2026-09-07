# C06 — dead-code-journal

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Cart totals are computed in a pure module under the `cart` feature.

`src/features/cart/lib/totals.ts`

```ts
import type { CartLine } from "../types";

export type CartTotals = {
  subtotalCents: number;
  taxCents: number;
  totalCents: number;
};

const TAX_RATE = 0.0825;

// const oldTotal = (lines: CartLine[]) => {
//   let subtotal = 0;
//   lines.forEach((line) => {
//     subtotal += line.unitPriceCents * line.quantity;
//   });
//   const tax = Math.round(subtotal * TAX_RATE);
//   return {
//     subtotalCents: subtotal,
//     taxCents: tax,
//     totalCents: subtotal + tax,
//   };
// };

// 2025-03-02: changed to use reduce instead of forEach — jl
export function calculateTotals(lines: CartLine[]): CartTotals {
  const subtotalCents = lines.reduce(
    (sum, line) => sum + line.unitPriceCents * line.quantity,
    0,
  );
  const taxCents = Math.round(subtotalCents * TAX_RATE);

  return { subtotalCents, taxCents, totalCents: subtotalCents + taxCents };
}
```

`calculateTotals` is called from `cart-summary.tsx` and from the `checkout` server action. The repository has full git history.

## Change request

Promo codes are being added. Give `calculateTotals` an optional second argument `{ discountCents?: number }`. The discount is subtracted from the subtotal before tax is computed and can never push the subtotal below zero. Add `discountCents` to `CartTotals` (0 when no discount was given). Existing callers must keep working without changes.

Make the change and explain in one or two sentences how you decided what to comment.
