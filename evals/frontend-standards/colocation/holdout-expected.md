# Holdout cases — writer's expected answers

Independent opinion of the case author. Written without reading the skill, its rules, or any other eval.

## H1 — shared undo hook

**Answer**
- Move: `src/features/inbox/hooks/use-undoable-delete.ts` -> `src/hooks/use-undoable-delete.ts`.
- Change signature to `useUndoableDelete({ onDelete: (id: string) => Promise<void>; message: string })`; drop the `deleteMessage` import.
- Edit: `src/features/inbox/components/message-row.tsx` passes `{ onDelete: deleteMessage, message: "Message deleted" }`.
- New consumer: `src/features/drafts/components/draft-card.tsx` passes `{ onDelete: deleteDraft, message: "Draft discarded" }`.
- No file left in `src/features/inbox/hooks/`.

**Reason**
The timer/pending-set/toast mechanics are the whole hook and are feature-neutral; only the delete call and the copy are feature-specific, and both fit cleanly as parameters. Two real consumers in different features is the threshold at which a copy would start to drift, and a cross-feature import from drafts into inbox is worse than either option.

**What makes this hard:** the hook imports a feature API today, so it looks feature-owned, and the cheap moves (copy it, or import across features) both look reasonable in the diff.

## H2 — cart context (corrected 2026-09-07 after the Layers decision)

- Provider, reducer, `useCart`, `CartItem` stay in `src/features/cart/` (split into `lib/cart-reducer.ts` and `components/cart-provider.tsx` if wanted). No `index.ts` barrel (Vercel `bundle-barrel-imports`); import files directly. `src/app/layout.tsx` mounts `CartProvider` and passes `<CartBadge />` (new client component `src/features/cart/components/cart-badge.tsx`) into `SiteHeader` through a slot prop or `children`; `site-header.tsx` stays free of feature imports. Checkout: the page under `src/app/(shop)/checkout` composes cart data into `OrderSummary` via props, or renders a cart-feature client component beside it; `features/checkout` never imports `features/cart`.
- Reason: cart owns its domain; consumers spreading does not move it. Cross-feature need is met by app-level composition, so imports keep flowing shared → features → app.
- Fails: moving the reducer to `src/hooks` or `src/lib`; `features/checkout` importing `features/cart`; a feature `index.ts`.

<details><summary>Writer's original H2 answer, kept for the record</summary>

## H2 — cart context

**Answer**
- Keep everything inside the feature. Split the file:
  - `src/features/cart/types.ts` — `CartItem`, `CartState`, `CartAction`.
  - `src/features/cart/lib/cart-reducer.ts` — `cartReducer` (pure, unit-testable).
  - `src/features/cart/components/cart-provider.tsx` — `CartProvider`, `useCart` (imports the two above).
  - `src/features/cart/index.ts` — exports `CartProvider`, `useCart`, `type CartItem` only.
- Move the mount: remove `<CartProvider>` from `src/app/(shop)/cart/page.tsx`; wrap `{children}` in `src/app/layout.tsx` (inside the existing providers).
- `src/components/site-header.tsx` becomes a client component (or extracts a small `CartBadge` client child) and imports `useCart` from `@/features/cart`.
- `src/features/checkout/components/order-summary.tsx` imports `useCart` from `@/features/cart`.
- Nothing moves to `src/components`, `src/hooks`, or `src/lib`.

**Reason**
The state, actions, and types are cart domain knowledge; a global component or another feature needing them changes who consumes, not who owns. Mounting the provider in the root layout is a wiring concern that belongs in `src/app`, and a single `index.ts` gives the header and checkout one sanctioned door instead of deep paths into the feature.

**What makes this hard:** a file in `src/components` importing from `src/features/*` feels like the dependency arrow pointing the wrong way, which tempts people to hoist the provider to `src/components/providers` or the reducer to `src/lib`.

</details>

## H3 — signup schema

