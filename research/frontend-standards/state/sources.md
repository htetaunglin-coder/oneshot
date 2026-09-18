# Sources: where state lives (React + Next.js App Router + TS)

Fetched 2026-09-17. Primary sources only: official docs, maintainer blogs, and the Vercel rules repo. No search engine, no Medium/dev.to/freeCodeCamp/LogRocket/aggregators/LLM-summary sites were used. Quotes are verbatim; nothing is paraphrased. "Silent on" lists questions from the brief the source does not address. Author verification is via the project itself, the author's GitHub profile, or the TanStack maintainers page, as noted.

Current versions used for validity checks: React 19 (react.dev, last commits 2025-11 to 2026-04), Next 16.2.10 (local docs), Zustand v5.0.15 (2026-08-13), nuqs v2.10.1 (2026-08-25), TanStack Query v5.90.x (2026-09-16).

---

## 1. Vercel agent-skills, composition-patterns rules

Repo: https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns/rules
Author: Fernando Rojo (GitHub `nandorojo`, "Head of v0 @ Vercel"); all four rule files below were last committed by him, 2026-01-26 (verified via commit history). Repo owner: vercel-labs.
Date: rules last modified 2026-01-26 (`state-*`), section file first committed 2026-01-28.
Read via GitHub API. Files present: `_sections.md`, `_template.md`, `architecture-avoid-boolean-props.md`, `architecture-compound-components.md`, `patterns-children-over-render-props.md`, `patterns-explicit-variants.md`, `react19-no-forwardref.md`, `state-context-interface.md`, `state-decouple-implementation.md`, `state-lift-state.md`. Only three `state-*` rules exist.

`_sections.md`: "State Management (state) — Impact: MEDIUM — Patterns for lifting state and managing shared context across composed components."

### state-lift-state (impact HIGH)
- Claim: "Move state management into dedicated provider components. This allows sibling components outside the main UI to access and modify state without prop drilling or awkward refs."
- Bad shapes: (a) `useState` trapped inside `ForwardMessageComposer`, sibling `ForwardButton` cannot reach it; (b) `useEffect(() => { onInputChange(state.input) }, [state.input])` with comment "Sync on every change 😬"; (c) reading state from a `stateRef` on submit.
- Good shape: `ForwardMessageProvider` holds `useState(initialState)` + `useRef` + `useForwardMessage()` and renders `<Composer.Provider state={state} actions={{ update: setState, submit: forwardMessage }} meta={{ inputRef }}>`; `ForwardButton` does `const { actions } = use(Composer.Context)`.
- "Key insight: Components that need shared state don't have to be visually nested inside each other—they just need to be within the same provider."

### state-context-interface (impact HIGH)
- "Define a generic interface for your component context with three parts: `state`, `actions`, and `meta`. This interface is a contract that any provider can implement—enabling the same UI components to work with completely different state implementations."
- "Core principle: Lift state, compose internals, make state dependency-injectable."
- Bad shape: `ComposerInput` calls `useChannelComposerState()` directly, comment "Tightly coupled to a specific hook".
- Good shape: `interface ComposerContextValue { state: ComposerState; actions: ComposerActions; meta: ComposerMeta }`; `createContext<ComposerContextValue | null>(null)`; two providers (`ForwardMessageProvider` with local `useState`, `ChannelProvider` with `useGlobalChannel(channelId)`) feed the same UI.
- "The UI is reusable bits you compose together. The state is dependency-injected by the provider. Swap the provider, keep the UI."

### state-decouple-implementation (impact MEDIUM)
- "The provider component should be the only place that knows how state is managed. UI components consume the context interface—they don't know if state comes from useState, Zustand, or a server sync."
- Bad shape: `ChannelComposer({ channelId })` calls `useGlobalChannelState(channelId)` and `useChannelSync(channelId)` inside the UI component ("UI component knows about global state implementation").
- Good shape: `ChannelProvider` wraps `useGlobalChannel` and renders `<Composer.Provider state actions meta>`; `ChannelComposer` is pure composition of `Composer.*` parts.

### architecture-compound-components (impact HIGH; context-related)
- "Structure complex components as compound components with a shared context. Each subcomponent accesses shared state via context, not props. Consumers compose the pieces they need."
- Good shape: `ComposerProvider({ children, state, actions, meta })` + `const Composer = { Provider, Frame, Input, Submit, ... }`; subparts call `use(ComposerContext)`.

Silent on: which tool holds state (useState vs reducer vs Zustand vs URL vs server); server vs client state; URL state; file location; Server Components.

