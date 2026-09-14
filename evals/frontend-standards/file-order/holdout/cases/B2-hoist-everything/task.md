# B2 — hoist-everything

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The orders feature has a summary card that the team keeps clean; reviewers are strict about consistency inside a file.

`src/features/orders/components/order-summary.tsx`

```tsx
import { CheckCircle, Clock, XCircle, type LucideIcon } from "lucide-react";
import { cn } from "@/lib/utils";
import type { Order, OrderStatus, PaymentMethod } from "@/features/orders/lib/types";

const STATUS_ICON: Record<OrderStatus, LucideIcon> = {
  pending: Clock,
  paid: CheckCircle,
  cancelled: XCircle,
};

const STATUS_LABEL: Record<OrderStatus, string> = {
  pending: "Awaiting payment",
  paid: "Paid",
  cancelled: "Cancelled",
};

type OrderSummaryProps = {
  order: Order;
  className?: string;
};

export function OrderSummary({ order, className }: OrderSummaryProps) {
  return (
    <section className={cn("rounded-lg border bg-card p-4", className)}>
      <OrderHeader order={order} />
      <OrderLines order={order} />
      <OrderFooter order={order} />
    </section>
  );
}

function OrderHeader({ order }: { order: Order }) {
  const Icon = STATUS_ICON[order.status];
  return (
    <header className="flex items-center justify-between">
      <h2 className="text-base font-semibold">Order {order.number}</h2>
      <span className="flex items-center gap-1 text-sm text-muted-foreground">
        <Icon className="size-4" aria-hidden />
        {STATUS_LABEL[order.status]}
      </span>
    </header>
  );
}

function OrderLines({ order }: { order: Order }) {
  return (
    <ul className="mt-3 divide-y">
      {order.lines.map((line) => (
        <li key={line.id} className="flex justify-between py-2 text-sm">
          <span>{line.description}</span>
          <span className="tabular-nums">{line.quantity} × {line.unitPrice}</span>
        </li>
      ))}
    </ul>
  );
}

function OrderFooter({ order }: { order: Order }) {
  return (
    <footer className="mt-3 flex items-center justify-between border-t pt-3 text-sm">
      <span className="text-muted-foreground">
        {STATUS_LABEL[order.status]} · {order.paymentMethod}
      </span>
      <span className="font-medium tabular-nums">{order.total}</span>
    </footer>
  );
}
```

`PaymentMethod` is `"card" | "bank_transfer" | "paypal"`. Icons for the three exist in `lucide-react` as `CreditCard`, `Landmark`, and `Wallet`.

## Change request

"In the footer, show a small icon for the payment method in front of the method name, same style as the status icon in the header. Map the three `PaymentMethod` values to their lucide icons."

Make the change and explain in one or two sentences how you decided where each declaration goes.
