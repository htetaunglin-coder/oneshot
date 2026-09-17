# Magic numbers / magic literals — sources

Collected 2026-09-16. Primary sources only. Quotes are verbatim unless marked `[paraphrase]` or `[from print edition]`.
`[from print edition]` = text is from the book; only partially web-verifiable (paywalled). Verify page refs before publishing.

---

## 1. Martin Fowler — Refactoring, 2nd ed. (2018), "Replace Magic Literal"

- URL: https://refactoring.com/catalog/replaceMagicLiteral.html
- Author: Martin Fowler. Book: Addison-Wesley, 2018, ch. 8 "Moving Features", p. 240. Alias: "Replace Magic Number with Symbolic Constant" (1st ed. name).
- Catalog page shows only the sketch, no motivation text (web edition is paywalled):

```js
function potentialEnergy(mass, height) { return mass * 9.81 * height; }
// ->
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, height) { return mass * STANDARD_GRAVITY * height; }
```

- Motivation `[from print edition, paraphrase]`: magic literals are numbers (or other literal values) in source whose meaning is not obvious. The two costs: the reader must work out what `9.81` means; and the same value appears in many places, so a change is unreliable.
- Caveat 1 `[from print edition, paraphrase]`: not every literal deserves a constant. A literal that is clear in context stays inline. Fowler's example: a constant named `ONE` for `1` adds nothing.
- Caveat 2 `[from print edition, paraphrase]`: when the literal is used in a comparison, a well-named function often beats a constant. Fowler's example: for `aValue === "M"` he prefers `isMale(aValue)` over `aValue === MALE_GENDER`. (Task prompt frames the same idea as `isGoldCustomer()` vs `GOLD_TOKEN`; Fowler's actual example is the `"M"` / `isMale` one.)
- Mechanics `[from print edition, paraphrase]`: declare a constant set to the literal; find all uses of the literal; for each, check it means the same thing as the constant; replace; test.
- Rule extracted: replace when the value has a meaning the reader must decode OR is duplicated; do not replace when the name only restates the value; prefer a function when the literal is a test/comparison.

---

## 2. Robert C. Martin — Clean Code (2008), ch. 17 "Smells and Heuristics", G25

- URL (partial verification): https://www.ersoftware.nl/clean-code/17-smells-and-heuristics/ ; outline: https://gist.github.com/scottashipp/88b3a4d97eaa542842bcf5b08f5bac6d
- Author: Robert C. Martin. Prentice Hall, 2008, pp. 300-301.
- G25 title: "Replace Magic Numbers with Named Constants".
- Rule `[from print edition]`: "In general it is a bad idea to have raw numbers in your code. You should hide them behind well-named constants. For example, the number 86,400 should be hidden behind the constant SECONDS_PER_DAY. If you are printing 55 lines per page, then the constant 55 should be hidden behind the constant LINES_PER_PAGE."
- Exception (web-verified fragment on ersoftware.nl): "some constants are so easy to recognize that they don't always need a named constant" `[print edition continues:]` "to hide behind so long as they are used in conjunction with very self-explanatory code." Examples `[from print edition]`:

```java
double milesWalked = feetWalked/5280.0;
int dailyPay = hourlyRate * 8;
double circumference = radius * Math.PI * 2;
```

  "Do we really need the constants FEET_PER_MILE, WORK_HOURS_PER_DAY, and TWO in the above examples? Clearly, the last case is absurd. There are some formulae in which constants are simply better written as raw numbers." `[from print edition]`
- On 5280 `[from print edition]`: "the number 5280 is so very well known and so unique a constant that readers would recognize it even if it stood alone on a page with no context surrounding it."
- On PI `[from print edition]`: constants like 3.141592653589793 are well known "However, the chance for error is too great to leave them raw" — use `Math.PI`.
- Magic strings `[from print edition]`: "The term 'Magic Number' does not apply only to numbers. It applies to any token that has a value that is not self-describing." Example `assertEquals(7777, Employee.find("John Doe").getEmployeeNumber());` — both `7777` and `"John Doe"` are magic.
- Rule extracted: obvious-in-formula numbers (2 = "twice", 8 hours, 5280) may stay inline; well-known-but-long constants (PI) must use the library constant; non-self-describing strings count as magic too.

