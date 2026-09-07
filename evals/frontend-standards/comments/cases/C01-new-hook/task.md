# C01 — new-hook

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `notifications` feature shows a banner when the browser loses its connection.

`src/features/notifications/components/connection-banner.tsx`

```tsx
"use client";

import { AlertTriangle } from "lucide-react";
import { cn } from "@/lib/utils";

type ConnectionBannerProps = {
  className?: string;
};

export function ConnectionBanner({ className }: ConnectionBannerProps) {
  const online = typeof navigator !== "undefined" ? navigator.onLine : true;

  if (online) return null;

  return (
    <div
      role="status"
      className={cn(
        "flex items-center gap-2 border-b bg-amber-50 px-4 py-2 text-sm text-amber-900",
        className,
      )}
    >
      <AlertTriangle className="size-4" aria-hidden />
      You are offline. Changes will sync when the connection returns.
    </div>
  );
}
```

The feature already has one hook, which shows the folder's conventions.

`src/features/notifications/hooks/use-unread-count.ts`

```ts
"use client";

import { useEffect, useState } from "react";
import { subscribeUnreadCount } from "../lib/unread-store";

export function useUnreadCount(initial: number) {
  const [count, setCount] = useState(initial);

  useEffect(() => subscribeUnreadCount(setCount), []);

  return count;
}
```

## Change request

The banner reads `navigator.onLine` once at render and never updates, so it stays hidden when the connection drops after the page loads and stays visible after it returns.

Add a `useOnlineStatus()` hook in `src/features/notifications/hooks/use-online-status.ts` that follows the browser's `online` / `offline` events, returns `true` during server rendering, and use it in `ConnectionBanner`.

Make the change and explain in one or two sentences how you decided what to comment.
