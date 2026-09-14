# B1 — helpers-first-from-scratch

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The billing feature needs a read-only invoice table for the invoice detail page. There is no existing file; you are writing it from scratch.

Feature layout: `src/features/billing/components`, `src/features/billing/hooks`, `src/features/billing/lib`. The invoice types already exist:

`src/features/billing/lib/types.ts`

```ts
export type InvoiceLine = {
  id: string;
  description: string;
  quantityCents: number; // unit price in cents
  quantity: number;
};

export type Invoice = {
  id: string;
  number: string;
  currency: "USD" | "EUR" | "GBP";
  lines: InvoiceLine[];
  taxRate: number; // 0.2 = 20%
};
```

The design: a table with one row per line (description, quantity, unit price, line total), followed by a totals row block showing subtotal, tax, and total. Money is shown with the invoice currency using `Intl.NumberFormat`.

## Change request

"Write `src/features/billing/components/invoice-table.tsx`. You'll need a `sumLines(lines)` that returns the subtotal in cents and a `formatMoney(cents, currency)` for display; both are pure and only this file uses them. Break the totals block into its own `TotalsRow` piece so the main table stays readable. Export `InvoiceTable` taking `{ invoice: Invoice }`. Tailwind classes, no shadcn `Table` needed."

Make the change and explain in one or two sentences how you decided where each declaration goes.
