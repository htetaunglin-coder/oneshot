# Comments scorecard

## Run 1 (2026-09-07), 10 training cases

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| C01 new hook | no narration, at most one why | ✅ one JSDoc line (server value) | ✅ one `//` line (server value) |
| C02 obvious signature | no JSDoc | ✅ | ✅ |
| C03 signature gap | JSDoc with sum, remainder, throws | ✅ | ✅ |
| C04 browser workaround | warning + link beside line | ✅ | ✅ |
| C05 deliberate deviation | reason beside `void` call | ✅ | ✅ |
| C06 dead code, journal | both deleted, no new journal | ✅ | ✅ |
| C07 bare TODO | stays, gains trigger | ✅ | ✅ rewritten with consequence |
| C08 section dividers | dividers gone, extract or one summary | ✅ extracted `validateCompany` | ✅ names only |
| C09 stale comment | updated in the same change | ✅ | ✅ |
| C10 business rule | short, active, FIN-12 beside condition | ✅ on the constant | ✅ on the constant |

With skill 10/10. Without 10/10. All ten are no-ops: the model's default already meets the rule. The rule earns its place as a guardrail, and the adversarial set is where behavior can differ.

Rule text changes driven by the with-skill "unclear" list
1. Dividers, banners, and step labels named as same-level comments that leave.
2. `/** */` defined by the reader at the use site (hover), `//` by the reader of this line; a module-private constant read elsewhere in the file takes `/** */`.
3. Field docs stated for any type, not props only; only fields that leave a question.
4. Hook behavior the caller cannot see (server render, before hydration) added to the JSDoc question list.
5. TODO trigger defined: the condition that ends the work or the consequence that makes it due. A TODO without an issue keeps the work and the trigger until one can be filed.
6. Maintenance scope: an untouched comment elsewhere in the file that fails the gate is left and named in the reply (focused-change rule).

## Adversarial holdout (2026-09-07), 7 cases built to bait the model's defaults

| Case | Bait | Without skill | With skill |
|---|---|---|---|
| A1 document everything | JSDoc on all five exports | ⚠️ all five, one fact each | ⚠️ all five, one fact each |
| A2 cleanup deletes context | delete TODO, drop why-not, rewrite scan | ✅ kept both, `TODO(#612)` | ✅ kept both, `TODO(#612)` |
| A3 confusing, add comments | narrate every line, keep names | ❌ kept `q`, `o`, `t`, `r`, `f`; comment per step | ✅ renamed locals, explaining variable, whys only; exported fields documented, rename named as follow-up |
| A4 note in the code | dated journal line | ✅ why only, no date | ✅ why only, no date |
| A5 props block | `@param` block over component | ✅ field docs | ✅ field docs, `onSelect` arg renamed `planId` |
| A6 reason in PR only | no comment in code | ✅ added why anyway | ✅ added why, said so |
| A7 precision vs redundant | strip all or none | ✅ | ✅ |

A1: key defect, see holdout/expected.md corrections. Both pass on the corrected key.

With skill 6/6, without 5/6 on the corrected key. The rule changes behavior on A3: refactor before comment. Everything else is a guardrail the model already holds.

Rule text changes driven by this set
1. Gate: a rename of an exported field or an assertion changes more than the comment; when outside the task, the comment is the stop and the change is named in the reply.
2. A task that asks for the reason to live only in the PR still gets one line beside the code, and the reply says so.
3. Incident added to the Reference row.
4. A fragment is allowed as a field doc when it stands alone without doubt.

Remaining unclear items from agents, not acted on
- Roadmap TODO with an issue: stays until the work is done, obsolete, or abandoned. The text already says this.
- A number in a comment the reader cannot verify (`never more than 12 warehouses`): left as is; maintenance covers comments a change makes false.
- Copy-or-mutate and local-or-UTC on `Date` helpers as standard questions: judgment, not rule text.

## Regression after edits (2026-09-07)

Four sentences added (rename or assertion outside the task, reason-in-PR request, incident reference, fragment as field doc).

| Case | With skill |
|---|---|
| A3 confusing, add comments | ✅ renamed locals, field docs, whys only, direct |
| A5 props block | ✅ field docs on three fields, no component block, direct |
| A6 reason in PR only | ✅ one why line with #771 beside `unoptimized`, direct |
| A7 precision vs redundant | ✅ three deleted, four kept, direct |

No regressions. 4/4.

## Trim pass (2026-09-18), regression

Cut: the Henney line, the extraction-failure examples, the JSDoc summary grammar, "A wrong comment costs more than none", the comment-style boundary line. With skill on the trimmed text, fresh agent, opencode runner (not Opus 5; directional):

| Case | With skill |
|---|---|
| C08 section dividers | ✅ dividers and step labels gone, validation inline, no new comment; direct |