---

## 3. Steve McConnell — Code Complete, 2nd ed. (2004), §12.1 "Numbers in General" and §12.7 "Named Constants"

- URL (TOC only; content paywalled, 403): https://www.oreilly.com/library/view/code-complete-2nd/0735619670/ch12s01.html and .../ch12s07.html
- Author: Steve McConnell. Microsoft Press, 2004, ch. 12 "Fundamental Data Types", pp. 292-293, 307-309.
- "Avoid 'magic numbers.'" `[from print edition]`: "Magic numbers are literal numbers, such as 100 or 47524, that appear in the middle of a program without explanation. If you program in a language that supports named constants, use them instead." Reasons: "Changes can be made more reliably", "Changes can be made more easily", "Your code is more readable".
- "Use hard-coded 0s and 1s if you need to." `[from print edition]`: "The values 0 and 1 are used to increment, decrement, and start loops at the first element of an array. The 0 in `for ( i = 0; i < CONSTANT; i++ )` is OK, and the 1 in `total = total + 1` is OK. A good rule of thumb is that the only literals that should occur in the body of a program are 0 and 1. Any other literals should be replaced with something more descriptive."
- "Anticipate divide-by-zero." `[from print edition, paraphrase]`: every time you use `/`, consider whether the denominator could be 0.
- "Make type conversions obvious." `[from print edition, paraphrase]`: write `y = x + (float) i` rather than relying on implicit conversion; make mixed-type expressions explicit.
- §12.7 "Avoid literals, even 'safe' ones" `[from print edition, paraphrase]`: `for i = 1 to 12` for months looks safe, but `MONTHS_PER_YEAR` is still better: it documents intent and survives change. Also "Use named constants consistently" and "Simulate named constants with appropriately scoped variables or classes" if the language lacks them.
- Rule extracted: hard rule "only 0 and 1 inline"; strictest of the three authors; even 12 (months) gets a name.

---

## 4. Lint rules

### 4a. ESLint `no-magic-numbers`

- URL: https://eslint.org/docs/latest/rules/no-magic-numbers
- Author: ESLint team. Status: **frozen** ("not accepting feature requests"). **Not in `eslint:recommended`** (`recommended=false` in page metadata).
- Description: "Disallow magic numbers." Definition: "'Magic numbers' are numbers that occur multiple times in code without an explicit meaning. They should preferably be replaced by named constants."
- Options and defaults:
  - `ignore` (default `[]`): "an array of numbers to ignore" (accepts numbers and bigint strings like `"100n"`).
  - `ignoreArrayIndexes` (default `false`): "set to true to ignore numbers used as array indexes" (`data[2]`; valid indexes `0` to `4294967294`).
  - `ignoreDefaultValues` (default `false`): "set to true to ignore numbers used as default values" (param defaults, destructuring defaults).
  - `ignoreClassFieldInitialValues` (default `false`): "set to true to ignore numbers used as initial values of class fields" (incl. private/static).
  - `enforceConst` (default `false`): "set to true to require the `const` keyword for numeric declarations" (rejects `let`/`var`).
  - `detectObjects` (default `false`): "set to true to detect numbers used as object property values".
- Note: base rule does not know TS syntax; use the TS extension below for enums/literal types.

### 4b. `@typescript-eslint/no-magic-numbers`

- URL: https://typescript-eslint.io/rules/no-magic-numbers/
- Author: typescript-eslint team. "This rule extends the base `no-magic-numbers` rule from ESLint core." Not in `recommended`/`strict`/`stylistic` configs (extension rule, opt-in).
- Extra options, all default `false`:
  - `ignoreEnums`: "Whether enums used in TypeScript are considered acceptable." (`enum foo { SECOND = 1000 }`)
  - `ignoreNumericLiteralTypes`: "Whether numbers appearing in TypeScript numeric literal types are considered acceptable." (`type SmallPrimes = 2 | 3 | 5 | 7 | 11;`)
  - `ignoreReadonlyClassProperties`: "Whether `readonly` class properties are considered acceptable." (`class Foo { readonly A = 1; }`)
  - `ignoreTypeIndexes`: "Whether numbers used to index types are considered acceptable." (`type Foo = Bar[0]; type Baz = Parameters<Foo>[2];`)

