# A2 — cleanup-deletes-context

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The inventory feature is about to get a multi-warehouse split-shipment change, and this file is where it will land. Backorders are on the roadmap for next quarter (issue #612, "Backorder flow"), not built yet. Tenants are small businesses; the largest has nine warehouses.

`src/features/inventory/lib/allocate.ts`

```ts
export type Warehouse = {
  id: string;
  name: string;
  stock: Record<string, number>;
};

export type OrderLine = {
  id: string;
  sku: string;
  quantity: number;
};

export type Allocation = {
  lineId: string;
  warehouseId: string;
  sku: string;
  quantity: number;
};

type AllocateInput = {
  lines: OrderLine[];
  warehouses: Warehouse[];
};

export type AllocateResult = {
  allocations: Allocation[];
  unallocated: OrderLine[];
};

export function allocate({ lines, warehouses }: AllocateInput): AllocateResult {
  const allocations: Allocation[] = [];
  const unallocated: OrderLine[] = [];
  const remaining = structuredClone(warehouses);

  // loop over lines
  for (const line of lines) {
    const warehouse = findWarehouseWithStock(remaining, line.sku, line.quantity);

    if (!warehouse) {
      // TODO: handle backorders
      unallocated.push(line);
      continue;
    }

    allocations.push({
      lineId: line.id,
      warehouseId: warehouse.id,
      sku: line.sku,
      quantity: line.quantity,
    });
    warehouse.stock[line.sku] -= line.quantity;
  }

  // return result
  return { allocations, unallocated };
}

// Linear scan on purpose: there are never more than 12 warehouses per tenant
// and this runs once per order, so a Map costs more than it saves.
function findWarehouseWithStock(
  warehouses: Warehouse[],
  sku: string,
  quantity: number,
): Warehouse | undefined {
  for (const warehouse of warehouses) {
    const available = warehouse.stock[sku] ?? 0;
    if (available >= quantity) {
      return warehouse;
    }
  }
  return undefined;
}
```

## Change request

"This file has accumulated some cruft; tidy it up before we build on it. No behaviour changes."

Make the change and explain in one or two sentences how you decided what to comment.
