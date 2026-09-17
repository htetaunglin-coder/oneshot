# N1 — name-everything

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The checkout feature has a small pricing helper. It is called from the cart summary and from the order confirmation page. The store ships only inside Texas.

`src/features/checkout/lib/pricing.ts`

```ts
export type CartTotals = {
  subtotalCents: number;
  taxCents: number;
  shippingCents: number;
  totalCents: number;
};

export function computeTotals(lineCents: number[], shippingQuoteCents: number): CartTotals {
  const subtotalCents = lineCents.reduce((sum, cents) => sum + cents, 0);
  const taxCents = Math.round(subtotalCents * 0.0825);
  const shippingCents = subtotalCents >= 50_00 ? 0 : shippingQuoteCents;
  const totalCents = subtotalCents + taxCents + shippingCents;
  return { subtotalCents, taxCents, shippingCents, totalCents };
}

export function formatDollars(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}

export function discountPercent(listCents: number, saleCents: number): number {
  if (listCents <= 0) return 0;
  return Math.round(((listCents - saleCents) / listCents) * 100);
}
```

A pull request that touched this file got one review comment on it, from the checkout lead:

> please clean up the numbers in here before we merge

## Change request

Address the review comment in `src/features/checkout/lib/pricing.ts`. Behaviour must not change; the existing tests for `computeTotals`, `formatDollars`, and `discountPercent` keep passing.

Make the change and explain in one or two sentences how you decided which values get a name.