### Supplemental: same repo, `react-best-practices` rules that touch state
Authors (last commit): Shu Ding (`shuding`, "@Vercel", Next.js core) for `rerender-derived-state-no-effect` (2026-01-23) and `server-no-shared-module-state` (2026-03-31); Rex (`rexwangcc`) for `rerender-derived-state` (2026-01-16); Andrew Qu (`quuu`) for `rerender-lazy-state-init` (2026-01-14). All under vercel-labs.
- `rerender-derived-state-no-effect` (MEDIUM): "If a value can be computed from current props/state, do not store it in state or update it in an effect. Derive it during render to avoid extra renders and state drift. Do not set state in effects solely in response to prop changes; prefer derived values or keyed resets instead." Bad: `fullName` in `useState` + `useEffect`. Good: `const fullName = firstName + ' ' + lastName`.
- `rerender-derived-state` (MEDIUM): "Subscribe to derived boolean state instead of continuous values to reduce re-render frequency." Bad: `useWindowWidth()` then `width < 768`. Good: `useMediaQuery('(max-width: 767px)')`.
- `rerender-lazy-state-init` (MEDIUM): "Pass a function to `useState` for expensive initial values. Without the function form, the initializer runs on every render even though the value is only used once." "For simple primitives (`useState(0)`), direct references (`useState(props.value)`), or cheap literals (`useState({})`), the function form is unnecessary."
- `server-no-shared-module-state` (HIGH): "For React Server Components and client components rendered during SSR, avoid using mutable module-level variables to share request-scoped data." "Treat module scope on the server as process-wide shared memory, not request-local state."

---

## 2. React docs (react.dev)

Author: React team (official docs; repo reactjs/react.dev). Dates below are last commit on each page's source file.

### Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure (2026-04-08)
- "If you always update two or more state variables at the same time, consider merging them into a single state variable."
- "If you can calculate some information from the component's props or its existing state variables during rendering, you should not put that information into that component's state."
- "When the same data is duplicated between multiple state variables, or within nested objects, it is difficult to keep them in sync. Reduce duplication when you can."
- "Deeply hierarchical state is not very convenient to update. When possible, prefer to structure state in a flat way."
- Don't mirror props in state: "if the parent component passes a different value of `messageColor` later (for example, `'red'` instead of `'blue'`), the `color` state variable would not be updated!"
- "instead of a `selectedItem` object (which creates a duplication with objects inside `items`), you hold the `selectedId` in state, and then get the `selectedItem` by searching the `items` array for an item with that ID."

### Sharing State Between Components — https://react.dev/learn/sharing-state-between-components (2026-04-08)
- "Remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as lifting state up, and it's one of the most common things you will do writing React code."
- "For each unique piece of state, you will choose the component that 'owns' it. This principle is also known as having a 'single source of truth' It doesn't mean that all state lives in one place--but that for each piece of state, there is a specific component that holds that piece of information."
- "you might say a component is 'controlled' when the important information in it is driven by props rather than its own local state." / "It is common to call a component with some local state 'uncontrolled'."

### Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state (2026-04-09)
- "React keeps state for as long as the same component is rendered at the same position."
- "You can force a subtree to reset its state by giving it a different key."

### Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer (2023-12-15)
- "We recommend using a reducer if you often encounter bugs due to incorrect state updates in some component, and want to introduce more structure to its code. You don't have to use reducers for everything: feel free to mix and match!"
- "`useState` is very easy to read when the state updates are simple. When they get more complex, they can bloat your component's code and make it difficult to scan. In this case, `useReducer` lets you cleanly separate the how of update logic from the what happened of event handlers."
- "A reducer is a pure function that doesn't depend on your component. This means that you can export and test it separately in isolation."
- "Each action describes a single user interaction."

### Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context (2026-04-09)
- "Start by passing props. If your components are not trivial, it's not unusual to pass a dozen props down through a dozen components."
- "Extract components and pass JSX as `children` to them. If you pass some data through many layers of intermediate components that don't use that data (and only pass it further down), this often means that you forgot to extract some components along the way."
- "If neither of these approaches works well for you, consider context."
- Use cases: "Theming", "Current account" ("Many components might need to know the currently logged in user."), "Routing", "Managing state" ("It is common to use a reducer together with context to manage complex state.")
- "If you pass a different value on the next render, React will update all the components reading it below!"

### Scaling Up with Reducer and Context — https://react.dev/learn/scaling-up-with-reducer-and-context (2026-04-08)
- "You can combine reducers and context together to manage state of a complex screen."
- "You don't have to do this, but you could further declutter the components by moving both reducer and context into a single file."
- "You can export a component like `TasksProvider` that provides context. You can also export custom Hooks like `useTasks` and `useTasksDispatch` to read it."

### You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect (2025-11-10)
- "When something can be calculated from the existing props or state, don't put it in state. Instead, calculate it during rendering."
- "You can cache (or 'memoize') an expensive calculation by wrapping it in a `useMemo` Hook"
- "By passing `userId` as a `key` to the `Profile` component, you're asking React to treat two `Profile` components with different `userId` as two different components that should not share any state."
- "Storing information from previous renders like this can be hard to understand, but it's better than updating the same state in an Effect." / "Always check whether you can reset all state with a key or calculate everything during rendering instead."
- "Keep in mind that modern frameworks provide more efficient built-in data fetching mechanisms than writing Effects directly in your components."

