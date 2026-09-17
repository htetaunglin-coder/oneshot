# N11 — platform-keys

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The command palette feature has a keyboard-navigation hook used by the palette list and by the assignee picker. It filters the visible items by a query and moves an active row with the arrow keys.

`src/features/command-palette/hooks/use-list-navigation.ts`

```ts
"use client";

import { useCallback, useState, type KeyboardEvent } from "react";

interface UseListNavigationOptions<T> {
  items: T[];
  query: string;
  matches: (item: T, query: string) => boolean;
  onSelect: (item: T) => void;
  onClose: () => void;
}

export function useListNavigation<T>({
  items,
  query,
  matches,
  onSelect,
  onClose,
}: UseListNavigationOptions<T>) {
  const [activeIndex, setActiveIndex] = useState(0);

  const visibleItems = query === "" ? items : items.filter((item) => matches(item, query));

  const onKeyDown = useCallback(
    (event: KeyboardEvent<HTMLElement>) => {
      const target = event.target as HTMLInputElement;
      if (target.type === "checkbox") return;

      if (event.key === "ArrowDown") {
        event.preventDefault();
        setActiveIndex((i) => Math.min(i + 1, visibleItems.length - 1));
      } else if (event.key === "ArrowUp") {
        event.preventDefault();
        setActiveIndex((i) => Math.max(i - 1, 0));
      } else if (event.key === "Enter") {
        event.preventDefault();
        const item = visibleItems[activeIndex];
        if (item !== undefined) onSelect(item);
      } else if (event.key === "Escape") {
        onClose();
      }
    },
    [visibleItems, activeIndex, onSelect, onClose],
  );

  return { visibleItems, activeIndex, setActiveIndex, onKeyDown };
}
```

The last PR that touched this file carries an open review comment from the feature owner on the `onKeyDown` body:

> These string comparisons are getting scattered as we add keys. Please tidy this up while you are in here.

## Change request

"Add `Home` and `End` support to the list navigation: `Home` moves the active row to the first visible item and `End` to the last, both with the default browser behaviour prevented like the arrows. Address the open review comment on the same file in the same change."

Make the change and explain in one or two sentences how you decided which values get a name.
