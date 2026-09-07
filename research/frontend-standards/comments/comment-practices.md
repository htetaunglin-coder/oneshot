# Research: code comment best practices (primary sources only)

Date of research: 2026-09-07. Sources read directly (author blog, official style guide, official docs, repo files), with two exceptions marked below: the Ousterhout quotes in §4 come from Goodreads, and the ASD-STE100 rules in §6 come from Wikipedia's summary of the spec; the full-text book reading for Ousterhout is in `self-documenting-code.md`. Rejected: Medium listicles, dev.to summaries, content farms.

Source counts in the consensus section count sources that state a rule, not independent evidence; several of them quote one another (Kernighan & Plauger via Henney, Spertus, McConnell, and Martin; Raskin via Atwood).

Not found / not verified (do not cite):
- Nelson Elhage: no post on code comments found on blog.nelhage.com. Closest is "Systems that defy detailed understanding" (mentions `# this can never happen` comments; suggests logging instead of trusting the comment). Not a comments post.
- TkDodo: no post on code comments found on tkdodo.eu. "Why I don't like comments" does not exist as far as search shows.
- Vercel `vercel-labs/agent-skills` `react-best-practices`: no rule on comments. Only hits for "comment" are `fetchComments()` in the async-parallel example.
- Google TypeScript Style Guide: has NO TODO section. TODO format lives in the C++ and Python guides.
- React docs: no guidance on comments beyond JSX `{/* */}` syntax.

---

## 1. Ellen Spertus, "Best practices for writing code comments"

- URL: https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/
- Author: Ellen Spertus. Date: 2021-12-23. Stack Overflow blog (invited author post).

The 9 rules:
1. "Comments should not duplicate the code." Example of a bad one: `i = i + 1; // Add one to i`. Such comments "add visual clutter" and "take time to write and read".
2. "Good comments do not excuse unclear code." Quotes Kernighan & Plauger: "Don't comment bad code — rewrite it." Fix the names, not the comment.
3. "If you can't write a clear comment, there may be a problem with the code." Cites the Unix v6 comment "You are not expected to understand this" — the code was later rewritten.
4. "Comments should dispel confusion, not cause it." If a comment puzzles the reader, delete it.
5. "Explain unidiomatic code in comments." Example: a comparison to a nullable Boolean that looks redundant; the comment stops a future editor from "simplifying" it.
6. "Provide links to the original source of copied code." Link the Stack Overflow answer or source so readers can see context, license, and later edits.
7. "Include links to external references where they will be most helpful." E.g. link the RFC next to the code that implements it.
8. "Add comments when fixing bugs." Reference the issue tracker; explain why the odd-looking code is needed.
9. "Use comments to mark incomplete implementations." Use TODO with an issue reference so debt is visible and trackable.

---

## 2a. Google TypeScript Style Guide — "Comments and documentation"

- URL: https://google.github.io/styleguide/tsguide.html#comments-documentation
- Author: Google. Undated (living document).

- Two kinds: "Use `/** JSDoc */` comments for documentation, i.e. comments a user of the code should read. Use `//` line comments for implementation comments, i.e. comments that only concern the implementation of the code itself."
- "JSDoc comments are understood by tools (such as editors and documentation generators), while ordinary comments are only for other humans."
- Multi-line implementation comments "must use multiple single-line comments (`//`-style), not block comment style (`/* */`)." "Comments are not enclosed in boxes drawn with asterisks or other characters."
- "Document all top-level exports of modules." "Avoid merely restating the property or parameter name. You should also document all properties and methods (exported/public or not) whose purpose is not immediately obvious from their name, as judged by your reviewer."
- Method descriptions "begin with a verb phrase ... written in the third person, as if there is an implied `This method ...` before it."
- "Method, parameter, and return descriptions may be omitted if they are obvious from the rest of the method's JSDoc or from the method name and type signature."
- "JSDoc type annotations are redundant in TypeScript source code. Do not declare types in `@param` or `@return` blocks, do not write `@implements`, `@enum`, `@private`, `@override` etc. on code that uses the ... keywords."
- "Make comments that actually add information." "Avoid comments that just restate the parameter name and type, e.g. `/** @param fooBarService The Bar service for the Foo application. */`" "Because of this rule, `@param` and `@return` lines are only required when they add information, and may otherwise be omitted."
- Good example kept: `@param amountLitres The amount to brew. Must fit the pot size!`
- Call-site "parameter name" comments: `someFunction(obviousParam, /* shouldRender= */ true, /* name= */ 'hello');` — but first "consider refactoring the method to instead accept an interface".
- JSDoc is Markdown; use `-` lists, plain indented lines collapse.
- `@deprecated` "must include simple, clear directions for people to fix their call sites."
- Put JSDoc before decorators.

