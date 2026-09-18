# frontend-standards: working process

Read this first after a context reset. It holds the method, the decisions, and what is next. The skill itself is at `.agents/skills/frontend-standards/`.

## Goal

One skill, `frontend-standards`, with the house rules for React + Next.js + Tailwind + shadcn code in a feature-based folder structure. It complements the Vercel skills (`react-best-practices`, `composition-patterns`, `nextjs`, `shadcn`) and never repeats them. Rules cover organization and standards, not UI aesthetics. One rule file per topic under `rules/`.

## Done

| Rule | Words | Evals | Commit |
|---|---|---|---|
| colocation | ~1,290 | 13 training, 5 holdout, 4 trigger | c113c19, f3b60fb |
| tailwind | ~950 | 9 training, 7 adversarial holdout | 73f6c62 |
| comments | ~1,600 | 10 training, 7 adversarial holdout | de00843 |
| file-order | ~1,040 | 8 training, 5 adversarial holdout | ebefb3a |
| magic-literals | ~1,270 | 8 training, 13 adversarial holdout in three rounds | 5fe8f73 |
| conditional-render | ~1,100 | 8 training, 5 adversarial holdout | 15955b2 |
| state | ~230 | 5 probe, control first; 2 regression | pending |
| error-loading (probe, no rule) | 0 | 5 probe, control 5/5 | pending |

Skill folder: `SKILL.md`, `rules/colocation.md`, `rules/tailwind.md`, `rules/comments.md`, `rules/file-order.md`, `rules/magic-literals.md`, `rules/conditional-render.md`, `rules/state.md`. Maintainer files stay here in `evals/`: `coverage-gaps.md`, this README.

## Method per rule

1. Research: read primary sources only (author blogs, official docs, the skills on skills.sh). Reject AI-written summaries. Save the findings under `research/frontend-standards/<rule>/` with a `README.md` that holds the consensus table and the house picks.
2. Write the rule with `mattpocock-skills:writing-for-agents`: leading words, positive phrasing, one checkable gate per decision, no-op test on every sentence, pointers to Vercel rule IDs with a one-line meaning.
A probe comes first when the rule would be a tool pick rather than a code shape (state was the case): control arm only on four or five failure shapes; text is written only for a captured failure, sourced, tool-neutral, and shorter than the pointer it replaces; a clean control is logged as evidence and closes the question.
3. Training fixtures: a fresh agent that has NOT read the rule writes N `cases/<id>/task.md` plus one `expected.md`. task.md never hints at the answer.
4. Two solvers in parallel, fresh agents: one reads the rule files then the cases; one reads only the cases. Both write a table: decision, files, justification, and (with-skill only) `Rule coverage: direct | by analogy | silent` plus a "Rule text I found unclear or missing" list. Neither may open `expected.md` or glob the folder.
5. Score against `expected.md`. Cases where both agree are no-ops (model default). Cases where with-skill fails are rule defects. Fix the text, re-run only the touched cases.
6. Adversarial holdouts: cases designed so the model's default is the wrong answer by our standard. This is the run that proves the rule changes behavior. Without-skill run is the control.
7. After any review-driven edit, regression re-run of the touched cases.
8. Keep: fixtures, keys, one `scorecard.md` per rule. Delete raw result tables. Evals live in `evals/frontend-standards/<rule>/`, outside the skill folder.
9. Commit only the skill and evals paths: `git add <paths> && git commit -m "..." -- <paths>`. The repo has ~190 unrelated staged files; never sweep them in.

## Decisions made (do not re-litigate)

