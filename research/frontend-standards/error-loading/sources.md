# Sources: error, loading, empty, not-found states (React 19 + Next.js App Router)

Fetched 2026-09-18. Primary sources only: local Next.js docs (next@16.2.10 in `node_modules`), react.dev (React 19.2.4 in `node_modules`), Vercel plugin skills installed locally (vercel@0.49.2), TkDodo (Dominik Dorfmeister, TanStack Query maintainer), MDN ARIA reference pages. No Medium/dev.to/LogRocket/aggregators/LLM-summary sites. Quotes are verbatim. "Silent on" lists what the source does not answer.

---

## 1. Next.js: Error Handling (getting started)

Path: `node_modules/next/dist/docs/01-app/01-getting-started/10-error-handling.md`
Author: Next.js docs (Vercel), shipped with next@16.2.10. Date: package release.

- Two categories: "Errors can be divided into two categories: expected errors and uncaught exceptions."
- Expected → return values, not throw: "For these errors, avoid using `try`/`catch` blocks and throw errors. Instead, model expected errors as return values."
- Expected via `useActionState`: "You can pass your action to the `useActionState` hook and use the returned `state` to display an error message." Example renders `{state?.message && <p aria-live="polite">{state.message}</p>}` and `<button disabled={pending}>`.
- Server Components: "you can use the response to conditionally render an error message or `redirect`." Example: `if (!res.ok) { return 'There was an error.' }`.
- Not found: "You can call the `notFound` function within a route segment and use the `not-found.js` file to show a 404 UI."
- Uncaught: "Uncaught exceptions are unexpected errors that indicate bugs or issues that should not occur during the normal flow of your application. These should be handled by throwing errors, which will then be caught by error boundaries."
- `error.tsx` shape: `'use client' // Error boundaries must be Client Components`; props `{ error: Error & { digest?: string }; unstable_retry: () => void }`; logs in `useEffect(() => { console.error(error) }, [error])`.
- Nesting: "Errors will bubble up to the nearest parent error boundary. This allows for granular error handling by placing `error.tsx` files at different levels in the route hierarchy."
- Component-level: "For component-level error recovery, the `unstable_catchError` function lets you create error boundaries that can wrap any part of your component tree".
- Not caught: "Error boundaries don't catch errors inside event handlers. They're designed to catch errors during rendering to show a **fallback UI** instead of crashing the whole app." "In general, errors in event handlers or async code aren’t handled by error boundaries because they run after rendering." "To handle these cases, catch the error manually and store it using `useState` or `useReducer`, then update the UI to inform the user."
- Transition exception: "Note that unhandled errors inside `startTransition` from `useTransition`, will bubble up to the nearest error boundary."
- Global: "you can handle errors in the root layout using the `global-error.js` file, located in the root app directory ... Global error UI must define its own `<html>` and `<body>` tags, since it is replacing the root layout or template when active."

Silent on: empty states; loading; client-fetch libraries; a11y beyond the `aria-live="polite"` example.

## 2. Next.js: Fetching Data (streaming, Suspense, loading)

Path: `node_modules/next/dist/docs/01-app/01-getting-started/06-fetching-data.md`
Author/date: as above.

