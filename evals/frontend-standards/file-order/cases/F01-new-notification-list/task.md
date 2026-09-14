# F01 — new-notification-list

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Features live under `src/features/<name>/components|hooks|lib`; shared UI under `src/components/ui`; shared helpers under `src/lib`.

`src/features/notifications/lib/types.ts`

```ts
export type Notification = {
  id: string;
  title: string;
  body: string;
  createdAt: Date;
  readAt: Date | null;
};
```

`src/features/notifications/lib/queries.ts`

```ts
import { db } from "@/lib/db";
import type { Notification } from "./types";

export async function getNotifications(userId: string): Promise<Notification[]> {
  const rows = await db.notification.findMany({ where: { userId }, take: 100 });
  return rows.sort((a, b) => b.createdAt.getTime() - a.createdAt.getTime());
}
```

`src/app/(app)/notifications/page.tsx`

```tsx
import { getCurrentUser } from "@/lib/auth";
import { getNotifications } from "@/features/notifications/lib/queries";

export default async function NotificationsPage() {
  const user = await getCurrentUser();
  const notifications = await getNotifications(user.id);

  return (
    <section className="space-y-4">
      <h1 className="text-xl font-semibold">Notifications</h1>
      <pre className="text-xs">{JSON.stringify(notifications, null, 2)}</pre>
    </section>
  );
}
```

There is no `src/features/notifications/components/` folder yet.

## Change request

Replace the `<pre>` dump with a real list. Create `src/features/notifications/components/notification-list.tsx` exporting a `NotificationList` component that takes `notifications: Notification[]`. It shows at most 20 items (name the cap `MAX_ITEMS`) and, when it truncates, a line above the list that reads "Showing 20 of N". Each item is rendered by a `NotificationRow` sub-component in the same file: title, body, and a relative time such as "3 min ago", "2 h ago", "5 d ago", computed by a `formatRelativeTime(date: Date): string` helper in the same file. Rows with `readAt === null` get a `font-medium` title. Wire the page to render `NotificationList`.

Make the change and explain in one or two sentences how you decided where each declaration goes.
