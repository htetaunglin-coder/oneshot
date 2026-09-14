# B5 — append-to-export-list

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The shared UI folder holds the Tabs primitive, a thin wrapper over Radix. Other features import from it as `import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs"`.

`src/components/ui/tabs.tsx`

```tsx
"use client";

import * as TabsPrimitive from "@radix-ui/react-tabs";
import { cn } from "@/lib/utils";

function Tabs({ className, ...props }: React.ComponentProps<typeof TabsPrimitive.Root>) {
  return (
    <TabsPrimitive.Root
      data-slot="tabs"
      className={cn("flex flex-col gap-2", className)}
      {...props}
    />
  );
}

function TabsList({ className, ...props }: React.ComponentProps<typeof TabsPrimitive.List>) {
  return (
    <TabsPrimitive.List
      data-slot="tabs-list"
      className={cn(
        "relative inline-flex h-9 w-fit items-center rounded-lg bg-muted p-1 text-muted-foreground",
        className,
      )}
      {...props}
    />
  );
}

function TabsTrigger({ className, ...props }: React.ComponentProps<typeof TabsPrimitive.Trigger>) {
  return (
    <TabsPrimitive.Trigger
      data-slot="tabs-trigger"
      className={cn(
        "inline-flex flex-1 items-center justify-center rounded-md px-2 py-1 text-sm font-medium transition-colors data-[state=active]:text-foreground",
        className,
      )}
      {...props}
    />
  );
}

function TabsContent({ className, ...props }: React.ComponentProps<typeof TabsPrimitive.Content>) {
  return (
    <TabsPrimitive.Content
      data-slot="tabs-content"
      className={cn("flex-1 outline-none", className)}
      {...props}
    />
  );
}

export { Tabs, TabsList, TabsTrigger, TabsContent };
```

## Change request

"Add a `TabsIndicator`: a `div` with `data-slot="tabs-indicator"` that renders inside `TabsList` as the sliding background behind the active trigger (absolute, `rounded-md bg-background shadow-sm`, plus a `transition-transform`). It takes `className` and the rest of a `div`'s props. Positioning logic comes in a later PR; this one only adds the piece and makes it importable alongside the others."

Make the change and explain in one or two sentences how you decided where each declaration goes.
