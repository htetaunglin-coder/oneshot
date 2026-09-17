# R08 — step-dot-pending

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/onboarding/components/step-dot.tsx`

```tsx
import { cn } from "@/lib/utils";

type StepDotProps = {
  index: number;
  label: string;
  isActive: boolean;
  isDisabled: boolean;
  onSelect: (index: number) => void;
};

export function StepDot({ index, label, isActive, isDisabled, onSelect }: StepDotProps) {
  return (
    <button
      type="button"
      disabled={isDisabled}
      aria-current={isActive ? "step" : undefined}
      onClick={() => onSelect(index)}
      className={cn(
        "flex size-8 items-center justify-center rounded-full border text-sm transition-colors",
        isActive
          ? isDisabled
            ? "border-muted bg-muted text-muted-foreground"
            : "border-primary bg-primary text-primary-foreground"
          : "border-border bg-background text-foreground hover:bg-accent",
      )}
    >
      <span className="sr-only">{label}</span>
      {index + 1}
    </button>
  );
}
```

`src/features/onboarding/components/step-list.tsx` (call site)

```tsx
{steps.map((step, index) => (
  <StepDot
    key={step.id}
    index={index}
    label={step.label}
    isActive={index === currentIndex}
    isDisabled={index > furthestIndex}
    onSelect={goTo}
  />
))}
```

## Change request

Steps now run async validation, and the dot for a step being validated should look pending: `border-amber-500 bg-amber-50 text-amber-900 animate-pulse`. Add an `isPending: boolean` prop and pass `isPending={step.id === validatingStepId}` from the call site (`validatingStepId: string | null` is already in scope there). Precedence for the look, highest first: disabled, pending, active, default. A pending dot stays clickable unless it is disabled.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