- "`fetch` requests are not cached by default and will block the page from rendering until the request is complete."
- "There are two ways you can use streaming in your application: 1. Wrapping a page with a `loading.js` file 2. Wrapping a component with `<Suspense>`"
- `loading.js` scope: "You can create a `loading.js` file in the same folder as your page to stream the **entire page** while the data is being fetched." "Behind the scenes, `loading.js` will be nested inside `layout.js`, and will automatically wrap the `page.js` file and any children below in a `<Suspense>` boundary."
- Layout caveat: "a layout that accesses uncached or runtime data (e.g. `cookies()`, `headers()`, or uncached fetches) does not fall back to a same route segment `loading.js`. Instead, it blocks navigation until the layout finishes rendering."
- Fix: "wrap the uncached access in its own `<Suspense>` boundary with a fallback, or move the data fetching into `page.js` where `loading.js` can cover it."
- Recommendation: "while `loading.js` works well for streaming route segments, using `<Suspense>` closer to the runtime or uncached data access is recommended."
- Suspense: "`<Suspense>` allows you to be more granular about what parts of the page to stream." Example fallback `<BlogListSkeleton />`.
- Meaningful loading: "An instant loading state is fallback UI that is shown immediately to the user after navigation. ... you can use skeletons and spinners, or a small but meaningful part of future screens such as a cover photo, title, etc."
- Client fetch: "You can use a community library like SWR or React Query to fetch data in Client Components." SWR example: `if (isLoading) return <div>Loading...</div>`, `if (error) return <div>Error: {error.message}</div>`.
- Sequential: "the page still waits for the artist data before displaying anything. To prevent this, you can wrap the entire page component in a `<Suspense>` boundary (for example, using a `loading.js` file)".

Silent on: empty states; error inside a Suspense boundary; order of checks.

## 3. Next.js: Mutating Data (actions, pending, redirect)

Path: `node_modules/next/dist/docs/01-app/01-getting-started/07-mutating-data.md`

- Pending: "While executing a Server Function, you can show a loading indicator with React's `useActionState` hook. This hook returns a `pending` boolean" — example `{pending ? <LoadingSpinner /> : 'Create Post'}` with `onClick={() => startTransition(action)}`.
- Redirect: "Calling `redirect` throws a framework handled control-flow exception. Any code after it won't execute. If you need fresh data, call `revalidatePath` or `revalidateTag` beforehand."
- `useTransition` with actions: `const [isPending, startTransition] = useTransition()` ... "// You can use `isPending` to give users feedback".

Silent on: error UI for actions beyond `useActionState`; optimistic rollback.

## 4. Next.js: `error.js` file convention

Path: `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/error.md`

- "`error.js` wraps a route segment and its nested children in a React Error Boundary. When an error throws within the boundary, the `error` component shows as the fallback UI."
- Bubbling on purpose: "If you want errors to bubble up to the parent error boundary, you can `throw` when rendering the `error` component."
- Scope: "`error.js` wraps `loading.js`, `not-found.js`, `page.js`, and nested `layout.js` files in a React error boundary. It does **not** wrap the `layout.js` or `template.js` above it in the same segment. To handle errors in the root layout, use `global-error.js`."
- `error` prop: "During development, the `Error` object forwarded to the client will be serialized and include the `message` of the original error for easier debugging. However, **this behavior is different in production** to avoid leaking potentially sensitive details included in the error to the client."
- `error.message`: "Errors forwarded from Client Components show the original `Error` message." "Errors forwarded from Server Components show a generic message with an identifier. This is to prevent leaking sensitive details. You can use the identifier, under `errors.digest`, to match the corresponding server-side logs."
- `error.digest`: "An automatically generated hash of the error thrown. It can be used to match the corresponding error in server-side logs."
- `unstable_retry`: "When executed, the function will try to re-fetch and re-render the error boundary's children. If successful, the fallback error component is replaced with the result of the re-render."
- `reset`: "In most cases, you should use `unstable_retry()` instead. However, if you have a specific reason to clear the error state and re-render the error boundary's children without re-fetching the contents, you can use the `reset()` function."
- Global: "Global error UI must define its own `<html>` and `<body>` tags, global styles, fonts, or other dependencies that your error page requires. This file replaces the root layout or template when active." "`metadata` and `generateMetadata` exports are not supported in `global-error.jsx`."
- Version: "`v16.2.0` | `unstable_retry` prop added."

## 5. Next.js: `loading.js` file convention

Path: `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/loading.md`

