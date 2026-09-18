# P5 — filter-state-location

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`. Server data is fetched with TanStack Query (`QueryClientProvider` in the root layout). The public catalog page lists products with a category filter and a sort control.

`src/features/catalog/types.ts`

```ts
export const CATEGORIES = ["shoes", "apparel", "accessories"] as const;
export type Category = (typeof CATEGORIES)[number];

export type SortKey = "newest" | "price-asc" | "price-desc";

export type Product = {
  id: string;
  slug: string;
  name: string;
  category: Category;
  priceCents: number;
  imageUrl: string;
  createdAt: string;
};
```

`src/features/catalog/hooks/use-products.ts` (`fetchProducts(filter: ProductsFilter): Promise<Product[]>` lives in `src/features/catalog/api.ts`)

```ts
import { useQuery } from "@tanstack/react-query";
import { fetchProducts } from "@/features/catalog/api";
import type { Category, SortKey } from "@/features/catalog/types";

export type ProductsFilter = {
  category: Category | "all";
  sort: SortKey;
};

export function useProducts(filter: ProductsFilter) {
  return useQuery({
    queryKey: ["products", filter] as const,
    queryFn: () => fetchProducts(filter),
    placeholderData: (previous) => previous,
  });
}
```

`src/features/catalog/components/product-list.tsx` (`ProductCard` and `ProductGridSkeleton` exist next to it)

```tsx
"use client";

import { useState } from "react";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { ProductCard } from "@/features/catalog/components/product-card";
import { ProductGridSkeleton } from "@/features/catalog/components/product-grid-skeleton";
import { useProducts } from "@/features/catalog/hooks/use-products";
import { CATEGORIES, type Category, type SortKey } from "@/features/catalog/types";

const SORT_OPTIONS: { value: SortKey; label: string }[] = [
  { value: "newest", label: "Newest" },
  { value: "price-asc", label: "Price: low to high" },
  { value: "price-desc", label: "Price: high to low" },
];

export function ProductList() {
  const [category, setCategory] = useState<Category | "all">("all");
  const [sort, setSort] = useState<SortKey>("newest");
  const { data: products, isPending } = useProducts({ category, sort });

  return (
    <section className="flex flex-col gap-4">
      <div className="flex flex-wrap gap-2">
        <Select value={category} onValueChange={(value) => setCategory(value as Category | "all")}>
          <SelectTrigger className="w-44" aria-label="Category">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">All categories</SelectItem>
            {CATEGORIES.map((c) => (
              <SelectItem key={c} value={c} className="capitalize">
                {c}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
        <Select value={sort} onValueChange={(value) => setSort(value as SortKey)}>
          <SelectTrigger className="w-52" aria-label="Sort by">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            {SORT_OPTIONS.map((option) => (
              <SelectItem key={option.value} value={option.value}>
                {option.label}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
      </div>
      {isPending || !products ? (
        <ProductGridSkeleton />
      ) : (
        <ul className="grid grid-cols-2 gap-4 md:grid-cols-3 lg:grid-cols-4">
          {products.map((product) => (
            <li key={product.id}>
              <ProductCard product={product} />
            </li>
          ))}
        </ul>
      )}
    </section>
  );
}
```

`src/app/(shop)/products/page.tsx`

```tsx
import { ProductList } from "@/features/catalog/components/product-list";

export const metadata = { title: "Products" };

export default function ProductsPage() {
  return (
    <div className="mx-auto flex max-w-6xl flex-col gap-6 px-4 py-8">
      <h1 className="text-2xl font-semibold">Products</h1>
      <ProductList />
    </div>
  );
}
```

## Change request

"Filters reset. Pick Shoes and Price: low to high on `/products`, hit refresh, and it is back to All categories / Newest. Same when I copy the page address from the browser and send it to a colleague: they land on the default list. The chosen category and sort should survive a refresh and come through when the link is shared."

Make the change and explain in one or two sentences how you decided where the value lives.
