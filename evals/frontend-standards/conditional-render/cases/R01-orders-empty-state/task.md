# R01 — orders-empty-state

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/orders/hooks/use-orders.ts` (signature only)

```ts
export function useOrders(customerId: string): {
  orders: Order[];
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
};
```

`src/features/orders/components/order-history.tsx`

```tsx
"use client";

import { Spinner } from "@/components/ui/spinner";
import { useOrders } from "../hooks/use-orders";
import { OrderRow } from "./order-row";
import { OrdersErrorView } from "./orders-error-view";

type OrderHistoryProps = {
  customerId: string;
};

export function OrderHistory({ customerId }: OrderHistoryProps) {
  const { orders, isLoading, error, refetch } = useOrders(customerId);

  return isLoading ? (
    <Spinner className="mx-auto my-8" />
  ) : error ? (
    <OrdersErrorView message={error.message} onRetry={refetch} />
  ) : (
    <ul className="divide-y">
      {orders.map((order) => (
        <OrderRow key={order.id} order={order} />
      ))}
    </ul>
  );
}
```

`src/components/ui/empty-state.tsx` exists and exports `EmptyState` with props `{ title: string; description?: string; action?: React.ReactNode }`.

## Change request

Customers with no orders currently see a blank area. When loading has finished, there is no error, and `orders` is empty, show an `EmptyState` with the title "No orders yet", the description "Your orders will appear here after checkout.", and a "Browse products" button that links to `/shop` (shadcn `Button` with `asChild` around a Next `Link`). Loading and error behaviour stay as they are.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
