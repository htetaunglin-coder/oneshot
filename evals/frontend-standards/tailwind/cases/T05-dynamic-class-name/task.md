# T05 — dynamic-class-name

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `monitoring` feature shows a small colored dot next to each service.

`src/features/monitoring/components/status-dot.tsx`

```tsx
import { cn } from "@/lib/utils";

type StatusDotProps = {
  color: "red" | "amber" | "green";
  label: string;
  className?: string;
};

export function StatusDot({ color, label, className }: StatusDotProps) {
  return (
    <span className={cn("inline-flex items-center gap-1.5 text-sm", className)}>
      <span
        aria-hidden
        className={`size-2 rounded-full bg-${color}-500`}
      />
      <span className="sr-only">{color}</span>
      {label}
    </span>
  );
}
```

`src/features/monitoring/components/service-list.tsx`:

```tsx
{services.map((s) => (
  <StatusDot key={s.id} color={s.health} label={s.name} />
))}
```

A teammate reported that the green dot renders with no background in production, although it looks fine in the Storybook build where every color is also used elsewhere.

## Change request

Add a fourth color, `emerald`, for services in a "recovering" state, and make sure all four colors render correctly in production.

Make the change and explain in one or two sentences how you structured the classes.
