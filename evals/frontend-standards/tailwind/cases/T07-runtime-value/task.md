# T07 — runtime-value

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `uploads` feature shows a progress bar.

`src/features/uploads/components/progress-bar.tsx`

```tsx
import { cn } from "@/lib/utils";

type ProgressBarProps = {
  /** 0–100 */
  pct: number;
  className?: string;
};

export function ProgressBar({ pct, className }: ProgressBarProps) {
  const clamped = Math.min(100, Math.max(0, pct));

  return (
    <div
      role="progressbar"
      aria-valuemin={0}
      aria-valuemax={100}
      aria-valuenow={clamped}
      className={cn("relative h-2 w-full overflow-hidden rounded-full bg-muted", className)}
    >
      <div
        className="h-full rounded-full bg-primary transition-[width] duration-300"
        style={{ width: `${clamped}%` }}
      />
    </div>
  );
}
```

Usage in `src/features/uploads/components/upload-row.tsx`:

```tsx
<ProgressBar pct={upload.progress} className="mt-2" />
```

## Change request

Add a small text label (e.g. `42%`) that sits at the right edge of the filled portion, so it moves along with the fill. The label is absolutely positioned inside the container, with `-translate-x-full` so its right edge lines up with the fill edge. The container height may grow to `h-5` to fit the text. The fill and the label must always agree on the same percentage.

Make the change and explain in one or two sentences how you structured the classes.
