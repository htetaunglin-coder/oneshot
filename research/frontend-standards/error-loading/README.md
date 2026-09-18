# Error, loading, empty, not-found states: fact sheet + delta

React 19.2 + Next.js App Router (next@16.2.10, local docs). Quotes, paths, authors, dates in `sources.md` (§ below). Fetched 2026-09-18.

## Facts

| Claim | Source |
|---|---|
| Expected errors are return values shown via `useActionState` state; uncaught exceptions throw to the nearest `error.tsx`; errors bubble up | Next §1, §4 |
| `error.tsx` is `'use client'`; props `error: Error & { digest?: string }`, `unstable_retry` (re-fetch, v16.2), `reset` (no re-fetch) | Next §4 |
| Production: Server Component errors arrive as "a generic message with an identifier"; `digest` matches server logs; client errors keep the message | Next §4 |
| `error.tsx` wraps `loading`, `not-found`, `page`, nested layouts; not its own `layout`/`template`; root layout → `global-error.tsx` with own `<html>`/`<body>` | Next §4 |
| `unstable_catchError` (v16.2): component-level boundary that ignores `redirect`/`notFound` throws | Next §11 |
| Boundaries miss event handlers, async callbacks, SSR, own errors; `startTransition` throws are caught; store handler errors in `useState` | React §18, Next §1 |
| Class or `react-error-boundary`; granularity: list and each item, not each avatar | React §18 |
| `redirect()` throws `NEXT_REDIRECT`, `notFound()` throws `NEXT_HTTP_ERROR_FALLBACK;404`; `never`-typed; a `try` catches them → call outside `try` or `unstable_rethrow(err)` first in `catch` | Next §8–10 |
| `loading.tsx` = Suspense around `page` and below, nested inside `layout`; no props; prefetched | Next §2, §5 |
| Layout's uncached data is not covered by `loading.tsx`; navigation blocks; docs prefer `<Suspense>` near the data | Next §2, §5, §12 |
| Streamed responses are 200; `notFound()` after streaming starts → `noindex` only; put it before any suspending `await` | Next §5, §6 |
| `not-found.tsx` sits inside `loading`'s Suspense and `error`'s boundary; no props; root one serves unmatched URLs. `forbidden()`/`unauthorized()` → 403/401 (experimental) | Next §6, §7 |
| Mid-stream throw: nearest `error.tsx` replaces only that section | Next §12 |
| Suspense subtree reveals together; nest for a sequence; ignores fetches in effects/handlers; SSR error → fallback, client retry, then boundary | React §17 |
| Re-suspend re-shows the fallback unless in `startTransition`/`useDeferredValue`; `useTransition` `isPending` = inline indicator | React §17, §20 |
| `useActionState` → `[state, dispatch, isPending]`; known errors returned, unknown thrown to boundary; `isPending` needs `startTransition` or form `action` | React §19 |
| Updates after `await` in `startTransition` need another `startTransition`; rejected promise → boundary | React §20 |
| `useOptimistic` equals `value` unless an Action pends; rolls back on failure | React §21 |
| Forms: `disabled={pending}`, `<p aria-live="polite">`, zod `fieldErrors` in state | Next §1, §12 |
| TanStack: `fetch` must throw on `!res.ok`; levels: `error` prop, global `onError`, boundary via `throwOnError` (fn form: 5xx up, 4xx local) | TkDodo §22 |
| Status `pending|error|success`; `isFetching` orthogonal; background failure keeps stale `data`; retries 3x; check `data` first; "no clear principle" | TkDodo §23 |
| `aria-busy` true→false around loading; `role="status"` = polite+atomic, no focus; prefer `polite`; live region must pre-exist | MDN §24–26 |

## Delta vs Vercel

- nextjs `references/error-handling.md`: `error.tsx`/`global-error.tsx` (old `reset`), navigation APIs out of `try`, `unstable_rethrow`, forbidden/unauthorized, `not-found.tsx`, hierarchy tree.
- nextjs `references/suspense-boundaries.md`: `useSearchParams`/`usePathname` need Suspense (CSR bailout).
- nextjs `references/data-patterns.md`: per-section Suspense skeletons; client `useEffect` fetch with `if (!data) return <Loading />`, no error branch. `app-router-files.md`, `file-conventions.md`: file tables only.
- react-best-practices `async-suspense-boundaries` (HIGH): Suspense around data components; layout-shift trade-off.
- react-best-practices `rendering-usetransition-loading` (LOW): `useTransition` over manual `isLoading`.
- react-best-practices `rerender-transitions` (MEDIUM): non-urgent updates. `rendering-conditional-render` (LOW): ternary, not `&&`. `client-swr-dedup`, `server-auth-actions`, `rendering-hoist-jsx`, `bundle-conditional`: incidental code only.
- composition-patterns (GitHub, absent from plugin 0.49.2): ten rules, zero hits.
- shadcn `SKILL.md`: "Empty/loading/error states | Card + Skeleton + Alert"; anti-pattern "without design treatment".
- verification `SKILL.md`: checklist "Missing error boundary".

NOT covered: room for a house rule

- Expected-vs-unexpected split.
- `error.message` generic in production; `digest`.
- `error.tsx` not wrapping its own layout; `unstable_retry` vs `reset`; `unstable_catchError`.
- `loading.tsx` scope; layout data blocking; Suspense-near-data.
- Streaming 200/`noindex`; `notFound()` before suspending awaits.
- Handler/async errors; fallback re-flash.
- Action pending UI: `isPending`, `disabled`, `useOptimistic` rollback.
- Empty state as its own branch and its order against pending/error.
- TanStack levels, `throwOnError` routing, data-first check.
- All a11y: `aria-busy`, `role="status"`, `aria-live`.

## Not sourced

- File location of error/loading/empty components in a feature folder. All silent.
- Fixed check order (pending → error → empty → data). TkDodo: data-first, "no clear principle".
- "Every list needs an empty state." shadcn design note only.
- Skeleton vs spinner; toast vs inline; boundary per feature vs per route. Examples, no rule.

## Gotchas an agent gets wrong (docs only)

1. `redirect()`/`notFound()` inside `try` is swallowed; move out or `unstable_rethrow` first.
2. `error.message` is generic in production for server errors; use `digest`.
3. `error.tsx` cannot catch its own `layout.tsx`; root needs `global-error.tsx` with `<html>`/`<body>`.
4. `loading.tsx` skips data fetched in the same segment's layout; navigation blocks.
5. Status is 200 once streaming starts; `notFound()` after a suspending `await` gives `noindex` only.
6. Boundaries miss `onClick`/`setTimeout`/promise errors; `useState` them.
7. `isPending` stays false when `dispatch` runs outside `startTransition`/form `action`.
8. Set-state after `await` inside `startTransition` is not a transition.
9. `useEffect`+`useState` fetching never triggers Suspense or `loading.tsx`.
10. Re-suspend on update flashes the fallback; use a transition.
11. TanStack + `fetch` without `throw` on `!res.ok` → no error state.
12. Live region rendered with its message pre-filled is not announced.
13. `{items.length && <Empty />}` renders `0`.