## 2b. Google C++ Style Guide — "Comments"

- URL: https://google.github.io/styleguide/cppguide.html#Comments
- Author: Google. Undated.

- "Comments are absolutely vital to keeping our code readable ... But remember: while comments are very important, the best code is self-documenting."
- "When writing your comments, write for your audience: the next contributor who will need to understand your code. Be generous — the next one may be you!"
- Function declaration comments "describe what the function does and how to use it"; "may be omitted only if the function is simple and obvious". Written "with an implied subject of This function ... 'Opens the file', rather than 'Open the file'."
- What to mention at a declaration: inputs/outputs; whether object keeps reference/pointer args; null-ability of pointer args; what happens to output-arg state; performance implications.
- Function definition comments: "describe any coding tricks you use, give an overview of the steps you go through, or explain why you chose to implement the function in the way you did rather than using a viable alternative."
- Class data members: "If there are any invariants (special values, relationships between members, lifetime requirements) not clearly expressed by the type and name, they must be commented." Example: `// -1 means that we don't yet know how many entries the table has.`
- "Do not state the obvious. In particular, don't literally describe what code does, unless the behavior is nonobvious to a reader who understands C++ well. Instead, provide higher-level comments that describe why the code does what it does, or make the code self-describing."
- Bad: `// Find the element in the vector.` Better: `// Process "element" unless it was already processed.` Best: `if (!IsAlreadyProcessed(element))` with no comment.
- Argument comments are a "last resort"; prefer named constants, enums instead of bool, options structs, named variables.
- "Comments should be as readable as narrative text, with proper capitalization and punctuation. In many cases, complete sentences are more readable than sentence fragments."
- TODO: "Use TODO comments for code that is temporary, a short-term solution, or good-enough but not perfect." "TODOs should include the string TODO in all caps, followed by the bug ID, name, e-mail address, or other identifier of the person or issue with the best context". Preferred forms, in order:
  - `// TODO: bug 12345678 - Remove this after the 2047q4 compatibility window expires.`
  - `// TODO: example.com/my-design-doc - Manually fix up this code the next time it's touched.`
  - `// TODO(bug 12345678): Update this list after the Foo service is turned down.`
  - `// TODO(John): Use a "\*" here for concatenation operator.`
- "If your TODO is of the form 'At a future date do something' make sure that you either include a very specific date ('Fix by November 2005') or a very specific event".

## 2c. Google Python Style Guide — 3.8 Comments and Docstrings, 3.12 TODO

- URL: https://google.github.io/styleguide/pyguide.html
- "Never describe the code. Assume the person reading the code knows Python better than you do."
- Block/inline comments: only for "tricky parts of the code".
- Docstrings mandatory for public API, nontrivial size, or "non-obvious logic". Summary line max 80 chars, ends with punctuation.
- TODO format: `TODO: <resource-link> - <explanation>`; "Avoid person-specific references like `@username`" (bug link preferred; a person may leave).

## 2d. Google Java Style Guide — 7 Javadoc

- URL: https://google.github.io/styleguide/javaguide.html#s7-javadoc
- Javadoc required for every visible (public/protected) class and member.
- Exception: "Javadoc is optional for 'simple, obvious' members ... such as a `getFoo()` method, if there really and truly is nothing else worthwhile to say but 'the foo'." But "it is not appropriate to cite this exception to justify omitting relevant information that a typical reader might need to know."
- Summary fragment: "a noun phrase or verb phrase, not a complete sentence"; must not "begin with `A {@code Foo} is a...`, or `This method returns...`". "A common mistake is to write simple Javadoc in the form `/** @return the customer ID */`" — rewrite it.

## 2e. Google Testing Blog (Testing on the Toilet) — "Code Health: To Comment or Not to Comment?"

- URL: https://testing.googleblog.com/2017/07/code-health-to-comment-or-not-to-comment.html
- Authors: Dori Reuveni and Kevin Bourrillion. Date: 2017-07-17.