### useState reference — https://react.dev/reference/react/useState (2026-04-09)
- "If you pass a function as `initialState`, it will be treated as an initializer function. It should be pure, should take no arguments, and should return a value of any type. React will call your initializer function when initializing the component, and store its return value as the initial state."
- "React saves the initial state once and ignores it on the next renders."
- "Although the result of `createInitialTodos()` is only used for the initial render, you're still calling this function on every render. This can be wasteful if it's creating large arrays or performing expensive calculations."

### useSyncExternalStore reference — https://react.dev/reference/react/useSyncExternalStore (2026-04-08)
- "`useSyncExternalStore` is a React Hook that lets you subscribe to an external store."
- "Most of your React components will only read data from their props, state, and context. However, sometimes a component needs to read some data from some store outside of React that changes over time. This includes: Third-party state management libraries that hold state outside of React. Browser APIs that expose a mutable value and events to subscribe to its changes."

Silent on (all React pages): Zustand vs context choice; URL state; server state as a category (only "frameworks provide ... data fetching mechanisms"); file location.

---

## 3. TkDodo (tkdodo.eu/blog)

Author: Dominik Dorfmeister (GitHub `TkDodo`, blog tkdodo.eu). Listed on https://tanstack.com/maintainers as a TanStack maintainer; 444 commits to TanStack/query, latest 2026-09-15. Also a Zustand contributor by his own statement ("I've also contributed a bit to the library").

### Working with Zustand — https://tkdodo.eu/blog/working-with-zustand (2022-11-20)
- "Only export custom hooks ... This is my number one tip for working with… everything in React, really." Code comment: `// ⬇️ not exported, so that no one can subscribe to the entire store`.
- "While selectors are optional in Zustand, I think they should always be used. Even if we have a store with just a single state value, I'd write a custom hook solely to be able to add more state in the future."
- "Prefer atomic selectors ... Zustand decides when to inform your component that the state it is interested in has changed, by comparing the result of the selector with the result of the previous render."
- "Separate Actions from State. Actions are functions which update values in your store. These are static and never change, so they aren't technically 'state'. Organising them into a separate object in our store will allow us to expose them as a single hook to be used in any our components without any impact on performance" — shape: `create((set) => ({ bears: 0, fish: 0, actions: { increasePopulation, eatFish } }))`; `useBearActions = () => useBearStore((state) => state.actions)`.
- "Model Actions as Events, not Setters ... It will help you keep your business logic inside your store, and not in your components ... The component just calls the action, and the store decides what to do with it."
- "Keep the scope of your store small. Unlike Redux, where you're supposed to have a single store for your whole app, Zustand encourages you to have multiple, small stores. Each store can be responsible for a single piece of state."
- "I honestly haven't needed to combine multiple Zustand stores very often, because most of the state in apps is either server or url state. I'm far more likely to combine a Zustand store with useQuery or useParams, for example, than I am to combine two separate stores."
- Validity in Zustand v5: `create`, selector hooks, and `useShallow` are unchanged; the `shallow` second-argument form shown in the post was removed in v5 in favour of `useShallow` (Zustand README 2026 shows `useShallow`).

### Zustand and React Context — https://tkdodo.eu/blog/zustand-and-react-context (2024-04-14)
- "Zustand is a great lib for global client-state management. It's simple, fast, and has a small bundle size. There is however one thing I don't necessarily like about it: The stores are global."
- On init-from-props via effect: "We don't really initialize our store with initialBears - we sync it."
- "I think it all stems from the fact that the store is global. If it were scoped to a component subtree, we could render those components and the store would be isolated to it, not needing any of those 'workarounds'."
- Shape: `const [store] = React.useState(() => createStore((set) => ({ bears: initialBears, actions: {...} })))` inside `BearStoreProvider`; "we have to make sure that the creation of the store only happens once. We can do this with refs, but I prefer useState for that."
- "The useState initializer function only runs once, so updates to the prop will not be passed to the store."
- Hook shape: `const useBearStore = (selector) => { const store = React.useContext(BearStoreContext); if (!store) { throw new Error('Missing BearStoreProvider') } return useStore(store, selector) }`
- "knowing how to combine store creation with React Context can come in quite handy in situations where encapsulation and reusability are required. I for one have used this abstraction more than truly global zustand stores."
- Post states it shows "the v5 syntax for combining zustand with React Context".

### React Query as a State Manager — https://tkdodo.eu/blog/react-query-as-a-state-manager (2021-08-20)
- "React Query is an async state manager. It can manage any form of asynchronous state - it is happy as long as it gets a Promise."
- "It is a proper, real, 'global state manager'. The QueryKey uniquely identifies your query, so as long you call the query with the same key in two different places, they will get the same data."
- "Because React Query manages async state (or, in terms of data fetching: server state), it assumes that the frontend application doesn't 'own' the data. And that's totally right. If we display data on the screen that we fetch from an API, we only display a 'snapshot' of that data"
- "As long as data is fresh, it will always come from the cache only."
- "React Query is great at managing async state globally in your app, if you let it. Only turn off the refetch flags if you know that make sense for your use-case, and resist the urge to sync server data to a different state manager. Usually, customizing staleTime is all you need to get a great ux while also being in control of how often background updates happen."
- Validity in TanStack Query v5: "Important Defaults" (2026-04-01) still says queries "by default consider cached data as stale" and "Setting `staleTime` is the recommended way to avoid excessive refetches". Holds.