- Precedence: repository standards → the Vercel rule the task touches → these rules. Surface conflicts, do not override.
- Layers: shared → features → app, one-way. Features never import each other. App is glue. No feature `index.ts` barrels (Vercel `bundle-barrel-imports`).
- Extract on a named problem that makes a typical change understandable with less context. Share on shared ownership (who changes it, not who reads it). Never on count.
- File size: hard limit under 1,000 lines, hand-written source only. Static data and generated files exempt. Over-limit file: focused change plus report; extract first only when the task touches the block.
- Tailwind: classes stay on the element. Repetition is evidence, not a command. Ladder `className` → `cn()` → variant map → `cva`. `cva` gate: five or more options across two or more axes, or compound rule, or exported type (house convention). No `@utility` escape, no spacing rule (styling, not organization). Three kinds of variable: runtime value, layout contract, design token.
- Tests: file beside module, setup beside assertion, no `beforeEach` shared state.
- File order: newspaper order. Directive and imports; constants then types the main export or two or more parts read; import-time expressions below every `const` they read; main export; sub-components in render order; helpers by first call. A one-reader constant or type sits above its reader and moves up when a second reader appears. `function` declarations for components, hooks, helpers. Exports at the declaration; `src/components/ui` keeps the registry shape. Existing files are not precedent; focused-change rule applies. Research in `research/frontend-standards/file-order/`.
- Magic literals: inline when the meaning is on the line (0, 1, `-1` from `indexOf`, formula-obvious, mechanism durations, library constants and arguments, protocol codes, one default, test fixture inputs). Named when decoded, a business limit, the same meaning twice, a compared or stored string, cross-file, or a reused JSX design number. Name says meaning and unit. Module-level `CONSTANT_CASE` including objects; no precedent from camelCase files. `as const` array plus derived union; no `enum`, even on request. Research in `research/frontend-standards/magic-numbers/`.
- Conditional rendering: the lowest rung that holds the branch. Early return after the last hook; `&&` for a boolean or comparison, ternary with `null` otherwise; one ternary for two; a map or a sub-component with early returns for three or more keyed by a value; a variable before the JSX for several flags; a sub-component when a branch has state, hooks, handlers, or a nested element condition; IIFE last, one branch only. Ternaries never nest. Extraction on length alone fails the colocation test. Research in `research/frontend-standards/conditional-render/`.
- State: which tool holds a value and the store shape are the repository's; the rule holds the one shape that leaks, a store with request data at module level, created per request inside its provider instead. Everything else passed the probe by model default and is written nowhere. Research in `research/frontend-standards/state/`.
- Error and loading: no rule. Probe 5/5 by model default; Vercel `nextjs` holds the redirect and not-found shapes. Research in `research/frontend-standards/error-loading/`.
- Comments: code takes the information first (name, explaining variable, union, assertion, extraction); the residue is a comment when a first-time reader would have to reconstruct it. Six kinds: why, why-not, warning, contract, coupling, reference. JSDoc on an export only when the signature leaves a question; never types in tags. TODO = work + trigger + issue when one exists; a TODO stays until done, obsolete, or abandoned. Simple English, about 20 words. Reviewer softenings taken: no forced extraction, precision comments allowed below the code, TODO never deleted for a missing issue.
- Reviews from other agents: take the technical corrections, reject policy reversals of the user's decisions.

## Rules of engagement

- Do not read `~/Desktop/development/htetaunglin-coder.dev` anymore. `.agents/skills/_Reference` in this repo is the example project when one is needed.
- Organization and standards only. No aesthetics.
- Skill stays in this repo, not `~/.claude/skills`. Link it globally only for a trigger test, then unlink.
- User reviews each change before commit. Ask before committing.
- Replies: concise, plain, STE-style. Tables for comparisons. Report numbers in tables, not prose.

## Next, in order

1. Trim pass is the last item. Business logic closed by delta: domain policy is never this skill, seam shapes are colocation by the Share test, component APIs are Vercel `composition-patterns` (see coverage-gaps). Testing dropped: too broad for the time. Error and loading closed: probe 5/5 by default, no text. State is closed: probe 4/5 by default, one sentence added, tool picks parked as repository decisions (see coverage-gaps, state).
2. Tooling closed: every rule's Tooling section names ESLint and Biome as examples with a version stamp and leaves the repository's linter in charge; `DECISIONS.md` dropped, this README holds the decisions.

## Open gaps

See `evals/frontend-standards/coverage-gaps.md`.

## Reviews taken (2026-09-16)

