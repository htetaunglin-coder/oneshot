# Magic-literals scorecard

## Run 1 (2026-09-17), 8 training cases

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| M01 retry delay cap | `MAX_ATTEMPTS`, `BASE_DELAY_MS`, `MAX_DELAY_MS` | ⚠️ `5` left, reported (no focused-change clause) | ✅ |
| M02 price separators | no constants for `100`, `2` | ✅ | ✅ |
| M03 sidebar width | one `SIDEBAR_WIDTH_PX`; text-length `320` untouched | ✅ | ✅ |
| M04 sort set | `as const` array + union, no `enum` | ✅ | ✅ |
| M05 storage key | one key constant, mismatch named | ✅ | ✅ |
| M06 durations, defaults | `300` param inline; `5 * 60 * 1000` either | ✅ | ✅ |
| M07 naming case | new limit `CONSTANT_CASE` | ✅ | ✅ |
| M08 comparison to function | `isPremiumTier` in lib | ✅ | ✅ |

With skill 7/8 plus one soft. Without 8/8. All no-ops. M01 exposed a rule gap: no focused-change clause; added.

Rule text changes driven by the with-skill "unclear" list
1. Focused-change clause: a literal on a touched line is named; one elsewhere in the edited function too; elsewhere in the file, reported.
2. Duration row applies to a mechanism; a duration that is a rule is a business limit.
3. Protocol codes and library arguments stay inline.
4. `??` fallback compared later is a sentinel.
5. Same digits, different meaning: two literals.
6. Subset of a union typed against the parent.

## Adversarial holdout round 1 (2026-09-17), 5 cases

| Case | Bait | Without skill | With skill |
|---|---|---|---|
| N1 name everything | constants for `100`, `2` | ✅ | ✅ |
| N2 constants bag | append to `src/lib/constants.ts` | ✅ | ✅ |
| N3 enum instinct | `enum` | ✅ | ✅ |
| N4 rename the value | `THIRTY` → `FIFTY` | ✅ | ✅ |
| N5 literal once | `ONE_SECOND_MS`, `ZERO` | ✅ | ✅ |

5/5 both. None of the baits bit; the model's defaults match the rule here.

## Adversarial holdout round 2 (2026-09-17), 5 sharper cases

| Case | Bait | Without skill | With skill |
|---|---|---|---|
| N6 camelCase precedent | match the file | ❌ `maxFiles` | ✅ `MAX_FILES`, mismatch named |
| N7 enum by request | comply with `enum` | ✅ const object + union | ✅ `as const` array, reason given |
| N8 loose test numbers | name fixture inputs | ❌ inputs named | ✅ inputs inline, rate imported |
| N9 card image width | separate literals, skeleton left | ✅ | ✅ shared constant, skeleton reads it |
| N10 second role check | second inline comparison | ✅ helper | ✅ `canManageWorkspace` + `MANAGER_ROLES` |

With skill 5/5, without 3/5. The rule changes behavior on N6 and N8.

Rule text changes driven by round 2
1. Naming: an existing file's camelCase constants are not a precedent, even when asked for consistency.
2. A literal compared against a union-typed value stays inline; a subset check is a function; an `enum` request is met with the `as const` form and one line why.
3. One-concept `lib` file is an owner. Two siblings sending a value: feature `lib` module.
4. Feature-owned value read by JSX and a class: JS constant is the source, class reads a variable set on the ancestor.
5. Test expected output written as the formula that produces it.

## Regression after edits (2026-09-17)

Thirteen sentences added across the gate, naming, strings, and location sections.

| Case | With skill |
|---|---|
| M01 retry delay cap | ✅ `MAX_ATTEMPTS` now named (focused-change clause), direct |
| M02 price separators | ✅ nothing named, direct |
| M06 durations | ✅ session warning named as a rule, `300` inline, direct |
| N6 camelCase precedent | ✅ `MAX_FILES`, direct |
| N7 enum by request | ✅ `as const`, no enum, direct |
| N9 card image width | ✅ constant in feature lib, skeleton reads a variable, direct |

No regressions. 6/6.

## Fourth review and holdout round 3 (2026-09-17)

Review verified by running ESLint, tsc, and Biome docs. Four gate misfires fixed (platform strings and `""`, array indexes, rate vs unit conversion, default vs enforced limit), four cross-rule contradictions fixed, two tooling claims corrected, four trims. Three new holdouts on the edges the reviewer named.

| Case | Bait | Without skill | With skill |
|---|---|---|---|
| N11 platform keys | `ENTER_KEY`, `EMPTY_QUERY` | ✅ switch on `event.key` | ✅ |
| N12 inbox page size | two literal 40s, or clamp left at 25 | ✅ one `INBOX_PAGE_SIZE` | ✅ in feature lib, both import |
| N13 processor fee | `0.029` left bare, or `100`/`2` named | ✅ | ✅ |

3/3 both. Regression on N1, N9, M02, M06 with the edited text: 4/4, all direct. Rule change from this run: a `"use server"` owner cannot export a constant, so the shared value goes to a feature lib module.

Final: training 8/8 with skill; holdouts 13 cases, with skill 13/13, without 11/13. The rule changes behavior on N6 (naming under precedent) and N8 (test fixtures); the rest is guardrail.

## Trim pass (2026-09-18), regression

Cut: the `z.enum(SORT_ORDERS)` sentence in Strings. With skill on the trimmed text, fresh agent, opencode runner (not Opus 5; directional):

| Case | With skill |
|---|---|
| N7 enum by request | ✅ `as const` array plus derived union in a feature lib, dropdown iterates, no enum; direct; the `"all"` sentinel took a shared name (key allows) |
