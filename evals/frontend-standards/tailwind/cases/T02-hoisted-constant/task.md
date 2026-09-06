# T02 — hoisted-constant

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `billing` feature has a summary card.

`src/features/billing/components/invoice-summary.tsx`

```tsx
import { formatPrice } from "@/lib/utils";

const CARD_CLASSES = "rounded-md border bg-card p-4";

type InvoiceSummaryProps = {
  invoiceNumber: string;
  total: number;
  dueDate: string;
};

export function InvoiceSummary({ invoiceNumber, total, dueDate }: InvoiceSummaryProps) {
  return (
    <div className={CARD_CLASSES}>
      <p className="text-sm text-muted-foreground">Invoice {invoiceNumber}</p>
      <p className="mt-1 text-2xl font-semibold tabular-nums">{formatPrice(total)}</p>
      <p className="mt-2 text-xs text-muted-foreground">Due {dueDate}</p>
    </div>
  );
}
```

`CARD_CLASSES` is not imported anywhere else; a search for it returns only this file.

## Change request

When the user hovers the card, it should gain a medium shadow (`shadow-md`) with a short transition.

Make the change and explain in one or two sentences how you structured the classes.
