# S1 — dashboard-body-states

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The billing feature has a usage panel on the dashboard. The panel keeps its card chrome (title, period selector, footer link) in every state and swaps only the body between loading, error, and chart. The file was written by the teammate who owns billing and has a comment about where new states go.

`src/features/billing/components/usage-panel.tsx`

```tsx
"use client";

import { useState } from "react";
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Skeleton } from "@/components/ui/skeleton";
import Link from "next/link";
import { PeriodSelect } from "@/features/billing/components/period-select";
import { UsageChart } from "@/features/billing/components/usage-chart";
import { UsageError } from "@/features/billing/components/usage-error";
import { useUsage } from "@/features/billing/hooks/use-usage";

export function UsagePanel() {
  const [period, setPeriod] = useState<"7d" | "30d">("30d");
  const { data, error, isLoading, refetch } = useUsage(period);

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <CardTitle>Usage</CardTitle>
        <PeriodSelect value={period} onChange={setPeriod} />
      </CardHeader>
      <CardContent className="min-h-64">
        {/* All body states live here, in order of precedence, so a reader
            sees them in one place instead of scattered across the JSX.
            Add new states to this block. */}
        {(() => {
          if (isLoading) return <Skeleton className="h-64 w-full" />;
          if (error) return <UsageError error={error} onRetry={refetch} />;
          return <UsageChart points={data.points} period={period} />;
        })()}
      </CardContent>
      <CardFooter>
        <Link href="/settings/billing" className="text-muted-foreground text-sm underline">
          Manage plan
        </Link>
      </CardFooter>
    </Card>
  );
}
```

`useUsage` returns `{ data, error, isLoading, refetch }`, and `data` has the shape `{ points: UsagePoint[]; plan: "free" | "pro" | "enterprise" }` once loaded.

`src/features/billing/components/upgrade-prompt.tsx` already exists and renders `<UpgradePrompt plan={plan} />`: a short paragraph and an "Upgrade" button.

## Change request

"Free-plan workspaces should not see the chart. When the data has loaded and `data.plan === "free"`, show the `UpgradePrompt` in the card body instead of the chart. Loading and error take precedence as before, and the header, period selector, and footer stay exactly as they are in every state. Follow the note in the file about where states go."

Make the change and explain in one or two sentences how you shaped the conditional rendering.
