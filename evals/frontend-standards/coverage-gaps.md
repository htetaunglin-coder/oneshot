# Coverage gaps

Maintainer file, outside the skill. Places the rules are silent or answered by analogy. Source: eval agents' "unclear or missing" reports. An entry leaves when a rule covers it or a decision says it stays out.

## colocation

Decided 2026-09-18
- Feature-to-feature import for a session or auth store: none needed. By the Share test an auth check is one concept several features keep consistent, so the store is shared scope (`src/lib`, `src/hooks`). Seen in state probe P1.

Open
- App-layer client glue (a `"use client"` file that reads one feature's hook and feeds another feature's props): lives in the route segment beside `page.tsx`. Not yet in the rule. H2 run 2.
- `import type` across features: default is no; the consumer declares the prop shape it needs. Not yet in the rule. H2 run 2.
- Under-limit file: "earns the extraction" reads as may, not must. Held until it fails a case.
- Boundaries enforcement on Biome/Ultracite repos: no `eslint-plugin-boundaries` equivalent, only `noRestrictedImports`. Tooling note, not rule text.

Out of scope, decided
- Hook shape when the caller supplies the varying part (argument vs options vs configurator). H1.
- Extracted-file naming; where "the size is reported". Formatting.
- `src/lib` sub-folder naming. Repo convention.

Closed 2026-09-07 by the Layers section and scope edits
- Cross-feature imports, domain state consumed by several features, route handler as a second consumer, config/constants scope, test utilities scope, `"use client"` leaf in a server component.

## tailwind

Open
- "The variant type is exported" when only the `Props` type is exported and `VariantProps` is not: reads as not exported. Regression 2026-09-16, T04.
- Opacity modifier on a variable read, `bg-(--x)/80`: allowed by Tailwind 4; not stated. A6.
- Existing shadcn primitive vs matching hand-written class string: when `Card` replaces a panel string. Pointed at the shadcn skill; no rule text.
- `tailwind-merge` configuration for custom theme utilities (`extendTailwindMerge`). Pointed at the shadcn skill.
- `@theme` token naming (`brand` vs reusing `primary`). Repo convention.

Closed 2026-09-07
- `@apply` and CSS modules. Layout-contract variables. `@utility` (removed). cva gate at exactly four. Shared look as a responsibility.

## comments

Open
- Copy-or-mutate and local-or-UTC on a `Date` helper as questions the signature leaves. Judgment; A1.
- A number in a comment the reader cannot verify. Left to maintenance; A2.
- How far a restructure may go on a frozen branch when an explaining variable is preferred. Colocation question; A3 regression.
- A config entry (`remotePatterns`) the change makes dead: maintenance speaks about comments only. A3/A6 regression.

Out of scope, decided
- Code shape of fire-and-forget (`void` vs `.catch`). Not a comment question; C05.
- A reference with no identifier (an on-call finding): write the reason, skip the anecdote; C09.

Closed 2026-09-07 by run 1 and the holdout
- Dividers and step labels. Field docs on any type. Hook server behavior in the JSDoc questions. TODO trigger and missing issue. Maintenance scope for untouched comments. Rename or assertion outside the task. Reason-in-PR-only request. Incident references. Fragment as field doc.

## file-order

Open
- Slot for a type that reads a `cva` value (`VariantProps<typeof x>`): after the `cva`, since it reads it; not stated. T04.
- A hand rewrite of a `src/components/ui` file: converts to registry shape or not. Not stated. B5.
- Two constants in the top section read by different parts: first use decides; watch for a case where first use is unclear.
- Mixed style after a focused change to an arrow-function file: accepted; the reply names it.

Out of scope, decided
- Import order: formatter or import sorter.
- Order between sibling exports in a non-component module (`src/lib/format.ts`): first use or alphabetical, repo convention.

Closed 2026-09-14 by run 1 and the holdout
- Helper order. Compound components. In-file precedent. `generate*` exports. `cva` with one reader. Registry files in `src/components/ui`.

## magic-literals

