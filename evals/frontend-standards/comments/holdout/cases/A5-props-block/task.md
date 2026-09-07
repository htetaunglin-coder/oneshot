# A5 — props-block

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). `PlanCard` is used by the pricing page and the in-app upgrade dialog. Last week a teammate passed `trialDays={0}` to mean "no trial" and shipped a card that read "0-day free trial"; the same person asked in the channel what `onSelect` is called with.

`src/features/billing/types.ts`

```ts
export type Plan = {
  id: string;
  name: string;
  tagline: string;
  features: string[];
};
```

`src/features/billing/components/plan-card.tsx`

```tsx
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { formatPrice } from "@/lib/format";
import { cn } from "@/lib/utils";
import type { Plan } from "../types";

type PlanCardProps = {
  plan: Plan;
  selected: boolean;
  onSelect: (id: string) => void;
  priceCents: number;
  trialDays: number | null;
  billingInterval: "month" | "year";
};

export function PlanCard({
  plan,
  selected,
  onSelect,
  priceCents,
  trialDays,
  billingInterval,
}: PlanCardProps) {
  return (
    <Card
      className={cn(
        "flex flex-col",
        selected && "border-primary ring-2 ring-primary",
      )}
    >
      <CardHeader>
        <CardTitle>{plan.name}</CardTitle>
        <p className="text-sm text-muted-foreground">{plan.tagline}</p>
      </CardHeader>
      <CardContent className="flex-1 space-y-2">
        <p className="text-3xl font-semibold">
          {formatPrice(priceCents)}
          <span className="text-sm font-normal text-muted-foreground">
            /{billingInterval}
          </span>
        </p>
        {trialDays !== null ? (
          <p className="text-sm">{trialDays}-day free trial</p>
        ) : null}
        <ul className="space-y-1 text-sm">
          {plan.features.map((feature) => (
            <li key={feature}>{feature}</li>
          ))}
        </ul>
      </CardContent>
      <CardFooter>
        <Button
          className="w-full"
          variant={selected ? "secondary" : "default"}
          onClick={() => onSelect(plan.id)}
        >
          {selected ? "Current plan" : "Choose plan"}
        </Button>
      </CardFooter>
    </Card>
  );
}
```

Prices come from a separate price table because the same plan costs different amounts per interval, which is why `priceCents` is not on `Plan`. Upstream, a plan either has no trial or a trial of at least one day; zero never reaches this component.

## Change request

From the teammate: "Can you document PlanCard's props so people know how to use it?"

Make the change and explain in one or two sentences how you decided what to comment.
