# Magic numbers and magic literals — research summary

Scope: JS/TS/React. Primary sources only (see `sources.md` for URLs and quotes). Collected 2026-09-16.
Quotes from Fowler, Martin, McConnell come from print editions; web verification is partial (paywalled). Page refs in `sources.md`.

## Consensus

| Question | Fowler (Refactoring 2e) | Martin (Clean Code G25) | McConnell (Code Complete 2e §12.1) | Linters (ESLint / TS-ESLint / Biome) |
|---|---|---|---|---|
| Replace a literal whose meaning must be decoded | Yes (`9.81` -> `STANDARD_GRAVITY`) | Yes (`86400` -> `SECONDS_PER_DAY`) | Yes ("Avoid magic numbers") | Yes (rule exists in all three) |
| Replace a literal that is duplicated | Yes (main motivation) | Yes | Yes ("changes can be made more reliably") | ESLint definition: "numbers that occur multiple times ... without an explicit meaning" |
| Magic strings count too | Yes ("magic literal", not "number") | Yes: "any token that has a value that is not self-describing" (`7777`, `"John Doe"`) | Numbers only in §12.1 | Numbers only (all three rules) |
| Name must add meaning | Yes; `ONE` is pointless | Yes; `TWO` is "absurd" | Yes | n/a |
| Prefer function over constant for comparisons | Yes: `isMale(aValue)` over `aValue === MALE_GENDER` | Not stated | Not stated | n/a |
| Where the constant lives | Not specified | Not specified | "appropriately scoped"; consistent use | ESLint `enforceConst`: must be `const` |
| Rule is opt-in in tooling | – | – | – | None of the three rules is in a recommended config; ESLint rule is frozen |

## Exceptions all sources allow

- `0` and `1` for loop start, increment, decrement, empty check. McConnell: "the only literals that should occur in the body of a program are 0 and 1." Biome ignores 0, 1, 2, 10, 24, 60 and their negatives.
- `-1` as "not found" / sentinel: not named by any book; ESLint `ignore: [-1]` and Biome's negative-form ignore cover it in practice. Treat as house rule, not sourced.
- Literals obvious in their formula. Martin: `hourlyRate * 8`, `radius * Math.PI * 2`, `feetWalked/5280.0` "are simply better written as raw numbers"; `TWO` would be "absurd". Fowler: a name that only restates the value (`ONE`) adds nothing.
- Well-known constants via library, not literal nor own constant: `Math.PI` (Martin: leaving 3.14159... raw is "too great" a chance for error).
- Self-describing arithmetic (`24 * 60 * 60`): only Biome encodes it (ignores 24/60 inside expressions). No book states it. See Disputed.
- Array indexes (`data[0]`, `pair[1]`): ESLint `ignoreArrayIndexes`, Biome built-in. No book covers it.
- Default parameter values and class field initialisers: ESLint `ignoreDefaultValues`, `ignoreClassFieldInitialValues`; Biome built-in. Lint pragmatics, not sourced from books.
- Test assertions: Martin flags `assertEquals(7777, ...)` as magic. No author gives tests a blanket exemption.
- Tailwind arbitrary values (`w-[347px]`): Tailwind allows "once in a while ... to get things pixel-perfect"; reused values go in `@theme`. Pointer only.

## Disputed

### Constant naming case

| Source | UPPER_CASE for | Not UPPER_CASE for |
|---|---|---|
| Google TS style guide | "global constant values, including enum values"; module-level only; objects allowed (`UNIT_SUFFIXES = {...}`) to signal "must not be modified" | anything "instantiated more than once" — local `const` inside functions, static fields of nested classes -> `lowerCamelCase` |
| Airbnb 23.10 | "only if it (1) is exported, (2) is a `const`, (3) ... never change"; top level of exported object (`EXPORTED_OBJECT.key`) | non-exported constants "within a file"; nested keys of an object (`MAPPING.KEY` is "bad") |
| sergiodxa `const-let-usage` | module-level primitives and regexes | module-level objects/arrays/Maps/Sets -> `camelCase`; anything inside a function (`let` + camelCase) |
| Fowler / Martin / McConnell | examples use `STANDARD_GRAVITY`, `SECONDS_PER_DAY`, `MONTHS_PER_YEAR` (Java/JS habit); no naming rule stated | – |

Conflicts: (a) Airbnb requires `export` for uppercase; Google requires module scope only. (b) Google uppercases constant objects; sergiodxa camelCases them; Airbnb uppercases the top-level name but not keys. (c) All three agree: never uppercase a local `const`.

### Enums vs union types vs `as const`