- "Sometimes the need for a comment can be a sign that the code should be refactored."
- "Use a comment when it is infeasible to make your code self-explanatory." Before commenting, try: introduce an explaining variable; extract a method; use a more descriptive identifier (`int width; // Width in pixels.` -> `int widthInPixels`); "Add a check in case your code has assumptions" (`// Safe since height is always > 0.` -> `checkArgument(height > 0);`).
- Cases where a comment helps:
  - "Reveal your intent: explain why the code does something (as opposed to what it does)." `// Compute once because it's expensive.`
  - "Protect a well-meaning future editor from mistakenly 'fixing' your code." `// Create a new Foo instance because Foo is not thread-safe.`
  - "Clarification: a question that came up during code review or that readers of the code might have." `// Note that order matters because...`
  - "Explain your rationale for what looks like a bad software engineering practice." `@SuppressWarnings("unchecked") // The cast is safe because...`
- "avoid comments that just repeat what the code does. These are just noise": `// Get all users.` / `// Check if the name is empty.`

## 2f. Google Testing Blog — "Less Is More: Principles for Simple Comments"

- URL: https://testing.googleblog.com/2024/08/less-is-more-principles-for-simple.html
- Author: David Bendory. Date: 2024-08-21. About source-code comments.

- "Adopt the mindset of someone unfamiliar with the project." "separate the process of writing your comments from reviewing them; proofreading your comments without code context in mind helps ensure they are clear and concise".
- "Use self-contained comments to clearly convey intent without relying on the surrounding code for context. If you need to read the code to understand the comment, you've got it backwards!"
- "Include only essential information in the comments and leverage external references to reduce cognitive load on the reader. For comments suggesting improvements, links to relevant bugs or docs keep comments concise". Example: `// TODO: Consider various factors to present the best transit option. See issuetracker.fake/bus-vs-subway`
- "Note that linked docs may be inaccessible, so use judgment in deciding how much context to include directly".
- "Avoid extensive implementation details in function-level comments. When implementations change, such details often result in outdated comments. Instead, describe the public API contract, focusing on what the function does."

## 2g. Google Testing Blog — "Code Health: Providing Context with Commit Messages and Bug Reports"

- URL: https://testing.googleblog.com/2017/09/code-health-providing-context-with.html
- Author: Chris Lewis. Date: 2017-09.
- "Code comments provide context, but comments alone sometimes can't provide enough." Two other places for the why: commit messages (first line stands alone) and bug reports ("Most commits should reference a bug report").

## 2h. Google Engineering Practices — Code review: "Comments"

- URL: https://google.github.io/eng-practices/review/reviewer/looking-for.html#comments
- "Did the developer write clear comments in understandable English? Are all of the comments actually necessary? Usually comments are useful when they explain why some code exists, and should not be explaining what some code is doing. If the code isn't clear enough to explain itself, then the code should be made simpler. There are some exceptions (regular expressions and complex algorithms often benefit greatly from comments that explain what they're doing, for example) but mostly comments are for information that the code itself can't possibly contain, like the reasoning behind a decision."
- "It can also be helpful to look at comments that were there before this CL. Maybe there is a TODO that can be removed now, a comment advising against this change being made, etc."
- "comments are different from documentation of classes, modules, or functions, which should instead express the purpose of a piece of code, how it should be used, and how it behaves when used."

---

## 3a. TSDoc (official)

- URL: https://tsdoc.org/
- Author: Microsoft / TSDoc project. Living doc.
- "TSDoc is a proposal to standardize the doc comments used in TypeScript code, so that different tools can extract content without getting confused by each other's markup."
- Motivation: "The JSDoc grammar is not rigorously specified, but rather inferred from the behavior of a particular implementation." JSDoc's type-annotation focus is unnecessary in TS because TS has native types.
- Core tags: `@param`, `@returns`, `@remarks`, `@example`, `@internal`, `@deprecated`. Consumers: TypeDoc, API Extractor, ESLint plugins.

## 3b. TypeScript Handbook — "JSDoc Reference"

- URL: https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html
- "Only documentation tags are supported in TypeScript files. The rest of the tags are only supported in JavaScript files." Documentation tags: `@deprecated`, `@see`, `@link`.
- Implication: in `.ts`, JSDoc `@type`/`@param {T}` do nothing for the checker; types belong in TS syntax. JSDoc in `.ts` is for humans and editor hover.

## 3c. When JSDoc adds value in TS (synthesis of 2a, 3a, 3b, 7, 8)

- Yes: exported/public API (Google TS: "Document all top-level exports"); units, ranges, invariants a type cannot express (`Must fit the pot size!`); `@deprecated` with migration path; `@example`; anything shown on hover to a caller.
- No: restating name + type (`@param fooBarService The Bar service for the Foo application.`); types in `@param {string}` (Google TS; eslint-plugin-jsdoc `no-types`); `@private` when `private` keyword exists.

---

## 4. Conventions: TODO/FIXME, "why not what", commented-out code, comment rot

