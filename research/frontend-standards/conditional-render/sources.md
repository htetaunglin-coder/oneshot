# Sources: shape of conditional rendering in a React component

Scope: early return, `&&`, ternary, nested ternary, variable-before-JSX, object/map lookup, `switch`, extract sub-component, IIFE in JSX. Primary sources only (official docs, well-known authors, lint rule docs, Airbnb guide). Fetched 2026-09-17.

---

## 1. React docs — Conditional Rendering

- URL: https://react.dev/learn/conditional-rendering
- Author: React team (official docs)
- Date: living doc (react.dev, 2023–)

Techniques, in the order the page shows them:

1. `if` / `return` different JSX trees ("Conditionally returning JSX").
2. `return null` ("Conditionally returning nothing with `null`"). Quote: "A component must return something. In this case, you can return `null`." And: "In practice, returning `null` from a component isn't common because it might surprise a developer trying to render it. More often, you would conditionally include or exclude the component in the parent component's JSX."
3. Ternary `? :` inline in JSX.
4. `&&` inline in JSX.
5. Assign JSX to a `let` variable, then `if` reassigns ("Conditionally assigning JSX to a variable"). Quote: "When the shortcuts get in the way of writing plain code, try using an `if` statement and a variable." And: "This style is the most verbose, but it's also the most flexible."

Pitfall box (verbatim): "**Don't put numbers on the left side of `&&`.** To test the condition, JavaScript converts the left side to a boolean automatically. However, if the left side is `0`, then the whole expression gets that value (`0`), and React will happily render `0` rather than nothing. For example, a common mistake is to write code like `messageCount && <p>New messages</p>`. It's easy to assume that it renders nothing when `messageCount` is `0`, but it really renders the `0` itself! To fix it, make the left side a boolean: `messageCount > 0 && <p>New messages</p>`."

Extract-a-component sentence (verbatim, on the ternary section): "This style works well for simple conditions, but use it in moderation. If your components get messy with too much nested conditional markup, consider extracting child components to clean things up."

Not mentioned on this page: `switch`, object/map lookup, IIFE, nested ternary by name.

## 1b. React docs — Rules of Hooks

- URL: https://react.dev/reference/rules/rules-of-hooks
- Author: React team
- Date: living doc

- Verbatim: "Don't call Hooks inside loops, conditions, nested functions, or `try`/`catch`/`finally` blocks. Instead, always use Hooks at the top level of your React function, before any early returns."
- Bullet list (verbatim): "Do not call Hooks inside conditions or loops." / "Do not call Hooks after a conditional `return` statement." / "Do not call Hooks in event handlers." / "Do not call Hooks in class components." / "Do not call Hooks inside functions passed to `useMemo`, `useReducer`, or `useEffect`." / "Do not call Hooks inside `try`/`catch`/`finally` blocks."
- Code example comment (verbatim): `// 🔴 Bad: after a conditional return (to fix, move it before the return!)`.
- Consequence for the house rule: early returns are allowed, but only after the last hook call.

---

## 2. Kent C. Dodds — "Use ternaries rather than && in JSX"

- URL: https://kentcdodds.com/blog/use-ternaries-rather-than-and-and-in-jsx
- Author: Kent C. Dodds
- Date: 2020-07-27

