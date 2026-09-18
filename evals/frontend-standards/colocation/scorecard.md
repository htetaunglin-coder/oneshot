# Scorecard: colocation rule, run 1 (2026-09-05)

| Case | Expected | With skill | Without skill |
|---|---|---|---|
| 01 small-helper-once | in place | ✅ in place | ✅ in place |
| 02 large-editor-one-consumer | extract to feature-local file | ✅ extracted `note-editor.tsx` | ⚠️ in place, size flagged |
| 03 two-features-one-policy | one owner in `src/lib`, delete copies | ✅ | ✅ |
| 04 three-lookalikes-diverge | keep separate | ✅ | ✅ |
| 05 button-legit-variant | add variant to ui/button | ✅ | ✅ |
| 06 helper-branches-on-caller | decline option, caller prepends | ✅ | ✅ |
| 07 feature-local-server-fetcher | update fetcher in place, no route handler | ✅ | ✅ |
| 08 api-contract-two-consumers | type owned by api module, `import type` | ✅ | ✅ |
| 09 1200-line-screen-with-editor | extract editor, file under limit | ❌ in place, size reported, split deferred | ❌ in place, no report |
| 10 1050-line-tightly-coupled | focused fix, report, refuse split | ✅ | ⚠️ fix, refused split, no size report |
| 11 700-line-mixed | separate unrelated responsibilities | ❌ in place, no extraction | ⚠️ export moved to lib, form left |
| 12 tiny-fix-in-large-file | one-line change | ✅ | ✅ |
| 13 scattered-short-files | consolidate | ✅ five files → one | ❌ change spread over three files |

With skill: 11/13. Without: 8/13 clean, 3 partial, 2 fail.
Skill changed the answer on 02, 10 (report), 13. Both agents agree on 01, 03–08, 12: the model does those by default.

## Rule defects found

1. **File-size rule 3 inverts the intent.** Over-limit file (09) gets less restructuring than an under-limit one with the same shape (02), because "no restructuring" is stated only above the limit. Agent followed the text and failed the case.
2. **Business-logic paragraph reintroduced a count.** "Moves to `features/<x>/lib` when a second file calls them." Agent used it to refuse 11. Contradicts "extract on a named problem".
3. Gaps the agent listed: task-mandated path vs rule (08), static-data exemption for a hand-written SVG component (12), which file absorbs a consolidation (13), `src/lib` sub-folder naming (03).

## Run 2 (after fixes), cases 02, 09, 11, 13 only

| Case | Run 1 | Run 2 |
|---|---|---|
| 02 | ✅ | ✅ |
| 09 | ❌ | ✅ editor extracted first, count in new file |
| 11 | ❌ | ✅ export run moved to `lib/profile-export.ts`; preferences form left (task did not touch it, accepted) |
| 13 | ✅ | ✅ |

Defects 1 and 2 closed. Untouched cases carried over: 13/13.

## Holdout, run 1 (2026-09-06)

Five cases written by an agent that never saw the rule. Judged against the writer's own answer, a no-skill baseline, and the skill agent's coverage tag.

| Case | Writer | No skill | With skill | Coverage | Verdict |
|---|---|---|---|---|---|
| H1 shared undo hook | `src/hooks`, generic, caller passes delete fn + text | same | same | direct | pass |
| H2 cart context | stays in `features/cart`, public `index.ts` | same | moved to `src/hooks/use-cart.tsx` | by analogy | diverges, see below |
| H3 signup schema | `features/signup/lib`, list in own file | same | same, list in same file | direct | pass |
| H4 route paths | `src/lib/routes.ts` | `src/config/routes.ts` | `src/lib/routes.ts` | by analogy | pass |
| H5 test render helper | `src/test/render.tsx` | same | same | silent | pass |

4/5 agree with both independent opinions.

## H2 finding

Skill agent read "code whose ownership is already shared goes to the shared scope directly" as "three consumers = shared ownership" and moved the cart domain out of its feature. Writer and baseline keep the domain in `features/cart` behind a public entry, which assumes cross-feature imports are allowed. The portfolio's `agent_docs/code-organization.md` forbids feature-to-feature imports, so under that repo rule the skill answer is consistent. Decision needed: public feature entry (A), no cross-feature imports (B), or layered core/leaf (C).

## Run 2 (2026-09-07), H2 only, after the Layers section

| Case | Run 1 | Run 2 |
|---|---|---|
| H2 cart context | moved domain to `src/hooks`; header badge in `src/components`; checkout imports `useCart` | provider stays in `features/cart`; `CartBadge` from cart feature passed into `SiteHeader` slot; `OrderSummary` takes props; app-layer client glue in the checkout route segment; no cross-feature import; coverage direct |

Matches the corrected answer key. Holdout total: 5/5.

## Trigger test, run 1 (2026-09-06)

Skill listed by description only, four fresh agents, plain requests, no mention of skills. 4/4 fired, including two prompts with no description keywords. Co-fired with `ponytail` (T3) and `codebase-design` (T4); both agreed with the rule.

## Trim pass (2026-09-18), regression

Cut: the size-check consistency summary under File size limit. With skill on the trimmed text, fresh agent, opencode runner (not Opus 5; directional):

| Case | With skill |
|---|---|
| 11 700-line mixed | ✅ export functions to a feature `lib` module, JSON beside CSV, preferences form left with reason; direct |