### PEP 8 — Comments
- URL: https://peps.python.org/pep-0008/#comments
- "Comments that contradict the code are worse than no comments. Always make a priority of keeping the comments up-to-date when the code changes!"
- "Comments should be complete sentences. The first word should be capitalized, unless it is an identifier".
- "Use inline comments sparingly." "Inline comments are unnecessary and in fact distracting if they state the obvious."

### Go — "Go Doc Comments" and "Go Code Review Comments"
- URLs: https://go.dev/doc/comment ; https://go.dev/wiki/CodeReviewComments
- "Doc comments typically use complete sentences." Start with the name: "Request represents a request to run a command." "Encode writes the JSON encoding of req to w."
- "A function's doc comment should explain what the function returns or, for functions called for side effects, what it does."
- "Doc comments should not explain internal details such as the algorithm used in the current implementation. Those are best left to comments inside the function body."
- Boolean funcs: use "reports whether"; "The phrase 'or not' is unnecessary."
- Types: document zero-value meaning and concurrency guarantees if stronger than default.
- `Deprecated:` paragraph is machine-readable.

### Kevlin Henney — "Comment Only What the Code Cannot Say"
- URL: https://accu.org/journals/overload/28/157/henney_2796/ (Overload 157, June 2020; revision of 97 Things ch. 17, 2010)
- "A comment is of zero (or negative) value if it is wrong." (quoting Kernighan & Plauger)
- Core rule: "Comment what the code cannot say, not simply what it does not say."
- Restating code in prose violates DRY: "repeated expression of knowledge".
- Commented-out code: version control already keeps history; dead code "becomes obsolete rapidly".
- Noise trains readers to ignore all comments.
- "Don't comment bad code – rewrite it." Improve names, extract functions, before commenting.

### antirez — "Writing system software: code comments"
- URL: http://antirez.com/news/124
- Author: Salvatore Sanfilippo. Date: ~2018-10 (page shows "2892 days ago" on 2026-09-07).
- "Many comments don't explain what the code is doing. They explain what you can't understand just from what the code does."
- Taxonomy: function comments ("prevent the reader from reading code in the first place"); design comments (algorithm rationale, alternatives); why comments ("explain the reason why the code is doing something, even if what the code is doing is crystal clear"); teacher comments (domain knowledge); checklist comments ("a set of actions to do when something is modified"); guide comments (rhythm/division); trivial comments (bad: "Increment the length of our array"); debt comments (TODO/FIXME); backup comments (bad).
- "Source code is not for making backups. If you want to save an older version of a function or code part, your work is not finished and cannot be committed."
- "Comments require always to have some design process ongoing, and to understand the code you are writing in a deeper sense."

### Jeff Atwood — "Code Tells You How, Comments Tell You Why"
- URL: https://blog.codinghorror.com/code-tells-you-how-comments-tell-you-why/
- Date: 2006-12-18.
- "Code can only tell you how the program works; comments can tell you why it works."
- "Code can't explain why the program is being written, and the rationale for choosing this or that method."
- Quotes Jef Raskin's example: "A binary search turned out to be slower than the Boyer-Moore algorithm for the data sets of interest..." — a rejected-alternative comment.

### John Ousterhout — A Philosophy of Software Design (ch. 12-16), quotes
- Source: book; quotes here via Goodreads (secondary): https://www.goodreads.com/work/quotes/61938796-a-philosophy-of-software-design . Primary reading of ch. 12–16 is in `self-documenting-code.md` §15.
- "Comments should describe things that are not obvious from the code."
- "The overall idea behind comments is to capture information that was in the mind of the designer but couldn't be represented in the code."
- "Comments augment the code by providing information at a different level of detail." (higher-level than code = fewer rot edits)
- "Some people believe that if code is written well, it is so obvious that no comments are needed. This is a delicious myth... it's simply not true."
- "Writing the comments first makes documentation part of the design process."
- Red flags: "Comment Repeats Code"; "Implementation Documentation Contaminates Interface".
- "put yourself in the mindset of the reader and ask yourself what are the key things he or she will need to know."

### Conventions summary (from 2b, 2c, 2f, 7, 8)
- TODO format: `TODO` in caps + owner or (better) bug/issue id + what to do + when/trigger. Google C++ prefers bug link over person; Google Python: "Avoid person-specific references like @username". sergiodxa: `// TODO(#1234): Remove after migration completes in Q2 2024`.
- Commented-out code: forbidden (Henney, antirez, Spertus implicit via VCS).
- Comment rot: keep comment adjacent to code (PEP 8, Google eng-practices review check); comment at a higher level than code (Ousterhout, Bendory) so small edits do not stale it; reviewers must check old comments and TODOs (Google eng-practices).
- Comments are reviewed like code: Google reviewer checklist asks "clear comments in understandable English" and "actually necessary".

