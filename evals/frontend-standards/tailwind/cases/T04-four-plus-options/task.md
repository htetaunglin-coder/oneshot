# T04 — four-plus-options

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The shared badge lives in the UI folder.

`src/components/ui/badge.tsx`

```tsx
import * as React from "react";
import { cn } from "@/lib/utils";

type BadgeVariant = "default" | "secondary" | "outline";

const variantClasses: Record<BadgeVariant, string> = {
  default: "border-transparent bg-primary text-primary-foreground",
  secondary: "border-transparent bg-secondary text-secondary-foreground",
  outline: "border-border text-foreground",
};

export type BadgeProps = React.HTMLAttributes<HTMLSpanElement> & {
  variant?: BadgeVariant;
};

export function Badge({ variant = "default", className, ...props }: BadgeProps) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-medium",
        variantClasses[variant],
        className
      )}
      {...props}
    />
  );
}
```

`class-variance-authority` is already a dependency (`src/components/ui/button.tsx` uses it).

Usage in `src/features/projects/components/project-card.tsx`:

```tsx
<Badge variant="secondary">{project.status}</Badge>
```

## Change request

Add a `size` prop with two options: `sm` (`px-2 py-0 text-[11px]`) and `md` (the current `px-2.5 py-0.5 text-xs`). `md` is the default. Existing call sites must keep compiling without changes.

Make the change and explain in one or two sentences how you structured the classes.