### Practical React Query — https://tkdodo.eu/blog/practical-react-query (2020-11-16, "Last Update: 2023-10-21")
- "when it comes to server state (think: A list of articles that you fetch, the details of a User you want to display, …), your app does not own it. We have only borrowed it to display the most recent version of it on the screen for the user. It is the server who owns the data."
- "If we can leverage the cache to display data that we do not own, there isn't really much left that is real client state that also needs to be made available to the whole app."
- "Keep server and client state separate ... If you get data from useQuery, try not to put that data into local state. The main reason is that you implicitly opt out of all background updates that React Query does for you, because the state 'copy' will not update with it."
- "This is fine if you want to e.g. fetch some default values for a Form, and render your Form once you have them."
- "Don't use the queryCache as a local state manager. If you tamper with the queryCache (queryClient.setQueryData), it should only be for optimistic updates or for writing data that you receive from the backend after a mutation. Remember that every background refetch might override that data"
- Filter example: "We would have some local state to store that filtering, and as soon as the user changes their selection, we would update that local state, and React Query will automatically trigger the refetch for us, because the query key changes."
- Validity: updated 2023-10 for v5; TanStack's own "Does this replace" page (2024) makes the same server/client split. Holds.

### Putting props to useState — https://tkdodo.eu/blog/putting-props-to-use-state (2020-10-25)
- "The initial value of a useState hook is always discarded on re-renders - it only has an effect when the component mounts."
- Three fixes: conditional render ("Showing the DetailView in a Modal will magically make our code above work, because Modals are usually rendered conditionally."); "take the draft state and move it further up the tree, thus making our DetailView a fully controlled component"; key: "you can also just put a key attribute on any component to tell React: 'Please mount this whenever the key changes. As long as the key is the same, please re-render'."
- "I would consider effects like these generally an anti-pattern. If effects are used for syncing, they should be used to sync React state with something outside of React, e.g. with localstorage. But here, we are syncing something that already lives inside React with React state."
- "Whatever you decide, please, don't use the syncing state 'solution'."
- Validity in React 19: useState reference (2026) "React saves the initial state once and ignores it on the next renders"; You Might Not Need an Effect (2025) documents the `key` reset. Holds.

### Don't over useState — https://tkdodo.eu/blog/dont-over-use-state (2020-08-29)
- "Can you compute it based on any other state or props in your component? If so, it isn't state."
- "Utilizing useEffect to sync two react states is rarely right."
- "Whenever a state setter function is only used synchronously in an effect, get rid of the state!"
- Validity in React 19: matches Choosing the State Structure (2026) "you should not put that information into that component's state". Holds.

### Things to know about useState — https://tkdodo.eu/blog/things-to-know-about-use-state (2021-02-13)
- Lazy initializer: "we can pass a function as initial value to useState. React will only invoke this function when it really needs the result (= when the component mounts)."
- Bailout: "you try to update to the same value that your state is currently holding. React uses Object.is to determine if the values are different."
- Validity in React 19: initializer function documented in useState reference (2026). Holds.

Silent on (all TkDodo posts): file location in a feature folder; Next.js Server Components; form libraries.

---

## 4. Zustand docs

Author: the pmndrs Zustand project (official docs, repo pmndrs/zustand). Maintainer: Daishi Kato (GitHub `dai-shi`, bio "React library author, maintaining three state management libraries, Zustand🐻, Jotai👻, Valtio🧙"; 425 commits to pmndrs/zustand). Docs are multi-contributor; the project is the author of record.
Note on URLs: the `/guides/<slug>` paths in the brief return 404. Live paths are `https://zustand.docs.pmnd.rs/learn/guides/<slug>`; markdown read from `docs/learn/guides/` in the repo. Dates are last commit per file. Current release v5.0.15 (2026-08-13).

### Practice with no store actions — /learn/guides/practice-with-no-store-actions (2026-02-23)
- "The recommended usage is to colocate actions and states within the store (let your actions be located together with your state)."
- Alternative: `export const inc = () => useBoundStore.setState((state) => ({ count: state.count + 1 }))` — "It doesn't require a hook to call an action; It facilitates code splitting." "While this pattern doesn't offer any downsides, some may prefer colocating due to its encapsulated nature."

### Flux inspired practice — /learn/guides/flux-inspired-practice (2026-02-23)
- "Single store. Your applications global state should be located in a single Zustand store. If you have a large application, Zustand supports splitting the store into slices."
- "Always use `set` (or `setState`) to perform updates to your store."
- "Colocate store actions. In Zustand, state can be updated without the use of dispatched actions and reducers found in other Flux libraries. These store actions can be added directly to the store"
- Redux-like: "If you can't live without Redux-like reducers, you can define a `dispatch` function on the root level of the store"; `redux` middleware also offered.