---

## 5. Hillel Wayne (Computer Things newsletter)

### "Comment the Why *and* the What"
- URL: https://buttondown.com/hillelwayne/archive/comment-the-why-and-the-what/
- Date: 2021-06-14.
- Argues against the slogan "comment the why, not the what". "What" comments earn their place when:
  1. Context: the code calls another service or file the reader cannot see — "include a brief description of the other service"; link to prod logs/dashboards.
  2. Essential algorithm buried under validation/logging: "stick the 'idealized' function in a comment".
  3. Optimized code: "performant code needs to be legible *and* performant".
  4. Skill differentials on the team: unfamiliar library, domain technique, novel abstraction.
  5. Language weirdness: `b=( "${a[@]}" ) # copy array`.
  Plus: ASCII diagrams; direct messages to the next developer (warnings, "negative information", speculation).
- Against "put it in the issue tracker": "readers must already know the information exists to find it; comments sit visibly alongside code."
- Against "comments go stale": an imperfect comment still lets you verify faster than rebuilding the model from zero.

### "Why Not Comments"
- URL: https://buttondown.com/hillelwayne/archive/why-not-comments/
- Date: 2024-09-10.
- Comments are the only place for "Negative information, drawing attention to what's *not* there. The 'why nots' of the system."
- Example from his epub script: kept an inefficient 16-pass string replace because "there are only 25 math strings in the book so far and most are <5 characters. So it's still fast enough." The comment records that the trade-off was evaluated, not overlooked.
- Names cannot encode both "what the function does" and "what tradeoffs it makes".

---

## 6. Plain / simple English sources

### ASD-STE100 Simplified Technical English
- Official: https://www.asd-ste100.org/ — "a controlled natural language and an international standard to write technical documentation." Owned by ASD (Brussels). Spec is free on request; rule text below via Wikipedia's summary of the spec (secondary; verify against the spec before quoting) (https://en.wikipedia.org/wiki/Simplified_Technical_English).
- Sentence length: "no more than 20 words in instructions (procedures)", "25 words in descriptive texts".
- "Write one instruction per sentence."
- "Write only one topic per paragraph." "Do not write more than six sentences in each paragraph."
- "Use the active voice. In descriptive writing, one should use the passive voice only when the agent is unknown."
- Approved dictionary (~900 words): "one word, one part of speech, one meaning". Use approved words "only as the part of speech and meaning given in the dictionary".
- "Do not write multi-word nouns that have more than three words."
- Tenses allowed: infinitive, imperative, simple present, simple past, simple future, past participle as adjective. "Do not use auxiliary verbs to make complex verb constructions."
- "Do not omit parts of the sentence (e.g. verb, subject, article) to make the text shorter."

### US Federal Plain Language Guidelines (plainlanguage.gov, now digital.gov)
- URLs: https://digital.gov/guides/plain-language/writing ; https://digital.gov/guides/plain-language/principles/short-simple
- "Use the active voice" — "makes it clear who should do what".
- "Use the present tense" — "simplest and strongest form"; other tenses "only when necessary for accuracy".
- "Avoid hidden verbs" (nouns ending -ment, -tion, -sion); use "the strongest, most direct form of the verb possible".
- Short words, short sentences, short sections.
- "Consider whether you need every word." Cut modifiers ("very", "really", "actually"). Avoid doublets ("cease and desist" -> "stop").
- Consistency: one term per concept; do not swap synonyms.
- Use "must" for requirements (federal guideline "Use 'must' to indicate requirements").

### Google developer documentation style guide
- Voice & tone: https://developers.google.com/style/tone — "conversational, friendly, and respectful"; avoid "please note", "at this time", exclamation marks, slang, "simple/easy/quick", pop-culture, humor, figurative language.
- Global audience: https://developers.google.com/style/translation — "The shorter the sentence, the easier it is to translate." "Use active voice." Present tense. Avoid phrasal verbs, idioms ("ballpark figure"), "commence/utilize" -> "start/use". "If you use a particular term for a concept in one place, then use that exact same term elsewhere." "Address the reader directly (`you`)". Unambiguous dates.
- Active voice: https://developers.google.com/style/voice — "Use active voice ... instead of passive voice"; passive acceptable to emphasize the object ("The file is saved."), de-emphasize actor, or when actor is irrelevant ("The database was purged in January.").

