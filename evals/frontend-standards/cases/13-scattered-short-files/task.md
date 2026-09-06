# Case 13 — scattered-short-files

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

`src/features/search/` currently contains these five files. Each imports from two or
three of the others; the last two changes to search (adding `sort`, adding
`pageSize`) each touched four of the five.

### `src/features/search/search-state.ts` (10 lines)

```ts
export type SearchState = { query: string; sort: "relevance" | "recent"; pageSize: number };
export const initialSearchState: SearchState = { query: "", sort: "relevance", pageSize: 20 };
```

### `src/features/search/search-actions.ts` (14 lines)

```ts
import type { SearchState } from "./search-state";

export type SearchAction =
  | { type: "set-query"; query: string }
  | { type: "set-sort"; sort: SearchState["sort"] }
  | { type: "set-page-size"; pageSize: number };

export function reduce(state: SearchState, a: SearchAction): SearchState {
  if (a.type === "set-query") return { ...state, query: a.query };
  if (a.type === "set-sort") return { ...state, sort: a.sort };
  return { ...state, pageSize: a.pageSize };
}
```

### `src/features/search/search-context.tsx` (20 lines)

```tsx
"use client";
import { createContext, useContext, useReducer, type Dispatch, type ReactNode } from "react";
import { initialSearchState, type SearchState } from "./search-state";
import { reduce, type SearchAction } from "./search-actions";

const Ctx = createContext<[SearchState, Dispatch<SearchAction>] | null>(null);

export function SearchProvider({ children }: { children: ReactNode }) {
  const value = useReducer(reduce, initialSearchState);
  return <Ctx.Provider value={value}>{children}</Ctx.Provider>;
}

export function useSearchContext() {
  const v = useContext(Ctx);
  if (!v) throw new Error("SearchProvider missing");
  return v;
}
```

### `src/features/search/use-search-query.ts` (12 lines)

```ts
import { useSearchContext } from "./search-context";

export function useSearchQuery() {
  const [state, dispatch] = useSearchContext();
  return {
    query: state.query,
    setQuery: (query: string) => dispatch({ type: "set-query", query }),
    setSort: (sort: typeof state.sort) => dispatch({ type: "set-sort", sort }),
  };
}
```

### `src/features/search/use-search-results.ts` (15 lines)

```ts
import { useQuery } from "@tanstack/react-query";
import { useSearchContext } from "./search-context";
import type { SearchState } from "./search-state";

async function fetchResults(s: SearchState) {
  const p = new URLSearchParams({ q: s.query, sort: s.sort, n: String(s.pageSize) });
  return (await fetch(`/api/search?${p}`)).json();
}

export function useSearchResults() {
  const [state] = useSearchContext();
  return useQuery({ queryKey: ["search", state], queryFn: () => fetchResults(state) });
}
```

Consumers: `features/search/components/search-box.tsx` uses `useSearchQuery`;
`features/search/components/results-list.tsx` uses `useSearchResults`;
`app/search/layout.tsx` wraps in `SearchProvider`.

## Task

Add a "clear search" control to `search-box.tsx` that resets query, sort and page
size back to their initial values. Make the change. Decide where the new/changed
code lives and explain in one or two sentences.
