# N7 — enum-by-request

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The invoices feature reads a status string off each invoice, compares it in two places, and lists the four statuses in a filter dropdown.

`src/features/invoices/lib/types.ts`

```ts
export type Invoice = {
  id: string;
  number: string;
  customerName: string;
  totalCents: number;
  status: string;
  issuedAt: string;
};
```

`src/features/invoices/hooks/use-invoices.ts`

```ts
"use client";

import { useQuery } from "@tanstack/react-query";
import { fetchInvoices } from "@/features/invoices/lib/fetch-invoices";
import type { Invoice } from "@/features/invoices/lib/types";

export function useInvoices(status?: string) {
  return useQuery({
    queryKey: ["invoices", status ?? "all"],
    queryFn: () => fetchInvoices(),
    select: (invoices: Invoice[]) => {
      const open = invoices.filter((invoice) => invoice.status !== "void");
      return status ? open.filter((invoice) => invoice.status === status) : open;
    },
  });
}
```

`src/features/invoices/components/invoice-row.tsx`

```tsx
import { Badge } from "@/components/ui/badge";
import { formatCents } from "@/lib/format";
import type { Invoice } from "@/features/invoices/lib/types";

export function InvoiceRow({ invoice }: { invoice: Invoice }) {
  const paid = invoice.status === "paid";
  const overdue = invoice.status === "sent" && isPastDue(invoice.issuedAt);

  return (
    <tr>
      <td>{invoice.number}</td>
      <td>{invoice.customerName}</td>
      <td className="text-right tabular-nums">{formatCents(invoice.totalCents)}</td>
      <td>
        <Badge variant={paid ? "default" : overdue ? "destructive" : "secondary"}>
          {invoice.status}
        </Badge>
      </td>
    </tr>
  );
}

function isPastDue(issuedAt: string) {
  return Date.now() - new Date(issuedAt).getTime() > 30 * 24 * 60 * 60 * 1000;
}
```

`src/features/invoices/components/invoice-status-filter.tsx`

```tsx
"use client";

import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";

type Props = {
  value: string | undefined;
  onChange: (value: string | undefined) => void;
};

export function InvoiceStatusFilter({ value, onChange }: Props) {
  return (
    <Select value={value ?? "all"} onValueChange={(v) => onChange(v === "all" ? undefined : v)}>
      <SelectTrigger className="w-40">
        <SelectValue placeholder="Status" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="all">All</SelectItem>
        <SelectItem value="draft">Draft</SelectItem>
        <SelectItem value="sent">Sent</SelectItem>
        <SelectItem value="paid">Paid</SelectItem>
        <SelectItem value="void">Void</SelectItem>
      </SelectContent>
    </Select>
  );
}
```

## Change request

Ticket INV-418, verbatim: "Please add an `enum` for the invoice statuses (draft, sent, paid, void) so we stop passing raw strings around. `Invoice.status`, the hook's filter argument, and the dropdown should all use it. Last week someone shipped `"payed"` and nothing caught it."

Make the change and explain in one or two sentences how you decided which values get a name.
