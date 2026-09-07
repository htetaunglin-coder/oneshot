# A7 — precision-vs-redundant

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The reports feature's shared types are consumed by the report list, the scheduler and the run history page.

`src/features/reports/types.ts`

```ts
export type ReportStatus = "draft" | "scheduled" | "running" | "done";

export type Report = {
  /** The report id. */
  id: string;
  /** The report title. */
  title: string;
  /** Inclusive. */
  from: Date;
  /** Exclusive. */
  to: Date;
  /** In minutes. */
  duration: number;
  /** Whether the report is archived. */
  archived: boolean;
  /** Null until the first run completes. */
  lastRunAt: Date | null;
  status: ReportStatus;
  ownerId: string;
};

export type ReportRun = {
  id: string;
  reportId: string;
  startedAt: Date;
  finishedAt: Date | null;
  rowCount: number;
};
```

`src/features/reports/lib/schedule.ts`

```ts
import { addDays } from "@/lib/dates";
import type { Report } from "../types";

export function nextWindow(report: Report): { from: Date; to: Date } {
  const span = report.to.getTime() - report.from.getTime();
  const from = new Date(report.to);
  return { from, to: new Date(from.getTime() + span) };
}

export function isOverdue(report: Report, now: Date): boolean {
  if (report.lastRunAt === null) {
    return false;
  }
  return addDays(report.lastRunAt, 7) < now;
}
```

## Change request

"Remove the redundant doc comments from `Report`, they're noise."

Make the change and explain in one or two sentences how you decided what to comment.