### Slices pattern — /learn/guides/slices-pattern (2026-02-23)
- "Your store can become bigger and bigger and tougher to maintain as you add more features. You can divide your main store into smaller individual stores to achieve modularity."
- Shape: `export const createBearSlice = (set) => ({ bears: 0, addBear: ... })`; `export const useBoundStore = create((...a) => ({ ...createBearSlice(...a), ...createFishSlice(...a) }))`.
- "Please keep in mind you should only apply middlewares in the combined store. Applying them inside individual slices can lead to unexpected issues."

### Initialize state with props — /learn/guides/initialize-state-with-props (2026-02-23)
- "In cases where dependency injection is needed, such as when a store should be initialized with props from a component, the recommended approach is to use a vanilla store with React.context."
- Shape: `createBearStore(initProps?: Partial<BearProps>)` returning `createStore<BearState>()(...)`; `export const BearContext = createContext<BearStore | null>(null)`; `const [store] = useState(() => createBearStore(props))`; hook `useBearContext<T>(selector)` that throws `'Missing BearContext.Provider in the tree'`.

### Auto generating selectors — /learn/guides/auto-generating-selectors (2026-02-23)
- "We recommend using selectors when using either the properties or actions from the store."
- Provides `createSelectors` adding `store.use.bears()`.

### How to reset state — /learn/guides/how-to-reset-state (2026-02-23)
- Shape: `create<State & Actions>()((set, get, store) => ({ ..., reset: () => { set(store.getInitialState()) } }))`.

### Setup with Next.js — /learn/guides/nextjs (2026-08-17)
- "Keep in mind that Zustand store is a global variable (AKA module state) making it optional to use a `Context`."
- "Per-request store: A Next.js server can handle multiple requests simultaneously. This means that the store should be created per request and should not be shared across requests."
- "No global stores - Because the store should not be shared across requests, it should not be defined as a global variable. Instead, the store should be created per request."
- "React Server Components should not read from or write to the store - RSCs cannot use hooks or context. They aren't meant to be stateful. Having an RSC read from or write values to a global store violates the architecture of Next.js."
- Shape: `src/stores/counter-store.ts` (`createCounterStore` via `createStore` from `zustand/vanilla`; `CounterState` + `CounterActions` types) and `src/providers/counter-store-provider.tsx` (`'use client'`; `const [store] = useState(() => createCounterStore())`; `useCounterStore(selector)` throws "`useCounterStore must be used within CounterStoreProvider`").
- Page banner: "We will be updating this guide soon based on our discussion in https://github.com/pmndrs/zustand/discussions/2740."

### Beginner TypeScript Guide — /learn/guides/beginner-typescript (2026-03-08)
- `export const useBearStore = create<BearState>()((set) => ({ ... }))` — "The `create` function uses the curried form".
- "Derived State with Selectors. Not all values need to be stored directly - some can be computed from existing state. You can derive values using selectors. This avoids duplication and keeps the store minimal."
- "Multiple Stores. You can create more than one store for different domains. For example, `BearStore` manages bears and `FishStore` manages fish. This keeps state isolated and easier to maintain in larger apps."

### README — https://github.com/pmndrs/zustand (2026-08-19)
- "Why zustand over context? Less boilerplate / Renders components only on changes / Centralized, action-based state management"
- "Why zustand over redux? Simple and un-opinionated / Makes hooks the primary means of consuming state / Doesn't wrap your app in context providers / Can inform components transiently (without causing render)"
- "Fetching everything. You can, but bear in mind that it will cause the component to update on every state change!"
- "It detects changes with strict-equality (old === new) by default, this is efficient for atomic state picks." Multi-pick: "you can use `useShallow` to prevent unnecessary rerenders".

Silent on: feature-folder placement (Next.js guide uses `src/stores/` and `src/providers/`); URL state (a separate "connect to state with URL hash" guide exists, not read); server state/TanStack Query.

---

## 5. nuqs

Author: François Best (GitHub `franky47`, bio "Building nuqs, a type-safe search params state manager for React"; org 47ng owns the repo). Official docs from `packages/docs/content/docs` in 47ng/nuqs. Current release v2.10.1 (2026-08-25). Dates are last commit per file.

