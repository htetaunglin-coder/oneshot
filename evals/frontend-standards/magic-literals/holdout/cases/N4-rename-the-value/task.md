# N4 — rename-the-value

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The activity feature has a hook that loads the latest events for a team and keeps them fresh by re-fetching on a timer.

`src/features/activity/hooks/use-activity-feed.ts`

```ts
"use client";

import { useEffect, useState } from "react";
import { fetchActivity } from "@/features/activity/lib/fetch-activity";
import type { ActivityEvent } from "@/features/activity/lib/types";

const THIRTY = 30;

export function useActivityFeed(teamId: string) {
  const [events, setEvents] = useState<ActivityEvent[]>([]);

  useEffect(() => {
    let cancelled = false;
    let timer: ReturnType<typeof setTimeout> | undefined;

    async function load() {
      const next = await fetchActivity({ teamId, take: THIRTY });
      if (cancelled) return;
      setEvents(next);
      timer = setTimeout(load, THIRTY * 1000);
    }

    load();

    return () => {
      cancelled = true;
      if (timer) clearTimeout(timer);
    };
  }, [teamId]);

  return events;
}
```

`fetchActivity({ teamId, take })` returns the `take` most recent events, newest first.

## Change request

"Product wants the activity feed to show 50 events instead of 30. Raise the list limit to 50."

Make the change and explain in one or two sentences how you decided which values get a name.