- "The special file `loading.js` helps you create meaningful Loading UI with React Suspense. With this convention, you can show an instant loading state from the server while the content of a route segment streams in."
- "Loading UI components do not accept any parameters."
- Navigation: "The Fallback UI is prefetched, making navigation immediate unless prefetching hasn't completed." "Shared layouts remain interactive while new route segments load."
- Scope: "`loading.js` wraps `not-found.js`, `page.js`, and nested `layout.js` files in a `<Suspense>` boundary. It does **not** wrap the `layout.js`, `template.js`, or `error.js` in the same segment."
- "If the layout accesses uncached or runtime data (e.g. `cookies()`, `headers()`, or uncached fetches), `loading.js` will not show a fallback for it." "**Without Cache Components:** Navigation blocks until the layout finishes rendering."
- Status: "When streaming, a `200` status code will be returned to signal that the request was successful." "Because the response headers have already been sent to the client, the status code of the response cannot be updated." "when a 404 page is streamed to the client, Next.js includes a `<meta name="robots" content="noindex">` tag in the streamed HTML."
- "If you need a 404 status, for compliance or analytics, ensure the resource exists before the response body is streamed".
- "The response body starts streaming when a Suspense fallback renders (for example, a `loading.tsx`) or when a Server Component suspends under a `Suspense` boundary. Place `notFound()` before those boundaries and before any `await` that may suspend."
- Suspense: "`<Suspense>` works by wrapping a component that performs an asynchronous action (e.g. fetch data), showing fallback UI (e.g. skeleton, spinner) while it's happening, and then swapping in your component once the action completes."

## 6. Next.js: `not-found.js` file convention

Path: `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/not-found.md`

- "The **not-found** file is used to render UI when the `notFound` function is thrown within a route segment. Along with serving a custom UI, Next.js will return a `200` HTTP status code for streamed responses, and `404` for non-streamed responses".
- Hierarchy: "`not-found.js` renders between `loading.js` and `page.js`. It is wrapped by the `<Suspense>` boundary from `loading.js` and the error boundary from `error.js` in the same segment."
- "`not-found.js` or `global-not-found.js` components do not accept any props."
- "In addition to catching expected `notFound()` errors, the root `app/not-found.js` and `app/global-not-found.js` files handle any unmatched URLs for your whole application."
- "By default, `not-found` is a Server Component. You can mark it as `async` to fetch and display data".
- `global-not-found.js` is experimental (`experimental.globalNotFound`), "must return a full HTML document".

## 7. Next.js: `forbidden.js` / `unauthorized.js` + functions

Paths: `03-file-conventions/forbidden.md`, `unauthorized.md`; `04-functions/forbidden.md`, `unauthorized.md`. All `version: experimental`.

- "The **forbidden** file is used to render UI when the `forbidden` function is invoked during authentication. Along with allowing you to customize the UI, Next.js will return a `403` status code."
- "The **unauthorized** file ... Next.js will return a `401` status code."
- "To start using `forbidden`, enable the experimental `authInterrupts` configuration option in your `next.config.js` file".
- "`forbidden` can be invoked in Server Components, Server Functions, and Route Handlers."

## 8. Next.js: `redirect` function

Path: `node_modules/next/dist/docs/01-app/03-api-reference/04-functions/redirect.md`

- "When used in a streaming context, this will insert a meta tag to emit the redirect on the client side. When used in a server action, it will serve a 303 HTTP redirect response to the caller. Otherwise, it will serve a 307 HTTP redirect response to the caller."
- "By default, `redirect` will use `push` ... in Server Actions and `replace` ... everywhere else."
- Behavior: "In Server Actions and Route Handlers, redirect should be called **outside** the `try` block when using `try/catch` statements." "`redirect` throws an error so it should be called **outside** the `try` block when using `try/catch` statements." "`redirect` can be called in Client Components during the rendering process but not in event handlers. You can use the `useRouter` hook instead."
- "Invoking the `redirect()` function throws a `NEXT_REDIRECT` error and terminates rendering of the route segment in which it was thrown."
- "`redirect` does not require you to use `return redirect()` as it uses the TypeScript `never` type."

## 9. Next.js: `notFound` function

