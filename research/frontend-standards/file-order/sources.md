# File-order research: sources

Topic: order of declarations inside one JS/TS/React source file (imports, constants, types, main component, sub-components, helpers, exports). Primary sources only. Retrieved 2026-09-14.

---

## 1. Robert C. Martin, *Clean Code* (Prentice Hall, 2008)

Book not online. Quotes verified against reader notes that transcribe the text (https://github.com/JuanCrg90/Clean-Code-Notes) and a search-index hit for the Newspaper Metaphor passage.

### Ch. 3 "Functions" — "Reading Code from Top to Bottom: The Stepdown Rule"
- "We want the code to read like a top-down narrative. We want every function to be followed by those at the next level of abstraction so that we can read the program, descending one level of abstraction at a time as we read down the list of functions."
- "...we want to be able to read the program as though it were a set of TO paragraphs, each of which is describing the current level of abstraction and referencing subsequent TO paragraphs at the next level down."
- "It turns out to be very difficult for programmers to learn to follow this rule and write functions that stay at a single level of abstraction. But learning this trick is also very important. It is the key to keeping functions short and making sure they do 'one thing.'"

### Ch. 5 "Formatting" — "The Newspaper Metaphor"
- "Think of a well-written newspaper article. ... At the top you expect a headline that will tell you what the story is about. The first paragraph gives you a synopsis of the whole story, hiding all the details ... As you continue downward, the details increase."
- "We would like a source file to be like a newspaper article. The name should be simple but explanatory. ... The topmost parts of the source file should provide the high-level concepts and algorithms. Detail should increase as we move downward, until at the end we find the lowest level functions and details in the source file."

### Ch. 5 — "Vertical Distance"
- Variable declarations: "Variables should be declared as close to their usage as possible. Because our functions are very short, local variables should appear at the top of each function."
- Instance variables: "Instance variables, on the other hand, should be declared at the top of the class. ... The important thing is for the instance variables to be declared in one well-known place."
- Dependent functions: "If one function calls another, they should be vertically close, and the caller should be above the callee, if at all possible. This gives the program a natural flow. If the convention is followed reliably, readers will be able to trust that function definitions will follow shortly after their use."
- Conceptual affinity: "Certain bits of code want to be near other bits. They have a certain conceptual affinity. The stronger that affinity, the less vertical distance there should be between them."

### Ch. 5 — "Vertical Ordering"
- "In general we want function call dependencies to point in the downward direction. That is, a function that is called should be below a function that does the calling. This creates a nice flow down the source code module from high level to low level."
- "(This is the exact opposite of languages like Pascal, C, and C++ that enforce functions to be defined, or at least declared, before they are used.)"

Position: **top-down, caller above callee**. Relies on hoisting (Java methods). Opposite of define-before-use.

---

## 2. Steve McConnell, *Code Complete*, 2nd ed. (Microsoft Press, 2004), ch. 31 "Layout and Style"

Text checked via flylib mirror of 31.7 (https://flylib.com/books/en/2.823.1.267/1/) and 31.8 (https://flylib.com/books/en/2.823.1.268/1/); TOC via O'Reilly (https://www.oreilly.com/library/view/code-complete-2nd/0735619670/ch31s08.html).

### 31.7 Laying Out Routines
- "Use blank lines to separate parts of a routine" (header, data and named-constant declarations, body).
- "Use standard indentation for routine arguments." Reason: "accuracy, consistency, readability, and modifiability."

### 31.8 Laying Out Classes
- Class interface order: "Header comment ... Constructors and destructors / Public routines / Protected routines / Private routines and member data".
- Class implementation order: "Header comment ... Class data / Public routines / Protected routines / Private routines".
- "If you have more than one class in a file, identify each class clearly ... A class is like a chapter in a book."
- "put only one class in each file unless you have a compelling reason to do otherwise (such as including a few small classes that make up a single pattern)."

### 31.8 "Laying Out Files and Programs"
- "Put one class in one file ... a file should hold a collection of routines that supports one and only one purpose."
- "Give the file a name related to the class name".
- "Separate routines within a file clearly. Separate each routine from other routines with at least two blank lines."
- "Sequence routines alphabetically. An alternative to grouping related routines in a file is to put them in alphabetical order." (Implicit default: **group related routines**.)
- "In C++, order the source file carefully. Here's a typical order of source-file contents in C++: File-description comment / #include files / Constant definitions that apply to more than one class / Enums ... / Macro function definitions / Type definitions ... / Global variables and functions imported / Global variables and functions exported / Variables and functions that are private to the file / Classes".

Position: **public before private; shared constants/types before classes; related routines grouped; one purpose per file**. Silent on caller-vs-callee direction.

---

## 3. Google style guides

### Angular Style Guide (legacy, angular.io v17) — Style 05-14 "Member sequence"
Source: https://github.com/angular/angular/blob/17.0.x/aio/content/guide/styleguide.md (rendered at https://v17.angular.io/guide/styleguide#member-sequence).
- "**Do** place properties up top followed by methods."
- "**Do** place private members after public members, alphabetized."
- "**Why**? Placing members in a consistent sequence makes it easy to read and helps instantly identify which members of the component serve which purpose."

### Angular Style Guide (current, angular.dev) — "Group Angular-specific properties before methods"
Source: https://angular.dev/style-guide
- "Components and directives should group Angular-specific properties together, typically near the top of the class declaration. This includes injected dependencies, inputs, outputs, and queries. Define these and other properties before the class's methods."
- Public-before-private and alphabetized rules are **dropped** in the current guide.

### Google JavaScript Style Guide — §3 "Source file structure"
Source: https://google.github.io/styleguide/jsguide.html
- "Files consist of the following, in order: 1. License or copyright information, if present 2. @fileoverview JSDoc, if present 3. goog.module statement, if a goog.module file 4. ES import statements, if an ES module 5. goog.require and goog.requireType statements 6. The file's implementation"
- "Exactly one blank line separates each section that is present, except the file's implementation, which may be preceded by 1 or 2 blank lines."
- §5.1.3 "Declared when needed, initialized as soon as possible": "Local variables are not habitually declared at the start of their containing block or block-like construct. Instead, local variables are declared close to the point they are first used (within reason), to minimize their scope".
- §5.1.1 "Use const by default, unless a variable needs to be reassigned."

### Google TypeScript Style Guide
Source: https://google.github.io/styleguide/tsguide.html
- "Source file structure": "Files consist of the following, in order: 1. Copyright information, if present 2. JSDoc with @fileoverview, if present 3. Imports, if present 4. The file's implementation". "Exactly one blank line separates each section that is present."
- Exports: "Use named exports in all code." "Do not use default exports." "Only export symbols that are used outside of the module. Generally minimize the exported API surface of modules."
- "Prefer function declarations for named functions" (top-level); "Do not use function expressions. Use arrow functions instead" for nested/inline.
- No rule on ordering of implementation-level declarations; no rule on export placement (inline vs bottom).

---

## 4. Airbnb JavaScript Style Guide
Source: https://github.com/airbnb/javascript (README.md, master).
- 7.1 "Use named function expressions instead of function declarations. eslint: func-style". Why: "Function declarations are hoisted, which means that it's easy - too easy - to reference the function before it is defined in the file. This harms readability and maintainability."
- 7.3 "Never declare a function in a non-function block (if, while, etc). Assign the function to a variable instead."
- 10.7 "Put all imports above non-import statements. eslint: import/first" — "Since imports are hoisted, keeping them all at the top prevents surprising behavior."
- 13.4 "Assign variables where you need them, but place them in a reasonable place." Why: "let and const are block scoped and not function scoped."
- 14.1 "var declarations get hoisted to the top of their closest enclosing function scope, their assignment does not. const and let declarations are blessed with a new concept called Temporal Dead Zones (TDZ)."
- 14.2 "Anonymous function expressions hoist their variable name, but not the function assignment."
- 14.3 "Named function expressions hoist the variable name, not the function name or the function body."
- 14.4 "Function declarations hoist their name and the function body."
- 14.5 "Variables, classes, and functions should be defined before they can be used. eslint: no-use-before-define" — Why: "When variables, classes, or functions are declared before being used, it can harm readability since a reader won't know what one of the fundamental building blocks is until they've read the source." (sic; the intent is define-before-use.)

Position: **define before use**, explicitly anti-hoisting. Direct opposite of Clean Code Vertical Ordering.

---

## 5. ESLint `no-use-before-define` / `@typescript-eslint/no-use-before-define`

### ESLint core
Source: https://eslint.org/docs/latest/rules/no-use-before-define
- "In JavaScript, prior to ES6, variable and function declarations are hoisted to the top of a scope, so it's possible to use identifiers before their formal declarations in code. This can be confusing and some believe it is best to always declare variables and functions before using them."
- "In ES6, block-level bindings (let and const) introduce a 'temporal dead zone' where a ReferenceError will be thrown with any attempt to access the variable before its declaration."
- "This rule will warn when it encounters a reference to an identifier that has not yet been declared."
- `functions` (default `true`): "The flag which shows whether or not this rule checks function declarations. If this is true, the rule warns every reference to a function before the function declaration. Otherwise, ignores those references. Function declarations are hoisted, so it's safe."
- `classes` (default `true`): "...If this is true, the rule warns every reference to a class before the class declaration. Otherwise, ignores those references if the declaration is in upper function scopes. Class declarations are not hoisted, so it might be danger."
- `variables` (default `true`): "This flag determines whether or not the rule checks variable declarations in upper scopes. If this is true, the rule warns every reference to a variable before the variable declaration. Otherwise, the rule ignores a reference if the declaration is in an upper function scope, while still reporting the reference if it's in the same scope as the declaration."
- `allowNamedExports` (default `false`): "If this flag is set to true, the rule always allows references in export {} declarations. These references are safe even if the variables are declared later in the code."
- Not in `eslint:recommended`.

### typescript-eslint
Source: https://typescript-eslint.io/rules/no-use-before-define/
- "This rule extends the base eslint/no-use-before-define rule. It adds support for type, interface and enum declarations."
- `enums` (default `true`): "Whether to check references to enums. If this is true, this rule warns every reference to a enum before the enum declaration. If this is false, this rule will ignore references to enums, when the reference is in a child scope."
- `typedefs` (default `true`): "Whether to check references to types. If this is true, this rule warns every reference to a type before the type declaration. If this is false, this rule will ignore references to types."
- `ignoreTypeReferences` (default `true`): "Whether to ignore type references, such as in type annotations and assertions. If this is true, this rule ignores all type references. If this is false, this will check all type references."
- Not in `recommended`/`strict` configs.

Net: with defaults `{functions: true}` the rule bans caller-above-callee even for hoisted function declarations; the common relaxation is `{functions: false, classes: true, variables: true, typedefs: false}`.

---

## 6. Sorting plugins

### eslint-plugin-perfectionist `sort-modules`
Source: https://perfectionist.dev/rules/sort-modules
- Sorts module-level `enum`, `interface`, `type`, `class`, `function` declarations. Does **not** sort imports, re-exports, or "other expressions to ensure compilation and runtime behavior" (so `const` statements are left in place).
- Default groups in order: `declare-enum, export-enum, enum` -> `[declare-interface, declare-type]` -> `[export-interface, export-type]` -> `[interface, type]` -> `declare-class, class, export-class` -> `declare-function, export-function, function`.
- Defaults: `type: 'alphabetical'`, `order: 'asc'`, `ignoreCase: true`, `partitionByComment: false`, `partitionByNewLine: false`, `newlinesBetween: 'ignore'`.
- Dependency safety: when sorting would create a use-before-define, the rule keeps dependencies first; `type: 'usage'` "enforces items referenced by other items within the same group to appear before the items that reference them."
- Net default shape: **enums -> types/interfaces -> classes -> functions**, exports first within each kind, alphabetical.

### eslint-plugin-react `sort-comp`
Source: https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/sort-comp.md
- "Enforce component methods order." Default `order`: `static-methods` -> `lifecycle` -> `everything-else` -> `render`.
- Default `lifecycle` group: displayName, propTypes, contextTypes, childContextTypes, mixins, statics, defaultProps, constructor, getDefaultProps, state, getInitialState, getChildContext, getDerivedStateFromProps, componentWillMount, UNSAFE_componentWillMount, componentDidMount, componentWillReceiveProps, UNSAFE_componentWillReceiveProps, shouldComponentUpdate, componentWillUpdate, UNSAFE_componentWillUpdate, getSnapshotBeforeUpdate, componentDidUpdate, componentDidCatch, componentWillUnmount.
- Options: `order` (array of group names / regex like `/^on.+$/`), `groups` (named custom groups). Fixable via react-codemod `sort-comp`.
- Applies to class components only; irrelevant to function components. Not deprecated but legacy in practice.

---

## 7. Delta check: vercel-labs/agent-skills and sergiodxa/agent-skills

Cloned 2026-09-14. No rule about declaration order, caller/callee direction, define-before-use, component-per-file, or export placement in either repo. `vercel-labs/agent-skills` has no `next-js` skill (skills present: composition-patterns, react-best-practices, react-native-skills, react-view-transitions, vercel-optimize, web-design-guidelines, writing-guidelines, deploy-to-vercel, vercel-cli-with-tokens).

Hits (all about **what goes to module scope**, never **where in the file**):

vercel-labs/agent-skills `react-best-practices`
- `rendering-hoist-jsx` "Hoist Static JSX Elements" (LOW): "Extract static JSX outside components to avoid re-creation." Example puts `const loadingSkeleton = (...)` above `function Container()`. Note: "If your project has React Compiler enabled, the compiler automatically hoists static JSX elements ... making manual hoisting unnecessary."
- `rerender-memo-with-default-value` (MEDIUM): "extract the default value into a constant." Example `const NOOP = () => {};` above the memo component.
- `server-hoist-static-io` "Hoist Static I/O to Module Level": "Module-level code runs once when the module is first imported, not on every request."
- `server-no-shared-module-state`: "Treat module scope on the server as process-wide shared memory, not request-local state."
- `js-hoist-regexp`: "Don't create RegExp inside render. Hoist to module scope or memoize with useMemo()."
- `js-cache-function-results`: "Use a module-level Map to cache function results".
- `advanced-init-once`: "Use a module-level guard or top-level init in the entry module instead."
- `composition-patterns`: no hits.

sergiodxa/agent-skills
- `frontend-react-best-practices/rendering-hoist-jsx`: identical text to Vercel's: "Extract static JSX outside components to avoid re-creation."
- `frontend-react-best-practices` SKILL.md: "Hoist default non-primitive props to constants."
- `frontend-js-best-practices/const-let-usage`: "Use `const` at module level and `let` inside functions/blocks." Table: "Module-level functions | `function` declaration".
- `frontend-js-best-practices/function-declarations`: "Prefer function declarations for named functions." Why list item 1: "**Hoisting** - Can be called before definition (useful for organizing code)". This is the only hit that touches ordering; it endorses relying on hoisting.
- `frontend-js-best-practices/hoist-regexp`, `cache-function-results`: module-scope caches, no order rule.

---

## 8. Kent C. Dodds, "Colocation" (2019-06-17)
Source: https://kentcdodds.com/blog/colocation
- Principle: "Place code as close to where it's relevant as possible".
- Cites Dan Abramov: "Things that change together should be located as close as reasonable."
- On in-file utilities: "If instead you had just left that function directly in the file that used it, the story would be completely different."
- "The more disconnected/indirect your state is from the UI that's using it, the harder it is to maintain."
- Scope: mostly cross-file (tests, styles, utils, docs). In-file guidance is limited to "keep the utility in the file that uses it"; no statement on above/below.

---

## 9. Extra primary voices

### Dan Abramov, "A Complete Guide to useEffect" (2019-03-09)
Source: https://overreacted.io/a-complete-guide-to-useeffect/
- "If a function doesn't use anything from the component scope, you can hoist it outside the component and then freely use it inside your effects".
- Tip: hoist functions that need no props/state outside the component; pull functions used only by one effect inside that effect.
- Says nothing about above/below placement.

### Josh W. Comeau, "Delightful React File/Directory Structure"
Source: https://www.joshwcomeau.com/react/file-structure/
- Directory-level only: `Widget.constants.ts`, `Widget.helpers.ts`, `Widget.types.ts` next to `Widget.tsx`. Explicitly no in-file ordering rule. Skipped as a source for in-file order.

---

## 10. MDN technical facts

### Hoisting (Glossary)
Source: https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
- "JavaScript Hoisting refers to the process whereby the interpreter appears to move the declaration of functions, variables, classes, or imports to the top of their scope, prior to execution of the code."
- Type 1 (value hoisting): `function`, `function*`, `async function`, `async function*` declarations. "Being able to use a variable's value in its scope before the line it is declared."
- Type 2 (declaration hoisting): `var`.
- Type 3: `let`, `const`, `class` "(also collectively called lexical declarations) are hoisted with type 3 behavior" (TDZ taints the scope; no usable value).
- `import` declarations "are hoisted with type 1 and type 4 behavior."

### `let` — Temporal dead zone
Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz
- "A variable declared with let, const, or class is said to be in a 'temporal dead zone' (TDZ) from the start of the block until code execution reaches the place where the variable is declared and initialized."
- "The term 'temporal' is used because the zone depends on the order of execution (time) rather than the order in which the code is written (position). For example, the code below works because, even though the function that uses the let variable appears before the variable is declared, the function is called outside the TDZ."
- `typeof` inside TDZ also throws ReferenceError.

### `const`
Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- "const declarations can only be accessed after the place of declaration is reached (see temporal dead zone). For this reason, const declarations are commonly regarded as non-hoisted."
- "An initializer for a constant is required."

### `function` declaration — Hoisting
Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function#hoisting
- "Function declarations in JavaScript are hoisted to the top of the enclosing function or global scope. You can use the function before you declared it".
- "Note that function expressions are not hoisted".
- Strict mode: "block-level function declarations are scoped to that block and are hoisted to the top of the block."

### Modules
Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- "Import declarations are hoisted. ... the imported values are available in the module's code even before the place that declares them, and that the imported module's side effects are produced before the rest of the module's code starts running."
- Modules execute once; module body is evaluated top to bottom on first load.
- Cyclic import example: synchronous access to a not-yet-evaluated `export const` throws "ReferenceError: Cannot access 'a' before initialization"; deferred access (setTimeout) works via live bindings.

### Derived rules (from the above)
- A `function` declaration may sit **below** its caller: hoisted with its body.
- A module-scope `const` referenced only inside a function body resolves at **call time**; it may sit below the function declaration as long as no call happens during module evaluation.
- A top-level expression that runs during module evaluation (`const styles = cva(...)`, `const Ctx = createContext(...)`, `const x = memo(Comp)`, `export default withX(Comp)`) reads its inputs immediately; every `const`/`class`/arrow-function it references must be declared **above** it, else ReferenceError (TDZ).
- `class` and arrow-function/`const` components are lexical: any top-level reference to them (e.g. `export default` of an HOC-wrapped arrow) must come after the declaration.
- `type`/`interface` are erased; TypeScript resolves them regardless of position.