- The bug: `{contacts.length && contacts.map(...)}`. Quote: "What would happen with the above code if `contacts` was `[]`? That's right! You'd render `0`!"
- Framing: "we're using `&&` to do conditional rendering. Said differently, we're using `&&` to do conditional argument passing."
- Rule: "If you want to do something *conditionally*, don't abuse the logical AND operator (`&&`)." Then: "I strongly suggest that rather than abusing `&&` (or `||` for that matter) in your JSX, you use actual branching syntax features, like ternaries (`condition ? trueConsequent : falseConsequent`) or even `if` statements".
- On `if` + variable-before-JSX: "I'm personally a fan of ternaries, but if you'd prefer `if` statements, that's cool too." Shows `let contactsElements = null; if (contacts.length) { contactsElements = ... }`.
- On readability: "Personally, I think the ternary is more readable (if you disagree, remember that 'readable' is subjective and the readability of something has much more to do with familiarity than anything else). Whatever you do, you do you, just don't do bugs."
- Extra benefit: "test coverage tools can detect and indicate when your tests are not covering one of the branches."
- Position on coercion fixes: "I'm well aware that we could've solved the contact problem by using `!!contacts.length && ...` or `contacts.length > 0 && ...` or even `Boolean(contacts.length) && ...`, but I still prefer not abusing the logical AND operator for rendering. I prefer being explicit by using a ternary."
- Lint: "you can also use `eslint-plugin-react`'s `jsx-no-leaked-render` rule to help catch cases where `&&` might accidentally leak something like `0`".
- Final form: `return error ? <div className="fancy-error">{error.message}</div> : null`.
- Nested ternaries: not addressed. Extracting a component: not addressed. `&&` with a true boolean: he still prefers ternary, but the article's problem statement is falsy leaks, not booleans.

## 2b. Kent C. Dodds — "When to break up a component into multiple components"

- URL: https://kentcdodds.com/blog/when-to-break-up-a-component-into-multiple-components
- Author: Kent C. Dodds
- Date: 2019-07-19

- Criteria are pain points, not size: performance ("Every state change results in a re-render of the entire application"), reuse ("Code sharing/reusability would be... not easy"), state ("Knowing which pieces of state and event handlers went with what parts of JSX would make your head hurt"), testing ("Testing would be 100% integration"), collaboration ("Can you imagine the git diffs and merge conflicts?!"), third-party libs, imperative abstractions ("leading to harder to follow code").
- Core rule (verbatim): "When you experience one of the problems above, that's when you break your component into multiple smaller components. NOT BEFORE."
- On long JSX (verbatim, the closest thing to a readability test): "So I don't mind if the JSX I return in my component function gets really long. Remember that JSX is just a bunch of JavaScript expressions using the declarative APIs given by components. Not a whole lot can go wrong with code like that and it's much easier to keep that code as it is than breaking out things into a bunch of smaller components and start Prop Drilling everywhere."
- Cost framing: "every abstraction comes with a cost" and Sandi Metz: "Duplication is far cheaper than the wrong abstraction."
- Conclusion (verbatim): "don't be afraid of a growing component until you start experiencing real problems. It's WAY easier to maintain it until it needs to be broken up than maintain a pre-mature abstraction."
- Note: there is no explicit "readability test" sentence in this article. The readability trigger is implied by "harder to follow code" and "make your head hurt".

---

## 3. Josh Comeau — "Common Beginner Mistakes with React"

- URL: https://www.joshwcomeau.com/react/common-beginner-mistakes/
- Author: Josh W. Comeau
- Date: 2023-03-06 (last updated 2026-01-30)

- Section: "Evaluating with zero". Bug: `{items.length && <ShoppingList items={items} />}`. Quote: "we wind up with a random `0` in the UI!"
- Cause (verbatim): "This happens because `items.length` evaluates to `0`. And since 0 is a falsy value in JavaScript, the `&&` operator short-circuits, and the entire expression resolves to `0`."
- Why `0` is special (verbatim): "Unlike other falsy values (`''`, `null`, `false`, etc), the number 0 is a valid value in JSX. After all, there are plenty of scenarios in which we really do want to print the number 0!"
- Fix 1 (verbatim): "Our expression should use a 'pure' boolean value (true/false)": `{items.length > 0 && (<ShoppingList items={items} />)}` — "`items.length > 0` will always evaluate to either `true` or `false`, and so we'll never have any issues."
- Fix 2: "Alternatively, we can use a ternary expression": `{items.length ? <ShoppingList items={items} /> : null}`.
- Verdict (verbatim): "Both options are perfectly valid, and it comes down to personal taste."
- `!!` is NOT one of Comeau's listed fixes (only `> 0` and ternary). `&&` with a real boolean is endorsed.