Path: `node_modules/next/dist/docs/01-app/03-api-reference/04-functions/not-found.md`

- "Invoking the `notFound()` function throws a `NEXT_HTTP_ERROR_FALLBACK;404` error and terminates rendering of the route segment in which it was thrown."
- "`notFound()` does not require you to use `return notFound()` due to using the TypeScript `never` type."
- Also injects `<meta name="robots" content="noindex" />`.

## 10. Next.js: `unstable_rethrow` function

Path: `node_modules/next/dist/docs/01-app/03-api-reference/04-functions/unstable_rethrow.md` (`version: unstable`)

- "calling the `notFound` function will throw an internal Next.js error and render the `not-found.js` component. However, if used inside the `try` block of a `try/catch` statement, the error will be caught, preventing `not-found.js` from rendering".
- "The following Next.js APIs rely on throwing an error which should be rethrown and handled by Next.js itself: `notFound()`, `redirect()`, `permanentRedirect()`".
- "This method should be called at the top of the catch block, passing the error object as its only argument. It can also be used within a `.catch` handler of a promise."
- "You may be able to avoid using `unstable_rethrow` if you encapsulate your API calls that throw and let the **caller** handle the exception."
- "Only use `unstable_rethrow` if your caught exceptions may include both application errors and framework-controlled exceptions (like `redirect()` or `notFound()`)."
- "Any resource cleanup (like clearing intervals, timers, etc) would have to either happen prior to the call to `unstable_rethrow` or within a `finally` block."

## 11. Next.js: `unstable_catchError` function

Path: `node_modules/next/dist/docs/01-app/03-api-reference/04-functions/catchError.md` (v16.2.0)

- "The `unstable_catchError` function creates a component that wraps its children in an error boundary. It provides a programmatic alternative to the `error.js` file convention, enabling component-level error recovery anywhere in your component tree."
- "**Framework-aware integration** — APIs like `redirect()` and `notFound()` work by throwing special errors under the hood. `unstable_catchError` handles these seamlessly, so they're not accidentally caught by your error boundary."
- "**Client navigation handling** — The error state automatically clears when you do a client navigation to a different route."
- "The `fallback` function must be a Client Component (or defined in a `'use client'` module)."
- "The `reset()` function only clears the error state and re-renders without re-fetching, which means it won't recover from Server Component errors."
- "You don't need to wrap `error.js` default exports with `unstable_catchError`."

## 12. Next.js: Streaming guide (mid-stream errors), Project structure (hierarchy), Forms guide

Paths: `02-guides/streaming.md`; `01-getting-started/02-project-structure.md`; `02-guides/forms.md`.

- Streaming: "If a component throws an error after streaming has started, the nearest `error.js` boundary catches it and renders the error UI in place of the failed component. The rest of the page remains intact, only the section that errored is replaced." "Because the HTTP status code (`200 OK`) has already been sent with the first chunk, it cannot be changed to a `4xx` or `5xx`."
- Streaming: "Prefer explicit `<Suspense>` boundaries close to the dynamic access. ... A `loading.js` high in the tree is a valid boundary, so the framework finds it and stops, but now the entire page falls back to a full-page skeleton instead of streaming granularly."
- Streaming: "a `redirect()` mid-stream becomes a client-side redirect rather than an HTTP redirect header."
- Hierarchy: "`layout.js` / `template.js` / `error.js` (React error boundary) / `loading.js` (React suspense boundary) / `not-found.js` (React error boundary for "not found" UI) / `page.js` or nested `layout.js`". "the components of a route segment will be nested **inside** the components of its parent segment."
- Forms: "For **server-side validation**, you can use a library like zod"; returns `errors: validatedFields.error.flatten().fieldErrors`. "To display validation errors or messages, turn the component that defines the `<form>` into a Client Component and use React `useActionState`." "The `useActionState` hook exposes a `pending` boolean that can be used to show a loading indicator or disable the submit button while the action is being executed."

