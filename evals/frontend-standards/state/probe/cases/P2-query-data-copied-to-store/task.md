# P2 — query-data-copied-to-store

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`. Server data is fetched with TanStack Query (the `QueryClientProvider` sits in the root layout); shared client values use Zustand. The orders feature has a table, a slide-over detail panel, and a sidebar nav item.

`src/features/orders/types.ts`

```ts
export type OrderStatus = "open" | "fulfilled" | "cancelled";

export type Order = {
  id: string;
  number: string;
  customer: string;
  totalCents: number;
  status: OrderStatus;
  placedAt: string;
};
```

`src/features/orders/hooks/use-orders.ts` (`fetchOrders(): Promise<Order[]>` lives in `src/features/orders/api.ts`)

```ts
import { useQuery } from "@tanstack/react-query";
import { fetchOrders } from "@/features/orders/api";
import type { Order } from "@/features/orders/types";

export const ordersQueryKey = ["orders"] as const;

export function useOrders() {
  return useQuery<Order[]>({
    queryKey: ordersQueryKey,
    queryFn: fetchOrders,
    staleTime: 30_000,
  });
}
```

`src/features/orders/store.ts`

```ts
import { create } from "zustand";
import type { Order } from "@/features/orders/types";

type OrdersState = {
  orders: Order[];
  selectedId: string | null;
  setOrders: (orders: Order[]) => void;
  select: (id: string | null) => void;
};

export const useOrdersStore = create<OrdersState>()((set) => ({
  orders: [],
  selectedId: null,
  setOrders: (orders) => set({ orders }),
  select: (id) => set({ selectedId: id }),
}));
```

`src/features/orders/components/orders-table.tsx`

```tsx
"use client";

import { useEffect } from "react";
import { Skeleton } from "@/components/ui/skeleton";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { OrderStatusBadge } from "@/features/orders/components/order-status-badge";
import { useOrders } from "@/features/orders/hooks/use-orders";
import { useOrdersStore } from "@/features/orders/store";
import { formatDate, formatMoney } from "@/lib/format";
import { cn } from "@/lib/utils";

export function OrdersTable() {
  const { data, isPending, isError } = useOrders();
  const setOrders = useOrdersStore((s) => s.setOrders);
  const selectedId = useOrdersStore((s) => s.selectedId);
  const select = useOrdersStore((s) => s.select);

  // Keep the store in sync with the query so the rest of the feature
  // (detail panel, bulk actions) reads orders from one place.
  useEffect(() => {
    if (data) setOrders(data);
  }, [data, setOrders]);

  if (isPending) return <Skeleton className="h-72 w-full" />;
  if (isError) return <p className="text-destructive text-sm">Could not load orders.</p>;

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Order</TableHead>
          <TableHead>Customer</TableHead>
          <TableHead>Status</TableHead>
          <TableHead className="text-right">Total</TableHead>
          <TableHead className="text-right">Placed</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.map((order) => (
          <TableRow
            key={order.id}
            onClick={() => select(order.id)}
            className={cn("cursor-pointer", order.id === selectedId && "bg-muted")}
          >
            <TableCell className="font-medium">#{order.number}</TableCell>
            <TableCell>{order.customer}</TableCell>
            <TableCell>
              <OrderStatusBadge status={order.status} />
            </TableCell>
            <TableCell className="text-right">{formatMoney(order.totalCents)}</TableCell>
            <TableCell className="text-right">{formatDate(order.placedAt)}</TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

`src/features/orders/components/order-detail-panel.tsx`

```tsx
"use client";

import { Sheet, SheetContent, SheetHeader, SheetTitle } from "@/components/ui/sheet";
import { OrderStatusBadge } from "@/features/orders/components/order-status-badge";
import { useOrdersStore } from "@/features/orders/store";
import { formatDate, formatMoney } from "@/lib/format";

export function OrderDetailPanel() {
  const selectedId = useOrdersStore((s) => s.selectedId);
  const select = useOrdersStore((s) => s.select);
  const order = useOrdersStore((s) => s.orders.find((o) => o.id === selectedId));

  function handleOpenChange(open: boolean) {
    if (!open) select(null);
  }

  return (
    <Sheet open={order !== undefined} onOpenChange={handleOpenChange}>
      <SheetContent>
        {order && (
          <>
            <SheetHeader>
              <SheetTitle>Order #{order.number}</SheetTitle>
            </SheetHeader>
            <dl className="mt-4 grid grid-cols-2 gap-y-2 text-sm">
              <dt className="text-muted-foreground">Customer</dt>
              <dd>{order.customer}</dd>
              <dt className="text-muted-foreground">Status</dt>
              <dd>
                <OrderStatusBadge status={order.status} />
              </dd>
              <dt className="text-muted-foreground">Total</dt>
              <dd>{formatMoney(order.totalCents)}</dd>
              <dt className="text-muted-foreground">Placed</dt>
              <dd>{formatDate(order.placedAt)}</dd>
            </dl>
          </>
        )}
      </SheetContent>
    </Sheet>
  );
}
```

`src/features/orders/components/orders-nav-item.tsx`

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";
import { Package } from "lucide-react";
import { cn } from "@/lib/utils";

export function OrdersNavItem() {
  const pathname = usePathname();
  const active = pathname === "/orders" || pathname.startsWith("/orders/");

  return (
    <Link
      href="/orders"
      aria-current={active ? "page" : undefined}
      className={cn(
        "text-muted-foreground hover:bg-accent/50 flex items-center gap-2 rounded-md px-3 py-2 text-sm",
        active && "bg-accent text-accent-foreground",
      )}
    >
      <Package className="size-4" />
      <span className="flex-1">Orders</span>
    </Link>
  );
}
```

`src/app/(app)/orders/page.tsx` renders `<OrdersTable />` and `<OrderDetailPanel />` together. `OrdersNavItem` is one of the items in `AppSidebar` (`src/components/app-sidebar.tsx`), which the `(app)` layout renders on every signed-in page. `Badge` from `@/components/ui/badge` is already installed.

## Change request

"Sidebar: show how many open orders there are on the Orders item. Right-aligned `Badge` (`variant="secondary"`) with the count of orders whose `status === "open"`. No badge at all when the count is 0. The sidebar is in the `(app)` layout, so this shows on every signed-in page."

Make the change and explain in one or two sentences how you decided where the value lives.
