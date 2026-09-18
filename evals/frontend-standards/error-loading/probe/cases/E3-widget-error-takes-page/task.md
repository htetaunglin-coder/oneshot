# E3 — widget-error-takes-page

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`, `src/app`. Server components fetch their own data with `fetch` or the Prisma client `db` from `src/lib/db.ts`. `src/lib/env.ts` exports a validated `env` object. `src/lib/auth.ts` exports `getSession(): Promise<Session | null>`, where `Session` has `userId` and `orgId`. Signed-in pages live under `src/app/(app)/`. Dependencies besides Next and React: `@tanstack/react-query`, `zustand`, `zod`, `lucide-react`, `sonner`, `class-variance-authority`, `tailwind-merge`. `src/components/` holds `ui/` (shadcn) plus `app-sidebar.tsx` and `page-header.tsx`.

`src/features/recommendations/api.ts`

```ts
import "server-only";
import { env } from "@/lib/env";

export type Recommendation = {
  id: string;
  sku: string;
  title: string;
  reason: string;
  upliftPct: number;
};

type MerlinItem = {
  id: string;
  sku: string;
  title: string;
  explanation: string;
  uplift: number;
};

export async function getRecommendations(orgId: string): Promise<Recommendation[]> {
  const res = await fetch(`${env.MERLIN_API_URL}/recommendations?org=${orgId}&limit=5`, {
    headers: { "x-api-key": env.MERLIN_API_KEY },
    signal: AbortSignal.timeout(4000),
    next: { revalidate: 300 },
  });

  if (!res.ok) {
    throw new Error(`Merlin responded ${res.status}`);
  }

  const body = (await res.json()) as { items: MerlinItem[] };
  return body.items.map((item) => ({
    id: item.id,
    sku: item.sku,
    title: item.title,
    reason: item.explanation,
    upliftPct: Math.round(item.uplift * 100),
  }));
}
```

`src/features/recommendations/components/recommendations-panel.tsx`

```tsx
import Link from "next/link";
import { Sparkles } from "lucide-react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { getRecommendations } from "@/features/recommendations/api";

type RecommendationsPanelProps = {
  orgId: string;
};

export async function RecommendationsPanel({ orgId }: RecommendationsPanelProps) {
  const items = await getRecommendations(orgId);

  return (
    <Card>
      <CardHeader className="flex flex-row items-center gap-2">
        <Sparkles className="size-4 text-muted-foreground" />
        <CardTitle className="text-base">Recommended for you</CardTitle>
      </CardHeader>
      <CardContent>
        {items.length === 0 ? (
          <p className="text-sm text-muted-foreground">Nothing to suggest yet.</p>
        ) : (
          <ul className="flex flex-col gap-3">
            {items.map((item) => (
              <li key={item.id} className="flex items-start justify-between gap-4">
                <div>
                  <Link
                    href={`/catalog/${item.sku}`}
                    className="text-sm font-medium hover:underline"
                  >
                    {item.title}
                  </Link>
                  <p className="text-xs text-muted-foreground">{item.reason}</p>
                </div>
                <span className="text-xs font-medium text-emerald-600">
                  +{item.upliftPct}%
                </span>
              </li>
            ))}
          </ul>
        )}
      </CardContent>
    </Card>
  );
}
```

`src/app/(app)/dashboard/page.tsx`

```tsx
import type { Metadata } from "next";
import { redirect } from "next/navigation";
import { getSession } from "@/lib/auth";
import { KpiCards } from "@/features/dashboard/components/kpi-cards";
import { RecentOrders } from "@/features/orders/components/recent-orders";
import { RecommendationsPanel } from "@/features/recommendations/components/recommendations-panel";

export const metadata: Metadata = {
  title: "Dashboard",
};

export default async function DashboardPage() {
  const session = await getSession();
  if (!session) redirect("/login");

  return (
    <div className="flex flex-col gap-6">
      <div>
        <h1 className="text-2xl font-semibold">Dashboard</h1>
        <p className="text-sm text-muted-foreground">Last 30 days</p>
      </div>
      <KpiCards orgId={session.orgId} />
      <div className="grid gap-6 lg:grid-cols-3">
        <div className="lg:col-span-2">
          <RecentOrders orgId={session.orgId} />
        </div>
        <RecommendationsPanel orgId={session.orgId} />
      </div>
    </div>
  );
}
```

`src/app/(app)/dashboard/error.tsx`

```tsx
"use client";

import { useEffect } from "react";
import { AlertTriangle } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";

type DashboardErrorProps = {
  error: Error & { digest?: string };
  unstable_retry: () => void;
};

export default function DashboardError({ error, unstable_retry }: DashboardErrorProps) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <Card className="mx-auto mt-12 max-w-md">
      <CardContent className="flex flex-col items-center gap-4 py-8 text-center">
        <AlertTriangle className="size-8 text-destructive" />
        <div>
          <h2 className="font-semibold">Something went wrong</h2>
          <p className="text-sm text-muted-foreground">
            The dashboard could not be loaded.
          </p>
        </div>
        <Button variant="outline" onClick={() => unstable_retry()}>
          Try again
        </Button>
      </CardContent>
    </Card>
  );
}
```

`KpiCards` and `RecentOrders` are async server components that read from `db`; both take `orgId`.

## Change request

"Merlin (the recommendations service) has been flaky all week, and every time it times out the whole dashboard is replaced by the 'Something went wrong' screen, so people can't see their orders either. When recommendations fail, the rest of the dashboard should still show as normal; put a small 'Recommendations unavailable' card in that slot instead."

Make the change and explain in one or two sentences how you decided where the failure or waiting state is handled.
