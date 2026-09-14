# F08 — export-list-card-footer

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Shared primitives live in `src/components/ui`.

`src/components/ui/card.tsx`

```tsx
import * as React from "react";
import { cn } from "@/lib/utils";

function Card({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card"
      className={cn("rounded-xl border bg-card text-card-foreground shadow-sm", className)}
      {...props}
    />
  );
}

function CardHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-header"
      className={cn("flex flex-col gap-1.5 px-6 pt-6", className)}
      {...props}
    />
  );
}

function CardContent({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-content" className={cn("px-6 py-4", className)} {...props} />;
}

export { Card, CardHeader, CardContent };
```

`src/app/(app)/settings/billing/page.tsx` (excerpt)

```tsx
import { Card, CardContent, CardHeader } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

export default function BillingSettingsPage() {
  return (
    <Card>
      <CardHeader>
        <h2 className="font-semibold">Payment method</h2>
      </CardHeader>
      <CardContent>
        <p className="text-sm">Visa ending in 4242</p>
      </CardContent>
      <div className="flex items-center gap-2 px-6 pb-6">
        <Button variant="outline">Remove</Button>
        <Button>Update card</Button>
      </div>
    </Card>
  );
}
```

## Change request

The billing page hand-rolls the action row that closes the card, and two more pages are about to do the same. Add a `CardFooter` primitive to `card.tsx` (a `div` with `data-slot="card-footer"`, base classes `flex items-center gap-2 px-6 pb-6`, accepting `React.ComponentProps<"div">` like the others) and use it on the billing page in place of the raw `div`.

Make the change and explain in one or two sentences how you decided where each declaration goes.