| Source | Position |
|---|---|
| TS Handbook "Objects vs Enums" | "you may not need an enum when an object with `as const` could suffice"; `as const` "keeps your codebase aligned with the state of JavaScript"; costs "an extra line to pull out the values" |
| TS Handbook `const enum` pitfalls | ambient const enums break `isolatedModules`, can inline stale values across dependency versions, and break import elision |
| TS 5.8 `--erasableSyntaxOnly` | `enum` is not erasable; disallowed for Node type-stripping targets |
| Google TS style guide | use `enum`, never `const enum`; enum values are CONSTANT_CASE |
| Matt Pocock | "if I were starting a project today, I would use `as const` instead of enums"; if enums, string enums only; concedes enums are more explicit (nominal) while `as const` accepts raw strings |
| typescript-eslint `no-magic-numbers` | `ignoreEnums`, `ignoreNumericLiteralTypes` default `false`: numbers inside enums and literal-type unions are flagged unless opted out |

Net: Handbook + Pocock + Node runtime constraint favour `as const` object + `(typeof X)[keyof typeof X]` union; Google favours plain `enum`. No source favours numeric enums. String literal unions alone (`type Sort = "asc" | "desc"`) have no runtime value list; use `as const` array/object when the values are iterated or validated (sergiodxa examples use `z.enum([...])` for that, not TS `enum`).

## Lint coverage

| Policy | ESLint `no-magic-numbers` | `@typescript-eslint/no-magic-numbers` | Biome `noMagicNumbers` |
|---|---|---|---|
| Allow 0 / 1 / -1 | `ignore: [0, 1, -1]` | same | built-in (0, 1, 2, 10, 24, 60, negatives, bigints) |
| Allow array indexes | `ignoreArrayIndexes: true` | same | built-in |
| Allow default param values | `ignoreDefaultValues: true` | same | built-in |
| Allow class field initialisers | `ignoreClassFieldInitialValues: true` | same | built-in (variable/property initialisers) |
| Allow readonly class props | – | `ignoreReadonlyClassProperties: true` | built-in (initialisers) |
| Allow numbers in `enum` | – | `ignoreEnums: true` | built-in |
| Allow numeric literal types `2 \| 3` | – | `ignoreNumericLiteralTypes: true` | not stated; type casts/`as const` ignored |
| Allow `Bar[0]` type index | – | `ignoreTypeIndexes: true` | not stated |
| Flag numbers in object values | `detectObjects: true` (default off) | same | never (object property values ignored) |
| Require `const` for the constant | `enforceConst: true` | same | – |
| Time arithmetic `24 * 60` | must list in `ignore` | same | built-in |
| JSX literals (`size={24}`) | allowed (any JSX parent) | allowed | built-in ignore |
| Magic strings | not covered | not covered | not covered |
| Recommended preset | no (frozen) | no | no (`style`, since v2.1.0, severity `information`) |
| Configurable | yes | yes | no options |

Gaps: no rule covers strings; ESLint reports every literal in an expression or call (`radius * 2`, `5 * 60 * 1000`) under any option set, so it cannot encode the formula exception; Biome's ignore list (esp. `2`, `10`) is broader than any book allows. Corrected 2026-09-17 after a reviewer ran the rule: JSX numbers are allowed by ESLint, not flagged.

## Delta vs Vercel / sergiodxa

- `vercel-labs/agent-skills` (react-best-practices, composition-patterns, react-view-transitions): **no rule** on magic numbers, named constants, enums vs `as const`, or string unions. `bundle-analyzable-paths` uses `as const` maps and says "a 2-value enum still hides the final path" — a bundler point, not a literal policy. `rerender-memo-with-default-value` hoists object defaults to constants for memo identity, not for readability.
- `sergiodxa/agent-skills` (frontend-js-best-practices, frontend-react-best-practices; no frontend-typescript skill): only `const-let-usage` touches this: module-level primitive constants and regexes `UPPER_SNAKE_CASE`, objects `camelCase`, local bindings `let` + camelCase. **No rule** on when to extract a literal, on enums, or on unions. Its object-camelCase rule conflicts with Google (uppercase constant objects) and Airbnb (uppercase top-level exported objects).
- A frontend-standards rule on magic literals therefore fills a gap in both repos rather than overlapping one.

## Not found

- No primary source for `5 * 60 * 1000` vs `FIVE_MINUTES_MS` or for a `_MS` unit-suffix convention. Only Biome's ignore of 24/60 and Martin's `SECONDS_PER_DAY` example touch it.
- No primary source giving `-1` sentinel an explicit exemption.
