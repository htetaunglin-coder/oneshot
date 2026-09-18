# Conditional-render scorecard

Runs on 2026-09-17. The first holdout control ran on Fable 5.1 after a usage-limit switch; it was re-run on Opus 5 after the fifth review (below) with the same result, so every comparison is same-model.

## Run 1, 8 training cases

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| R01 empty state | early returns after hooks | ✅ | ✅ |
| R02 hooks before return | hooks above the guard | ✅ | ✅ |
| R03 numeric `&&` | `> 0` or ternary, no `!!` | ✅ | ✅ |
| R04 boolean `&&` | keep `&&` or ternary, no restructure | ✅ `&&` | ✅ ternary |
| R05 IIFE switch | map or sub-component | ✅ sub-component | ✅ sub-component |
| R06 inline component | module-level `function` | ✅ | ✅ |
| R07 flag precedence | variable before JSX or sub-component | ✅ sub-component | ✅ sub-component |
| R08 nested prop ternary | variable above JSX | ✅ `let look` chain | ✅ map |

8/8 both. All no-ops. The training set is a regression guard.

Rule text changes driven by the with-skill "unclear" list
1. `&&` is the default for a boolean or comparison; ternary otherwise and in a repo that runs `jsx-no-leaked-render`.
2. Map when every branch takes the same props; sub-component when they differ.
3. Class string chosen by precedence: variant map keyed by a derived state, pointer to tailwind.
4. A sub-component that owns a precedence chain may end in `return null`.

## Adversarial holdout, 5 cases

| Case | Bait | Without skill (Fable) | With skill (Opus) |
|---|---|---|---|
| S1 IIFE by precedent, in-file comment | grow the IIFE | ❌ grew it | ✅ sub-component with early returns |
| S2 long flat block | extract for length | ❌ extracted | ✅ extracted on a real second condition, not length |
| S3 nested ternary, "keep it small" | three-level ternary | ✅ | ✅ by analogy |
| S4 `return null` in the child | second condition in the child | ❌ | ✅ parent decides presence |
| S5 effect after guard | hook below the return | ✅ | ✅ |

With skill 5/5, without 2/5. The rule changes behavior on S1, S2, S4.

Rule text changes driven by this set
1. An over-rung block is not a precedent, whatever a comment beside it says; a change that touches it moves it.
2. A hook that needs a value the guard produces: optional value plus `enabled`, or a sub-component.
3. New rung: one element chosen by several flags with precedence.
4. A text ternary is not a nested branch; a hook moves into the sub-component only when it is the only reader.
5. `return null` is for a source the parent does not hold.

## Regression after edits (2026-09-17)

Nine sentences changed.

| Case | With skill |
|---|---|
| R04 boolean `&&` | ✅ `&&`, direct |
| R05 IIFE switch, different props | ✅ sub-component, direct |
| R07 flag precedence | ✅ sub-component, four early returns, `null` last, direct |
| R08 nested prop ternary | ✅ derived state plus variant map, direct |
| S1 IIFE by precedent | ✅ sub-component, direct |
| S3 nested ternary | ✅ sub-component, direct (was by analogy) |

No regressions. 6/6. S3 moved from analogy to direct on the new precedence rung.

Final: training 8/8 both; holdout with skill 5/5, without 2/5. The rule changes behavior on the IIFE-by-precedent, extract-for-length, and child-`return null` baits.

## After the fifth review (2026-09-17)

Holdout control re-run on Opus 5, without skill: S1 ❌ grew the IIFE, S2 ❌ extracted `ContractDetails` for length, S3 ✅, S4 ❌ kept `return null` in the child with no ownership reason, S5 ✅. 2/5, the same three failures as the Fable run. The mixed-model caveat is removed.

Regression with skill after the review edits (rows 2, 3, 6, 8; Guards; nested-prop sentence; ladder scan wording; tailwind override sentence moved):

| Case | With skill |
|---|---|
| R04 boolean `&&` with a callback prop on the element | ✅ `&&`, direct |
| R06 inline component, `!!` ban | ✅ module `function`, no `!!`, direct |
| R08 nested prop ternary | ✅ `if` chain picks a variant-map key, direct |
| S1 IIFE by precedent | ✅ sub-component, hook moved as sole reader, direct |
| tailwind A3 two axes, three options | ✅ two maps, direct |

5/5. Final: training 8/8 both; holdout with skill 5/5, without 2/5, same model.

## Trim pass (2026-09-18), regression

Cut: the ladder intro's last-resort clause, the React quote in Extraction, the map intro line, the IIFE prose section and its `switch` sentence. With skill on the trimmed text, fresh agent per case, opencode runner (not Opus 5; directional):

| Case | With skill |
|---|---|
| S1 dashboard body states | ✅ sub-component with early returns, IIFE removed; direct |
| S2 long enterprise block | ✅ hoisted variable, no new component; direct |
| S4 return null from child | ✅ presence moved to the parent, child guard removed; direct |
| R05 order status cancelled | ✅ sub-component with early returns; direct |
| R06 comment editor counter | ✅ module-level function, state inside, counter; direct |

R06's reply omitted the remount reason the key's rationale names; the Extraction clause was restored, which re-attaches the `rerender-no-inline-components` citation in Sources to text. All five pass on the restored text.
