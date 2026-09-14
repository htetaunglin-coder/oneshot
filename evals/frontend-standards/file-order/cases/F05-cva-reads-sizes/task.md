# F05 — cva-reads-sizes

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui, `class-variance-authority` installed); features live under `src/features/<name>/components|hooks|lib`.

`src/features/inventory/lib/types.ts`

```ts
export type StockItem = {
  sku: string;
  quantity: number;
  reorderPoint: number;
};
```

`src/features/inventory/components/stock-level.tsx`

```tsx
import { cn } from "@/lib/utils";
import type { StockItem } from "@/features/inventory/lib/types";

const SIZES = {
  sm: "h-5 px-1.5 text-xs",
  md: "h-6 px-2 text-sm",
} as const;

type Size = keyof typeof SIZES;

type StockLevelProps = {
  item: StockItem;
  size?: Size;
};

export function StockLevel({ item, size = "md" }: StockLevelProps) {
  return (
    <div className="flex items-center gap-2">
      <QuantityBadge quantity={item.quantity} size={size} />
      <ReorderBadge item={item} size={size} />
    </div>
  );
}

function QuantityBadge({ quantity, size }: { quantity: number; size: Size }) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full border font-medium",
        SIZES[size],
        quantity === 0 ? "border-red-300 bg-red-50 text-red-700" : "border-border",
      )}
    >
      {quantity} in stock
    </span>
  );
}

function ReorderBadge({ item, size }: { item: StockItem; size: Size }) {
  if (item.quantity > item.reorderPoint) return null;

  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full border font-medium",
        SIZES[size],
        "border-amber-300 bg-amber-50 text-amber-800",
      )}
    >
      Reorder
    </span>
  );
}
```

## Change request

Both badges repeat the base classes and pick their colour ad hoc. Replace that with one `cva` definition named `badgeVariants` with a `size` variant (`sm`, `md`; reuse `SIZES` so the two cannot drift) and a `tone` variant (`neutral`, `danger`, `warning`). Both badges call `badgeVariants({ size, tone })` and stop spelling out the shared classes. No visual change.

Make the change and explain in one or two sentences how you decided where each declaration goes.
