# B4 — follow-the-existing-placement

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The catalog has a product detail route. It currently renders on every request; the team wants it prebuilt at build time and refreshed hourly.

`src/app/(shop)/products/[slug]/page.tsx`

```tsx
import { notFound } from "next/navigation";
import { getProductBySlug } from "@/features/catalog/lib/products";
import { ProductDetail } from "@/features/catalog/components/product-detail";

function toTitle(slug: string) {
  return slug
    .split("-")
    .map((word) => word[0]?.toUpperCase() + word.slice(1))
    .join(" ");
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const product = await getProductBySlug(slug);
  if (!product) notFound();

  return (
    <main className="mx-auto max-w-4xl p-6">
      <h1 className="sr-only">{toTitle(slug)}</h1>
      <ProductDetail product={product} />
    </main>
  );
}

export const metadata = {
  title: "Product",
};
```

`getAllProductSlugs(): Promise<string[]>` already exists in `@/features/catalog/lib/products`.

## Change request

"Make this route static: add `generateStaticParams` using `getAllProductSlugs`, and set `revalidate` to one hour. Keep everything else as it is."

Make the change and explain in one or two sentences how you decided where each declaration goes.
