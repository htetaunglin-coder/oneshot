---
title: Conditional rendering, the ladder from early return to IIFE
impact: MEDIUM
tags: conditional-rendering, early-return, ternary, sub-component, iife, jsx
---

# Conditional rendering: the ladder

A branch in JSX takes the first row that holds it. Each later row trades locality for room; the IIFE is the last row and is reached only when every row before it has failed.

## The ladder

| Branch | Shape |
|---|---|
| The whole component changes: loading, error, empty, no access | Early return, after the last hook. When a hook needs a value the guard produces, `session.user.id`, the hook takes the optional value and disables itself, `enabled: session != null`, or the guarded part becomes a sub-component that receives the value |
| One element in or out | `&&` when the left side is a `boolean` or a comparison, `isOpen && <Menu />`; `cond ? <X /> : null` otherwise, and in every case in a repository that runs `jsx-no-leaked-render` (see Guards). A callback passed to the element, `<Menu onClose={() => setOpen(false)} />`, keeps it on this row |
| One of two elements | A ternary, on one line when it fits; otherwise each branch in parentheses |
| One of three or more, keyed by a value | A map, `VIEW[status]`, when every branch takes the same props; a sub-component with early returns when the branches take different props |
| One element chosen by several flags with a precedence order | A variable before the JSX, assigned by an `if` chain in precedence order, or a sub-component with early returns in that order when the chain has four or more branches |
| A branch with state, hooks, or handlers of its own, declared inside the branch rather than passed to it | A sub-component, a `function` declaration at module level |
| Several elements decided by several conditions in one place | A variable assigned before the JSX, `let banner = null; if (...) banner = <A />;` |
| Statements one branch needs, where extraction fails the context test and the variable would sit far from its use | An IIFE, `{(() => { ... })()}`, the last row. One branch, one `if` at most; a second branch or a `switch` goes to a helper or a sub-component |

Check the rows from the top; the first row whose situation matches is the shape. An existing block that sits on a later row than its branches need, an IIFE with three flag branches, is not a precedent, whatever a comment beside it says; a change that touches the block moves it to its row in the same change. A ternary is never nested: the inner branch becomes a variable, a map entry, or a sub-component. A nested condition inside a prop, `className={a ? (b ? x : y) : z}`, is the same case: the value is assigned above the JSX by an `if` chain, never by a ternary chain; a class string chosen by precedence is a variant map keyed by a state the `if` chain picks (see [tailwind](tailwind.md)).

```tsx
// Incorrect: three branches in one nested ternary
return isLoading ? <Spinner /> : error ? <ErrorView error={error} /> : <List items={items} />;

// Correct: whole-component branches return early, after the hooks
const { data, error, isLoading } = useItems();
if (isLoading) return <Spinner />;
if (error) return <ErrorView error={error} />;
return <List items={data} />;
```

## Guards

A number or a string on the left of `&&` renders `0` or the string itself; Vercel `rendering-conditional-render` covers that case and its fix, the ternary. A `!!` coercion is not written, in JSX or in a hook option; the comparison says what is tested, `count > 0`, `session != null`. A repository that runs `react/jsx-no-leaked-render` flags a bare boolean variable on the left, `isOpen && <Menu />`, so there every one-element guard is the ternary with `null`; when the lint config is not visible, the default holds.

`return null` from a component is for a component that decides its own presence from a source the parent does not hold, a portal target or a feature flag read inside, and for the end of a sub-component that owns a precedence chain, where `null` is the last branch; a parent that holds the value the condition reads decides presence itself and renders nothing instead of mounting a component that returns nothing.

## Extraction

A branch becomes a sub-component when it has state, hooks, or handlers of its own, or when the markup inside it nests a second condition that chooses an element; a ternary that picks a string, `{enforced ? " (enforced)" : null}`, is text, not a branch, and does not count. A hook moves into the sub-component when the sub-component is its only reader; otherwise the value is a prop. The React docs' line: "too much nested conditional markup, consider extracting child components." The sub-component is a `function` declaration below the parent (see [file-order](file-order.md)); a component defined inside another component is Vercel `rerender-no-inline-components`, remounted on every render.

Extraction on a count of lines alone fails the colocation context test (see [colocation](colocation.md)). A branch that is only long, with no state and no nested condition, is a variable before the JSX.

A branch chain that reads a mode prop, `variant === "compact" ? … : variant === "card" ? … : …`, is a component API problem before it is a rendering one: Vercel `patterns-explicit-variants` (one component per variant) and `architecture-avoid-boolean-props` (a mode prop that branches inside) decide it.

## The map

Content keyed by a value is a lookup, not a chain. The map is a module constant when every branch is static JSX (Vercel `rendering-hoist-jsx`), or a record of components when a branch takes props:

```tsx
const STATUS_VIEW: Record<Status, ComponentType<{ order: Order }>> = {
  pending: PendingView,
  shipped: ShippedView,
  delivered: DeliveredView,
};
const View = STATUS_VIEW[order.status];
return <View order={order} />;
```

A `switch` that returns JSX lives in a helper function or a sub-component, never inline in the render.

## The IIFE

An IIFE is a function body with no name, placed where a value is expected. It is reached when a branch needs statements, the block fails the context test for extraction, and a variable before the JSX would sit a screen away from the element it fills. When the IIFE is written, it holds one branch and nothing else. When it grows a second condition or a hook, it is a sub-component.

## Tooling

`react/jsx-no-leaked-render` is off in `plugin:react/recommended`; turned on with `validStrategies: ["ternary", "coerce"]`, it accepts comparisons and `!!` and flags a bare boolean variable on the left of `&&` (see Guards). ESLint `no-nested-ternary` holds the nesting rule. Checked 2026-09 against eslint-plugin-react 7.37, ESLint 9.39.

## Sources

React docs: Conditional Rendering; Rules of Hooks. Kent C. Dodds: Use ternaries rather than && in JSX; When to break up a component into multiple components. Josh Comeau: Common Beginner Mistakes with React. Airbnb JavaScript Style Guide 15.6, 15.7; Airbnb React Style Guide, parentheses. ESLint `no-nested-ternary`; eslint-plugin-react `jsx-no-leaked-render`. Vercel `react-best-practices`: `rendering-conditional-render`, `rerender-no-inline-components`, `rendering-hoist-jsx`; `composition-patterns`: `patterns-explicit-variants`, `architecture-avoid-boolean-props`. The IIFE and map rows are house opinion; no primary source covers them.
