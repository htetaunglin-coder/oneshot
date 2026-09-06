# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui` (`badge.tsx` and `button.tsx` are present and follow the standard shadcn structure), `cn` is exported from `src/lib/utils`, theme lives in `src/app/globals.css`.

### `src/features/inbox/components/chip.tsx`

```tsx
import type { ReactNode } from "react";
import { cn } from "@/lib/utils";

type ChipProps = {
  tone?: "neutral" | "accent";
  size?: "sm" | "md";
  className?: string;
  children: ReactNode;
};

export function Chip({ tone = "neutral", size = "sm", className, children }: ChipProps) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full font-medium whitespace-nowrap",
        tone === "accent"
          ? size === "md"
            ? "bg-primary text-primary-foreground px-3 py-1 text-sm"
            : "bg-primary text-primary-foreground px-2 py-0.5 text-xs"
          : size === "md"
            ? "bg-muted text-muted-foreground px-3 py-1 text-sm"
            : "bg-muted text-muted-foreground px-2 py-0.5 text-xs",
        className,
      )}
    >
      {children}
    </span>
  );
}
```

### `src/features/inbox/components/thread-row.tsx` (usage, for reference)

```tsx
import { Chip } from "./chip";
import type { Thread } from "../types";

export function ThreadRow({ thread }: { thread: Thread }) {
  return (
    <li className="flex items-center gap-3 border-b py-2">
      <span className="flex-1 truncate text-sm">{thread.subject}</span>
      {thread.unread > 0 ? <Chip tone="accent">{thread.unread}</Chip> : null}
      {thread.labels.map((l) => (
        <Chip key={l} size="md">
          {l}
        </Chip>
      ))}
    </li>
  );
}
```

## Change request

The nested ternary in `Chip` is hard to read and a reviewer flagged it. Rewrite the class handling so it is obvious at a glance what each `tone` and `size` does. Keep the props and the rendered classes the same. It is fine to follow whatever pattern `src/components/ui/badge.tsx` uses if that helps.

Make the change and explain in one or two sentences how you structured the classes.