---

## 4a. Airbnb JavaScript Style Guide

- URL: https://github.com/airbnb/javascript
- Author: Airbnb
- Date: living doc

- 15.6 (verbatim): "Ternaries should not be nested and generally be single line expressions. eslint: `no-nested-ternary`". Example shows "bad" nested ternary, then "split into 2 separated ternary expressions" into a `const maybeNull = ...`, then "best" single-line `const foo = maybe1 > maybe2 ? 'bar' : maybeNull;`.
- 15.7 (verbatim): "Avoid unneeded ternary statements. eslint: `no-unneeded-ternary`". Bad: `a ? a : b`, `c ? true : false`, `c ? false : true`, `a != null ? a : b`. Good: `a || b`, `!!c`, `!c`, `a ?? b`.
- 7.2 IIFE (verbatim): "Wrap immediately invoked function expressions in parentheses. eslint: `wrap-iife`" and the Why: "An immediately invoked function expression is a single unit - wrapping both it, and its invocation parens, in parens, cleanly expresses this. Note that in a world with modules everywhere, you almost never need an IIFE."

## 4b. Airbnb React/JSX Style Guide

- URL: https://github.com/airbnb/javascript/tree/master/react
- Author: Airbnb
- Date: living doc

- Parentheses (verbatim): "Wrap JSX tags in parentheses when they span more than one line. eslint: `react/jsx-wrap-multilines`". Single-line exception: `const body = <div>hello</div>; return <MyComponent>{body}</MyComponent>;`.
- Alignment section shows `&&` and ternary as accepted forms; only formatting is prescribed. Good: `{showButton && (<Button />)}`, `{showButton && <Button />}`, `{someConditional ? (<Foo />) : (<Foo ... />)}`. Bad: `{showButton &&\n <Button />\n}` (no parens around multiline JSX).
- No rule on nested ternaries in JSX, `switch`, map lookup, extract-a-component, or IIFE in JSX. The JS guide's 15.6 applies by inheritance.

## 4c. ESLint core `no-nested-ternary`

- URL: https://eslint.org/docs/latest/rules/no-nested-ternary
- Author: ESLint
- Description (verbatim): "Disallow nested ternary expressions" — "Nesting ternary expressions can make code more difficult to understand."
- Not in `eslint:recommended`. Marked "Frozen". No options.
- Correct example replaces the nesting with `if / else if / else` assigning to a `let`.

## 4d. `unicorn/no-nested-ternary`

- URL: https://github.com/sindresorhus/eslint-plugin-unicorn/blob/main/docs/rules/no-nested-ternary.md
- Author: Sindre Sorhus
- Verbatim: "Improved version of the `no-nested-ternary` ESLint rule, which allows cases where the nested ternary is only one level and wrapped in parens."
- Rationale (verbatim): "Unparenthesized or deeply nested ternaries make readers track multiple conditions and branches at once, so this rule permits only clearly parenthesized single-level nesting."
- Fail: `i > 5 ? i < 100 ? true : false : true`. Pass: `i > 5 ? (i < 100 ? true : false) : true`.
- In unicorn `recommended`; off in `unopinionated`.

## 4e. `react/jsx-no-leaked-render` (eslint-plugin-react)

- URL: https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/jsx-no-leaked-render.md
- Author: jsx-eslint maintainers
- Description (verbatim): "Disallow problematic leaked values from being rendered." Fixable.
- Rationale (verbatim): "Using the `&&` operator to render some element conditionally in JSX can cause unexpected values being rendered, or even crashing the rendering." "In React, you might end up rendering unexpected values like `0` or `NaN`. In React Native, your render method will even crash if you render these values".
- Fixes (verbatim): "coercing the conditional to a boolean: `{!!someValue && <Something />}`" or "transforming the binary expression into a ternary expression which returns `null` for falsy values: `{someValue ? <Something /> : null}`".
- Options: `validStrategies` array, default `["ternary", "coerce"]`. `ternary` autofixes `{count && title}` to `{count ? title : null}`; `coerce` autofixes to `{!!count && title}`. `ignoreAttributes` (default `false`) skips non-children attributes.
- Correct code includes `{elements.length > 0 && <List />}` and `{!!elements.length && <List />}`.
- NOT in `plugin:react/recommended` (README table shows only the fixable icon).

