# E2 — error-message-in-production

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`, `src/app`. Server components fetch their own data with `fetch`. `src/lib/env.ts` exports a validated `env` object (`METERING_API_URL`, `METERING_API_KEY`, and others). `src/lib/auth.ts` exports `getSession(): Promise<Session | null>`, where `Session` has `userId` and `orgId`. Signed-in pages live under `src/app/(app)/`; that group's layout renders the sidebar. The app is deployed on Vercel and customers use it daily.

`src/features/usage/api.ts`

```ts
import "server-only";
import { env } from "@/lib/env";

export type UsageRow = {
  metric: string;
  used: number;
  included: number;
  overageCents: number;
};

export type UsageReport = {
  periodStart: string;
  periodEnd: string;
  rows: UsageRow[];
};

type UpstreamError = {
  code: string;
  message: string;
};

export async function getUsageReport(orgId: string): Promise<UsageReport> {
  const res = await fetch(`${env.METERING_API_URL}/v1/orgs/${orgId}/usage`, {
    headers: { Authorization: `Bearer ${env.METERING_API_KEY}` },
    cache: "no-store",
  });

  if (!res.ok) {
    const upstream = (await res.json().catch(() => null)) as UpstreamError | null;
    throw new Error(upstream?.message ?? `Metering API responded ${res.status}`);
  }

  return (await res.json()) as UsageReport;
}
```

`src/app/(app)/reports/usage/page.tsx`

```tsx
import type { Metadata } from "next";
import { redirect } from "next/navigation";
import { getSession } from "@/lib/auth";
import { getUsageReport } from "@/features/usage/api";
import { UsageSummary } from "@/features/usage/components/usage-summary";
import { UsageTable } from "@/features/usage/components/usage-table";

export const metadata: Metadata = {
  title: "Usage",
};

export default async function UsageReportPage() {
  const session = await getSession();
  if (!session) redirect("/login");

  const report = await getUsageReport(session.orgId);

  return (
    <div className="flex flex-col gap-6">
      <div>
        <h1 className="text-2xl font-semibold">Usage</h1>
        <p className="text-sm text-muted-foreground">
          {report.periodStart} – {report.periodEnd}
        </p>
      </div>
      <UsageSummary rows={report.rows} />
      <UsageTable rows={report.rows} />
    </div>
  );
}
```

`src/app/(app)/reports/usage/error.tsx`

```tsx
"use client";

import { useEffect } from "react";
import { AlertTriangle } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";

type UsageErrorProps = {
  error: Error & { digest?: string };
  unstable_retry: () => void;
};

export default function UsageError({ error, unstable_retry }: UsageErrorProps) {
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
            The usage report could not be loaded.
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

`UsageSummary` and `UsageTable` are presentational server components under `src/features/usage/components/` that take `rows: UsageRow[]`.

## Change request

"Support gets tickets from customers on the Usage page that only say 'something went wrong', and then has to dig through logs to work out why. When the metering API rejects the request, show the customer the real reason on that screen — the API's message is written for humans, e.g. 'Rate limit exceeded for org acme' or 'Billing account suspended' — so support can read it off the customer's screen on the call. Keep the Try again button."

Make the change and explain in one or two sentences how you decided where the failure or waiting state is handled.
