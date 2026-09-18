---
title: State, what the repository decides and what leaks
impact: MEDIUM
tags: state, store, provider, request-data, zustand
---

# State: what the repository decides and what leaks

Which tool holds a value, and the shape of the store, is the repository's choice: its store library, its URL-state tool, its form library. This rule covers the one shape that leaks.

## Request data

A store that holds request data, the session, the org, a flag map, is created per request, inside its provider, `useState(() => createStore(init))`, never at module level: on the server one module serves every request, so a module-level store or `let` leaks one user's data into another's render. An existing module-level store converts when the change touches it, even when the request asks to "add it the same way". Vercel `server-no-shared-module-state`: module scope on the server is process-wide memory. Where the provider lives is the colocation rule (see [colocation](colocation.md)).

```tsx
// Incorrect: one store object for every request the server handles
export const useSessionStore = create<SessionState>()(() => initialState);

// Correct: one store per provider instance, created on the client tree
"use client";
export function SessionProvider({ session, children }: SessionProviderProps) {
  const [store] = useState(() => createSessionStore(session));
  return <SessionStoreContext value={store}>{children}</SessionStoreContext>;
}
```

## Sources

Zustand docs: Setup with Next.js (per-request store, no global stores, server components never touch the store); Initialize state with props. TkDodo: Zustand and React Context. Vercel `react-best-practices`: `server-no-shared-module-state`.