### Microsoft Writing Style Guide — "Top 10 tips for style and voice"
- URL: https://learn.microsoft.com/en-us/style-guide/top-10-tips-style-voice (updated 2026-07-02)
- "Use bigger ideas, fewer words. ... Shorter is always better."
- "Write like you speak. Read your text aloud. Avoid jargon".
- "Get to the point fast. Lead with what's most important. Front-load keywords for scanning."
- "Be brief. ... Prune every excess word."
- "When in doubt, don't capitalize" (sentence-style caps).
- "Revise weak writing. Most of the time, start each statement with a verb. ... Avoid weak phrasing like there is, there are".
- Also: contractions OK; serial comma; one space after periods.

---

## 7. React / TypeScript ecosystem

### sergiodxa/agent-skills — `comments-meaningful-only` (full text)
- URL: https://github.com/sergiodxa/agent-skills/blob/main/skills/frontend-js-best-practices/rules/comments-meaningful-only.md (note: repo is `agent-skills`, not `skills`)
- Author: Sergio Xalambrí. Impact: MEDIUM.

Full text:

```
---
title: Meaningful Comments Only
impact: MEDIUM
impactDescription: code readability and maintenance
tags: javascript, comments, documentation, conventions
---

## Meaningful Comments Only

Don't write comments that repeat what the code says. Only comment when adding information the code cannot express.

### Why

1. **Noise reduction** - Meaningless comments make important ones harder to find
2. **Maintenance burden** - Comments that restate code get outdated
3. **Code should be self-documenting** - Good names and structure reduce need for comments
4. **Comments are for "why", not "what"** - Code shows what, comments explain why

### Bad: Meaningless Comments

// Bad: restates the code
// Set the user's name
let userName = user.name;

// Bad: obvious from the code
// Loop through the items
for (let item of items) {
  // Process the item
  processItem(item);
}

// Bad: describes the function name
// Calculate the total
function calculateTotal(items: Item[]) { ... }

// Bad: describes variable assignment
// Create an empty array
let results = [];

### Good: Meaningful Comments

#### Business Rules
// Transactions under $250 don't require written acknowledgment per policy
if (transaction.amount < 250) { return { requiresAcknowledgment: false }; }

#### Non-Obvious Behavior
// API returns amounts in cents, convert to dollars for display
let displayAmount = apiAmount / 100;

#### Edge Cases and Workarounds
// Safari doesn't support smooth scrolling in iframes, use instant instead
let behavior = isSafari && isInIframe ? "instant" : "smooth";

#### Performance Decisions
// Using Map for O(1) lookups instead of array.find() which is O(n)
// This matters because we check membership for every item in the list
let userById = new Map(users.map((u) => [u.id, u]));

#### External Dependencies
// Copied from lodash/debounce to avoid adding the full dependency
// https://github.com/lodash/lodash/blob/main/debounce.js
function debounce(fn: Function, wait: number) { ... }

#### Intentional Behavior
// Intentionally not awaiting - we want fire-and-forget analytics
analytics.track("page_view", { path });

#### TODO with Context
// TODO(#1234): Remove after migration completes in Q2 2024
let useLegacyApi = featureFlags.useLegacyApi;

### JSDoc for Public APIs

Use JSDoc for exported functions, especially utilities:

/**
 * Formats a number as USD currency
 * @param amount - The amount in dollars (not cents)
 * @param showDecimals - Whether to show cents (default: false)
 * @returns Formatted string like "$1,234" or "$1,234.56"
 */
export function formatCurrency(amount: number, showDecimals = false): string { ... }

### When to Comment

| Comment when...               | Example                                       |
| Business rule isn't obvious   | IRS requirements, legal constraints           |
| Working around a bug          | Browser quirks, API limitations               |
| Code is intentionally unusual | Performance optimization, deliberate no-await |
| External context needed       | Links to specs, ticket numbers                |
| Trade-off was made            | Why this approach over alternatives           |

### When NOT to Comment

| Don't comment...         | Why                              |
| What the code does       | Code already says it             |
| Variable assignments     | Name should be clear             |
| Loop iterations          | Standard patterns are understood |
| Function purpose         | Name should convey it            |
| Obvious type conversions | TypeScript shows types           |
```

### Vercel `react-best-practices`: no comments rule (verified by grep of AGENTS.md and rules/).
### TkDodo / React docs: nothing primary found on comment policy.

---

## 8. Lint rules that enforce comment policy

