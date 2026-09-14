# F02 — single-reader-label-map

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/billing/lib/types.ts`

```ts
export type PaymentMethod = "card" | "bank_transfer" | "paypal" | "invoice";

export type Invoice = {
  id: string;
  subtotalCents: number;
  taxRate: number;
  paymentMethod: PaymentMethod;
  paidAt: Date | null;
};
```

`src/features/billing/components/invoice-summary.tsx`

```tsx
import { formatPrice } from "@/lib/format";
import type { Invoice } from "@/features/billing/lib/types";

type InvoiceSummaryProps = {
  invoice: Invoice;
};

export function InvoiceSummary({ invoice }: InvoiceSummaryProps) {
  return (
    <div className="rounded-lg border p-4">
      <InvoiceTotals invoice={invoice} />
      <InvoicePayment invoice={invoice} />
    </div>
  );
}

function InvoiceTotals({ invoice }: { invoice: Invoice }) {
  const tax = Math.round(invoice.subtotalCents * invoice.taxRate);
  const total = invoice.subtotalCents + tax;

  return (
    <dl className="grid grid-cols-2 gap-1 text-sm">
      <dt>Subtotal</dt>
      <dd className="text-right">{formatPrice(invoice.subtotalCents)}</dd>
      <dt>Tax</dt>
      <dd className="text-right">{formatPrice(tax)}</dd>
      <dt className="font-medium">Total</dt>
      <dd className="text-right font-medium">{formatPrice(total)}</dd>
    </dl>
  );
}

function InvoicePayment({ invoice }: { invoice: Invoice }) {
  return (
    <p className="mt-3 text-sm text-muted-foreground">
      Paid via {invoice.paymentMethod}
      {invoice.paidAt ? ` on ${invoice.paidAt.toLocaleDateString()}` : ""}
    </p>
  );
}
```

## Change request

The payment line prints the raw enum value ("bank_transfer"). Show human wording instead: "Card", "Bank transfer", "PayPal", "Pay by invoice". Keep the wording in one lookup object typed `Record<PaymentMethod, string>` so that adding a new payment method fails type-checking until a label is added. Nothing else in the file changes behaviour.

Make the change and explain in one or two sentences how you decided where each declaration goes.