### 4c. Biome `noMagicNumbers`

- URL: https://biomejs.dev/linter/rules/no-magic-numbers/
- Author: Biome team. Group: **`style`** (`lint/style/noMagicNumbers`). Since **v2.1.0**. **Not recommended** ("isn't recommended, so you need to enable it"). Default severity: `information`. No fix. Sources: inspired by ESLint `no-magic-numbers`, same as `@typescript-eslint/no-magic-numbers`.
- Description: reports "'magic numbers' — numbers used directly instead of being assigned to named constants."
- Built-in ignores (no options; opinions baked in): "non-magic values (like 0, 1, 2, 10, 24, 60, and their negative or bigint forms) found anywhere, including arithmetic expressions and function calls"; array indices; enum values; variable/property initializers; function parameter defaults; destructuring defaults; arguments to `JSON.stringify` and `parseInt`; bitwise operands; JSX expressions; object property values. Type casts / `as const` wrappers are ignored (commit e53f2fe).
- Example: invalid `let total = price * 1.23;` valid `const TAX_RATE = 1.23; let total = price * TAX_RATE;`.
- Note: 24 and 60 in the ignore list = Biome treats hours/minutes/seconds arithmetic as self-describing.

---

## 5. Constant naming

### 5a. Google TypeScript Style Guide — "Identifiers"

- URL: https://google.github.io/styleguide/tsguide.html#identifiers
- Author: Google.
- Naming table: `CONSTANT_CASE` applies to "global constant values, including enum values".
- "Immutable: CONSTANT_CASE indicates that a value is intended to not be changed, and may be used for values that can technically be modified (i.e. values that are not deeply frozen) to indicate to users that they must not be modified."
- Scope caveat: "Only symbols declared on the module level, static fields of module level classes, and values of module level enums, may use CONST_CASE. If a value can be instantiated more than once over the lifetime of the program (e.g. a local variable declared within a function, or a static field on a class nested in a function) then it must use lowerCamelCase."
- Example in guide: `const UNIT_SUFFIXES = { milliseconds: 'ms', seconds: 's' };` — an object in CONSTANT_CASE signals "must not be modified" even though JS cannot freeze it.
- Enums: "Enums: Always use `enum` and not `const enum`. TypeScript enums already cannot be mutated; `const enum` is a separate language feature related to optimization that makes the enum invisible to JavaScript users of the module." Also: "must not use `const enum`; use plain `enum` instead."
- No rule about magic numbers as such in the TS guide.

### 5b. Airbnb JavaScript Style Guide — 23.10

- URL: https://github.com/airbnb/javascript#naming--uppercase
- Author: Airbnb.
- "23.10 You may optionally uppercase a constant only if it (1) is exported, (2) is a `const` (it can not be reassigned), and (3) the programmer can trust it (and its nested properties) to never change."
- "Why? This is an additional tool to assist in situations where the programmer would be unsure if a variable might ever change. UPPERCASE_VARIABLES are letting the programmer know that they can trust the variable (and its properties) not to change."
- "What about all `const` variables? - This is unnecessary, so uppercasing should not be used for constants within a file. It should be used for exported constants however."
- "What about exported objects? - Uppercase at the top level of export (e.g. `EXPORTED_OBJECT.key`) and maintain that all nested properties do not change."
- Examples: `// bad const PRIVATE_VARIABLE = 'should not be unnecessarily uppercased within a file';` `// better in most cases export const API_KEY = 'SOMEKEY';` `// bad export const MAPPING = { KEY: 'value' }; // good export const MAPPING = { key: 'value' };`
- No "magic number" rule in the Airbnb guide.

---

## 6. Enums vs unions vs `as const`

