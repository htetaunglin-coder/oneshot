# N12 — inbox-page-size

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui, TanStack Query, Prisma). The inbox feature lists a user's messages through a server action, paged by a cursor. The list is fetched from a client hook; a search box in the same feature debounces its input before it becomes the query.

`src/features/inbox/hooks/use-inbox.ts`

```ts
"use client";

import { useInfiniteQuery } from "@tanstack/react-query";
import { listMessages } from "@/features/inbox/lib/list-messages";

interface UseInboxOptions {
  pageSize?: number;
  search?: string;
}

export function useInbox({ pageSize = 25, search = "" }: UseInboxOptions = {}) {
  return useInfiniteQuery({
    queryKey: ["inbox", { pageSize, search }],
    queryFn: ({ pageParam }) => listMessages({ pageSize, search, cursor: pageParam }),
    initialPageParam: undefined as string | undefined,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });
}
```

`src/features/inbox/lib/list-messages.ts`

```ts
"use server";

import { db } from "@/lib/db";
import { getSession } from "@/lib/auth";

interface ListMessagesInput {
  pageSize: number;
  search: string;
  cursor?: string;
}

export async function listMessages({ pageSize, search, cursor }: ListMessagesInput) {
  const session = await getSession();
  const take = Math.min(pageSize, 25);

  const rows = await db.message.findMany({
    where: {
      recipientId: session.userId,
      ...(search === "" ? {} : { subject: { contains: search, mode: "insensitive" } }),
    },
    orderBy: { createdAt: "desc" },
    take: take + 1,
    ...(cursor ? { cursor: { id: cursor }, skip: 1 } : {}),
  });

  const hasMore = rows.length > take;
  const messages = hasMore ? rows.slice(0, take) : rows;

  return { messages, nextCursor: hasMore ? messages[messages.length - 1]?.id : undefined };
}
```

`src/features/inbox/hooks/use-debounced-value.ts`

```ts
"use client";

import { useEffect, useState } from "react";

export function useDebouncedValue<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}
```

`src/features/inbox/components/inbox-list.tsx` calls `useInbox({ search: debouncedSearch })` with no `pageSize`, where `debouncedSearch` comes from `useDebouncedValue(search)`.

## Change request

"Raise the inbox page size to 40. On the wide layout, 25 rows leaves a third of the screen empty and people hit 'Load more' on every visit."

Make the change and explain in one or two sentences how you decided which values get a name.
