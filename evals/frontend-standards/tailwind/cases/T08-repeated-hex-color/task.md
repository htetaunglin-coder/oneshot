# T08 — repeated-hex-color

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The brand purple appears in three components across two features.

`src/features/marketing/components/hero-cta.tsx`

```tsx
import Link from "next/link";

export function HeroCta() {
  return (
    <Link
      href="/signup"
      className="inline-flex h-11 items-center rounded-md bg-[#5b4bff] px-6 font-medium text-white hover:bg-[#5b4bff]/90"
    >
      Start free trial
    </Link>
  );
}
```

`src/features/marketing/components/pricing-highlight.tsx`

```tsx
export function PricingHighlight({ plan }: { plan: string }) {
  return (
    <p className="text-sm">
      Most popular: <span className="font-semibold text-[#5b4bff]">{plan}</span>
    </p>
  );
}
```

`src/features/dashboard/components/usage-meter.tsx`

```tsx
export function UsageMeter({ pct }: { pct: number }) {
  return (
    <div className="h-2 w-full rounded-full bg-muted">
      <div className="h-full rounded-full bg-[#5b4bff]" style={{ width: `${pct}%` }} />
    </div>
  );
}
```

`src/app/globals.css` (excerpt)

```css
@import "tailwindcss";

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-muted: var(--muted);
}
```

## Change request

Marketing has updated the brand purple from `#5b4bff` to `#4f46e5`. Apply the new color everywhere it is used, and make sure a future change does not require touching every component again.

Make the change and explain in one or two sentences how you structured the classes.