- README (2026-07-20): "Type-safe search params state manager for React frameworks. Like `useState`, but stored in the URL query string." / "Simple: the URL is the source of truth" / "Shallow mode by default for URL query updates, opt-in to notify server components" / "Server cache for type-safe searchParams access in nested server components"
- Basic usage (2026-06-23): "If you are using `React.useState` to manage your local UI state, you can replace it with `useQueryState` to sync it with the URL." Shape: `'use client'`; `const [name, setName] = useQueryState('name')`; "Setting `null` as a value will remove the key from the query string." Parsers: `parseAsInteger.withDefault(0)`.
- Options (2026-07-12): "By default, state updates are done by replacing the current history entry with the updated query when state changes." / "By default, query state updates are done in a client-first manner: there are no network calls to the server. ... To opt-in to notifying the server on query updates, you can set `shallow` to `false`"
- Server-side (2026-08-21): `createLoader(coordinatesSearchParams)`; comment "Describe your search params, and reuse this in useQueryStates / createSerializer"; App Router page `const { latitude, longitude } = await loadSearchParams(searchParams)` with `searchParams: Promise<SearchParams>`. Cache: "If you wish to access the searchParams in a deeply nested Server Component (ie: not in the Page component), you can use `createSearchParamsCache` to do so in a type-safe manner. Think of it as a loader combined with a way to propagate the parsed values down the RSC tree, like Context would on the client." Comment: "import from 'nuqs/server' to avoid the 'use client' directive". "Loaders don't validate your data."
- Adapters (2026-08-21): App Router wraps `{children}` in `NuqsAdapter` from `nuqs/adapters/next/app` in the root layout.
- SEO (2025-06-27): "If your page uses query strings for local-only state, you should add a canonical URL to your page, to tell SEO crawlers to ignore the query string ... If however the query string is defining what content the page is displaying (eg: YouTube's watch URLs ...), your canonical URL should contain relevant query strings"

Silent on: a rule for which state belongs in the URL (only the "local UI state" replacement framing and the SEO "local-only" vs "defining what content" split); Zustand/context; file location.

---

## 6. Kent C. Dodds (kentcdodds.com/blog)

Author: Kent C. Dodds (GitHub `kentcdodds`, bio "Dev Educator ... EpicReact.dev"); author of Epic React, former React trainer. Dates as shown on each page.

### Application State Management with React — /blog/application-state-management-with-react (page dated July 21st, 2020)
- "React is a state management library"
- "Not everything in your application needs to be in a single state object. Keep things logically separated (user settings does not necessarily have to be in the same context as notifications). You will have multiple providers with this approach. Not all of your context needs to be globally accessible! Keep state as close to where it's needed as possible."
- "Don't reach for context too soon!"
- "every type of state can fall into one of two buckets: Server Cache - State that's actually stored on the server and we store in the client for quick-access (like user data). UI State - State that's only useful in the UI for controlling the interactive parts of our app (like modal isOpen state). We make a mistake when we combine the two. Server cache has inherently different problems from UI state and therefore needs to be managed differently."
- "If you embrace the fact that what you have is not actually state at all but is instead a cache of state, then you can start thinking about it correctly and therefore managing it correctly."
- Shape: `CountProvider` + `useCount()` throwing "`useCount must be used within a CountProvider`"; `increment` helper lives in the hook.
- Validity: pre-React 18; the server-cache/UI-state split matches TanStack Query v5 docs (2024/2026); context behaviour matches React 19 docs. Holds.

### State Colocation will make your React app faster — /blog/state-colocation-will-make-your-react-app-faster (September 23rd, 2019)
- "One of the leading causes to slow React applications is global state, especially the rapidly changing variety."
- "The principle of colocation is: Place code as close to where it's relevant as possible"
- "Where I see this principle apply in real-world applications is when people put things into a global Redux store or in a global context that don't really need to be global."
- "people are pretty good at 'lifting state' as things change, but we don't often think to 'colocate' state as things change in our codebase."
- Validity: React 19 Sharing State (2026) still lifts only to "closest common parent"; the React Compiler reduces but does not remove the re-render cost argument. Holds.

### Don't Sync State. Derive It! — /blog/dont-sync-state-derive-it (September 30th, 2019)
- "`nextValue`, `winner`, and `status` are what are called 'derived state.' That means that their value can be derived (or calculated) based on other values rather than managed on their own."
- "The biggest problem with this is some of that state may fall out of sync with the true component state"
- "We don't need to worry about updating the derived state values because they're simply calculated every render."
- "derived state can sometimes be even faster than state synchronization because it will result in fewer unnecessary re-renders" ; on useMemo: "Measure first!"
- Validity: identical to React 19 Choosing the State Structure / You Might Not Need an Effect. Holds.

### How to use React Context effectively — /blog/how-to-use-react-context-effectively (June 5th, 2021)
- "context does NOT have to be global to the whole app, but can be applied to one part of your tree and you can (and probably should) have multiple logically separated contexts in your app."
- "99% of the time that you're going to be creating and using context in your application, you want your context consumers (those using useContext) to be rendered within a provider which can provide a useful value."
- Shape: `count-context.js` exports `CountProvider` + `useCount` (throws if `undefined`); value is `{state, dispatch}`.
- "You shouldn't be reaching for context to solve every state sharing problem that crosses your desk."
- Validity: React 19 replaces `useContext` with `use(Context)` and `<Context>` as provider (Vercel `react19-no-forwardref` section note); the provider+hook module shape matches React 19 Scaling Up doc. Holds with API rename.

Silent on: Zustand; URL state; Next.js Server Components; file location.

---