### 6a. TypeScript Handbook — "Enums"

- URL: https://www.typescriptlang.org/docs/handbook/enums.html
- Author: TypeScript team.
- "Objects vs Enums": "In modern TypeScript, you may not need an enum when an object with `as const` could suffice". Example: `const ODirection = { Up: 0, Down: 1, Left: 2, Right: 3 } as const;` then "It requires an extra line to pull out the values: `type Direction = typeof ODirection[keyof typeof ODirection];`". "The biggest argument in favour of this format over TypeScript's `enum` is that it keeps your codebase aligned with the state of JavaScript, and when/if enums are added to JavaScript then you can move to the additional syntax."
- Reverse mappings: "numeric enums members also get a reverse mapping from enum values to enum names." "Keep in mind that string enum members do not get a reverse mapping generated at all."
- `const enum` pitfalls (ambient / `.d.ts` case): "1. ... isolatedModules ... is fundamentally incompatible with ambient const enums." "2. You can easily inline values from version A of a dependency at compile time, and import version B at runtime. Version A and B's enums can have different values ... resulting in surprising bugs, like taking the wrong branches of `if` statements." "3. `importsNotUsedAsValues: "preserve"` will not elide imports for const enums used as values, but ambient const enums do not guarantee that runtime `.js` files exist."

### 6b. Matt Pocock — Total TypeScript

- URL 1: https://www.totaltypescript.com/why-i-dont-like-typescript-enums ("Why I Don't Like TypeScript Enums")
- URL 2: https://www.totaltypescript.com/books/total-typescript-essentials/deriving-types (book chapter "Deriving Types", section on enums vs `as const`)
- URL 3: https://www.totaltypescript.com/concepts/as-const
- Author: Matt Pocock. Undated on page (article ~2023).
- Objections (URL 1): numeric enums: "you can pass a raw number in places where the enum is expected"; numeric enums create 6 keys (bidirectional) vs 3 for string enums; nominal: "you can't use an enum in place of another enum, even if the values are the same"; the TS repo has many enum bugs, "many of these are uncloseable due to the way enums are implemented." Position: TypeScript's enum implementation is "weird enough" that he does not recommend them; if you must, use string enums only.
- Recommendation (URL 2): "If you're working with a team that's used to `enum`, you should use `enum`. But if I were starting a project today, I would use `as const` instead of enums." Trade-off stated: enums are nominal ("Enums with the same values that come from different enums are not compatible"), `as const` is structural ("if two POJOs have the same values, they are compatible"); with enums "you have to pass the enum value, which is more explicit", with `as const` "you can pass a raw string".
- Derivation (URL 2): `type AlbumType = (typeof albumTypes)[keyof typeof albumTypes];`
- URL 3: "Objects and arrays marked with `as const` get inferred as their literal types, not the wider types. This can be useful for creating type-safe enums without needing the `enum` keyword."

### 6c. TypeScript 5.8 — `--erasableSyntaxOnly`

- URL: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-8.html#the---erasablesyntaxonly-option
- Author: TypeScript team, Feb 2025.
- Purpose: compatibility with Node.js 23.6+ type stripping (`--experimental-strip-types`), which "requires that TypeScript-specific syntax must be erasable".
- Disallowed under the flag: "`enum` declarations", "`namespace`s and `module`s with runtime code", "Parameter properties in classes", "Non-ECMAScript `import =` and `export =` assignments". Example error: `// error: An enum declaration  enum Direction { Up, Down, Left, Right }`.
- Consequence: a project targeting Node type-stripping cannot use `enum`; `as const` objects + derived unions are the erasable form.

---

## 7. Delta check — `vercel-labs/agent-skills` and `sergiodxa/agent-skills`

