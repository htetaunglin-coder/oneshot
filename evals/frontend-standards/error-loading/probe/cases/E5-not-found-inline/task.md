# E5 — not-found-inline

## Scenario

A Next.js App Router storefront (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`, `src/app`. Server components fetch their own data with `fetch`. `src/lib/env.ts` exports a validated `env` object. `src/lib/format.ts` exports `formatPrice(cents: number): string`. Public pages live under `src/app/(shop)/`; that group's layout renders the shop header and footer. The folder `src/app/(shop)/products/[slug]/` holds only `page.tsx`.

`src/features/catalog/api.ts`

```ts
import "server-only";
import { env } from "@/lib/env";

export type Product = {
  id: string;
  slug: string;
  name: string;
  description: string;
  priceCents: number;
  imageUrl: string;
  inStock: boolean;
};

type CatalogProduct = Omit<Product, "inStock"> & {
  stock: number;
};

export async function getProductBySlug(slug: string): Promise<Product | null> {
  const res = await fetch(
    `${env.CATALOG_API_URL}/products/by-slug/${encodeURIComponent(slug)}`,
    { next: { revalidate: 60, tags: [`product:${slug}`] } },
  );

  if (res.status === 404) return null;
  if (!res.ok) throw new Error(`Catalog API responded ${res.status}`);

  const item = (await res.json()) as CatalogProduct;
  const { stock, ...rest } = item;
  return { ...rest, inStock: stock > 0 };
}
```

`src/app/(shop)/products/[slug]/page.tsx`

```tsx
import type { Metadata } from "next";
import Image from "next/image";
import { AddToCartButton } from "@/features/cart/components/add-to-cart-button";
import { getProductBySlug } from "@/features/catalog/api";
import { formatPrice } from "@/lib/format";

type ProductPageProps = {
  params: Promise<{ slug: string }>;
};

export async function generateMetadata({ params }: ProductPageProps): Promise<Metadata> {
  const { slug } = await params;
  const product = await getProductBySlug(slug);
  if (!product) return { title: "Product not found" };
  return {
    title: product.name,
    description: product.description.slice(0, 160),
  };
}

export default async function ProductPage({ params }: ProductPageProps) {
  const { slug } = await params;
  const product = await getProductBySlug(slug);

  if (!product) {
    return (
      <div className="mx-auto max-w-2xl py-24 text-center">
        <p className="text-lg text-muted-foreground">Product not found</p>
      </div>
    );
  }

  return (
    <div className="mx-auto grid max-w-5xl gap-10 py-10 md:grid-cols-2">
      <Image
        src={product.imageUrl}
        alt={product.name}
        width={800}
        height={800}
        className="rounded-lg object-cover"
        priority
      />
      <div className="flex flex-col gap-4">
        <h1 className="text-3xl font-semibold">{product.name}</h1>
        <p className="text-2xl">{formatPrice(product.priceCents)}</p>
        <p className="text-muted-foreground">{product.description}</p>
        <AddToCartButton productId={product.id} disabled={!product.inStock} />
        {!product.inStock && (
          <p className="text-sm text-muted-foreground">Currently out of stock.</p>
        )}
      </div>
    </div>
  );
}
```

`src/app/not-found.tsx`

```tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";

const POPULAR = [
  { href: "/collections/new", label: "New arrivals" },
  { href: "/collections/bestsellers", label: "Bestsellers" },
  { href: "/collections/sale", label: "Sale" },
];

export default function NotFound() {
  return (
    <div className="mx-auto flex max-w-md flex-col items-center gap-4 py-24 text-center">
      <p className="text-sm font-medium text-muted-foreground">404</p>
      <h1 className="text-3xl font-semibold">Page not found</h1>
      <p className="text-muted-foreground">
        The page you are looking for does not exist or has moved.
      </p>
      <Button asChild>
        <Link href="/">Back to the shop</Link>
      </Button>
      <ul className="mt-4 flex gap-4 text-sm">
        {POPULAR.map((item) => (
          <li key={item.href}>
            <Link href={item.href} className="underline">
              {item.label}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

`AddToCartButton` is a client component under `src/features/cart/components/`.

## Change request

"Bug: Search Console lists a few hundred product URLs for products we deleted last quarter and they are still indexed; analytics shows those URLs coming back as 200. A deleted product should behave like any other missing page on the site, with the same look as the rest of the site's missing pages."

Make the change and explain in one or two sentences how you decided where the failure or waiting state is handled.