## 7. Mark Erikson — Why React Context is Not a "State Management" Tool (and Why It Doesn't Replace Redux)
https://blog.isquaredsoftware.com/2021/01/context-redux-differences/ (2021-01-18)
Author: Mark Erikson (GitHub `markerikson`, bio "Redux maintainer"; 734 commits to reduxjs/redux).
- "Is Context a 'state management' tool? No. Context is a form of Dependency Injection. It is a transport mechanism - it doesn't 'manage' anything. Any 'state management' is done by you and your own code, typically via useState/useReducer."
- "Context does not 'store' anything itself. The parent component that renders a `<MyContext.Provider>` is responsible for deciding what value is passed into the context"
- "when useReducer produces a new state value, all components that are subscribed to that context will be forced to re-render ... With React-Redux, components can subscribe to specific pieces of the store state, and only re-render when those values change."
- "at that point you're just reinventing React-Redux, poorly."
- "If the only thing you need to do is avoid prop-drilling, then use Context. If you've got some moderately complex React component state, or just really don't want to use an external library, go with Context + useReducer. If you want better traceability of the changes to your state over time, need to ensure that only specific components re-render when the state changes, need more powerful capabilities for managing side effects, or have other similar problems, use Redux"
- "It's also important to point out that these are not mutually exclusive"
- Validity: predates `useSyncExternalStore` (React 18), which is now the documented way external stores subscribe; React 19 Passing Data Deeply (2026) still says a new context value updates "all the components reading it below". The Context-vs-store argument holds; "Redux" reads as "any subscribable store" (Zustand uses `useSyncExternalStore`).

Silent on: Zustand by name; URL state; server state; Server Components.

---

## 8. Local Next.js docs (next@16.2.10, `/Users/htetaunglin/Desktop/development/skills/node_modules/next/dist/docs/`)

Author: Vercel / Next.js team (official docs shipped in the package). Date: package version 16.2.10 (no per-page dates in the bundle).

### `01-app/03-api-reference/04-functions/use-search-params.md`
- "`useSearchParams` is a Client Component hook that lets you read the current URL's query string." / "returns a read-only version of the `URLSearchParams` interface"
- "`useSearchParams` is a Client Component hook and is not supported in Server Components to prevent stale values during partial rendering."
- "If you want to fetch data in a Server Component based on search params, it's often a better option to read the `searchParams` prop of the corresponding Page. You can then pass it down by props to any component (Server or Client) within that Page."
- "If a route is prerendered, calling `useSearchParams` will cause the Client Component tree up to the closest `Suspense` boundary to be client-side rendered."
- "During production builds, a static page that calls `useSearchParams` from a Client Component must be wrapped in a `Suspense` boundary, otherwise the build fails with the Missing Suspense boundary with useSearchParams error."
- "If a route is dynamically rendered, `useSearchParams` will be available on the server during the initial server render of the Client Component."
- "You can also pass the Page `searchParams` prop directly to a Client Component and unwrap it with React's `use()`. Although this will suspend, so the Client Component should be wrapped with a `Suspense` boundary."
- "You can use `useRouter` or `Link` to set new `searchParams`. After a navigation is performed, the current `page.js` will receive an updated `searchParams` prop." Shape: `router.push(pathname + '?' + createQueryString('sort', 'asc'))`.

### `01-app/03-api-reference/03-file-conventions/page.md`
- "`searchParams` (optional): A promise that resolves to an object containing the search parameters of the current URL." Type: `Promise<{ [key: string]: string | string[] | undefined }>`.
- "Since the `searchParams` prop is a promise. You must use `async/await` or React's `use` function to access the values."
- "`searchParams` is a Request-time API whose values cannot be known ahead of time. Using it will opt the page into dynamic rendering at request time."
- "`searchParams` is a plain JavaScript object, not a `URLSearchParams` instance."
- "Client Component pages can also access `searchParams` using React's `use` hook"
- "You can type pages with `PageProps` to get strongly typed `params` and `searchParams` from the route literal. `PageProps` is a globally available helper."

### `01-app/01-getting-started/03-layouts-and-pages.md`
- "On navigation, layouts preserve state, remain interactive, and do not rerender."
- "Using `searchParams` opts your page into dynamic rendering because it requires an incoming request to read the search parameters from."
- "Use the `searchParams` prop when you need search parameters to load data for the page (e.g. pagination, filtering from a database)."
- "Use `useSearchParams` when search parameters are used only on the client (e.g. filtering a list already loaded via props)."
- "As a small optimization, you can use `new URLSearchParams(window.location.search)` in callbacks or event handlers to read search params without triggering re-renders."

### `01-app/01-getting-started/04-linking-and-navigating.md`
- "Next.js allows you to use the native `window.history.pushState` and `window.history.replaceState` methods to update the browser's history stack without reloading the page. `pushState` and `replaceState` calls integrate into the Next.js Router, allowing you to sync with `usePathname` and `useSearchParams`."
- `pushState`: "Use it to add a new entry to the browser's history stack. The user can navigate back to the previous state. For example, to sort a list of products". `replaceState`: "The user is not able to navigate back to the previous state. For example, to switch the application's locale".

