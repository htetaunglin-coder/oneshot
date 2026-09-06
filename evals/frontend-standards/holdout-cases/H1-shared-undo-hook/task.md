# H1 — A feature hook that a second feature now wants

## Scenario

`src/features/inbox/hooks/use-undoable-delete.ts` delays a delete for five seconds, shows a toast with an Undo button, and only runs the real delete when the timer expires. It was written for the inbox message list and has exactly one consumer, `src/features/inbox/components/message-row.tsx`.

```ts
// src/features/inbox/hooks/use-undoable-delete.ts
"use client";

import { useCallback, useEffect, useRef, useState } from "react";
import { toast } from "sonner";
import { deleteMessage } from "@/features/inbox/api/delete-message";

const UNDO_WINDOW_MS = 5000;

export function useUndoableDelete() {
  const [pendingIds, setPendingIds] = useState<Set<string>>(new Set());
  const timers = useRef(new Map<string, ReturnType<typeof setTimeout>>());

  const clearPending = (id: string) =>
    setPendingIds((prev) => {
      const next = new Set(prev);
      next.delete(id);
      return next;
    });

  const cancel = useCallback((id: string) => {
    const t = timers.current.get(id);
    if (t) clearTimeout(t);
    timers.current.delete(id);
    clearPending(id);
  }, []);

  const remove = useCallback(
    (id: string) => {
      setPendingIds((prev) => new Set(prev).add(id));
      const t = setTimeout(async () => {
        timers.current.delete(id);
        await deleteMessage(id);
        clearPending(id);
      }, UNDO_WINDOW_MS);
      timers.current.set(id, t);
      toast("Message deleted", {
        action: { label: "Undo", onClick: () => cancel(id) },
        duration: UNDO_WINDOW_MS,
      });
    },
    [cancel],
  );

  useEffect(() => () => timers.current.forEach(clearTimeout), []);

  return { remove, cancel, isPending: (id: string) => pendingIds.has(id) };
}
```

```tsx
// src/features/inbox/components/message-row.tsx (excerpt)
const { remove, isPending } = useUndoableDelete();
// ...
<Button variant="ghost" disabled={isPending(message.id)} onClick={() => remove(message.id)}>
  Delete
</Button>
```

## Change request

The Drafts feature (`src/features/drafts`) needs the same behavior on draft cards: delayed delete, a toast that reads "Draft discarded" with an Undo button, and `deleteDraft(id)` from `src/features/drafts/api/delete-draft.ts` as the real delete. Decide where the new/changed code lives and explain in one or two sentences.
