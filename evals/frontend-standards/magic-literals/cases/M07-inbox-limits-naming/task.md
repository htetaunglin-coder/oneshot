# M07 — inbox-limits-naming

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/inbox/lib/types.ts`

```ts
export type MessageStatus = "unread" | "read" | "archived";

export type Message = {
  id: string;
  subject: string;
  preview: string;
  status: MessageStatus;
};
```

`src/features/inbox/components/inbox-list.tsx`

```tsx
import type { Message, MessageStatus } from "@/features/inbox/lib/types";
import { cn } from "@/lib/utils";

const maxItems = 20;

type InboxListProps = {
  messages: Message[];
};

export function InboxList({ messages }: InboxListProps) {
  const visible = messages.slice(0, maxItems);
  const hiddenCount = messages.length - visible.length;

  return (
    <div>
      <ul className="divide-y">
        {visible.map((message) => (
          <InboxRow key={message.id} message={message} />
        ))}
      </ul>
      {hiddenCount > 0 ? (
        <p className="p-3 text-sm text-muted-foreground">
          {hiddenCount} more in the archive
        </p>
      ) : null}
    </div>
  );
}

function InboxRow({ message }: { message: Message }) {
  return (
    <li className="flex items-start gap-3 p-3">
      <div className="min-w-0 flex-1">
        <p className={cn("truncate", message.status === "unread" && "font-medium")}>
          {message.subject}
        </p>
        <p className="text-sm text-muted-foreground">{message.preview}</p>
      </div>
      <StatusBadge status={message.status} />
    </li>
  );
}

function StatusBadge({ status }: { status: MessageStatus }) {
  const STATUS_LABELS: Record<MessageStatus, string> = {
    unread: "Unread",
    read: "Read",
    archived: "Archived",
  };

  return (
    <span className="rounded-full border px-2 py-0.5 text-xs">
      {STATUS_LABELS[status]}
    </span>
  );
}
```

## Change request

Two tweaks from product: show 25 messages instead of 20, and the preview line wraps across several lines on long messages, so cut the preview to its first 140 characters followed by an ellipsis ("…") when it is longer than that.

Make the change and explain in one or two sentences how you decided which values get a name.