ESLint core:
- `no-warning-comments` — reports comments containing configured terms (default `todo`, `fixme`, `xxx`); options `terms`, `location` (`start`|`anywhere`), `decoration`. Use to block TODOs without tracking, or to fail CI on FIXME. https://eslint.org/docs/latest/rules/no-warning-comments
- `capitalized-comments` — requires (or forbids) a capital first letter; `ignorePattern`, `ignoreInlineComments`, `ignoreConsecutiveComments`; fixable; frozen. https://eslint.org/docs/latest/rules/capitalized-comments
- `multiline-comment-style` — `starred-block` | `bare-block` | `separate-lines`; deprecated in ESLint v9.3.0, moved to `@stylistic/eslint-plugin`. https://eslint.org/docs/latest/rules/multiline-comment-style
- `spaced-comment` — requires/forbids a space after `//` or `/*`; exceptions/markers options; deprecated (v8.53.0), moved to `@stylistic`. https://eslint.org/docs/latest/rules/spaced-comment

eslint-plugin-jsdoc:
- `jsdoc/require-jsdoc` — requires JSDoc on functions/classes; `publicOnly` (exports only), `require` (per node type), `contexts` with `minLineCount`, `exemptEmptyFunctions`. https://github.com/gajus/eslint-plugin-jsdoc/blob/main/docs/rules/require-jsdoc.md
- `jsdoc/informative-docs` — the opposite: flags docs that only restate the name (`/** The user id. */` on `userId`); options `aliases`, `uselessWords`, `excludedTags`. https://github.com/gajus/eslint-plugin-jsdoc/blob/main/docs/rules/informative-docs.md
- `jsdoc/no-types` — "reports types being used on `@param` or `@returns`"; "prevent the indication of types on tags where the type information would be redundant with TypeScript". https://github.com/gajus/eslint-plugin-jsdoc/blob/main/docs/rules/no-types.md

eslint-plugin-unicorn:
- `unicorn/expiring-todo-comments` — TODO/FIXME/XXX must carry a condition in brackets: date `[2200-12-25]`, package version `[>=1.0.0]`, engine `[engine:node@>=12]`, dependency added/removed `[+react]`/`[-lodash]`, dependency version `[lodash@>10]`, peer `[peer:eslint@>=9]`; fails when the condition is met. Options: `checkDates` (default false), `allowWarningComments` (default true; set false to require a condition on every TODO), `terms`, `ignore`. https://github.com/sindresorhus/eslint-plugin-unicorn/blob/main/docs/rules/expiring-todo-comments.md

Biome (verified against `crates/biome_js_analyze/src/lint`):
- `suspicious/noCommentText` — comments inside JSX children must be `{/* */}`, not bare `//` text. https://biomejs.dev/linter/rules/no-comment-text/
- `correctness/useSingleJsDocAsterisk` — JSDoc lines start with exactly one `*`. https://biomejs.dev/linter/rules/use-single-js-doc-asterisk/
- No Biome equivalent of `no-warning-comments`, `require-jsdoc`, `informative-docs`, or `expiring-todo-comments` as of this check.

---

## Consensus rules (ranked by number of agreeing sources)

1. Do not restate the code. Comment what the code cannot say. — Spertus (R1), Google TS, Google C++, Google Python, Google Java, ToTT 2017, Google eng-practices, Henney, antirez, Atwood, Ousterhout, sergiodxa, PEP 8, Bendory. (14)
2. Explain why: intent, rationale, constraints, business rules, rejected alternatives. — Spertus (R5, R8), ToTT 2017, Google C++, Google eng-practices, Henney, antirez, Atwood, Ousterhout, Hillel Wayne (both), sergiodxa. (11)
3. Fix the code before commenting: better name, extracted function, explaining variable, assertion, enum instead of bool. — Spertus (R2, R3), ToTT 2017, Google C++, Henney, antirez, sergiodxa. (7)
4. Document public API with doc comments (JSDoc/Javadoc/docstring/Go doc): purpose, contract, how to use, not implementation. — Google TS/C++/Python/Java, Go, Bendory, Ousterhout, sergiodxa. (8)
5. Link out: issue, RFC, spec, source of copied code. Keep the link next to the code. — Spertus (R6, R7, R8, R9), Bendory, Google C++ TODO, Google Python TODO, Hillel Wayne, sergiodxa. (6)
6. TODO = `TODO` + issue id (or owner) + action + trigger/date. — Spertus (R9), Google C++, Google Python, sergiodxa, unicorn rule. (5)
7. Protect against future "fixes": warn when code looks wrong but is deliberate (unidiomatic code, thread-safety, ordering, fire-and-forget). — Spertus (R5), ToTT 2017, Hillel Wayne, sergiodxa, antirez (checklist). (5)
8. Keep comments in sync; a wrong comment is worse than none; reviewers check comments and stale TODOs. — PEP 8, Henney (Kernighan), Google eng-practices, Bendory, Spertus (R4), Ousterhout. (6)
9. No commented-out code; VCS is the backup. — Henney, antirez, Google eng-practices (remove old TODOs). (3)
10. Comment at a higher level than the code so small edits do not stale it; keep implementation detail out of interface docs. — Ousterhout, Bendory, Go doc comments, Google C++ (decl vs def). (4)
11. Write comments as readable prose: complete sentences, capitalised, punctuated, correct grammar, "understandable English". — Google C++, PEP 8, Go, Google eng-practices. (4)
12. Do not repeat types in JSDoc in TS; `@param`/`@return` only when they add information. — Google TS, TS handbook, TSDoc, `jsdoc/no-types`, `jsdoc/informative-docs`. (5)
13. Verb-phrase, third-person summary: "Opens the file", "Encode writes ...", "HasPrefix reports whether ...". — Google C++, Google TS, Google Java, Go. (4)
14. Write for the reader who is unfamiliar; make the comment self-contained. — Google C++, Bendory, Ousterhout, Hillel Wayne. (4)

