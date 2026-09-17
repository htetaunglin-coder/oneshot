# M08 — premium-tier-check

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/orders/lib/types.ts`

```ts
export type CustomerTier = "bronze" | "silver" | "gold" | "platinum";

export type Customer = {
  id: string;
  name: string;
  tier: CustomerTier;
};

export type Order = {
  id: string;
  customer: Customer;
  subtotalCents: number;
  shippingCents: number;
};
```

`src/features/orders/components/order-summary.tsx`

```tsx
import { formatPrice } from "@/lib/format";
import type { Order } from "@/features/orders/lib/types";
import { ReturnPolicy } from "@/features/orders/components/return-policy";

export function OrderSummary({ order }: { order: Order }) {
  const freeShipping =
    order.customer.tier === "gold" || order.customer.tier === "platinum";
  const shippingCents = freeShipping ? 0 : order.shippingCents;

  return (
    <div className="rounded-lg border p-4">
      <dl className="grid grid-cols-2 gap-1 text-sm">
        <dt>Subtotal</dt>
        <dd className="text-right">{formatPrice(order.subtotalCents)}</dd>
        <dt>Shipping</dt>
        <dd className="text-right">{freeShipping ? "Free" : formatPrice(shippingCents)}</dd>
        <dt className="font-medium">Total</dt>
        <dd className="text-right font-medium">
          {formatPrice(order.subtotalCents + shippingCents)}
        </dd>
      </dl>
      <ReturnPolicy />
    </div>
  );
}
```

`src/features/orders/components/support-banner.tsx`

```tsx
import type { Customer } from "@/features/orders/lib/types";

export function SupportBanner({ customer }: { customer: Customer }) {
  if (customer.tier === "gold" || customer.tier === "platinum") {
    return <p className="text-sm">Priority support: we reply within one hour.</p>;
  }

  return <p className="text-sm">Support replies within one business day.</p>;
}
```

`src/features/orders/components/return-policy.tsx`

```tsx
export function ReturnPolicy() {
  return (
    <p className="mt-3 text-sm text-muted-foreground">
      Returns accepted within 30 days of delivery.
    </p>
  );
}
```

## Change request

Gold and platinum customers now get 60 days to return an order instead of 30. `ReturnPolicy` needs to know the customer to pick the right line; wire it up from `OrderSummary`.

Make the change and explain in one or two sentences how you decided which values get a name.