Third review, all four rules. Technical corrections applied: `route.ts` has no default export; `middleware.ts` is `proxy.ts` on Next 16; default non-primitive props point at `rerender-memo-with-default-value`; `style={{ "--x": v }}` needs a `CSSProperties` cast; `extendTailwindMerge` is needed for custom `@utility` and ambiguous names, not `@theme` colors; `no-use-before-define` runs with `variables: false` so a function body may read a later `const`, matching Biome. Ambiguities fixed: "who changes it"; import-time reader; ladder scan direction and the five-option sum; conflict clause covers Vercel rules. No-ops cut: hooks above early return, blank-line sentence. Kept on purpose: test-beside-module sentence, comments filler list, tailwind hoisted-string example (the user's original complaint). `coverage-gaps.md` moved here; colocation fixtures moved under `colocation/`. Version stamps added to the Tooling sections. Regression after the edits: colocation 06, H2; tailwind T04, A3, A6; file-order F07, B4. 7/7, all direct.

## Review taken (2026-09-17), magic-literals

Fourth review, claims verified by the reviewer against ESLint 9.39.4, tsc with zod 3 and 4, Biome docs. Gate misfires fixed: platform strings and `""` inline; array indexes inline and the 0/1 row wins over the duplicate row; formula row narrowed to unit conversion or identity, a rate or fee takes a name; a default that is a limit enforced elsewhere takes a name. Contradictions fixed: tailwind example `tone` → `TONE`; `CONSTANT_CASE` scoped to constant data, call results keep camelCase; file-order example `MORE_LABEL` replaced with `MAX_LABEL_CHARS`; `CSSProperties` cast added to the variable snippet. Technical: ESLint allows JSX numbers (rule text and research row corrected); the option recipe removed, since the rule cannot encode the formula exception. Trims: comparison-to-function sentence, render-only-string clause, "the union types the props", THREE example compressed. Kept: none of the reviewer's trims were refused. Two new holdout rounds requested by the reviewer: platform strings, business default, formula vs rate (N11 to N13).

## Review taken (2026-09-17), conditional-render

Fifth review, claims verified by the reviewer against eslint-plugin-react 7.37.5 (`jsx-no-leaked-render` with the rule's exact config), TanStack Query 5.83 types, ESLint docs status. No technical errors found; the fixes are cue-word fixes. Ladder rows fixed: row 6 now says "of its own, declared inside the branch rather than passed to it" and row 2 says a callback passed to one element keeps it there; row 8 carries "one branch, one `if` at most"; the `jsx-no-leaked-render` repo rule is stated once in Guards and row 2 points there. Contradictions fixed: `enabled: !!session` → `enabled: session != null`; the nested-prop sentence now says `if` chain, never a ternary chain, matching the R08 key; magic-literals gets the `PascalCase` exemption for a component picked from a map. Two-way reads fixed: "one line when it fits; otherwise each branch in parentheses"; `&&` leak narrowed to numbers and strings, since `null` renders nothing and a truthy string renders itself. Trims: the Guards restatement of row 2; the prop clause in Extraction. Kept: the `switch` sentence (cheap guard). Ladder scan unified top-down across tailwind and conditional-render: the tailwind override sentence moved above its table; "rung" replaced by "row" so no directional word disagrees with the scan. Holdout control re-run on Opus 5 to remove the mixed-model caveat; see the scorecard.

## Review taken (2026-09-18), state

Sixth review. Blocker: `SKILL.md` description was 1,134 characters against the Agent Skills cap of 1,024; trimmed to 471, per-rule summaries dropped from the field (the table carries them), trigger terms kept. State findings: P1 key scored a reason the rule never asks for, now recorded not scored; P1 fixture had `features/reports` importing `features/session`, corrected to shared scope and re-run (✅ direct); `"use client"` added to the state.md snippet. Reviewer verified the fourth and fifth review fixes in the files. Process endorsed: probe first, control only, bar written before the run.

## Review taken (2026-09-18), tooling and business logic

Seventh review. Factual fix: Biome has `style/noExcessiveLinesPerFile` (2.3.12, `maxLines` default 300, off by default); the colocation Tooling sentence that said Biome has no line-count rule is corrected and the stamp reads Biome 2.3. Refinement taken: business logic split into domain policy (never this skill) and seam shapes (already colocation by the Share test); reopen only on a captured failure on a seam shape. Endorsed: close without a probe; a best-practices section would be the only part the no-op test cannot score.
