# E4 — loading-scope

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`, `src/app`. Server components fetch their own data with `fetch` or the Prisma client `db` from `src/lib/db.ts`. `src/lib/env.ts` exports a validated `env` object. `src/lib/format.ts` exports `formatRelative(iso: string): string`. Signed-in pages live under `src/app/(app)/`.

`src/features/projects/api.ts`

```ts
import "server-only";
import { db } from "@/lib/db";
import { env } from "@/lib/env";

export type Project = {
  id: string;
  name: string;
  client: string;
  status: "active" | "paused" | "done";
};

export type ProjectStats = {
  openTasks: number;
  hoursLogged: number;
  budgetUsedPct: number;
};

export type ActivityItem = {
  id: string;
  actor: string;
  verb: string;
  target: string;
  at: string;
};

export async function getProject(id: string): Promise<Project | null> {
  return db.project.findUnique({
    where: { id },
    select: { id: true, name: true, client: true, status: true },
  });
}

export async function getProjectStats(id: string): Promise<ProjectStats> {
  const [openTasks, hours, project] = await Promise.all([
    db.task.count({ where: { projectId: id, done: false } }),
    db.timeEntry.aggregate({ where: { projectId: id }, _sum: { hours: true } }),
    db.project.findUniqueOrThrow({ where: { id }, select: { budgetHours: true } }),
  ]);
  const hoursLogged = hours._sum.hours ?? 0;
  return {
    openTasks,
    hoursLogged,
    budgetUsedPct: Math.round((hoursLogged / project.budgetHours) * 100),
  };
}

// Event history lives in the warehouse; this call takes 2–3 s at p50.
export async function getActivity(id: string): Promise<ActivityItem[]> {
  const res = await fetch(`${env.WAREHOUSE_URL}/projects/${id}/events?limit=30`, {
    headers: { Authorization: `Bearer ${env.WAREHOUSE_TOKEN}` },
    cache: "no-store",
  });
  if (!res.ok) throw new Error(`Warehouse responded ${res.status}`);
  const body = (await res.json()) as { events: ActivityItem[] };
  return body.events;
}
```

`src/app/(app)/projects/[id]/page.tsx`

```tsx
import { notFound } from "next/navigation";
import { getActivity, getProject, getProjectStats } from "@/features/projects/api";
import { ActivityFeed } from "@/features/projects/components/activity-feed";
import { ProjectHeader } from "@/features/projects/components/project-header";
import { StatsRow } from "@/features/projects/components/stats-row";

type ProjectPageProps = {
  params: Promise<{ id: string }>;
};

export async function generateMetadata({ params }: ProjectPageProps) {
  const { id } = await params;
  const project = await getProject(id);
  return { title: project ? project.name : "Project" };
}

export default async function ProjectPage({ params }: ProjectPageProps) {
  const { id } = await params;
  const project = await getProject(id);
  if (!project) notFound();

  const [stats, activity] = await Promise.all([getProjectStats(id), getActivity(id)]);

  return (
    <div className="flex flex-col gap-6">
      <ProjectHeader project={project} />
      <StatsRow stats={stats} />
      <ActivityFeed items={activity} />
    </div>
  );
}
```

`src/features/projects/components/activity-feed.tsx`

```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import type { ActivityItem } from "@/features/projects/api";
import { formatRelative } from "@/lib/format";

type ActivityFeedProps = {
  items: ActivityItem[];
};

export function ActivityFeed({ items }: ActivityFeedProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Activity</CardTitle>
      </CardHeader>
      <CardContent>
        {items.length === 0 ? (
          <p className="text-sm text-muted-foreground">No activity yet.</p>
        ) : (
          <ol className="flex flex-col gap-3">
            {items.map((item) => (
              <li key={item.id} className="flex flex-col gap-0.5 text-sm">
                <span>
                  <span className="font-medium">{item.actor}</span> {item.verb}{" "}
                  <span className="font-medium">{item.target}</span>
                </span>
                <time dateTime={item.at} className="text-xs text-muted-foreground">
                  {formatRelative(item.at)}
                </time>
              </li>
            ))}
          </ol>
        )}
      </CardContent>
    </Card>
  );
}
```

`src/app/(app)/projects/[id]/loading.tsx`

```tsx
import { Skeleton } from "@/components/ui/skeleton";

const FEED_ROWS = 5;

export default function ProjectLoading() {
  return (
    <div className="flex flex-col gap-6">
      <div className="flex flex-col gap-2">
        <Skeleton className="h-7 w-64" />
        <Skeleton className="h-4 w-40" />
      </div>
      <div className="grid gap-4 sm:grid-cols-3">
        <Skeleton className="h-24" />
        <Skeleton className="h-24" />
        <Skeleton className="h-24" />
      </div>
      <div className="rounded-xl border p-6">
        <Skeleton className="mb-4 h-5 w-24" />
        <div className="flex flex-col gap-3">
          {Array.from({ length: FEED_ROWS }).map((_, i) => (
            <div key={i} className="flex flex-col gap-1.5">
              <Skeleton className="h-4 w-3/4" />
              <Skeleton className="h-3 w-24" />
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

`ProjectHeader` (takes `project: Project`) and `StatsRow` (takes `stats: ProjectStats`) are presentational server components under `src/features/projects/components/`.

## Change request

"Opening a project shows the grey skeleton for the whole page for two or three seconds because the activity list comes from the warehouse. The header and the stats are quick — they should appear right away, and only the activity list should show its own skeleton (the same rows as the current one) until it arrives."

Make the change and explain in one or two sentences how you decided where the failure or waiting state is handled.