## 13. Vercel plugin skill `nextjs` (installed)

Path: `/Users/htetaunglin/.claude/plugins/cache/claude-plugins-official/vercel/0.49.2/skills/nextjs/` (references/ identical to upstream/, verified by `diff -rq`).
Author: Vercel (plugin `vercel` 0.49.2). Date: plugin cache.

`references/error-handling.md`:
- `error.tsx` / `global-error.tsx` examples use `reset` prop, not `unstable_retry`. "**Important:** `error.tsx` must be a Client Component." "**Important:** Must include `<html>` and `<body>` tags."
- "**Do NOT wrap navigation APIs in try-catch.** They throw special errors that Next.js handles internally." Shows bad `try { redirect() } catch { return { error } }`, good "Call navigation APIs outside try-catch", and a manual re-throw `if (error instanceof Error && error.message === 'NEXT_REDIRECT') { throw error }`.
- "Same applies to: `redirect()` - 307 ... `permanentRedirect()` - 308 ... `notFound()` - 404 ... `forbidden()` - 403 ... `unauthorized()` - 401".
- "Use `unstable_rethrow()` to re-throw these errors in catch blocks".
- Auth: `unauthorized() // Renders unauthorized.tsx (401)`, `forbidden() // Renders forbidden.tsx (403)`.
- `not-found.tsx` + `notFound()  // Renders closest not-found.tsx`.
- Hierarchy tree: "`error.tsx` # Catches errors from all children" ... "`layout.tsx` # Errors here go to global-error.tsx".
Silent on: `digest` / production message stripping; `error.tsx` not wrapping its own layout; `loading.tsx`; streaming status codes; `unstable_catchError`; expected-vs-unexpected split; event-handler errors; pending state; empty state; a11y.

`references/suspense-boundaries.md`: "`useSearchParams` Always requires Suspense boundary in static routes. Without it, the entire page becomes client-side rendered." `usePathname` "Requires Suspense boundary when route has dynamic parameters." Table: `useParams()` No, `useRouter()` No.

`references/data-patterns.md`: "Solution 2: Streaming with Suspense" — per-section `<Suspense fallback={<UserSkeleton />}>`; "Option 2: Fetch on Mount (When Necessary)" — `if (!data) return <Loading />;` (no error branch).

`references/app-router-files.md` / `file-conventions.md`: tables — "`loading.tsx` | Suspense fallback | Server (default)", "`error.tsx` | Error boundary | Client (required)", "`not-found.tsx` | 404 UI | Server (default)", "`global-error.tsx` # Global error UI".

## 14. Vercel plugin skill `react-best-practices` (installed)

Path: `/Users/htetaunglin/.claude/plugins/cache/claude-plugins-official/vercel/0.49.2/skills/react-best-practices/rules/`. Upstream (GitHub `vercel-labs/agent-skills`) has 6 extra rules; none mention error/loading/suspense (checked by name).

- `async-suspense-boundaries` (HIGH): "Instead of awaiting data in async components before returning JSX, use Suspense boundaries to show the wrapper UI faster while data loads." "**When NOT to use this pattern:** Critical data needed for layout decisions ... SEO-critical content above the fold ... Small, fast queries ... When you want to avoid layout shift". "**Trade-off:** Faster initial paint vs potential layout shift."
- `rendering-usetransition-loading` (LOW): "Use `useTransition` instead of manual `useState` for loading states. This provides built-in `isPending` state and automatically manages transitions." "**Error resilience**: Pending state correctly resets even if the transition throws".
- `rerender-transitions` (MEDIUM): "Mark frequent, non-urgent state updates as transitions to maintain UI responsiveness." (scroll example; not loading.)
- `rendering-conditional-render` (LOW): "Use explicit ternary operators (`? :`) instead of `&&` for conditional rendering when the condition can be `0`, `NaN`, or other falsy values that render."
- `rendering-hoist-jsx` (LOW), `bundle-conditional` (HIGH): skeletons appear only as incidental example code (`{loading && loadingSkeleton}`, `if (!frames) return <Skeleton />`).
- `client-swr-dedup` (MEDIUM-HIGH): SWR for dedup; silent on error/loading rendering.
- `server-auth-actions` (CRITICAL): `throw unauthorized('Must be logged in')` inside actions; silent on how the client shows it.

