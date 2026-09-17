---
title: File order, top-down by importance
impact: MEDIUM
tags: file-order, declarations, hoisting, constants, types, exports
---

# File order: top-down by importance

A file reads like a newspaper. The headline is the export the file is named after; the detail that serves it follows, most important first. A reader who stops after the first screen knows what the file is for.

## The order

1. Directive (`"use client"`), then imports.
2. Constants, then types, that the main export or two or more parts of the file use. Inside the section, in order of first use.
3. Top-level expressions that run at import time and that other top-level code reads: `cva(...)`, `createContext(...)`. One with a single reader follows the ownership rule below and sits above that reader, still below every `const` it reads.
4. The main export: the component, hook, or function the file is named after.
5. Sub-components, in the order the main export renders them.
6. Helpers, in the order a reader meets their first call, reading the file top-down. Slot order wins over call order: a helper the main export calls before it renders a sub-component still sits in slot 6.

A file with several peer exports, such as a compound component (`Tabs`, `TabsList`, `TabsTrigger`), has no single headline: the root comes first, then its parts in the order a consumer nests them.

Nothing marks the sections; no divider comments (see [comments](comments.md)).

```tsx
"use client";

import { useState } from "react";
import { cn } from "@/lib/utils";

const MAX_VISIBLE_TAGS = 3;

type TagListProps = { tags: string[]; className?: string };

export function TagList({ tags, className }: TagListProps) {
  const [expanded, setExpanded] = useState(false);
  const visible = expanded ? tags : tags.slice(0, MAX_VISIBLE_TAGS);
  const hidden = tags.length - visible.length;
  return (
    <ul className={cn("flex flex-wrap gap-1", className)}>
      {visible.map((tag) => <Tag key={tag} label={tag} />)}
      {hidden > 0 ? <MoreButton count={hidden} onClick={() => setExpanded(true)} /> : null}
    </ul>
  );
}

const MAX_LABEL_CHARS = 12;

function Tag({ label }: { label: string }) {
  return <li className="rounded-full border px-2 text-xs">{label.slice(0, MAX_LABEL_CHARS)}</li>;
}

function MoreButton({ count, onClick }: { count: number; onClick: () => void }) {
  return <button onClick={onClick}>+{count} more</button>;
}
```

## Ownership decides the slot

A constant or a type goes to the top section when the main export reads it, or when two or more parts of the file read it. A constant or a type that one sub-component or one helper reads sits directly above that part, as `MAX_LABEL_CHARS` does above `Tag`. When a second reader appears, the declaration moves up to the top section in the same change. The colocation rule decides when it leaves the file (see [colocation](colocation.md)).

An existing file that breaks this order is not a precedent. A new declaration takes its slot by this rule; a new component, hook, or helper is a `function` declaration. An existing declaration moves, or converts from an arrow `const` to a `function`, when the change touches it or when the task is the file's structure; otherwise it stays and the reply names it, the same focused-change rule as the colocation file size limit.

## Declarations that let the order hold

Components, hooks, and helpers are `function` declarations. A `function` declaration is hoisted with its body, so a caller sits above its callee and the file stays top-down. A `const` arrow function is in the temporal dead zone until its line runs, so it cannot sit below a reader that runs at import time.

Function bodies read at call time. A `function` component may read a `const` declared below it, because the component runs at render, after the module has evaluated. Top-level expressions run at import time. `const styles = cva(...)`, `const Ctx = createContext(...)`, `export default memo(Component)`, and any wrapper call sit below every `const` and `class` they read. When such a wrapper is needed, the wrapped component is still a `function` declaration above it, and the wrapper is the last line of that component's slot.

Types and interfaces are erased and may sit anywhere; they follow the ownership rule. An `enum` emits code and follows the `const` rule.

## Exports

An export is written at the declaration: `export function`, `export const`, `export type`. The file carries no export list at the bottom. The exception is `src/components/ui`: a file there keeps the registry's shape, its bottom export list included, so a later `shadcn diff` or reinstall reads cleanly; a new export there joins the list, and its declaration takes its slot by nesting order. A default export exists only where the framework reads it: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, and the other App Router special files. `route.ts` exports named HTTP methods and has no default. The default export is the main export in slot 4. Framework config exports, the route segment config (`dynamic`, `revalidate`) and every `generate*` export (`generateMetadata`, `generateStaticParams`), and `metadata` itself, are configuration and sit in slot 2.

Static values hoisted out of a component follow the Vercel rules: `rendering-hoist-jsx` (static elements to a module constant), `js-hoist-regexp` (regex literal at module scope), `rerender-memo-with-default-value` (a non-primitive default prop to a constant). This rule only says where they sit: slot 2 or above their one reader.

## Inside a component

Hooks, then derived values, then handlers, then early returns, then JSX. A value is declared on the line before its first use, not at the top of the function.

## Tooling

The order relies on hoisting, so the linter guards the one runtime hazard, a `const` or `class` read at import time before its line, and allows both a `function` declaration below its caller and a function body that reads a `const` declared below it: ESLint `no-use-before-define` with `{ functions: false, classes: true, variables: false, allowNamedExports: false }` (`variables: false` ignores a reference from an inner function scope and still reports one in the same scope), or `@typescript-eslint/no-use-before-define` with the same plus `typedefs: false, ignoreTypeReferences: true`. Biome `correctness/noInvalidUseBeforeDeclaration` has the same behavior with no options. Import order belongs to the formatter or import sorter, not to this rule. `perfectionist/sort-modules` orders by kind and name and conflicts with the ownership slot, so it stays off. Checked 2026-09 against Next 16.2, ESLint 9.39, Biome 2.

## Sources

Robert C. Martin: Clean Code, ch. 3 The Stepdown Rule, ch. 5 Vertical Formatting. Steve McConnell: Code Complete, ch. 31.8 Laying Out Classes and Files. Google TypeScript Style Guide: Source file structure, Exports, Function declarations. Airbnb JavaScript Style Guide: Hoisting, Functions (the define-before-use position, resolved here by `function` declarations). MDN: Hoisting, Temporal dead zone, Modules. ESLint and typescript-eslint: `no-use-before-define`. Kent C. Dodds: Colocation. Dan Abramov: A Complete Guide to useEffect, hoisting functions that use no component scope.
