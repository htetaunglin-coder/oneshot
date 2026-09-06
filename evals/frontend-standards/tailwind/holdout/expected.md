# Expected decisions (holdout set A)

Solvers see only `cases/*/task.md`. Grade the diff and the one-or-two-sentence explanation against the decision below. Every case is built so the tempting first move is the wrong one; the change request in each `task.md` deliberately nudges toward that tempting move.

## A1-long-string-five-files

**Decision: edit the five strings in place. No shared component, no shared string.**

The five wrappers (`<form>`, `<div>`, `<figure>`, `<aside>`, `<section>`) share a class string by coincidence, not a UI responsibility. Each one will change for a different reason (form validation layout, chart sizing, list density, export affordances, settings grouping). Tying them together means a change requested for one panel silently lands on the other four.

Pass:
- All five wrappers gain `dark:border-border/60` (position inside the string does not matter).
- No new shared component wrapping the five, no exported string/array holding the classes, no `@utility`, no `@apply` rule.
- Explanation says something like "they only look the same; different reasons to change; edited each in place".
- Optional, acceptable: a note that `src/components/ui/card.tsx` already exists and could replace the wrapper on any element that is actually a card. Not required.

Fail:
- A new `Panel`/`Surface`/`CardShell` component, or migrating the five onto `Card` as the mechanism for the change.
- A `panelClasses`-style shared string imported into five files.
- A `@utility panel` or `.panel { @apply ... }` used across the five.
- Fewer than five wrappers updated.

## A2-named-constant-three-uses

**Decision: the ring is one visual contract applied to three different element types. Move it to a single `@utility focus-ring { ... }` in `src/app/globals.css` (or reuse the ring already carried by the repo's `button` variants if the solver reaches for that), apply the class `focus-ring` on the input, the `role="button"` div and the link, and delete `focusRing` from `styles.ts`.**

Pass:
- `src/app/globals.css` gains a `@utility focus-ring` (or equivalently named) block containing `outline: none` / `ring` styles including the offset and the new `ring-offset-background` colour under `focus-visible`, and the three elements use `focus-ring` in `className`. Idiomatic Tailwind v4 form is acceptable, e.g. `@utility focus-ring { @apply focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 focus-visible:ring-offset-background; }` or hand-written CSS with `&:focus-visible`.
- OR: the solver reuses the ring styles from `src/components/ui/button.tsx`'s variants (acceptable, since shadcn's button already carries the same ring) and removes the string from `styles.ts`.
- `styles.ts` no longer exports `focusRing` (file deleted or the export removed).
- Explanation identifies the ring as a single shared contract across unrelated elements, which is why it lives in CSS rather than a JS string.

Fail:
- Appending `focus-visible:ring-offset-background` to the existing `focusRing` string and leaving the string in place. This is the tempting one-line fix; it is a fail because the string remains a JS-side style contract that Tailwind cannot see as one utility.
- Creating a wrapper component (`Focusable`, `FocusRing`) around the three elements.
- Adding the classes inline to each of the three elements separately (three copies).

## A3-two-axes-three-options

**Decision: two small lookup objects, one per axis, composed with `cn(base, toneClasses[tone], sizeClasses[size], className)`. Not the shadcn badge pattern.**

There are two independent axes with two options each, no compound rule (accent+md does nothing special), and no exported type derived from the variants. The badge pattern the task points at adds a dependency, a `VariantProps` type and a config object to express what two four-line records already say. That is the tempting path and it is a fail.

Pass:
- A record for `tone` (`neutral` -> `bg-muted text-muted-foreground`, `accent` -> `bg-primary text-primary-foreground`) and a record for `size` (`sm` -> `px-2 py-0.5 text-xs`, `md` -> `px-3 py-1 text-sm`), keyed by the union types, composed with `cn(...)`. Naming and `as const`/`satisfies Record<...>` details do not matter.
- Rendered classes unchanged for all four combinations; props unchanged.
- Explanation says the axes are independent so one map per axis is enough.

Fail:
- Importing `class-variance-authority` / calling its variant builder, or copying `badge.tsx`'s structure.
- Keeping any ternary chain on `tone` or `size`.
- A single flattened four-key map (`"accent-md"`, `"neutral-sm"`, ...) is a soft fail: it works but re-couples the axes; mark it partial.

## A4-single-reader-runtime-value

**Decision: keep the plain inline `style={{ height: `${clamped}%` }}`. Add the clamp in JS. Do not introduce a CSS custom property or an arbitrary-value class.**

Exactly one element reads the value, and it is a per-item runtime number. A CSS variable (`style={{ "--bar-h": ... }}` plus `h-[var(--bar-h)]` or `h-(--bar-h)`) only pays for itself when more than one declaration or more than one element reads the same value. Here nothing else does, so the indirection is pure ceremony. The "inline style looks out of place" remark in the task is the tempting nudge and should be declined.

Pass:
- Height clamped to `[0, 100]` (e.g. `Math.min(100, Math.max(0, value))`), applied in `SparklineBar` or at the `values.map` call site.
- `style={{ height: `${...}%` }}` remains a plain inline style.
- No new `--*` custom property, no `h-[...]`/`h-(...)` arbitrary class, no `@utility`.
- Explanation notes a single reader of a runtime value is exactly what inline `style` is for.

Fail:
- `style={{ "--h": ... }}` with `h-(--h)` / `h-[var(--h)]` / `[height:var(--h)]`.
- Any attempt to express the height purely as Tailwind classes (bucketed `h-1/4`..`h-full` lookups etc.).
- Clamping missing or done only in the aria label.

## A5-apply-in-component-css

**Decision: put the classes on the elements in the TSX and delete the CSS module and its `@apply` rules. No `@utility`.**

