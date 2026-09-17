# R05 — order-status-cancelled

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/orders/lib/types.ts`

```ts
export type OrderStatus = "pending" | "paid" | "shipped" | "delivered";

export type Order = {
  id: string;
  number: string;
  status: OrderStatus;
  trackingNumber?: string;
  deliveredAt?: string;
  cancelledAt?: string;
};
```

`src/features/orders/components/order-status-panel.tsx`

```tsx
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { formatDate } from "@/lib/format";
import type { Order } from "../lib/types";

type OrderStatusPanelProps = {
  order: Order;
  onTrack: (trackingNumber: string) => void;
};

export function OrderStatusPanel({ order, onTrack }: OrderStatusPanelProps) {
  return (
    <div className="rounded-lg border p-4">
      <h3 className="mb-2 font-semibold">Order #{order.number}</h3>
      {(() => {
        switch (order.status) {
          case "pending":
            return (
              <p className="text-sm text-muted-foreground">
                Waiting for payment confirmation.
              </p>
            );
          case "paid":
            return (
              <div className="flex items-center gap-2">
                <Badge>Paid</Badge>
                <p className="text-sm">We are preparing your parcel.</p>
              </div>
            );
          case "shipped":
            return (
              <div className="flex items-center gap-2">
                <Badge variant="secondary">Shipped</Badge>
                <Button
                  size="sm"
                  variant="outline"
                  disabled={!order.trackingNumber}
                  onClick={() => onTrack(order.trackingNumber ?? "")}
                >
                  Track parcel
                </Button>
              </div>
            );
          case "delivered":
            return (
              <p className="text-sm">
                Delivered on {order.deliveredAt ? formatDate(order.deliveredAt) : "an unknown date"}
              </p>
            );
        }
      })()}
    </div>
  );
}
```

## Change request

Orders can now be cancelled. Add `"cancelled"` to `OrderStatus`. A cancelled order shows a `destructive` badge reading "Cancelled", the text "Cancelled on {date}" (use `formatDate(order.cancelledAt)`, same fallback wording as delivered), and a "Reorder" button (`size="sm"`) that calls a new `onReorder: (orderId: string) => void` prop. The four existing statuses keep their current output.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
