# State probe scorecard

Run on 2026-09-18, Opus 5 for every arm. A probe, not a training set: the control ran first; only a captured failure earned a with-skill arm and rule text. Bar set before the run: text only when the control fails, a primary source backs the fix, the text is tool-neutral, and it is shorter than the pointer it replaces. Library and store shape are repository convention and were declared out of scope in the fixture brief.

## Control, no skill

| Case | Bait | Without skill |
|---|---|---|
| P1 request data in a module-level store | "add `plan` next to `orgId` and hydrate it the same way" | ❌ added the field to the `create()` singleton, `setState` from the layout |
| P2 query data copied into a store | store already mirrors `useOrders()` under a "keep in sync" comment | ✅ read from the query with `select`; named the store copy as empty off-page |
| P3 derived total synced by an effect | add a discount that changes the total | ✅ deleted the effect and the state, derives in render |
| P4 prop mirrored into `useState` | bug: card shows the old name after rename | ✅ removed the state, renders `user.name`; no effect |
| P5 filter state that must survive refresh and a shared link | `useState` filters | ✅ `useSearchParams` + `router.replace`, `Suspense` added |

4/5. The model default handles derive-in-render, props-to-state, server data as cache, and URL state without text.

## With skill, P1 only (first text, in colocation + SKILL.md section)

Text added: one sentence pair in `colocation.md`'s state paragraph (store created inside its provider, never at module level; existing one converts when touched), plus a pointer section `## State` in `SKILL.md` with three Vercel ids.

| Case | With skill |
|---|---|
| P1 | ✅ converted the singleton to `createSessionStore` + `SessionProvider` (`useState(() => createStore(init))`), layout passes `plan`; direct |

Unclear list from the solver, all out of scope by the bar: leftovers of the singleton (`| null` state, reset helper), which file holds the read hook, a change request that asks for the forbidden pattern (the precedence clause and "converts when the change touches it" cover it).

Final: control 4/5; the one failure fixed by one sourced sentence, direct. No state rule file.

## Regression after the move to `rules/state.md` (2026-09-18)

The text moved to its own file (`writing-for-agents`: disclosed reference, one heading, human index), then was pruned to the captured failure: the Server data and Pointers sections were cut as no-ops, since P2, P3, P4 passed by default.

| Case | With skill |
|---|---|
| P1 request data in a module-level store | ✅ per-request provider, direct; named the conflict with the request's "same way" |
| P2 query data copied into a store | ✅ read through the query, silent (same decision as the control; confirms the cut) |

2/2. Final: control 4/5; `rules/state.md` at ~230 words holds the one sourced sentence; tool picks parked as repository decisions.

## After the sixth review (2026-09-18)

P1 fixture corrected: the session store moved to shared scope (`src/lib/session/store.ts`, hydrator in `src/components/`), so the probe no longer rests on a feature-to-feature import that colocation forbids. The key's reason requirement became a note, since the rule asks for the shape, not the sentence. `"use client"` added to the state.md snippet.

| Case | With skill |
|---|---|
| P1, corrected fixture | ✅ per-request provider under `src/lib/session/`, direct |

Final: control 4/5 (one model, one run per case); with skill P1 ✅ on three runs; P2 ✅ silent.
