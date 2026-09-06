---
title: Colocate by ownership
impact: HIGH
tags: abstraction, colocation, components, types, business-logic, tests
---

# Colocate by ownership

Code lives in the narrowest scope that matches who owns it and who uses it today. It moves when a concrete problem is named, never because a count was reached.

## Scopes

1. **Inline** in the expression that uses it.
2. **Same file** as its single consumer: a named helper, sub-component, `Props` type, or constant. Naming a thing does not require a new file.
3. **Feature folder** (`features/<x>/…`) when the code belongs to one feature but not to one file.
4. **Shared** (`src/lib`, `src/hooks`, `src/components/ui`, `src/constants` for app-wide config such as route maps and flags, `src/test` for test utilities) when the code is one concept that several features must keep consistent.

These are scopes to choose from, not steps to walk. Code whose ownership is already shared goes to the shared scope directly. Ownership is who changes the code, not who reads it: many consumers do not make a concept shared.

## Layers

Imports flow one way: shared → features → app. Shared imports only shared. A feature imports shared and itself. `src/app` imports both and owns nothing feature-specific: a route file, a route handler, `middleware.ts`, and the root layout are glue that composes features. Features never import each other. Enforce this with a linter (`eslint-plugin-boundaries` with three element types and one allow-list), and keep this section as the reason.

When a feature or a shared component needs another feature's state, the app layer composes them: it mounts the provider and passes data or a component in as props or `children`. Vercel `patterns-children-over-render-props` (pass markup as children) carries the form. A file that reaches into several features is split per feature or inverted the same way.

## Two decisions, two questions

**Extract** (give it a name, maybe a file): write the problem the separation solves in one sentence. "The parent stops reading top to bottom." "This function needs a test the component cannot host." "The editor is 300 lines of cohesive behavior and drowns the page." The sentence must pass one test: a typical change to this code is now understandable with less context, not more. A real sentence earns the extraction, and one consumer is enough. "It might be reused" is not a problem yet, so the code stays. The same test runs in reverse: several short files that every change has to trace together are consolidated into the file that owns the state. A file path the task itself mandates wins over this rule.

**Share** (move to a scope more than one feature imports): the consumers rely on one concept that must change together. A pricing policy, an auth check, a date format the whole site shows. Two features that need the same policy get one owner in the shared scope, even at two. Two helpers that merely look alike and have different reasons to change stay separate.

Shared placement is decided by ownership, so a repository that sets a count threshold for `src/lib` may disagree with this rule. Follow the repository and say that the conflict exists.

## File size limit

1. A hand-written source file has fewer than 1,000 lines. A change to a file under the limit keeps it under. Generated files and static-data files are exempt. A static-data file holds data, not behavior: a JSON-like table, an icon path list. A hand-written component is behavior, whatever it renders.
2. When a file must shrink, one cohesive responsibility moves to a feature-local file. One consumer is enough.
3. A file already over the limit may take a focused change without restructuring, and the size is reported. When the task touches a block that the two questions above would extract, the extraction happens first and the change lands in the new file.
4. When meeting the limit would force unrelated restructuring or a worse separation, the conflict is surfaced instead of the split.

The number makes the size check consistent. Choosing what to extract still follows the two questions above.

## Options that branch on the caller

An option is fine when it describes the abstraction's own behavior. `variant="destructive"` on a button is a presentation choice the button owns. An option is evidence of a wrong abstraction when it makes shared code branch on who is calling.

**Incorrect (the formatter knows about a page):**

```tsx
function formatPrice(cents: number, { isCheckoutPage = false } = {}) {
  const value = (cents / 100).toFixed(2);
  return isCheckoutPage ? `Total: $${value}` : `$${value}`;
}
```

**Correct (the caller supplies the varying part):**

```tsx
function formatPrice(cents: number) {
  return `$${(cents / 100).toFixed(2)}`;
}
// checkout-summary.tsx
<p>Total: {formatPrice(total)}</p>
```

When an option like that appears, reassess whether the callers still share one concept. Separate diverging behavior into its own function or component, or let the caller pass the varying part as an argument, a callback, or `children`. Vercel `architecture-avoid-boolean-props` (compose instead of adding mode flags) and `patterns-children-over-render-props` (pass markup as children) carry the component form.

## Per kind

**Components.** A sub-component used once lives in the same file when it makes the parent read top to bottom. A cohesive block that is large enough to hide the page around it gets a feature-local file, still with one consumer. Either way it is defined at module scope, never inside the parent body (Vercel `rerender-no-inline-components`: an inner definition is a new component type on every render and remounts its subtree). A `"use client"` leaf inside a server component is a runtime boundary and takes its own file.

**Types.** A type belongs to the file that is its source of truth. `Props` belongs to its component. An API contract belongs to the data module that fetches it, and UI code imports it from there with `import type`. Number of readers does not decide ownership, and no parallel definition of the same shape is written to satisfy a folder rule. A `types.ts` exists only for a shape with no owning module.

**Business logic.** Mapping, validation, pricing, and permission calculations are pure functions. They start in the file that calls them and move by the two questions above: to `features/<x>/lib` when separation makes a typical change cheaper to understand, and to the shared scope when the concept is shared. A run of pure functions with a different responsibility from the component around them is a named problem on its own. A derived one-liner (`const isEmpty = items.length === 0`) stays in the component. A pure permission calculation in the UI is a display decision only. Enforcement happens where the framework and repository put authoritative checks.

Domain state and its provider stay with the feature that owns them. Consumers spreading to other features or to shared components does not move it; the app layer composes them (see Layers).

Data access and mutations colocate inside their runtime boundary. A server-only fetcher can live in `features/<x>/api` and be called from a server component. Follow the framework and repository rules for where server code, client code, and mutation entrypoints go. Placement guidance never invents a route handler.

**Tests.** A test file sits beside the module it tests, in a repository that has a test runner. Inside a test, the same rule applies: setup stays next to the assertion that depends on it, so two similar tests show their difference without scrolling. Shared state built in `beforeEach` hides that difference.

## Tailwind

Class strings stay inline on the element. A repeated markup pattern becomes a small component with its classes inline. A hoisted className constant is not colocation, it is a name for a string, and the string reads better on the element. Vercel `rendering-hoist-jsx` (static JSX to a module constant for reuse) is a different case and stays allowed.

## Exceptions that skip the scopes

- Framework-mandated files: route files, `instrumentation.ts`, config. They live where the framework looks.
- Design-system primitives in `src/components/ui`. Shared by definition.
- End-to-end tests at the project root. They outlive any internal layout.

## Sources

Kent C. Dodds: AHA Programming, AHA Testing, Colocation, Inversion of Control, When to break up a component. Sandi Metz: The Wrong Abstraction.
