# M04 — sort-add-trending

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/catalog/lib/types.ts`

```ts
export type Product = {
  id: string;
  name: string;
  createdAt: string;
  salesCount: number;
  trendScore: number;
};
```

`src/features/catalog/lib/sort-products.ts`

```ts
import type { Product } from "@/features/catalog/lib/types";

export function sortProducts(products: Product[], sort: string): Product[] {
  const copy = [...products];

  switch (sort) {
    case "newest":
      return copy.sort((a, b) => b.createdAt.localeCompare(a.createdAt));
    case "oldest":
      return copy.sort((a, b) => a.createdAt.localeCompare(b.createdAt));
    case "popular":
      return copy.sort((a, b) => b.salesCount - a.salesCount);
    default:
      return copy;
  }
}
```

`src/features/catalog/components/product-sort.tsx`

```tsx
"use client";

import { useRouter, useSearchParams } from "next/navigation";

type Sort = "newest" | "oldest" | "popular";

export function ProductSort() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const sort = (searchParams.get("sort") ?? "newest") as Sort;

  function onChange(event: React.ChangeEvent<HTMLSelectElement>) {
    const next = new URLSearchParams(searchParams);
    next.set("sort", event.target.value);
    router.replace(`?${next.toString()}`);
  }

  return (
    <select
      value={sort}
      onChange={onChange}
      aria-label="Sort products"
      className="rounded-md border bg-background px-2 py-1 text-sm"
    >
      <option value="newest">Newest</option>
      <option value="oldest">Oldest</option>
      <option value="popular">Most popular</option>
    </select>
  );
}
```

## Change request

Add a fourth sort, "Trending", that orders by `trendScore` descending. It must appear in the dropdown and work through the URL like the other three.

Make the change and explain in one or two sentences how you decided which values get a name.
