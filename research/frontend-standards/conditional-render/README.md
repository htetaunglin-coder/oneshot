# Conditional rendering shape — research summary

Question: which shape to use for a conditional branch inside a React component (early return, `&&`, ternary, nested ternary, variable before JSX, object/map lookup, `switch`, extract sub-component, IIFE in JSX). Primary sources only; see `sources.md` for quotes and URLs. Fetched 2026-09-17.

## Consensus

| Shape | Verdict | Sources that agree |
|---|---|---|
| `if` + `return` (early return / guard clause) | Allowed. First technique the React docs show. Must sit after every hook call. | React docs (Conditional Rendering, Rules of Hooks); Kent ("if statements, that's cool too"); Kondov |
| `return null` | Allowed but uncommon; prefer the parent to include/exclude the component. | React docs: "returning `null` from a component isn't common because it might surprise a developer" |
| Ternary `cond ? <A/> : <B/>` / `cond ? <A/> : null` | Default explicit form. Safe against falsy leaks. | React docs; Kent (preferred); Comeau (valid); Vercel + sergiodxa rules; eslint `jsx-no-leaked-render` ternary strategy; Kondov |
| `&&` with a number, string, or nullable on the left | Never. Renders `0`/`NaN` (React), crashes (React Native). | React docs Pitfall box; Kent; Comeau; `jsx-no-leaked-render`; Vercel; sergiodxa; Kondov |
| Fix for numeric guard | `x > 0 &&`, `!!x &&`, or ternary. `> 0` is the React docs' own fix; `!!` is the lint's `coerce` strategy. | React docs (`messageCount > 0 &&`); Comeau (`> 0` or ternary; not `!!`); `jsx-no-leaked-render` (`!!` or ternary); Kent lists all three but still prefers ternary |
| Variable assigned before JSX (`let content = null; if (...) content = ...`) | Allowed; "most verbose, but also the most flexible". Use when shortcuts get in the way. | React docs; Kent (his `if` version); Airbnb 15.6 (split nested ternary into a `const`); ESLint `no-nested-ternary` correct example |
| Nested ternary | Avoid. Split into a variable, `if/else if`, or a sub-component. | Airbnb 15.6 ("should not be nested and generally be single line"); ESLint `no-nested-ternary`; unicorn (one parenthesised level max); React docs ("too much nested conditional markup, consider extracting child components"); Kondov |
| Extract a sub-component | Do it when the conditional markup gets messy, or when a Kent-style pain point shows up. Not pre-emptively. | React docs; Kent (break up "NOT BEFORE" a real problem); Kondov (nested ternary → component with guard clauses) |
| Multiline JSX inside `&&` / ternary | Wrap in parentheses. | Airbnb React (`react/jsx-wrap-multilines`) |
| Hooks vs early return | All hooks above the first `return`. | React docs Rules of Hooks: "before any early returns"; "Do not call Hooks after a conditional `return` statement." |

## Disputed

**1. Nested ternaries**
- Hard line: Airbnb 15.6 and ESLint core `no-nested-ternary` — never nest.
- Soft line: `unicorn/no-nested-ternary` — one level is fine if parenthesised. Vercel's own `architecture-avoid-boolean-props` "incorrect" example shows a nested ternary as the smell, but Vercel has no lint-level rule on nesting itself.
- Kent: silent. Comeau: silent. React docs: implied only ("too much nested conditional markup").
- House rule can pick either; the primary-source majority is "never nest" with the escape hatch "extract a component or a variable".

