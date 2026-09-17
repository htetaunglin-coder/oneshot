# R03 — inbox-header-counts

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/inbox/lib/types.ts`

```ts
export type InboxItem = {
  id: string;
  subject: string;
  isRead: boolean;
  receivedAt: string;
};
```

`src/features/inbox/components/inbox-header.tsx`

```tsx
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import type { InboxItem } from "../lib/types";

type InboxHeaderProps = {
  title?: string;
  items: InboxItem[];
  onMarkAllRead: () => void;
};

export function InboxHeader({ title, items, onMarkAllRead }: InboxHeaderProps) {
  return (
    <header className="flex items-center justify-between border-b px-4 py-3">
      <div className="flex items-center gap-2">
        {title && <h2 className="text-base font-semibold">{title}</h2>}
        {items.length && <Badge variant="secondary">{items.length}</Badge>}
      </div>
      <Button variant="ghost" size="sm" onClick={onMarkAllRead}>
        Mark all read
      </Button>
    </header>
  );
}
```

`src/features/inbox/components/inbox-page.tsx` (call site)

```tsx
<InboxHeader
  title={folder.label}
  items={folder.items}
  onMarkAllRead={() => markAllRead(folder.id)}
/>
```

## Change request

Add an `unreadCount: number` prop to `InboxHeader`. When there are unread items, show a second badge (default variant) reading `{unreadCount} unread` after the total badge, and disable the "Mark all read" button when nothing is unread. Update the call site to pass `unreadCount={folder.unreadCount}` (`folder.unreadCount` is already a `number`). While you are there: QA reports that an empty folder shows a stray `0` next to the title.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
