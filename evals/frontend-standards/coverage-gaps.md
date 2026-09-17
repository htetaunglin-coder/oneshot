# Coverage gaps

Maintainer file, outside the skill. Places the rules are silent or answered by analogy. Source: eval agents' "unclear or missing" reports. An entry leaves when a rule covers it or a decision says it stays out.

## colocation

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
