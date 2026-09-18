# File-order scorecard

## Run 1 (2026-09-14), 8 training cases

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| F01 new file | constants, types, main, sub, helper | ✅ | ✅ |
| F02 single-reader map | above its one reader | ✅ | ✅ |
| F03 second reader | moves to top section | ✅ | ✅ |
| F04 add Footer | after Body, own type above it | ✅ | ✅ |
| F05 cva reads SIZES | below SIZES, above main | ✅ | ✅ |
| F06 arrow file, add helper | `function` declaration at bottom | ✅ | ❌ `const` arrow at bottom |
| F07 page metadata | metadata top, sub-component below page | ✅ | ✅ |
| F08 export list in `components/ui` | corrected key: list stays, new export joins it | ✅ kept list, flagged | ✅ appended |

With skill 8/8. Without 7/8. Six cases are no-ops. F06 is the one training case where the rule changes the shape (`function` over `const` arrow). F08 exposed a key defect: `src/components/ui` is registry-owned, so the list stays; the rule now says so.

Rule text changes driven by the with-skill "unclear" list
1. Order inside the top section: first use.
2. A `cva` with one reader follows ownership: above that reader, below every `const` it reads.
3. Legacy arrow files: new declarations are `function`; existing ones convert when touched.
4. `src/components/ui` keeps the registry shape, export list included.

## Adversarial holdout (2026-09-14), 5 cases built to bait the model's defaults

| Case | Bait | Without skill | With skill |
|---|---|---|---|
| B1 helpers first, from scratch | helpers as arrows above the component | ✅ helpers below | ✅ |
| B2 hoist everything | one-reader map at the top | ❌ top | ✅ above `OrderFooter` |
| B3 second reader left in place | read `TONE` from main, leave it | ❌ left | ✅ moved to top |
| B4 follow existing placement | new config exports after the page | ❌ after page | ✅ above page; old `metadata` left with reason (soft pass) |
| B5 append to export list, `components/ui` | append | ⚠️ appended (correct on corrected key) | ⚠️ converted to inline (key defect at the time) |

On the corrected key: with skill 4/4 plus B5 re-run below; without 1/4 plus B5. The rule changes behavior on B2, B3, B4.

Rule text changes driven by this set
1. Helpers ordered by first call reading top-down.
2. Compound components: root first, then parts in nesting order.
3. An existing file that breaks the order is not a precedent; new declarations take their slot, existing ones move when touched.
4. Config exports generalized: route segment config and every `generate*` export.

## Regression after edits (2026-09-14)

Run A, after the holdout edits (helper order, compound components, precedent, `generate*`):

| Case | With skill |
|---|---|
| B1 helpers first | ✅ direct |
| B4 follow existing placement | ✅ new exports above page; old left and named, soft pass |
| B5 export list | converted; ran before the registry exception landed, superseded by run B |

Run B, after the training edits and the registry exception:

| Case | With skill |
|---|---|
| F06 arrow file | ✅ `function` helper at bottom; touched `ShiftList` converted, `ShiftRow` left |
| F08 card.tsx in `components/ui` | ✅ list kept, `CardFooter` joins it, declaration in nesting order |
| B5 tabs.tsx in `components/ui` | ✅ list kept, `TabsIndicator` in nesting order |

No regressions. Final: training with skill 8/8, without 7/8; holdout with skill 5/5, without 2/5 on the corrected keys (B1 and B5 pass by default).

## Trim pass (2026-09-18), regression

Cut: the first-screen rationale, the divider cross-reference, the `enum` fallback sentence. With skill on the trimmed text, fresh agent, opencode runner (not Opus 5; directional):

| Case | With skill |
|---|---|
| B4 follow the existing placement | ✅ new config exports in slot 2 above the page; old `metadata` and `toTitle` left and named; direct |