## 15. Vercel `composition-patterns` (GitHub only; not in local plugin 0.49.2)

Repo: https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns/rules. Ten rule files (`architecture-avoid-boolean-props`, `architecture-compound-components`, `patterns-children-over-render-props`, `patterns-explicit-variants`, `react19-no-forwardref`, `state-context-interface`, `state-decouple-implementation`, `state-lift-state`, plus `_sections`, `_template`). grep for suspense / error boundary / loading / skeleton / pending / useActionState / empty state: 0 hits in every file. Silent on the whole topic.

## 16. Vercel `shadcn` and `verification` skills (incidental)

- `shadcn/SKILL.md`: table row "Empty/loading/error states | `Card` + `Skeleton` + `Alert` | Gives non-happy paths a designed surface instead of placeholder text"; anti-pattern "Shipping empty/loading/error states without design treatment".
- `verification/SKILL.md`: checklist item "**Missing error boundary** — unhandled rejection crashes the page silently".

## 17. React: `<Suspense>` reference

URL: https://react.dev/reference/react/Suspense
Author: React team (react.dev). Date: live docs, React 19.2 (fetched 2026-09-18).

- "`<Suspense>` lets you display a fallback until its children have finished loading."
- `fallback`: "in practice, a fallback is a lightweight placeholder view, such as a loading spinner or skeleton. ... If `fallback` suspends while rendering, it will activate the closest parent Suspense boundary."
- Caveats: "Suspense does not detect when data is fetched inside an Effect or event handler." "If Suspense was displaying content for the tree, but then it suspended again, the fallback will be shown again unless the update causing it was caused by `startTransition` or `useDeferredValue`." "React reveals suspended content at most once every 300ms, measured from the last reveal."
- Together: "By default, the whole tree inside Suspense is treated as a single unit. For example, even if only one of these components suspends waiting for some data, all of them together will be replaced by the loading indicator". "Components that load data don’t have to be direct children of the Suspense boundary."
- Nested: "When a component suspends, the closest parent Suspense component shows the fallback. This lets you nest multiple Suspense components to create a loading sequence."
- Hiding: "When a component suspends, the closest parent Suspense boundary switches to showing the fallback. This can lead to a jarring user experience if it was already displaying some content." "mark the navigation state update as a Transition with `startTransition`". "A Transition doesn’t wait for all content to load. It only waits long enough to avoid hiding already revealed content."
- Indicator: "you can replace `startTransition` with `useTransition` which gives you a boolean `isPending` value."
- Server errors: "If a component throws an error on the server, React will not abort the server render. Instead, it will find the closest `<Suspense>` component above it and include its fallback ... On the client, React will attempt to render the same component again. If it errors on the client too, React will throw the error and display the closest Error Boundary."

## 18. React: `Component` — Catching rendering errors with an error boundary

URL: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

- "By default, if your application throws an error during rendering, React will remove its UI from the screen. To prevent this, you can wrap a part of your UI into an Error Boundary."
- "Error boundaries do not catch errors for: Event handlers; Server side rendering; Errors thrown in the error boundary itself (rather than its children); Asynchronous code (e.g. `setTimeout` or `requestAnimationFrame` callbacks); an exception is the usage of the `startTransition` function returned by the `useTransition` Hook. Errors thrown inside the transition function are caught by error boundaries".
- "To implement an Error Boundary component, you need to provide `static getDerivedStateFromError` ... You can also optionally implement `componentDidCatch` ... to log the error to an analytics service."
- Granularity: "You don’t need to wrap every component into a separate Error Boundary. ... in a messaging app, it makes sense to place an Error Boundary around the list of conversations. It also makes sense to place one around every individual message. However, it wouldn’t make sense to place a boundary around every avatar."
- "There is currently no way to write an Error Boundary as a function component. However, you don’t have to write the Error Boundary class yourself. For example, you can use `react-error-boundary` instead."

