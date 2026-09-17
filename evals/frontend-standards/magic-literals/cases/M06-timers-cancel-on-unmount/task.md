# M06 — timers-cancel-on-unmount

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`, shared hooks under `src/hooks`.

`src/hooks/use-debounced-value.ts`

```ts
"use client";

import { useEffect, useState } from "react";

export function useDebouncedValue<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    setTimeout(() => setDebounced(value), delay);
  }, [value, delay]);

  return debounced;
}
```

`src/features/search/components/search-box.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { Input } from "@/components/ui/input";
import { useDebouncedValue } from "@/hooks/use-debounced-value";

type SearchBoxProps = {
  onSearch: (query: string) => void;
};

export function SearchBox({ onSearch }: SearchBoxProps) {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebouncedValue(query);

  useEffect(() => {
    onSearch(debouncedQuery);
  }, [debouncedQuery, onSearch]);

  return (
    <Input
      type="search"
      value={query}
      onChange={(event) => setQuery(event.target.value)}
      placeholder="Search products"
    />
  );
}
```

`src/features/auth/components/session-warning.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { Button } from "@/components/ui/button";

type SessionWarningProps = {
  onExtend: () => void;
};

export function SessionWarning({ onExtend }: SessionWarningProps) {
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    setTimeout(() => setVisible(true), 5 * 60 * 1000);
  }, []);

  if (!visible) return null;

  return (
    <div
      role="alert"
      className="fixed bottom-4 right-4 rounded-lg border bg-background p-4 shadow-md"
    >
      <p className="text-sm">Your session is about to expire.</p>
      <Button
        size="sm"
        className="mt-2"
        onClick={() => {
          onExtend();
          setVisible(false);
        }}
      >
        Stay signed in
      </Button>
    </div>
  );
}
```

## Change request

Both timers keep running after their owner unmounts: tests that unmount `SearchBox` or `SessionWarning` early see state updates on unmounted components. Make each timer stop when its owner unmounts (and, for the debounce, when the value changes before the timer fires). Behaviour while mounted stays the same.

Make the change and explain in one or two sentences how you decided which values get a name.