`.cell` and `.header-cell` are two elements inside one component; they are not a cross-element contract shared by other components. Component-local `@apply` hides the styling from the place it is used and forces a hop to a second file for every change (which is exactly what this request is). Because the two rules are private to `DataTable`, a `@utility` in `globals.css` is not warranted either; that would promote a component detail to the global layer.

Pass:
- `<td className="border-b px-3 py-2 text-sm whitespace-nowrap">` and `<th className="border-b px-3 py-2 text-xs font-medium uppercase whitespace-nowrap">` (order of utilities does not matter; `cn` or a plain string are both fine).
- `data-table.module.css` deleted and the `styles` import removed.
- Explanation says the classes belong on the elements because they are local to this component, and the module added a level of indirection for nothing.

Fail:
- Adding `whitespace-nowrap` inside the `@apply` lines and keeping the module. This is the tempting one-line fix.
- Replacing the module with `@utility table-cell` / `@utility header-cell` in `globals.css`.
- Keeping the module for one rule and inlining the other.
- Pulling the `td`/`th` into new `Cell`/`HeaderCell` components solely to hold the class strings is a soft fail: acceptable structure, but flag that it was not asked for and the class string should still be on the element.


## Corrections 2026-09-07 after run 1

- A2: `@utility` removed from the rule. Expected is now: inline the ring classes on the three elements and delete `styles.ts`, or call the button `cva` if it already carries the ring. `@utility` is no longer a pass.
- A3: gate moved to five or more options. 2x2 is two maps. Unchanged expectation, now consistent with the rule text.


## A6-single-element-hover-color

**Decision: inject the runtime color once as a CSS variable on the element and consume it from utilities for every state. No JS hover state, no template-literal class.**

The color is a runtime value, so it must enter through `style`, but it is read in three states (rest, hover, focus-visible). Putting the raw value in `style` covers only the rest state; hover and focus cannot be expressed in `style` without JS event state, and `` `bg-[${color}]` `` is never generated by Tailwind because the class does not exist at build time. A single variable set on the element gives the utilities a stable name to read at every state. One element is enough to justify the variable; the justification is the number of states, not the number of elements.

Pass:
- `style={{ "--tag": color } as React.CSSProperties}` (any name, any cast style) on the `<button>`.
- Classes read the variable: `bg-(--tag) hover:bg-(--tag)/80 focus-visible:ring-(--tag)` or the long forms `bg-[var(--tag)]`, `hover:bg-[var(--tag)]/80`, `focus-visible:ring-[var(--tag)]`, or `[background:var(--tag)]` equivalents. Opacity via `color-mix` in an arbitrary value is also fine.
- `focus-visible:ring-2` (or reusing the existing `ring-2` for focus) so the ring is visible.
- No `backgroundColor` left in `style`.
- Explanation says the value enters once via the variable and the utilities read it per state, so no JS is needed for hover or focus.

Fail:
- `useState` + `onMouseEnter`/`onMouseLeave` (or `onFocus`/`onBlur`) toggling a computed hover color in `style`.
- `` className={`bg-[${color}] hover:bg-[${color}]/80`} `` or any template literal that builds a utility from the runtime value.
- Computing the 80% color in JS (hex-to-rgba helper) and swapping `style` values.
- A wrapper element or second element added only to "share" the variable; the variable is fine on the single element.
- Soft fail: `style={{ "--tag": color }}` for the background and hover but the focus ring still hardcoded to a theme color. The request asked for the tag color on the ring.


## A7-header-height-contract

**Decision: declare the header height once as a responsive CSS variable on the shell and have the header and every dependent read it. The number exists in one place.**

Four elements in three feature folders depend on the same number (`h-14`, `pt-14`, `top-14`, `calc(3.5rem+1rem)`, plus the `max-h` calc on the sidebar). Changing the literal in each place and adding `md:` twins everywhere makes eight or more literals that must be kept in sync by hand; the next height change repeats the whole hunt. A variable on the shell that the header and the dependents read means the responsive value is written once and the dependents never mention a number.

Pass:
- The shell root carries the value with a responsive override, either as classes `[--header-height:3.5rem] md:[--header-height:4rem]` or a `style` plus a media rule in `globals.css`, or a small block in `globals.css` under a selector the shell owns. Any name is fine.
- `Header` reads it: `h-(--header-height)` or `h-[var(--header-height)]`. No `h-14 md:h-16` on the header.
- `main`: `pt-(--header-height)`.
- `TocSidebar`: `top-(--header-height)` and `max-h-[calc(100dvh-var(--header-height))]`.
- `ScrollButton`: `top-[calc(var(--header-height)+1rem)]` (or `top-[calc(var(--header-height)+--spacing(4))]`).
- Declaring it at `:root` in `globals.css` under `@theme` or a plain rule is acceptable if the responsive value is still written once and every dependent reads it; note in grading that the shell is the better owner because it is the element that renders the header.
- Explanation says the height is written once, the header and every dependent read it, and a future change touches one line.

Fail:
- Editing the literal in each file: `h-14 md:h-16`, `pt-14 md:pt-16`, `top-14 md:top-16`, `top-[calc(3.5rem+1rem)] md:top-[calc(4rem+1rem)]`. This is the tempting fix and it is the failure the case exists to catch.
- A React context, hook, or prop that passes the height number to dependents.
- Measuring the header with a ref / `ResizeObserver` and writing the height to state.
- Partial: variable declared but one or more dependents still carry a literal (`top-14`, `3.5rem`).
- Variable declared on the `<header>` itself rather than on the shell or `:root`, so siblings (`main`, sidebar, button) cannot read it and the solver falls back to literals for them.