## 19. React: `useActionState`

URL: https://react.dev/reference/react/useActionState

- "`useActionState` returns an array with exactly three values: The current state ... A `dispatchAction` function that you call inside Actions ... The `isPending` flag that tells you if any dispatched Actions for this Hook are pending."
- "React queues and executes multiple calls to `dispatchAction` sequentially."
- "When using Server Functions, `initialState` needs to be serializable".
- "If `dispatchAction` throws an error, React cancels all queued actions and shows the nearest Error Boundary."
- Errors: "For known errors, such as “quantity not available” validation errors from your backend, you can return it as part of your `reducerAction` state and display it in the UI." "For unknown errors, such as `undefined is not a function`, you can throw an error. React will cancel all queued Actions and shows the nearest Error Boundary by rethrowing the error from the `useActionState` hook."
- Troubleshooting: "My `isPending` flag is not updating — If you’re calling `dispatchAction` manually (not through an Action prop), make sure you wrap the call in `startTransition`".
- "If you want to provide immediate feedback, such as immediately updating the quantity, you can use `useOptimistic`."

## 20. React: `useTransition`

URL: https://react.dev/reference/react/useTransition

- "The `isPending` flag that tells you whether there is a pending Transition."
- "the `isPending` state switches to `true` at the first call to `startTransition`, and stays `true` until all Actions complete and the final state is shown to the user."
- "State updates marked as Transitions will be non-blocking and will not display unwanted loading indicators."
- Error: "If a function passed to `startTransition` throws an error or returns a rejected Promise, you can display an error to your user with an error boundary. To use an error boundary, wrap the component where you are calling the `useTransition` in an error boundary."
- Troubleshooting: "When you use `await` inside a `startTransition` function, the state updates that happen after the `await` are not marked as Transitions. You must wrap state updates after each `await` in a `startTransition` call".
- "Hiding the entire tab container to show a loading indicator leads to a jarring user experience. If you add `useTransition` to `TabButton`, you can instead display the pending state in the tab button instead."

## 21. React: `useOptimistic`

URL: https://react.dev/reference/react/useOptimistic

- "`useOptimistic` is a React Hook that lets you optimistically update the UI."
- "`optimisticState`: The current optimistic state. It is equal to `value` unless an Action is pending, in which case it is equal to the state returned by `reducer`".
- "The set function returned by `useOptimistic` lets you update the state for the duration of an Action."
- "When deleting items optimistically, you should handle the case where the Action fails. This example shows how to display an error message when a delete fails, and the UI automatically rolls back to show the item again."

## 22. TkDodo: React Query Error Handling

URL: https://tkdodo.eu/blog/react-query-error-handling
Author: Dominik Dorfmeister (TkDodo), TanStack Query maintainer. Date: Sep 10, 2021; "Last Update: 2023-10-21" (v5).

- Prerequisite: "React Query needs a rejected Promise in order to handle errors correctly." "If you are working with the fetch API or other libraries that do not give you a rejected Promise on erroneous status codes like 4xx or 5xx, you’ll have to do the transformation yourself in the `queryFn`."
- Level 1: "we’re handling error situations by checking for the `isError` boolean flag (which is derived from the `status` enum) given to us by React Query."
- Level 2 (boundaries): "One thing that Error Boundaries cannot do is catch asynchronous errors, because those do not occur during rendering. So to make Error Boundaries work in React Query, the library internally catches the error for you and re-throws it in the next render cycle". "pass the `throwOnError` flag to your query". "you can even customize which errors should go towards an Error Boundary ... `throwOnError: (error) => error.response?.status >= 500`". "Errors in the 4xx range can be handled locally (e.g. if some backend validation failed), while all 5xx server errors can be propagated to the Error Boundary." "Before v5, the `throwOnError` flag was known as `useErrorBoundary`."
- Level 3 (callbacks): "The `onError` and `onSuccess` callbacks have been removed from `useQuery` in v5." "the `onError` callback on `useQuery` is called for every Observer, which means if you call `useTodos` twice in your application, you will get two error toasts". "The global callbacks need to be provided when you create the `QueryCache`".
- Summary: "The three main ways to handle errors in React Query are: the `error` property returned from `useQuery`; the `onError` callback (on the query itself or the global `QueryCache` / `MutationCache`); using Error Boundaries". "what I personally like to do is show error toasts for background refetches (to keep the stale UI intact) and handle everything else locally or with Error Boundaries".