## Disputed points

- "Why not what": Hillel Wayne (2021, 2024) argues "what" comments are needed for context, buried algorithms, optimized code, unfamiliar libraries, odd syntax. Google eng-practices concedes exceptions (regex, complex algorithms). Strict form (Atwood, sergiodxa, ToTT) says "what" belongs in code.
- Self-documenting code: Ousterhout calls "no comments needed" a "delicious myth"; antirez agrees. Henney and ToTT lean toward refactor-first. Both camps agree on the no-restating rule; they differ on how much remains after refactoring.
- Comments vs issue tracker: Bendory says link out to keep comments short but warns links may be inaccessible; Hillel Wayne says readers cannot find tracker info unless the comment points to it, so keep key context inline.
- Doc comments on everything vs only when informative: Google Java requires Javadoc on every visible member (allows omission for `getFoo`); Google TS requires JSDoc for all exports but lets `@param` be omitted when it adds nothing; `jsdoc/require-jsdoc` and `jsdoc/informative-docs` pull in opposite directions and must be tuned together.
- TODO owner: Google C++ still lists `TODO(John)` as acceptable (lowest preference); Google Python says avoid `@username`, use a bug link.
- Complete sentences vs fragments: Google C++ / PEP 8 / Go want complete sentences; Google Java summary fragment must NOT be a complete sentence; Google C++ allows informal end-of-line comments.
- Comment style: Google TS forbids `/* */` for multi-line implementation comments; ESLint `multiline-comment-style` default is `starred-block`.
- Whether the Google TS guide has a TODO format: it does not; teams borrow the C++ or Python form.

## Simple-English rules for a comment (transferable subset from item 6)

1. One idea per sentence. One instruction per sentence. (STE; plain language)
2. Keep sentences short: under 20 words for instructions, under 25 for descriptions. (STE; Google global audience: "The shorter the sentence, the easier it is to translate.")
3. Use active voice. Say who does what: "The API returns cents", not "Cents are returned." Passive only when the actor is unknown or irrelevant. (STE; plainlanguage; Google voice)
4. Use present tense and the simplest verb form. No "will be", "has been" unless the time matters. (STE; plainlanguage)
5. Use common short words. "use" not "utilize", "start" not "commence". No idioms, no humor, no "please note", no "simply/easy". (Google tone/translation; plainlanguage; Microsoft)
6. One word, one meaning, one term per concept. Do not switch between "user", "account", "member" for the same thing. (STE; Google; plainlanguage)
7. Cut every word that does no work: "very", "really", "actually", "in order to", "there is". (plainlanguage; Microsoft "Prune every excess word.")
8. Lead with the point. Front-load the key word or the action: "Retry twice because the API rate-limits bursts." (Microsoft "Get to the point fast"; Bendory "self-contained")
9. Do not omit sentence parts to save space; a comment that is a fragment must still be unambiguous. (STE) For doc-comment summaries, start with a verb phrase: "Opens the file." (Google C++/TS)
10. Use "must" for requirements, not "should" or "needs to be". (plainlanguage)
11. No noun stacks longer than three words: "cache invalidation retry policy timer" -> "timer for the cache retry policy". (STE)
12. Sentence-style capitalisation; end with a period. (Microsoft; PEP 8; Google C++)