Open
- Where the `as const` array lives when a feature `types.ts` already holds the type: the owning module takes the array and the type derives there; `types.ts` keeps no runtime value. Stated; watch for friction. N7.
- Exporting a private constant from an implementation for a test: allowed when the test asserts against it; not stated. N8.
- A derived constant (`HEIGHT = WIDTH * 9 / 16`): named beside its source; not stated. N9 regression.
- A unit conversion factor (`1024` bytes per KB): formula-obvious with a named variable, else named; judgment. M01 regression.
- A mechanism factor or ratio, `waitMs * 2`, `16 / 9`: inline once as a formula; named when the business owns it. Not a row; N13, N9 round 3.

Out of scope, decided
- Query-key segments inside a query factory: inline; the factory is the name.
- HTTP status codes: inline.

Closed 2026-09-17 by run 1 and two holdout rounds
- Focused change. Business durations. Library arguments. `??` sentinel. Same digits, different meaning. Subset of a union. Naming precedent. Union-typed comparison. Enum request. One-concept lib file. Sibling tie-break. JSX plus class value. Test expected output.

## conditional-render

Open
- A file shown only in part in a task: report the changed lines. Eval harness question, not rule text. R-run.
- Whether a four-line sub-component counts as "small" when a requester asks to keep a line tiny: judgment. S3.
- What counts as a branch toward "four or more" in a precedence chain: each `if` including the final default. Judgment; regression note.

Out of scope, decided
- `switch` vs map when branch props differ: sub-component with early returns. Stated.

Closed 2026-09-17 by run 1 and the holdout
- Ladder scan direction: both tailwind and conditional-render now check rows from the top, override sentence first; "rung" removed. Fifth review.
- Boolean `&&` default. Precedence chains. `return null` at the end of an owning sub-component. Map vs sub-component by props. Hook needing a guard value. Text ternary. Over-rung block as precedent.

## state

Decided 2026-09-18, short rule file
- Probe first, control only: 4/5 by model default (derive in render, props to state, server data as cache, URL for shareable filters). Evidence in `state/probe/scorecard.md`. This question does not come back to planning unless a new failure is captured.
- The one failure, request data in a module-level store, took one sourced sentence, now `rules/state.md` (~230 words, own file for disclosure and the human index). Server-data and pointer sections were cut as no-ops after P2 passed without them.
- Parked permanently as repository decisions, not house standards: Zustand vs context vs Jotai; one store vs many; actions key; URL library (nuqs vs `useSearchParams`); form library; store file name.
- Research kept in `research/frontend-standards/state/` for a later rule if a project produces a real case.

## error-loading

Decided 2026-09-18, no rule file
- Probe, control only: 5/5 by model default (redirect outside `try`, production message redaction, boundary around the widget, `Suspense` around the slow part, `notFound()` for a missing entity). Evidence in `error-loading/probe/scorecard.md`. Closed unless a new failure is captured.
- Version facts recorded for later fixtures: Next 16.2 `error.tsx` receives `unstable_retry`; `unstable_catchError` from `next/error`; `notFound()` throws `NEXT_HTTP_ERROR_FALLBACK;404`; a streamed `not-found` returns 200 with `noindex`.
- Parked as repository or design decisions: skeleton vs spinner, toast vs inline, empty-state wording, a11y live regions. Placement of error, loading, and empty components is colocation.
- Research kept in `research/frontend-standards/error-loading/`.

## business-logic

Decided 2026-09-18, no rule, no probe
- Domain policy (money in minor units, dates and timezones, rounding, tax, permission enforcement): never this skill. Repository or domain docs own it; a house pick here binds every repo to one domain convention, the same harm as a state tool pick.
- Seam shapes (one validation schema imported by client and server, DTO to view-model mapping at the data boundary, where the authoritative check lives): in charter and already covered, colocation Business logic (pure, placement, display vs enforcement) and the Share test (one concept two runtimes keep consistent gets one owner). Delta near zero.
- Component API half (data passing, dumb components, variants): Vercel `composition-patterns`.
- Reopen condition: a captured failure in a real project, on a seam shape only. A probe of money, time, or rounding can find failures this skill is forbidden to fix.
