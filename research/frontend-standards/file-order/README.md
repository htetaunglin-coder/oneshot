# File order: research summary

Order of declarations inside one JS/TS/React source file. Primary sources only; see `sources.md` for URLs and quotes. Retrieved 2026-09-14.

## Consensus

| Rule | Sources |
| --- | --- |
| Imports first, above all non-import statements; header comment / `@fileoverview` before imports | Google JS §3, Google TS "Source file structure", Airbnb 10.7 (`import/first`), MDN Modules (imports are hoisted) |
| One purpose (one class / one component) per file; file named after it | Code Complete 31.8 ("Put one class in one file"), Google TS (minimize export surface) |
| Related code sits vertically close; declare variables close to first use | Clean Code "Vertical Distance", Code Complete 31.8 (group related routines), Google JS 5.1.3, Airbnb 13.4, Kent C. Dodds "Colocation" |
| Shared constants, enums, and types come before the things that use them | Code Complete 31.8 C++ file order, perfectionist `sort-modules` default groups (enum -> type/interface -> class -> function) |
| Class members: fields/properties at top, then methods; public before private | Clean Code (instance variables at top), Code Complete 31.8 (public -> protected -> private), Angular 05-14 and angular.dev, `sort-comp` (statics -> lifecycle -> custom -> render) |
| Prefer `function` declarations for named top-level functions | Google TS, sergiodxa `function-declarations`; Airbnb 7.1 dissents (named function expressions) |
| Named exports over default exports; export only what other modules use | Google TS |
| Hoist static values (JSX, regexes, default non-primitive props, config) to module scope | Vercel `rendering-hoist-jsx`, `rerender-memo-with-default-value`, `js-hoist-regexp`, `server-hoist-static-io`; sergiodxa equivalents; Dan Abramov (hoist functions that use no component scope) |
| Use blank lines to separate declarations; consistent section order across files | Google JS/TS (exactly one blank line between sections), Code Complete 31.8 (two+ blank lines between routines), Angular 05-14 ("consistent sequence") |

## Disputed

| Position | Says | Sources |
| --- | --- | --- |
| Top-down / newspaper | "function call dependencies to point in the downward direction"; caller above callee; high-level concept at top, detail at bottom; explicitly rejects C/Pascal declare-before-use | Clean Code ch. 3 Stepdown Rule, ch. 5 Newspaper Metaphor, Vertical Ordering; sergiodxa `function-declarations` ("Hoisting - can be called before definition (useful for organizing code)") |
| Define before use | "Variables, classes, and functions should be defined before they can be used"; function declarations rejected because hoisting makes it "too easy" to reference before definition | Airbnb 14.5 (`no-use-before-define`), Airbnb 7.1; ESLint `no-use-before-define` defaults (`functions: true`) |
| Neutral / structural only | Order by kind (enum, type, class, function) and name, not by call graph; `usage` mode available | perfectionist `sort-modules`; Code Complete ("sequence routines alphabetically" as an alternative to grouping) |
| Silent | No in-file rule | Google JS/TS (only file sections), angular.dev, Kent C. Dodds, Josh Comeau, Vercel skills |

Note: ESLint itself is neutral. `no-use-before-define` is not in `eslint:recommended`; `{ functions: false }` is the documented escape hatch for top-down order and the docs state "Function declarations are hoisted, so it's safe."

## Technical constraints

Any ordering rule must respect these (MDN):

1. `import` declarations are hoisted; the imported module's side effects run before the rest of the file. Position of imports is cosmetic but must be at top for `import/first`.
2. `function`, `function*`, `async function` declarations are hoisted with their body (value hoisting). A caller may sit above its callee at module scope or inside a function.
3. `let`, `const`, `class` are lexical: in the TDZ from the start of the scope until the declaration executes. Any read before that line throws `ReferenceError` (`typeof` included).
4. TDZ is temporal, not positional. A module-scope `const` read only inside a function body resolves when the function is called. So `const STYLES = ...` may sit below `function Card()` as long as `Card` is not invoked during module evaluation.
5. Top-level expressions run during module evaluation: `const styles = cva(...)`, `const Ctx = createContext(...)`, `const Memo = memo(Comp)`, `export default withX(Comp)`, `forwardRef(...)`, `styled.div`. Every `const`/`class`/arrow-function they reference must be declared above them.
6. Arrow-function components (`const Foo = () => ...`) are `const`, so item 5 applies to them: a `memo(Foo)`, `forwardRef`, or HOC wrapper below must come after `Foo`; a `function Foo()` declaration has no such constraint.
7. `class` declarations are not value-hoisted; `new Foo()` or `extends Foo` at top level must come after `class Foo`.
8. In strict mode (all ES modules) block-level function declarations are scoped to their block and hoisted within it only.
9. TypeScript `type` and `interface` are erased; position is free. `enum` and `namespace` emit runtime code and follow rule 3 (`@typescript-eslint/no-use-before-define` has `enums`, `typedefs`, `ignoreTypeReferences` for exactly this split).
10. Cyclic imports: synchronous top-level read of a not-yet-evaluated `export const` throws; deferred reads work through live bindings.

Practical shape that satisfies both camps: imports -> module constants and types -> top-level expressions that other top-level expressions read (cva, createContext) -> main component (`function` declaration, exported) -> sub-components -> helpers (`function` declarations). Function declarations below the caller are legal (2, 4); every `const`-based top-level value stays above its first top-level reader (5).

## Delta vs Vercel / sergiodxa

Already covered by them (do not duplicate):
- Hoist static JSX out of components: Vercel `rendering-hoist-jsx`, sergiodxa `rendering-hoist-jsx` (identical text). Both note React Compiler makes it unnecessary.
- Hoist default non-primitive props to a module constant: Vercel `rerender-memo-with-default-value`, sergiodxa SKILL.md.
- Hoist RegExp / caches / static I/O to module scope: Vercel `js-hoist-regexp`, `js-cache-function-results`, `server-hoist-static-io`, `server-no-shared-module-state`, `advanced-init-once`; sergiodxa `hoist-regexp`, `cache-function-results`.
- `const` at module level, `function` declarations for module-level functions: sergiodxa `const-let-usage`, `function-declarations`.

Not covered by either (gap the frontend-standards skill can fill):
- Order of sections inside a file (imports, constants, types, main component, sub-components, helpers, exports).
- Caller-above-callee vs define-before-use; which `no-use-before-define` options to run.
- Where `cva`/`createContext`/`memo`/HOC wrappers go relative to what they read (TDZ hazard).
- Export placement (inline `export function` vs export list at bottom).
- Component-per-file / when private sub-components may share a file.
- `vercel-labs/agent-skills` has no `next-js` skill; `composition-patterns` has no ordering rules.
