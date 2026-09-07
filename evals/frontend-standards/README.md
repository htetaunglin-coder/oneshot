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

Skill folder: `SKILL.md`, `rules/colocation.md`, `rules/tailwind.md`, `rules/comments.md`, `coverage-gaps.md`.

## Method per rule

1. Research: read primary sources only (author blogs, official docs, the skills on skills.sh). Reject AI-written summaries. Save the findings under `research/frontend-standards/<rule>/` with a `README.md` that holds the consensus table and the house picks.
2. Write the rule with `mattpocock-skills:writing-for-agents`: leading words, positive phrasing, one checkable gate per decision, no-op test on every sentence, pointers to Vercel rule IDs with a one-line meaning.
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
- Comments: code takes the information first (name, explaining variable, union, assertion, extraction); the residue is a comment when a first-time reader would have to reconstruct it. Six kinds: why, why-not, warning, contract, coupling, reference. JSDoc on an export only when the signature leaves a question; never types in tags. TODO = work + trigger + issue when one exists; a TODO stays until done, obsolete, or abandoned. Simple English, about 20 words. Reviewer softenings taken: no forced extraction, precision comments allowed below the code, TODO never deleted for a missing issue.
- Reviews from other agents: take the technical corrections, reject policy reversals of the user's decisions.

## Rules of engagement

- Do not read `~/Desktop/development/htetaunglin-coder.dev` anymore. `.agents/skills/_Reference` in this repo is the example project when one is needed.
- Organization and standards only. No aesthetics.
- Skill stays in this repo, not `~/.claude/skills`. Link it globally only for a trigger test, then unlink.
- User reviews each change before commit. Ask before committing.
- Replies: concise, plain, STE-style. Tables for comparisons. Report numbers in tables, not prose.

## Next, in order

1. state: zustand vs context vs URL vs server. TkDodo posts already read (Working with Zustand; Zustand and React Context: context is DI, store for state; useCallback posts). wshobson five-row table.
2. file order, magic numbers, conditional-render ladder (early return → sub-component → IIFE last), error and loading, testing, business logic.
3. Move the 1,000-line check to a linter (`max-lines` on ESLint; Biome has none). Add `DECISIONS.md` per rule change.

## Open gaps

See `.agents/skills/frontend-standards/coverage-gaps.md`.
