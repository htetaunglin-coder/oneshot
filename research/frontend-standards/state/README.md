# State placement: research summary

Where a piece of state lives and which tool holds it, in a feature-organised React + Next.js App Router + TS app. Quotes, URLs, authors, dates and "silent on" notes in `sources.md`. Fetched 2026-09-17; Next.js docs read locally (next@16.2.10).

Verification: every source is an official doc or a verified maintainer/educator blog (author line per source in `sources.md`); none excluded; no Medium/dev.to/aggregator/LLM-summary content used. Pre-2022 posts (TkDodo 2020–21, Kent 2019–21, Erikson 2021) were checked against React 19 / TanStack v5 / Zustand v5 docs; all still hold, with one API rename (`useContext` → `use`).

## Consensus

| Question | Verdict | Sources that agree |
|---|---|---|
| Value computable from props/state | Derive in render; never `useState` + `useEffect` sync. `useMemo` for expensive calcs, "Measure first!". | React (State Structure, No Effect), Kent (Derive It), TkDodo (Don't over useState), Vercel `rerender-derived-state-no-effect` |
| Props copied to `useState` | Initial value read once; copying is a bug unless a deliberate draft. Fix: lift (controlled), `key` remount, or conditional mount. Never an effect. | React (Don't mirror props, `key`), TkDodo (Props to useState), Vercel `state-lift-state` |
| Local state location | As close to use as possible; lift only to closest common parent. One owner per piece. | React (Sharing State), Kent (Colocation), Vercel `state-lift-state` |
| `useState` vs `useReducer` | Equivalent; reducer when many handlers update related state or update bugs recur. Actions describe events. | React (Reducer), Erikson, TkDodo |
| "Server state" | Server-owned, async, shared ownership, goes stale. A cache, not state. | TanStack (Overview, Does-this-replace), TkDodo (Practical RQ), Kent (Server Cache vs UI State) |
| Server data into store | Do not copy `useQuery` data into `useState`/Zustand; it opts out of sync. Exception (TkDodo): form defaults. | TkDodo, TanStack, React |
| Client state feeding a query | Filter/page lives in client or URL state and goes into `queryKey`. | TkDodo, Next.js (`searchParams` prop loads data) |
| Context's job | Transport / DI, not storage; state lives in `useState`/reducer/store above the provider. All consumers re-render on change. | Erikson, React (Context), Kent, Zustand README |
| Before context | Props; extract components, pass `children`; then context. | React, Kent |
| Context scope | Many small contexts, not global by default; providers "as deep as possible". | Kent, React, Next.js |
| Context module shape | One file: `XProvider` + `useX()` that throws outside provider; no default value. | React (Scaling Up), Kent, Zustand, TkDodo |
| Zustand store shape | Actions colocated in store; export selector hooks, not raw store; atomic selectors (`useShallow` for multi-pick). Actions under one `actions` key: TkDodo only. | Zustand (No-store-actions, Selectors, README), TkDodo (Working with Zustand) |
| Zustand in App Router | No module-level global store; create per request in a `'use client'` provider via `useState(() => createStore())`; expose `useXStore(selector)`; RSCs never touch it. | Zustand (Next.js guide), Vercel `server-no-shared-module-state`, TkDodo |
| Store-in-context | When state initialises from props, needs per-subtree instances, or test isolation. | Zustand (Init with props), TkDodo (Zustand+Context) |
| State in Server Components | None: no state, context, hooks. State starts at `'use client'`; put the boundary on the leaf. | Next.js, Zustand |
| URL on server vs client | `searchParams` prop (async, dynamic rendering) to load data; `useSearchParams` in `Suspense` for client-only filtering. | Next.js, nuqs |

## Disputed

- **One store vs many.** Zustand Flux guide: "global state should be located in a single Zustand store", split by slices. TkDodo and Zustand TS guide: multiple small domain stores.
- **Actions in store vs module-level.** Zustand: colocate is "recommended", module-level `setState` actions "doesn't offer any downsides". TkDodo: `actions` object inside store.
- **Context + useReducer as app store.** React docs: for "a complex screen". Erikson: moderate complexity only; splitting contexts to avoid re-renders is "reinventing React-Redux, poorly". Zustand README: "Renders components only on changes" over context.
- **Reducer for derived values.** Kent shows it as acceptable, prefers derive-on-render.

## Not sourced (house opinion candidates)

- Store/provider file location inside a feature folder. Zustand's Next.js guide uses global `src/stores/` + `src/providers/`; every other source is silent.
- Complete list of what belongs in the URL. Next.js: pagination/filtering, dialog-open; nuqs SEO: "local-only state" vs "defining what content the page is displaying". No full rule anywhere.
- "Form state belongs to a form library, never a store." No source says it; React/Next only show `useActionState`.
- The decision ladder (derive → local → lift → URL → server → store-in-context → global store). Rungs sourced; order is synthesis.

## Delta vs Vercel

composition-patterns `state-*` rules (only three exist; author Fernando Rojo, 2026-01):

- `state-lift-state` (HIGH): lift into a provider so siblings outside the visual tree can read/act; effect-sync and ref-read are the bad shapes.
- `state-context-interface` (HIGH): context typed `{ state, actions, meta }`; UI consumes the interface.
- `state-decouple-implementation` (MEDIUM): provider is the only place that knows if state is `useState`, Zustand, or server sync.
- Related `architecture-compound-components` (HIGH): subparts read context, not props.
- Same repo, `react-best-practices`: `rerender-derived-state-no-effect`, `rerender-derived-state`, `rerender-lazy-state-init`, `server-no-shared-module-state` already cover derive-on-render, lazy init, no module-level request state.

NOT covered by Vercel: room for the house rule

- Which tool holds a given piece of state (the ladder). Vercel assumes state is already in a provider and only says how to expose it.
- Server vs client state definition; no `useQuery` data in stores.
- URL as a state location: `searchParams` prop vs `useSearchParams` vs nuqs; `Suspense`; shallow vs server-notifying updates; what belongs there.
- Zustand: per-request store, provider via `useState(() => createStore())`, selector-hook-only exports, atomic selectors, actions object, one-vs-many stores.
- Context vs Zustand criteria (re-render granularity, props-init, test isolation).
- `useState` vs `useReducer`.
- Props → state copying and `key` reset (Vercel flags effect-sync only in passing).
- `'use client'` boundary placement for state.
- Store/provider file location in a feature folder.
- Next.js 16 `<Activity>` preservation: what to reset and how.

## Next.js facts (local docs, next@16.2.10)

- `searchParams` page prop: `Promise<{[k]: string | string[] | undefined}>`; `await` or `use()`. Request-time API; opts page into dynamic rendering. Typed via global `PageProps<'/route'>`.
- `useSearchParams`: client-only, read-only `URLSearchParams`. In a prerendered route it forces CSR up to nearest `Suspense`; static production build fails without `Suspense`.
- Docs' split: `searchParams` prop to load data; `useSearchParams` for client-only filtering; `new URLSearchParams(window.location.search)` in handlers to avoid re-renders.
- URL writes: `router.push`/`Link`, or native `history.pushState/replaceState`, both sync with `usePathname`/`useSearchParams`.
- Client Components for state, handlers, effects, browser APIs, custom hooks. `'use client'` is a module-graph boundary; keep it on leaf components. Crossing props must be serializable. Server output passes as `children` into stateful client components.
- Context unsupported in Server Components; providers are `'use client'` and wrap `{children}` "as deep as possible".
- Layouts preserve state across navigation and do not re-render.
- Client fetching: `use(promise)` streamed from a server component, or SWR / React Query.
- `useActionState` returns `pending` and result for server actions; after an action, "Client state is preserved for re-rendered components".
- With Cache Components, routes stay mounted in `<Activity>` (up to 3); state and DOM survive navigation. Reset transient UI (dropdowns) in `useLayoutEffect` cleanup; derive dialog-open from a search param.