- Method: cloned both repos (2026-09-16), grep for `magic (number|literal|string)`, `named constant`, `as const`, `enum`, `literal union`, `UPPER_CASE`, `CONSTANT_CASE`, `hard-?coded`.
- **vercel-labs/agent-skills, react-best-practices**: no rule on magic numbers or constant naming. Nearest hits:
  - `rules/bundle-analyzable-paths.md` "Prefer Statically Analyzable Paths" (impact HIGH): uses `as const` maps of import thunks; "Incorrect (a 2-value enum still hides the final path from static analysis)". About bundler analysis, not literals.
  - `rules/rerender-memo-with-default-value.md` "Extract Default Non-primitive Parameter Value from Memoized Component to Constant": hoist object/array defaults to module constants for memo identity. Not about literals.
  - `rules/rerender-lazy-state-init.md`: "For simple primitives (`useState(0)`) ... the function form is unnecessary." Not a literal rule.
- **vercel-labs/agent-skills, composition-patterns**: none. `patterns-explicit-variants.md` is about variant props, not string unions/enums as constants.
- **vercel-labs/agent-skills, react-view-transitions**: `references/patterns.md`: "Use `as const` arrays and derived types to prevent ID clashes" (`const transitionTypes = [...] as const`). Domain-specific; not a general rule.
- **sergiodxa/agent-skills, frontend-js-best-practices**: `rules/const-let-usage.md` "Const vs Let Usage" (impact MEDIUM): "Single value constants - UPPER_SNAKE_CASE" (`const MAX_RETRIES = 3; const DEFAULT_TIMEOUT = 5000; const API_BASE_URL = "/api/v1";`); "Objects, arrays, Maps, Sets - camelCase"; "Regex patterns - UPPER_SNAKE_CASE"; "UPPER_SNAKE_CASE - Instantly identifies true constants (primitives that never change)". Summary table: module-level primitive constants = `UPPER_SNAKE_CASE`; module-level objects = `camelCase`; inside functions = `let` + camelCase. No rule on when a literal must become a constant.
- **sergiodxa/agent-skills, frontend-react-best-practices**: none (same `rerender-memo-with-default-value` mirror). No `frontend-typescript` skill exists in the repo.
- Net: neither repo states a magic-number rule, an enum-vs-`as const` rule, or a string-literal-union rule. Only naming-case convention (sergiodxa) exists, and it conflicts with Google/Airbnb on objects and on local `const`.

---

## 8. Tailwind CSS v4 — arbitrary values vs theme tokens

- URL: https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values ; https://tailwindcss.com/docs/theme
- Author: Tailwind Labs.
- "While you can usually build the bulk of a well-crafted design using a constrained set of design tokens, once in a while you need to break out of those constraints to get things pixel-perfect. When you find yourself really needing something like `top: 117px` to get a background image in just the right spot, use Tailwind's square bracket notation to generate a class on the fly with any arbitrary value."
- "If you want to change things like your color palette, spacing scale, typography scale, or breakpoints, add your customizations using the `@theme` directive in your CSS."
- Theme page: "Use `@theme` when you want a design token to map directly to a utility class, and use `:root` for defining regular CSS variables that shouldn't have corresponding utility classes."
- Mapping to this topic: `w-[347px]` is the CSS-side magic literal; a reused value belongs in `@theme` (named token), a one-off pixel fix may stay arbitrary.

---

## 9. Time / units: self-describing expression vs named constant

- Searched: Fowler catalog, Google style guides (TS/JS/Java/C#), ESLint/Biome docs. **No primary source states a rule** that `5 * 60 * 1000` is preferable to `FIVE_MINUTES_MS` or vice versa.
- Closest primary evidence:
  - Biome `noMagicNumbers` ignore list includes `24` and `60` "found anywhere, including arithmetic expressions" (https://biomejs.dev/linter/rules/no-magic-numbers/) — i.e. the linter treats `24 * 60 * 60` as self-describing but flags `86400`.
  - Clean Code G25 `[from print edition]`: "the number 86,400 should be hidden behind the constant SECONDS_PER_DAY" — names the collapsed value, says nothing against the expanded product.
  - Google TS style guide example object `UNIT_SUFFIXES = { milliseconds: 'ms', seconds: 's' }` shows unit-bearing names but no duration rule.
- Conclusion: unit suffix (`_MS`) and expanded products are house-style decisions; no authority to cite.