---

## 5. IIFE inside JSX `{(() => { ... })()}`

Checked: react.dev (no hit for "IIFE"/"immediately invoked"), Kent (no hit), Comeau (no hit), eslint-plugin-react README (no rule mentions IIFE), Airbnb React guide (no hit), Airbnb JS 7.2 (generic: "in a world with modules everywhere, you almost never need an IIFE"; not about JSX), TypeScript React cheatsheet (no hit).

Nearest named-author mention:

- Ben Ilegbodu, "Conditional rendering in React", https://www.benmvp.com/blog/conditional-rendering-react/, 2021-05-09. Lists IIFE only as a curiosity: "I've never in my 6+ years of writing React code ever done this. Or even thought to do it really. But I recently saw it and had to include it just for funsies." His recommendation: single condition → `&&`; two branches → ternary; multiple → element variables.

Verdict: no primary source recommends or bans IIFE in JSX; only community posts (Medium/dev.to) advocate it.

---

## 6. Delta check — agent-skill repos

### 6a. vercel-labs/agent-skills — react-best-practices

- URL: https://github.com/vercel-labs/agent-skills/blob/main/skills/react-best-practices/rules/rendering-conditional-render.md
- Rule `rendering-conditional-render` — title "Use Explicit Conditional Rendering", impact LOW, "prevents rendering 0 or NaN". Verbatim: "Use explicit ternary operators (`? :`) instead of `&&` for conditional rendering when the condition can be `0`, `NaN`, or other falsy values that render." Incorrect: `{count && <span className="badge">{count}</span>}`. Correct: `{count > 0 ? <span className="badge">{count}</span> : null}`.
- Scoped to falsy-leak only. Does not ban `&&` with booleans (their own `rendering-hoist-jsx` example uses `{loading && <LoadingSkeleton />}`).
- Rule `js-early-exit` / AGENTS.md 7.9 "Early Return from Functions" — impact LOW-MEDIUM, "Return early when result is determined to skip unnecessary processing." About plain functions/loops, not component guard clauses.
- Rule `rerender-memo` / 5.6 "Extract to Memoized Components" — "Extract expensive work into memoized components to enable early returns before computation." Performance-motivated extraction only.
- Rule `rerender-no-inline-components` — do not define components inside components (related to "nested render functions", not conditional shape).
- No rule on nested ternary, `switch`, map lookup, IIFE, variable-before-JSX, or hooks-before-early-return.

### 6b. vercel-labs/agent-skills — composition-patterns

- URL: https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns/rules
- Rule ids and one-line meaning:
  - `architecture-avoid-boolean-props` (1.1, CRITICAL) — "Don't add boolean props like `isThread`, `isEditing`, `isDMThread` to customize component behavior. Each boolean doubles possible states and creates unmaintainable conditional logic. Use composition instead." Incorrect example is a component whose body is nested ternaries (`isDMThread ? ... : isThread ? ... : null`). Correct: "composition eliminates conditionals". FLAG: variants + conditional branches inside a component.
  - `architecture-compound-components` (1.2, HIGH) — "Structure complex components as compound components with a shared context." Incorrect example uses `{showAttachments && <Attachments />}` flags; closing line: "Consumers explicitly compose exactly what they need. No hidden conditionals." FLAG: compound components.
  - `state-decouple-implementation` (2.1) — provider is the only place that knows how state is managed.
  - `state-context-interface` (2.2) — generic `{state, actions, meta}` context contract.
  - `state-lift-state` (2.3) — lift state into provider components.
  - `patterns-explicit-variants` (3.1, MEDIUM) — "Instead of one component with many boolean props, create explicit variant components. Each variant composes the pieces it needs." Subtitle: "self-documenting code, no hidden conditionals". FLAG: variants.
  - `patterns-children-over-render-props` (3.2) — "Use `children` for composition instead of `renderX` props."
  - `react19-no-forwardref` (4.1) — `ref` as prop, `use()` over `useContext()`.
