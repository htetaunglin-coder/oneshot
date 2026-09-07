---
title: Comments, what the code cannot say
impact: MEDIUM
tags: comments, jsdoc, todo, naming, self-documenting
---

# Comments: what the code cannot say

Code carries the what and the how. A comment carries what the code cannot say: the why, the why-not, the warning, and the part of a contract a type cannot express.

## The gate

A comment is the residue after the code has been made to say it. Before a comment is written, the code takes the information where one of these fits:

- A name: `isEligibleForFullBenefits()` replaces `// check if eligible for full benefits`.
- An explaining variable: `const isStale = updatedAt < cutoff` replaces a comment over a condition.
- A union or an options object: `mode: "instant" | "smooth"` replaces `/* smooth= */ false`.
- An assertion or a narrower type: `invariant(items.length > 0)` replaces `// items is never empty here`.
- An extraction, when the block passes the colocation context test (see [colocation](colocation.md)).

A local rename or an explaining variable is inside any task that touches the function. A rename of an exported name or an assertion changes more than the comment would: the rename touches every caller, and the assertion turns wrong output into a crash. When that change is outside the task, the comment is the stop, and the rename or the assertion is named in the reply.

What is left passes one test: delete the comment. When a first-time reader recovers everything it said from the code in front of them, it stays deleted. When they must reconstruct it, by tracing other files, by re-measuring a trade-off, or by guessing at a reason, the comment stays. Henney's line: comment what the code *cannot* say, not what it *does not* say.

A comment describes the code as it is, never the edit that produced it. Git holds the history, so commented-out code is deleted and "changed X to Y" lines are deleted.

## What the code cannot say

| Kind | Carries | Example |
|---|---|---|
| Why | The reason behind a decision: a business rule, a constraint, an intent. | `// Orders under $250 skip the signature step. Policy FIN-12.` |
| Why not | The alternative that was rejected, and the trade-off measured. | `// Linear scan. The list holds at most 30 rows, so a Map costs more than it saves.` |
| Warning | Code that looks wrong and is deliberate. Protects it from a well-meant fix. | `// No await. Analytics is fire-and-forget; a failure must not block navigation.` |
| Contract | A unit, a range, an inclusive or exclusive end, the meaning of `null` or `-1`, a side effect, a precondition. Stated after an assertion or a narrower type has been tried. | `/** Amount in cents. Callers convert for display. */` |
| Coupling | A change here needs a change elsewhere, when a type or a test cannot enforce it. | `// Keep in sync with the STATUS enum in the Prisma schema.` |
| Reference | The issue, incident, spec, RFC, or source of copied code. The link sits beside the code. | `// Workaround for Safari smooth scroll in iframes. https://bugs.webkit.org/…` |

The why lives in the code, beside the line it explains. A commit message repeats it; it never replaces it, because the next reader sees the file, not the log. When the task asks for the reason to live only in the PR, one line still goes beside the code, and the reply says so.

## Level

A comment sits at a different level from the code it describes. Above it, the comment states the goal or the reason: `// Compute once; this runs on every keystroke.` sits above `useMemo`. Below it, the comment adds precision the code lacks: a unit, an inclusive or exclusive end, the meaning of a sentinel. There the exact identifier is repeated when it removes doubt. A comment at the same level repeats the code: `// Memoize the result` over `useMemo` fails the gate. Dividers, banners, and step labels (`// ---- handlers ----`, `// Step 2: build payload`) are the same level as the code and leave; the function name or the block order carries the structure.

A block that needs a section comment is a candidate for the colocation context test. When extraction passes the test, the function name replaces the comment. When it fails, the block stays and one sentence above it states what the block achieves. This is the case for the essential algorithm buried under validation and logging, a call path that crosses files, or code shaped by a measured optimization.

## Doc comments

`/** */` is for the reader at the use site, who sees it on hover. `//` is for the reader of this line. A module-private constant read elsewhere in the file takes `/** */` when the fact is needed at the use site. Multi-line implementation notes use several `//` lines. Inside JSX, a comment is `{/* */}`.

