# T03 — one-axis-three-states

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `onboarding` feature has an inline callout.

`src/features/onboarding/components/callout.tsx`

```tsx
import { cn } from "@/lib/utils";
import { Info, AlertOctagon } from "lucide-react";

type CalloutProps = {
  kind: "info" | "danger";
  title: string;
  children: React.ReactNode;
  className?: string;
};

export function Callout({ kind, title, children, className }: CalloutProps) {
  const Icon = kind === "danger" ? AlertOctagon : Info;

  return (
    <div
      role="note"
      className={cn(
        "flex gap-3 rounded-md border p-4 text-sm",
        kind === "danger"
          ? "border-red-300 bg-red-50 text-red-900"
          : "border-blue-300 bg-blue-50 text-blue-900",
        className
      )}
    >
      <Icon className="mt-0.5 size-4 shrink-0" aria-hidden />
      <div>
        <p className="font-medium">{title}</p>
        <div className="mt-1">{children}</div>
      </div>
    </div>
  );
}
```

Usage in `src/features/onboarding/components/connect-step.tsx`:

```tsx
<Callout kind="info" title="Read-only access">
  We only request permission to list your repositories.
</Callout>
```

## Change request

Add a third `kind`, `"warning"`, that uses `AlertTriangle` from `lucide-react` and the amber palette (`border-amber-300 bg-amber-50 text-amber-900`).

Make the change and explain in one or two sentences how you structured the classes.
