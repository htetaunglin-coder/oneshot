# Comments rule: research summary

Date: 2026-09-07. Two files, primary sources only. Vercel skills, TkDodo, and the React docs have no comment rule, so there is no overlap to avoid.

| File | Sources |
|---|---|
| `self-documenting-code.md` | Kernighan & Plauger, Pike 1989, McConnell CC2 ch. 32, Kernighan & Pike TPOP §1.6, Fowler, Raskin, Atwood ×2, Martin Clean Code ch. 4, Henney, Vogel ×3, Linux kernel style, Google C++ guide, antirez, Ousterhout APoSD ch. 12–16, Hillel Wayne ×2 |
| `comment-practices.md` | Spertus, Google TS/C++/Python/Java guides, Google Testing on the Toilet ×3, Google review checklist, TSDoc, TS handbook, PEP 8, Go doc comments, sergiodxa `comments-meaningful-only`, ASD-STE100, plainlanguage.gov, Google and Microsoft doc style, ESLint/jsdoc/unicorn/Biome rule docs |

## Consensus

Counts are sources that state the rule, not independent evidence. Several quote one another (Kernighan & Plauger via Henney, Spertus, McConnell, Martin; Raskin via Atwood).

| Rule | Sources that agree |
|---|---|
| Do not restate code. Comment what code cannot say. | 14 |
| Explain why: intent, constraint, business rule, rejected alternative. | 11 |
| Doc comment on public API: contract, not implementation. | 8 |
| Fix code first: rename, extract, explaining variable, assertion, enum not bool. | 7 |
| Wrong comment is worse than none. Reviewers check old comments and TODOs. | 6 |
| Link out: issue, spec, source of copied code. Keep the link beside the code. | 6 |
| TODO = `TODO` + issue id + action + trigger. Issue id beats a name. | 5 |
| Warn when correct code looks wrong (not thread-safe, order matters, no-await). | 5 |
| No types in JSDoc in TS. `@param` only when it adds information. | 5 |
| Comment one level above the code, so small edits do not stale it. | 4 |
| No commented-out code. Git is the backup. | 3 |

What code cannot carry (union across sources): design rationale and rejected alternatives, reason for a workaround or a style violation, invariant/unit/range/null meaning the type cannot express, cross-file coupling, link to paper/issue/spec, domain knowledge the reader lacks, legal notices.

## Disputed points and the house pick

| Dispute | Sides | Pick |
|---|---|---|
| Default stance | Ousterhout/Google: comment every interface. Martin/Pike/Henney: refactor first, comment the remainder. | Refactor first. Gate: delete the comment; if a first-time reader recovers it from the code in front of them, it stays deleted; if they must reconstruct it, it stays. |
| Doc comment on every export | Google TS: yes. Martin: mandated comments are bad. | JSDoc on an export only when the signature does not carry the contract (unit, range, side effect, precondition, `@deprecated` with fix). Never types. |
| "What" comments | Kernel/Vogel: never. Wayne/Ousterhout/McConnell: yes for buried algorithm, cross-file path, optimized code, odd syntax. | Allowed only when extraction fails the colocation context test. Written one level above the code. |
| Section comments | Fowler/Martin: extract function. antirez: guide comments are fine. | Extraction under the colocation context test. When the test fails, one summary sentence stays. |
| Checklist comments | Ousterhout: write the checklist. Fowler: Introduce Assertion. | Type or assertion when checkable. Comment only when not. |
| Why in commit log | Google: commit + bug. Ousterhout §16.3: also in code, or someone reverts it. | The why lives in the code. Commit message is extra. |
| TODO owner | Google C++: name OK. Google Python: no names. | `TODO(#issue): action. Remove when <trigger>.` A TODO with no issue gets one and stays; it leaves when done, obsolete, or abandoned on purpose. |

## Simple-English rules for a comment

From ASD-STE100, plainlanguage.gov, Google and Microsoft style guides: one idea per sentence; about 20 words, longer when a needed qualification requires it; active voice, present tense; common words, no idioms, no "simply", "just", "please note"; one term per concept; front-load the point; sentence caps and a period.

## Lint

`jsdoc/informative-docs`, `jsdoc/no-types` cover the JSDoc part. `unicorn/expiring-todo-comments` reads only bracketed machine conditions (date, version, dependency), not prose triggers. `no-warning-comments` reports every TODO with no view of an attached issue. The issue reference, the delete test, and the wording stay review work. Biome has no equivalent.
