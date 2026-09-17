# N3 — enum-instinct

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The notifications feature stores the kind of a notification as a plain `string`, and the three known kinds are spelled out wherever they are checked.

`src/features/notifications/lib/types.ts`

```ts
export type Notification = {
  id: string;
  kind: string;
  message: string;
  readAt: string | null;
  createdAt: string;
};
```

`src/features/notifications/components/notification-item.tsx`

```tsx
import { AlertTriangle, Info, XCircle } from "lucide-react";
import { cn } from "@/lib/utils";
import type { Notification } from "@/features/notifications/lib/types";

type NotificationItemProps = {
  notification: Notification;
};

export function NotificationItem({ notification }: NotificationItemProps) {
  const Icon =
    notification.kind === "error" ? XCircle : notification.kind === "warning" ? AlertTriangle : Info;
  return (
    <li
      className={cn(
        "flex items-start gap-2 rounded-md border p-3 text-sm",
        notification.kind === "error" && "border-destructive/40 bg-destructive/5",
        notification.kind === "warning" && "border-amber-400/40 bg-amber-50",
      )}
    >
      <Icon className="mt-0.5 size-4 shrink-0" aria-hidden />
      <span>{notification.message}</span>
    </li>
  );
}
```

`src/features/notifications/hooks/use-notifications.ts`

```ts
"use client";

import { useQuery } from "@tanstack/react-query";
import { fetchNotifications } from "@/features/notifications/lib/fetch-notifications";

export function useNotifications(kind?: string) {
  return useQuery({
    queryKey: ["notifications", kind ?? "all"],
    queryFn: () => fetchNotifications(),
    select: (items) => (kind ? items.filter((item) => item.kind === kind) : items),
  });
}
```

`src/features/notifications/components/notification-filter.tsx`

```tsx
"use client";

import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

type NotificationFilterProps = {
  value: string | undefined;
  onChange: (kind: string | undefined) => void;
};

export function NotificationFilter({ value, onChange }: NotificationFilterProps) {
  return (
    <Select value={value ?? "all"} onValueChange={(v) => onChange(v === "all" ? undefined : v)}>
      <SelectTrigger className="w-40">
        <SelectValue placeholder="All kinds" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="all">All kinds</SelectItem>
        <SelectItem value="info">Info</SelectItem>
        <SelectItem value="warning">Warning</SelectItem>
        <SelectItem value="error">Error</SelectItem>
      </SelectContent>
    </Select>
  );
}
```

Last week a typo (`"warnng"`) shipped in a filter call and nothing caught it.

## Change request

"Add a proper type for the notification kinds so the compiler catches typos. `Notification.kind`, the hook's `kind` parameter, and the filter's `value`/`onChange` should all use it, and the picker should list the same kinds from one place rather than spelling them out again."

Make the change and explain in one or two sentences how you decided which values get a name.
