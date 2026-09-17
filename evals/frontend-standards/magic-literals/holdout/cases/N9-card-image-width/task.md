# N9 — card-image-width

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The listings feature renders property cards with a cover image, plus a skeleton and a form validator.

`src/features/listings/components/listing-card.tsx`

```tsx
import Image from "next/image";
import { Card, CardContent } from "@/components/ui/card";
import { formatCents } from "@/lib/format";
import type { Listing } from "@/features/listings/lib/types";

export function ListingCard({ listing }: { listing: Listing }) {
  return (
    <Card className="overflow-hidden">
      <Image src={listing.coverUrl} alt={listing.title} width={640} height={360} className="w-full object-cover" />
      <CardContent className="p-4">
        <h3 className="font-medium">{listing.title}</h3>
        <p className="text-muted-foreground text-sm">{formatCents(listing.priceCents)}</p>
      </CardContent>
    </Card>
  );
}
```

`src/features/listings/components/featured-listing-card.tsx`

```tsx
import Image from "next/image";
import { Badge } from "@/components/ui/badge";
import { Card, CardContent } from "@/components/ui/card";
import type { Listing } from "@/features/listings/lib/types";

export function FeaturedListingCard({ listing }: { listing: Listing }) {
  return (
    <Card className="overflow-hidden border-primary">
      <div className="relative">
        <Image src={listing.coverUrl} alt={listing.title} width={640} height={360} priority className="w-full object-cover" />
        <Badge className="absolute top-2 left-2">Featured</Badge>
      </div>
      <CardContent className="p-4">
        <h3 className="font-medium">{listing.title}</h3>
      </CardContent>
    </Card>
  );
}
```

`src/features/listings/components/listing-card-skeleton.tsx`

```tsx
import { Skeleton } from "@/components/ui/skeleton";

export function ListingCardSkeleton() {
  return (
    <div className="w-full max-w-[640px] overflow-hidden rounded-xl border">
      <Skeleton className="aspect-video w-full" />
      <div className="space-y-2 p-4">
        <Skeleton className="h-4 w-3/4" />
        <Skeleton className="h-4 w-1/3" />
      </div>
    </div>
  );
}
```

`src/features/listings/lib/validate-listing.ts`

```ts
export function validateDescription(description: string): string | null {
  if (description.trim().length === 0) return "Description is required";
  if (description.length > 640) return "Description must be 640 characters or fewer";
  return null;
}
```

## Change request

"Design bumped the card art: the card images should be 720 wide now, same 16:9 ratio. Both the regular and the featured card. Make sure the loading skeleton still matches the card."

Make the change and explain in one or two sentences how you decided which values get a name.
