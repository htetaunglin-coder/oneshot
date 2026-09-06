# T01 — repeated-utilities

## Scenario

The `catalog` feature in a Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui) has two components that share the same horizontal layout.

`src/features/catalog/components/toolbar.tsx`

```tsx
import { Button } from "@/components/ui/button";

export function Toolbar({ onRefresh }: { onRefresh: () => void }) {
  return (
    <div className="flex items-center justify-between gap-2 border-b pb-3">
      <h2 className="text-lg font-semibold">Products</h2>
      <Button variant="outline" size="sm" onClick={onRefresh}>
        Refresh
      </Button>
    </div>
  );
}
```

`src/features/catalog/components/price-row.tsx`

```tsx
import { formatPrice } from "@/lib/utils";

type PriceRowProps = { label: string; amount: number };

export function PriceRow({ label, amount }: PriceRowProps) {
  return (
    <div className="flex items-center justify-between gap-2 py-1 text-sm">
      <span className="text-muted-foreground">{label}</span>
      <span className="font-medium tabular-nums">{formatPrice(amount)}</span>
    </div>
  );
}
```

`src/features/catalog/components/product-page.tsx` renders `<Toolbar />` and then a list of `<PriceRow />`.

## Change request

Add a new component `src/features/catalog/components/filter-bar.tsx` that sits directly below the toolbar. It should use the same horizontal layout as the toolbar and price row: a `<span>` reading "Filters" on the left and a `<Select>` from `@/components/ui/select` for category (`"all" | "books" | "music"`) on the right. Render it in `product-page.tsx` between the toolbar and the price rows.

Make the change and explain in one or two sentences how you structured the classes.