**2. `&&` with a genuine boolean on the left**
- Allowed: React docs (their own fix is `messageCount > 0 && ...`); Comeau ("Both options are perfectly valid, and it comes down to personal taste"); Airbnb React alignment examples (`{showButton && <Button />}` is "good"); `jsx-no-leaked-render` (passes `> 0 &&` and `!! &&`); Vercel and sergiodxa (scoped to "when the condition can be `0`, `NaN`"; Vercel's own examples use `{loading && <LoadingSkeleton />}`).
- Never: Kent ("I still prefer not abusing the logical AND operator for rendering. I prefer being explicit by using a ternary"); Kondov ("default to using ternary operators").
- Kent's extra argument for ternary: branch coverage tools see both branches.
- Net: official docs and the lint rule permit boolean `&&`; the "always ternary" stance is one author's preference. A house rule that bans `&&` outright goes beyond the docs and should say so.

**3. `!!` coercion**
- Lint rule and Kent list it; Comeau does not offer it; React docs use a comparison instead. Style-level choice, no correctness dispute.

## Not sourced

**IIFE inside JSX `{(() => { ... })()}`** — no primary source recommends or forbids it. react.dev, Kent, Comeau, eslint-plugin-react rule list, Airbnb React guide: no mention. Airbnb JS 7.2 is the only adjacent primary text: "in a world with modules everywhere, you almost never need an IIFE" (not about JSX). Ben Ilegbodu (named author, not top-tier): "I've never in my 6+ years of writing React code ever done this." Advocacy exists only in Medium/dev.to posts. Verdict: no primary source; only community posts. If the house rule bans it, cite the React docs' variable-before-JSX and extract-a-component techniques as the sanctioned alternatives, not a ban from an authority.

**Object/map lookup for value-keyed branches (`STATUS_VIEW[status]`)** — no primary source. Not in react.dev, Kent, Comeau, Airbnb, Vercel, or sergiodxa. Only community posts. Nearest primary neighbours: Airbnb 15.6 (move branches into a `const`), ESLint `no-nested-ternary` correct example (`if / else if / else` into a `let`). Verdict: house rule may recommend it, but as house opinion.

**`switch` in a component body** — no primary source addresses it for rendering. React docs use `if` chains; ESLint `no-nested-ternary` doc uses `if/else if`. No source says "use `switch`" or "avoid `switch`".

## Delta vs Vercel / sergiodxa (do not repeat these)

Already covered by both skills:
- `rendering-conditional-render`: ternary over `&&` when the left side can be `0`/`NaN`/falsy. Example `count && <Badge/>` → `count > 0 ? <Badge/> : null`. Scope is numeric/falsy only; both are silent on boolean `&&`.
- `composition-avoid-boolean-props` (Vercel id `architecture-avoid-boolean-props`, CRITICAL): boolean mode props → nested conditionals inside one component is the smell; fix is composition/variants. Vercel's incorrect example is literally a nested ternary chain.
- `composition-explicit-variants` (Vercel `patterns-explicit-variants`): one component per variant instead of prop combinations; "no hidden conditionals".
- `composition-compound-components` (Vercel `architecture-compound-components`): compound + context; "No hidden conditionals" (Vercel's incorrect example uses `{showX && <X/>}` flag props).
- `composition-children-over-render-props`, `composition-state-provider` / `state-lift-state`, `state-decouple-implementation`, `state-context-interface`, `react19-no-forwardref`, `composition-avoid-overabstraction`, `composition-typescript-namespaces`: composition/state, not conditional shape.
- `rerender-memo` / Vercel 5.6 "Extract to Memoized Components": extraction motivated by perf ("enables early returns before computation"), not readability.
- `rendering-hoist-jsx`: static JSX outside the component.
- Vercel `js-early-exit` / 7.9 "Early Return from Functions": early exit in plain functions/loops for perf; not a component guard-clause rule.
- Vercel `rerender-no-inline-components`: no component definitions inside components.

NOT covered by either skill (room for the house rule):
- Early return / guard clause as the preferred shape for whole-component branches, and its ordering after hooks.
- Nested ternary ban and the `unicorn` one-level exception.
- Boolean `&&` policy (allow vs ternary-always).
- `!!` vs `> 0` vs ternary as the numeric-guard fix.
- Variable-before-JSX as the multi-branch shape.
- Extract-a-component trigger for readability (React docs' "too much nested conditional markup"), as opposed to Kent's pain-point list and Vercel's perf trigger.
- Multiline JSX parentheses inside `&&`/ternary (Airbnb).
- IIFE in JSX, object/map lookup, `switch` — no source on either side; any house position is opinion.
- Turning on `react/jsx-no-leaked-render` (not in `plugin:react/recommended`) and choosing `validStrategies`.
