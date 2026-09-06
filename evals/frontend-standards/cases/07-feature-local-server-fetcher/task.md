# Case 07 — feature-local-server-fetcher

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
src/app/(routes)/…
```

### `src/features/home/api/contributions.ts`

```ts
export type ContributionDay = { date: string; count: number };

const ENDPOINT = "https://api.example-forge.dev/v1/contributions";

export async function getContributions(): Promise<ContributionDay[]> {
  const res = await fetch(`${ENDPOINT}?user=${process.env.FORGE_USER}`, {
    headers: { Authorization: `Bearer ${process.env.FORGE_TOKEN}` },
    next: { revalidate: 60 * 60 },
  });
  if (!res.ok) throw new Error(`contributions: ${res.status}`);
  const json = (await res.json()) as { days: ContributionDay[] };
  return json.days;
}
```

### `src/features/home/components/contribution-graph.tsx` (server component)

```tsx
import { getContributions } from "../api/contributions";
import { cn } from "@/lib/cn";

function level(count: number) {
  if (count === 0) return "bg-zinc-800";
  if (count < 3) return "bg-emerald-900";
  if (count < 6) return "bg-emerald-700";
  return "bg-emerald-500";
}

export async function ContributionGraph() {
  const days = await getContributions();
  return (
    <div className="grid grid-flow-col grid-rows-7 gap-1">
      {days.map((d) => (
        <span
          key={d.date}
          title={`${d.date}: ${d.count}`}
          className={cn("h-3 w-3 rounded-sm", level(d.count))}
        />
      ))}
    </div>
  );
}
```

`ContributionGraph` is rendered by `src/app/(routes)/page.tsx`. `getContributions`
has no other callers. The external API accepts a `year=YYYY` query parameter.

## Task

The home page should show a chosen year (from the `?year=` search param, defaulting
to the current year) instead of always the current year. Make the change. Decide
where the new/changed code lives and explain in one or two sentences.
