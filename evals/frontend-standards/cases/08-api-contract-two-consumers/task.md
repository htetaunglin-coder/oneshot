# Case 08 — api-contract-two-consumers

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/features/orders/api/orders.ts`

```ts
export type Order = {
  id: string;
  placedAt: string;
  totalCents: number;
  itemCount: number;
};

export async function getOrders(customerId: string): Promise<Order[]> {
  const res = await fetch(`/api/customers/${customerId}/orders`, {
    next: { revalidate: 30 },
  });
  if (!res.ok) throw new Error(`orders: ${res.status}`);
  return (await res.json()) as Order[];
}
```

### `src/features/orders/components/order-list.tsx`

```tsx
import { getOrders, type Order } from "../api/orders";
import { formatPrice } from "@/lib/format-price";

function OrderRow({ order }: { order: Order }) {
  return (
    <li className="flex justify-between py-2">
      <span>{new Date(order.placedAt).toLocaleDateString()}</span>
      <span>{order.itemCount} items</span>
      <span>{formatPrice(order.totalCents)}</span>
    </li>
  );
}

export async function OrderList({ customerId }: { customerId: string }) {
  const orders = await getOrders(customerId);
  return (
    <ul className="divide-y divide-zinc-800">
      {orders.map((o) => (
        <OrderRow key={o.id} order={o} />
      ))}
    </ul>
  );
}
```

The backend now returns a `status` field on each order with the value `"paid"` or
`"refunded"`. It is not yet reflected anywhere in the frontend.

## Task

Add a new `OrderBadge` component in
`src/features/orders/components/order-badge.tsx` that takes an order and renders
a small pill — green for paid, grey for refunded — and render it inside `OrderRow`.
Make whatever type changes are needed. Decide where the new/changed code lives and
explain in one or two sentences.