A JSDoc block goes on an export when the signature leaves the caller a question: a unit, a range, a side effect, a precondition, what happens on failure, what a hook returns where the caller cannot see (the server render, before hydration), or `@deprecated` with the replacement named. The name and the types answer the rest, so an export whose signature answers every question has no block.

A function's summary is one verb phrase in the third person: `/** Formats cents as USD for display. */`. A property or a type is summarized as a noun phrase: `/** Amount in cents. */`. Types never appear in a JSDoc tag; TypeScript owns them. A `@param` or `@returns` line exists only when it adds a fact the type lacks. The fields of a type, props included, are documented on the field, where hover shows them, and only the fields that leave a question; a field doc carries a why when the field's placement needs one. A component takes no block that lists its props.

```tsx
// Incorrect: restates the signature
/**
 * Formats a price.
 * @param {number} amount - The amount
 * @returns {string} The formatted price
 */
export function formatPrice(amount: number): string

// Correct: carries what the type cannot
/** Formats cents as USD. Rounds half up, so 1005 → "$10.05" and 1004.5 → "$10.05". */
export function formatPrice(cents: number): string
```

## TODO

A `TODO` names the work and the trigger, and the issue when the repository tracks issues: `// TODO(#412): Drop the legacy parser. Remove when the v1 API is off.` The trigger is the condition that ends the work or the consequence that makes it due: `Remove when the v1 API is off`, or `take: 50 cuts off orgs above 50 members`. A TODO with the issue missing gets one when an issue can be filed; until then it carries the work and the trigger, and it stays, because it is evidence of unfinished work. A TODO leaves when the work is done, obsolete, or abandoned on purpose. `FIXME` and `HACK` follow the same form.

## Wording

Comments are read more often than written, and by people whose first language may differ, so they use simple English:

- One idea per sentence. About 20 words; a qualification the reader needs survives even when it makes the sentence longer.
- Active voice, present tense: `The API returns cents.`, not `Cents will be returned by the API.`
- Common words. `use`, `start`, `because`. One term per concept across the file.
- The first words carry the point: `Retry twice because the API rate-limits bursts.`
- A complete sentence with a capital first letter and a period. A fragment is allowed at the end of a line, or as a field doc, when it stands alone without doubt: `const limit = 50; // Per page, API maximum.`, `/** Start of the window, inclusive. */`
- No filler: `note that`, `simply`, `just`, `basically`, `in order to` are cut.

## Maintenance

A change to a line re-reads the comments above it and beside it. A comment the change makes false is edited in the same change; a comment the change makes unnecessary is deleted in the same change. A comment elsewhere in the file that fails the gate and that the change does not touch is left and named in the reply, the same focused-change rule as the colocation file size limit. A wrong comment costs more than none.

## Tooling

Linters hold parts of this rule, and only parts. `jsdoc/informative-docs` flags a doc that restates the name. `jsdoc/no-types` flags a type in a tag. `unicorn/expiring-todo-comments` reads a machine-checkable condition in brackets, `TODO [2027-01-01]` or `TODO [-legacy-parser]`, and fails when it is met; it cannot read a prose trigger. `no-warning-comments` reports every TODO, FIXME, or XXX with no view of whether an issue is attached. The issue reference, the delete test, and the wording are review work. Biome has no equivalent rule. The rule text carries no comment-style convention beyond the `//` and `/** */` split.

## Sources

Kevlin Henney: Comment Only What the Code Cannot Say. John Ousterhout: A Philosophy of Software Design, ch. 12–16. Steve McConnell: Code Complete, ch. 32. Martin Fowler: Refactoring, the Comments smell. Robert C. Martin: Clean Code, ch. 4. Hillel Wayne: Comment the Why and the What; Why Not Comments. antirez: Writing system software, code comments. Google: TypeScript Style Guide, Comments and documentation; Testing on the Toilet, To Comment or Not to Comment; Less Is More. Ellen Spertus: Best practices for writing code comments. sergiodxa: comments-meaningful-only. ASD-STE100 and plainlanguage.gov for wording.