## 23. TkDodo: Status Checks in React Query

URL: https://tkdodo.eu/blog/status-checks-in-react-query
Author: as above. Date: Mar 27, 2021; "Last Update: 2023-10-21".

- States: "`success`: Your query was successful, and you have data for it; `error`: Your query did not work, and an error is set; `pending`: Your query has no data". "the `isFetching` flag is not part of the internal state machine ... You can be fetching and success, you can be fetching and error". "Before v5, `pending` was named `loading`".
- Standard order: "Here, we check for pending and error first, and then display our data. This is probably fine for some use-cases, but not for others."
- Background errors: "we will have both an `error` and the stale `data` available." "React Query will retry failed queries three times per default with exponential backoff, so it might take a couple of seconds until the stale data is replaced with the error screen."
- "This is why I usually check for data-availability first" — `if (todos.data) ... if (todos.error) ... return 'Loading...'`. "Again, there is no clear principle of what is right, as it is highly dependent on the use-case."

## 24. MDN: `aria-busy`

URL: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-busy
Author: MDN contributors. Date: "last modified on Jun 23, 2025".

- "The `aria-busy` attribute is a global ARIA state that indicates whether an element is currently being modified. It helps assistive technologies understand that changes to the content are not yet complete, and that they may want to wait before informing users of the update."
- "When multiple parts of a live region need to be loaded before changes are announced to the user, set `aria-busy="true"` until loading is complete. Then set to `aria-busy="false"`."
- "it can also be used outside of live regions—for example, in widgets or feeds—to signal ongoing changes or loading."

## 25. MDN: `status` role

URL: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles/status_role
Date: "last modified on Jun 23, 2025".

- "The `status` role defines a live region containing advisory information for the user that is not important enough to be an alert."
- "Do not give focus to the status when its content updates."
- "Elements with the role `status` have an implicit `aria-live` value of `polite` and an implicit `aria-atomic` value of `true`."

## 26. MDN: `aria-live` (+ `alert` role, one line)

URL: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-live
Date: "last modified on Sep 23, 2025".

- "The `aria-live` attribute is set on an empty element. When an update to the page occurs, the empty element with that `aria-live` attribute should be updated with a brief announcement".
- "`polite` ... assistive technologies will notify users of updates but generally do not interrupt the current task ... `assertive`, assistive technologies immediately notify the user, potentially clearing the speech queue of previous updates."
- "Screen readers buffer content when the page is loaded. Because of this, content added after the initial accessibility tree is built may not be noticed ... you can let users know the page has been updated by setting `aria-live="polite"`."
- "don't use the `assertive` value unless the interruption is imperative."
- `off (default)`: "updates to the region should not be presented to the user unless the user is currently focused on that region."

`alert` role (https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles/alert_role, "last modified on Mar 10, 2026"): "Setting `role="alert"` is equivalent to setting `aria-live="assertive"` and `aria-atomic="true"`." "Make sure that the element with the role is present in the page's markup first - this will "prime" the browser and screen reader to keep watching the element for changes. ... Do not try to dynamically add/generate an element with `role="alert"` that is already populated with the alert message you want announced - this generally does not lead to an announcement".
