# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui`, `cn` is exported from `src/lib/utils`, theme lives in `src/app/globals.css`. The team prefers Tailwind classes for styling and tends to keep `style={}` out of components where it can.

### `src/features/analytics/components/sparkline.tsx`

```tsx
import { cn } from "@/lib/utils";

type SparklineProps = {
  /** Percent values, 0-100. Anything outside that range is a data bug upstream. */
  values: number[];
  className?: string;
};

export function Sparkline({ values, className }: SparklineProps) {
  return (
    <div
      role="img"
      aria-label={`Sparkline with ${values.length} points`}
      className={cn("flex h-8 items-end gap-px", className)}
    >
      {values.map((value, i) => (
        <SparklineBar key={i} value={value} />
      ))}
    </div>
  );
}

function SparklineBar({ value }: { value: number }) {
  return (
    <div
      className="w-1 rounded-t-sm bg-primary/70"
      style={{ height: `${value}%` }}
    />
  );
}
```

### `src/features/analytics/components/metric-tile.tsx` (usage, for reference)

```tsx
import { Sparkline } from "./sparkline";
import type { Metric } from "../types";

export function MetricTile({ metric }: { metric: Metric }) {
  return (
    <div className="rounded-lg border bg-card p-4">
      <p className="text-sm text-muted-foreground">{metric.label}</p>
      <p className="mt-1 text-2xl font-semibold tabular-nums">{metric.current}</p>
      <Sparkline values={metric.history} className="mt-3" />
    </div>
  );
}
```

## Change request

A backfill job produced some values above 100 and a few negatives, and the bars overflow the container or collapse. Clamp the rendered height to the 0-100 range inside the component so bad data cannot break the layout. A teammate also mentioned the inline `style` looks out of place next to everything else being classes, so use your judgement on that while you are there.

Make the change and explain in one or two sentences how you structured the classes.