### `01-app/01-getting-started/05-server-and-client-components.md`
- "Use Client Components when you need: State and event handlers. E.g. `onClick`, `onChange`. Lifecycle logic. E.g. `useEffect`. Browser-only APIs. E.g. `localStorage`, `window`, `Navigator.geolocation`, etc. Custom hooks."
- "`'use client'` is used to declare a boundary between the Server and Client module graphs (trees)."
- "To reduce the size of your client JavaScript bundles, add `'use client'` to specific interactive components instead of marking large parts of your UI as Client Components."
- "Props passed to Client Components need to be serializable by React."
- "A common pattern is to use `children` to create a slot in a `<ClientComponent>`. For example, a `<Cart>` component that fetches data on the server, inside a `<Modal>` component that uses client state to toggle visibility."
- "React context is commonly used to share global state like the current theme. However, React context is not supported in Server Components. To use context, create a Client Component that accepts `children`"
- "You should render providers as deep as possible in the tree – notice how `ThemeProvider` only wraps `{children}` instead of the entire `<html>` document. This makes it easier for Next.js to optimize the static parts of your Server Components."

### `01-app/01-getting-started/06-fetching-data.md`
- "There are two ways to fetch data in Client Components, using: 1. React's `use` API 2. A community library like SWR or React Query"
- "These libraries have their own semantics for caching, streaming, and other features."

### `01-app/01-getting-started/07-mutating-data.md`
- "While executing a Server Function, you can show a loading indicator with React's `useActionState` hook. This hook returns a `pending` boolean" (`const [state, action, pending] = useActionState(createPost, false)`).
- "The server update applies to the current React tree, re-rendering, mounting, or unmounting components, as needed. Client state is preserved for re-rendered components, and effects re-run if their dependencies changed."

### `01-app/02-guides/preserving-ui-state.md` (requires `cacheComponents: true`)
- "Before Cache Components, preserving page-level state across navigations required workarounds like hoisting state to a shared layout or using an external store. With Cache Components, Next.js preserves state and DOM out of the box."
- "Instead of unmounting pages on navigation, Next.js hides them using React's `<Activity>` component." / "Next.js preserves up to 3 routes."
- "When to reset it: A dropdown menu or popover triggered by a button click. These are transient interactions, not persistent view state." Reset: `useLayoutEffect` cleanup calling `setIsOpen(false)`.
- "To fix this, derive the dialog state from something outside the preserved component state like a search param" / "With this approach, `isDialogOpen` derives from the URL rather than component state."
- "When to keep it: A search page with filters, a draft the user was composing, or a settings form with unsaved changes."

Silent on: Zustand, context-vs-store, form libraries, TanStack Query beyond "community library", feature-folder placement.

---

## 9. TanStack Query docs

Author: TanStack project (official docs, repo TanStack/query; maintainers listed at tanstack.com/maintainers include Tanner Linsley and Dominik Dorfmeister). Current v5.90.x (2026-09-16). Dates are last commit per file.

### Overview — https://tanstack.com/query/latest/docs/framework/react/overview (2026-09-02)
- "it makes fetching, caching, synchronizing and updating server state in your web applications a breeze."
- "While most traditional state management libraries are great for working with client state, they are not so great at working with async or server state. This is because server state is totally different. For starters, server state: Is persisted remotely in a location you may not control or own; Requires asynchronous APIs for fetching and updating; Implies shared ownership and can be changed by other people without your knowledge; Can potentially become 'out of date' in your applications if you're not careful"

### Does TanStack Query replace Redux, MobX or other global state managers? — /guides/does-this-replace-client-state (2024-01-25)
- "TanStack Query is a server-state library, responsible for managing asynchronous operations between your server and client"
- "Redux, MobX, Zustand, etc. are client-state libraries that can be used to store asynchronous data, albeit inefficiently when compared to a tool like TanStack Query"
- "For a vast majority of applications, the truly globally accessible client state that is left over after migrating all of your async code to TanStack Query is usually very tiny."
- "TanStack Query is not a replacement for local/client state management. However, you can use TanStack Query alongside most client state managers with zero issues."
- Example: `globalState = { projects, teams, tasks, users, themeMode, sidebarStatus }` → after: `{ themeMode, sidebarStatus }`.

### Important Defaults — /guides/important-defaults (2026-04-01)
- "Query instances via `useQuery` or `useInfiniteQuery` by default consider cached data as stale."
- "Stale queries are refetched automatically in the background when: New instances of the query mount; The window is refocused; The network is reconnected"
- "Setting `staleTime` is the recommended way to avoid excessive refetches"
- "Use `'static'` for data that cannot change while the app is running: feature flags fetched at boot, user permissions loaded at login, static reference tables."

Silent on: which client tool to use for the leftover client state; URL state; Server Components.

---

## Excluded

- None excluded for author verification: every source above is an official project doc, a Vercel-owned rules repo with named Vercel authors, or a blog whose author is a verified maintainer/educator (TanStack maintainers page; GitHub bios and commit counts).
- Not consulted at all: Medium, dev.to, freeCodeCamp, LogRocket, aggregators, LLM-summary sites, web search results.
- Zustand "Connect to state with URL hash" guide and the pmndrs discussion #2740 (referenced by the Next.js guide) were not read; the Next.js guide itself flags a pending rewrite.