**Answer**
- Create `src/features/signup/lib/signup-schema.ts` exporting `signupSchema` (password `.min(10)`, `email` refined against the disposable list) and `type SignupInput = z.infer<typeof signupSchema>`.
- Create `src/features/signup/lib/disposable-domains.ts` exporting `DISPOSABLE_DOMAINS: ReadonlySet<string>` and `isDisposableEmail(email)`.
- Delete the inline `schema` in `src/features/signup/components/signup-form.tsx`; import `signupSchema`.
- Delete the inline `bodySchema` in `src/app/api/signup/route.ts`; import `signupSchema` from `@/features/signup/lib/signup-schema`.
- Error copy lives in the schema (`.refine(..., { message: "Disposable email addresses are not allowed" })`) so both surfaces read the same string.

**Reason**
The schema is the signup contract and there is exactly one feature that owns it; the route handler is a thin transport that should call into the feature, not host a second copy of its rules. Keeping the domain list beside the schema (not in `src/lib`) is right until a second feature needs disposable-email detection.

**What makes this hard:** the route handler lives outside the feature tree, so importing "into" a feature from `src/app/api` looks inverted, and Zod schemas have a strong gravitational pull toward a global `src/lib/validations` folder.

## H4 — route paths

**Answer**
- Create `src/lib/routes.ts`:
  ```ts
  export const routes = {
    dashboard: "/dashboard",
    projects: "/projects",
    settings: "/settings",
    login: "/login",
    billing: {
      root: "/account/billing",
      invoices: "/account/billing/invoices",
      invoice: (id: string) => `/account/billing/invoices/${id}`,
      paymentMethods: "/account/billing/payment-methods",
    },
  } as const;
  ```
- Delete `src/features/billing/lib/routes.ts`; billing components import `routes.billing.*` from `@/lib/routes`.
- `src/components/main-nav.tsx` builds `items` from `routes`.
- `src/middleware.ts` imports `routes` for `startsWith(routes.billing.root)` and the `/login` redirect; `config.matcher` stays a literal array (`"/account/billing/:path*"`) because Next reads it statically, with a one-line comment pointing at `routes.ts`.
- Move the segment folder `src/app/(app)/billing` -> `src/app/(app)/account/billing`.

**Reason**
Paths are an app-wide contract consumed by the nav, middleware, and any feature that links across the app, so the single map belongs in `src/lib`, not inside the feature whose pages happen to sit at those paths. Feature-local route maps only work while nothing outside the feature links in, and here two things already do.

**What makes this hard:** colocation says billing owns its paths, the existing code already does that, and the middleware matcher cannot consume the constant, so a full "one source of truth" is impossible and the author has to decide how far to push it.

## H5 — test render helper

**Answer**
- Create `src/test/render.tsx` exporting `renderWithProviders` (and re-exporting `screen`, `waitFor`, `within` from `@testing-library/react`), with `makeQueryClient` and `Providers` as module-private helpers.
- Remove `makeQueryClient`, `Providers`, and `renderWithProviders` from `src/features/cart/components/cart-drawer.test.tsx`; import `renderWithProviders` from `@/test/render`.
- `src/features/orders/components/order-list.test.tsx` imports from `@/test/render`.
- Add `src/test/**` to `tsconfig` `exclude` for the production build if the project's `tsconfig.json` does not already exclude test files.
- Do not put it in `src/lib`, `src/hooks`, or under either feature.

**Reason**
The wrapper contains no cart or orders knowledge, only app-wide providers, so it is shared test infrastructure; `src/lib` and `src/hooks` are production trees and should not import `@testing-library/react`. A new top-level `src/test/` is the smallest honest home, and importing test helpers across features (orders from cart) is the option to avoid.

**What makes this hard:** the sanctioned folder list has no slot for test-only shared code, so every candidate is a compromise: pollute `src/lib`, reach across features, or invent a new top-level directory.