- Not covered: `&&` vs ternary shape, nested ternary lint, early return, `switch`, map lookup, IIFE.

### 6c. sergiodxa/agent-skills — frontend-react-best-practices

- URL: https://github.com/sergiodxa/agent-skills/blob/main/skills/frontend-react-best-practices/SKILL.md
- `rendering-conditional-render` — "Use ternary, not && for conditionals with numbers." Bad: `count && <Badge>{count}</Badge>` ("renders '0' when count is 0"). Good: `count > 0 ? <Badge>{count}</Badge> : null`. Same scope as Vercel: numbers only.
- `rendering-hoist-jsx` — "Extract static JSX outside components."
- `rerender-memo` — "Extract expensive work into memoized components." (perf)
- `composition-avoid-boolean-props` — "Don't add boolean props to customize behavior. Use composition instead." Good: `<ThreadComposer />`, `<EditComposer />` "explicit variants".
- `composition-compound-components` — "Structure complex components as compound components with shared context."
- `composition-state-provider` — "Lift state into provider components for cross-component access."
- `composition-explicit-variants` — "Create explicit variant components instead of prop combinations."
- `composition-children-over-render-props` — "Prefer children for composition. Use render props only when passing data back."
- `composition-avoid-overabstraction` — "Avoid rigid configuration props; prefer composable children APIs."
- `composition-typescript-namespaces` — types-only namespaces; not about rendering.
- No rule on early return, nested ternary, `switch`, map lookup, IIFE, variable-before-JSX, extract-for-readability, or hooks-before-return.

---

## 7. Object / map lookup instead of `switch` / nested ternary

Checked Kent (site search: no hit), Comeau (no hit), react.dev conditional-rendering (not shown), Airbnb JS/React (no rule), Vercel and sergiodxa skills (no rule). The pattern `STATUS_VIEW[status]` appears only in Medium/dev.to posts.

Verdict: no primary source. Closest primary neighbours: Airbnb 15.6 pushes multi-branch out of ternaries into a `const`; ESLint `no-nested-ternary` "correct" example uses `if / else if / else` into a `let`; Vercel `js-index-maps` / `js-set-map-lookups` are about O(1) data lookup, not render branching.

---

## 8. Supporting (secondary, named author): Alex Kondov — "Tao of React"

- URL: https://alexkondov.com/tao-of-react/
- Author: Alex Kondov (author of the book "Tao of React")
- Date: 2021-01-18
- "Conditional Rendering" (verbatim): "In some situations using short circuit operators for conditional rendering may backfire and you may end up with an unwanted 0 in your UI. To avoid this default to using ternary operators." And: "Ternaries are more verbose but there is no chance to get it wrong. Plus, adding the alternative condition is less of a change."
- "Avoid Nested Ternary Operators" (verbatim): "Ternary operators become hard to read after the first level. Even if they seem to save space at the time, it's better to be explicit and obvious in your intentions." Good example: extract `CallToActionWidget` with guard-clause early returns (`if (subscribed) return ...; if (registered) return ...; return ...`).
- "Avoid Nested Render Functions" (verbatim): "When you need to extract markup from a component or logic, don't put it in a function living in the same component... Move it in its own component, name it and rely on props instead of a closure."
- Kondov is opinion, not official; listed for the nested-ternary → extract-with-early-returns pattern that the official docs only imply.
