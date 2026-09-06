# Tailwind scorecard

## Run 1 (2026-09-07), 10 training cases

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| T01 repeated utilities | inline | ✅ | ✅ |
| T02 hoisted constant | inline, delete constant | ✅ | ✅ |
| T03 one axis, three states | variant map | ✅ | ✅ |
| T04 two axes, five options | cva | ✅ | ✅ |
| T05 dynamic class name | whole-class map | ❌ cva | ✅ map |
| T07 runtime value, two readers | CSS variable | ✅ | ✅ |
| T08 repeated hex | @theme token | ✅ | ✅ |
| T09 button look on link | buttonVariants() | ✅ | ✅ |
| T10 template string merge | cn(), className last | ✅ | ✅ |

With skill 8/9. Without 9/9. (T06 removed with the spacing rule.)

Findings
1. The count gate "four or more options → cva" fired on a one-axis lookup. Fixed: one axis is always a map; cva needs two or more axes and four or more options, or a compound rule, or an exported type.
2. Nine of ten cases are no-ops: the model does them by default. The training set was written from the rule and tests recall only. An adversarial holdout set is needed to show where the rule changes behavior.
3. Added: shadcn token form pointer. Spacing rule later cut as styling, not organization.

## Adversarial holdout (2026-09-07), 5 cases built to bait the model's defaults

| Case | Bait | Without skill | With skill run 1 | Run 2 | Run 3 |
|---|---|---|---|---|---|
| A1 same 9-class string, five files, three features | shared constant or component | ❌ constant in `ui/` | ❌ `@utility` | ❌ `Panel` with `asChild` | ✅ in place |
| A2 named constant, three users | keep the constant | ❌ | ✅ inlined, deleted | | |
| A3 two axes, 2x2 | `cva` | ❌ | ❌ `cva` | ✅ two maps | |
| A4 single-reader runtime value | CSS variable | ✅ | ✅ | | |
| A5 `@apply` in CSS module | edit the CSS | ✅ | ✅ by analogy | | |

Final: with skill 5/5, without 2/5. The rule changes behavior on A1, A2, A3.

Rule text changes driven by this set
1. `@utility` escape removed. Cross-element look calls an existing `cva` or writes the classes on each element.
2. `cva` gate moved from four to five named options across two or more axes. 2x2 is two maps.
3. `@apply` in a stylesheet named as a hoisted string under a selector.
4. Responsibility defined: structure, behavior, or semantics. A look is none of those; a wrapper whose only prop is `className` is a hoisted string in JSX syntax.

Remaining unclear items from agents, not acted on
- Repeated coordinated edits of a class string read as "change together" pressure. The responsibility sentence answers it; watch for recurrence.
- When an existing shadcn primitive (`Card`) should replace a matching hand-written string. Pointed at the shadcn skill.

## Regression after review edits (2026-09-07)

Four sentences changed (runtime write target, variable trigger, cn wording, ladder precedence); spacing section cut.

| Case | With skill | Without skill |
|---|---|---|
| T04 two axes, five options | ✅ cva, direct | |
| T10 template string merge | ✅ cn, direct | |
| A3 two axes, 2x2 | ✅ two maps, direct | |
| A4 single reader, unconditional | ✅ style prop, direct | |
| A6 single element, hover reads color (new) | ✅ variable on element, direct | ✅ same |
| A7 header height contract (new) | ✅ variable on shell, by analogy | ✅ variable in :root |

No regressions. A6 and A7 are no-ops: the model reaches for the variable by default. Kept as ownership guardrails. Closed from A7: responsive declaration form; one read syntax, `(--x)` shorthand, bracket only inside `calc()`.
