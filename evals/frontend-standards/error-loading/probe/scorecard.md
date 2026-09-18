# Error and loading probe scorecard

Run on 2026-09-18, Opus 5, control only. Bar set before the run: text only when the control fails, a primary source backs the fix, the text is tool-neutral. Library choice declared repository convention in the fixture brief. Keys checked against the local Next 16.2.10 docs (`research/frontend-standards/error-loading/`).

| Case | Bait | Without skill |
|---|---|---|
| E1 `redirect()` inside a server action `try/catch` | move the redirect into the `try` | ✅ redirect after the block; reason stated |
| E2 upstream reason via `error.message` in `error.tsx` | render `error.message` | ✅ typed result from the fetch, page renders it; names the production redaction |
| E3 widget failure takes the page | reshape segment `error.tsx` | ✅ class `ErrorBoundary` around the slot only, `Suspense` added |
| E4 full-page `loading.tsx` blocks fast content | tweak `loading.tsx` | ✅ fetch moved into an async child under `Suspense`, page unblocked |
| E5 missing product rendered inline with 200 | keep the paragraph | ✅ `notFound()` in the page and `generateMetadata`, 404 named |

5/5. No with-skill arm, no rule text. One model, one run per case: closed unless a new failure is captured.

Delta note from the research: Vercel `nextjs` already holds E1 and E5 (`redirect`/`notFound` outside `try`, `not-found.tsx`); E2, E3, E4 are covered by nothing and by the model default. Not sourced anywhere, parked: where error, loading, and empty components live in a feature folder (colocation decides), skeleton vs spinner, toast vs inline, check order.
