# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui`, `cn` is exported from `src/lib/utils`, theme and `@theme` values live in `src/app/globals.css`.

### `src/features/onboarding/components/styles.ts`

```ts
export const focusRing =
  "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2";
```

### `src/features/onboarding/components/workspace-name-step.tsx`

```tsx
"use client";

import { cn } from "@/lib/utils";
import { focusRing } from "./styles";

export function WorkspaceNameStep({
  value,
  onChange,
}: {
  value: string;
  onChange: (next: string) => void;
}) {
  return (
    <label className="grid gap-2">
      <span className="text-sm font-medium">Workspace name</span>
      <input
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder="Acme Inc."
        className={cn(
          "h-10 rounded-md border bg-background px-3 text-sm placeholder:text-muted-foreground",
          focusRing,
        )}
      />
    </label>
  );
}
```

### `src/features/onboarding/components/plan-option.tsx`

```tsx
"use client";

import { cn } from "@/lib/utils";
import { focusRing } from "./styles";

export function PlanOption({
  name,
  price,
  selected,
  onSelect,
}: {
  name: string;
  price: string;
  selected: boolean;
  onSelect: () => void;
}) {
  return (
    <div
      role="button"
      tabIndex={0}
      aria-pressed={selected}
      onClick={onSelect}
      onKeyDown={(e) => (e.key === "Enter" || e.key === " ") && onSelect()}
      className={cn(
        "flex cursor-pointer items-center justify-between rounded-lg border p-4",
        selected && "border-primary bg-primary/5",
        focusRing,
      )}
    >
      <span className="font-medium">{name}</span>
      <span className="text-sm text-muted-foreground">{price}</span>
    </div>
  );
}
```

### `src/features/onboarding/components/skip-link.tsx`

```tsx
import Link from "next/link";
import { cn } from "@/lib/utils";
import { focusRing } from "./styles";

export function SkipLink() {
  return (
    <Link
      href="/dashboard"
      className={cn("rounded-sm text-sm text-muted-foreground underline-offset-4 hover:underline", focusRing)}
    >
      Skip for now
    </Link>
  );
}
```

## Change request

On the dark onboarding background the ring offset renders white, which looks wrong. Add `focus-visible:ring-offset-background` so the offset picks up the page colour. All three elements should get it.

Make the change and explain in one or two sentences how you structured the classes.
