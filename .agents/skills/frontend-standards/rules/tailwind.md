---
title: Tailwind classes, where they live and how they compose
impact: HIGH
tags: tailwind, classes, cn, cva, css-variables, spacing
---

# Tailwind classes: where they live and how they compose

Classes stay on the element they style. Everything below decides the exceptions.

## Repetition is evidence

The same class list in two places is not a reason to extract. Tailwind's own guidance: edit nearby duplicates together, render repeated data with a loop, and reach for a component only for reusable UI. The colocation test applies unchanged: the instances become one component when they must change together and own one UI responsibility. A responsibility is structure, behavior, or semantics the consumers share. A look is none of those, so a shared class list is never a responsibility on its own, however many files carry it or how often they were edited together. A wrapper whose only prop is `className` is a hoisted string in JSX syntax. A toolbar and a price row that both say `flex items-center justify-between` stay two elements, in one file or across three features. A `FormField` that owns label, help text, and error layout is one component.

A hoisted string is a name for a string and reads worse than the classes on the element:

```tsx
// Incorrect
const CARD = "rounded-md border p-4";
<div className={CARD} />

// Correct
<div className="rounded-md border p-4" />
```

`@apply` in a stylesheet or a CSS module is the same hoisted string under a selector; the classes move onto the element. Vercel `rendering-hoist-jsx` (hoist static *elements* to a module constant for reuse) is a different case and stays allowed.

## The ladder

Pick the lowest rung that holds the decision.

| Situation | Tool |
|---|---|
| Static classes | `className="..."` |
| One on/off condition, or merging a `className` prop | `cn()` |
| One axis, any count; or two axes with four or fewer named options in total | Variant map, one per axis |
| Two or more axes with five or more named options in total, or a compound rule where two axes together change a class, or the variant type is exported | `cva` |

Rows are checked from the bottom: an exported variant type or a compound rule goes to `cva` whatever the count. The five-option line is the house gate, chosen for determinism.

A variant map is an object of complete class strings keyed by state. It encodes a decision; a lone constant does not.

```tsx
const tone = {
  info: "border-blue-200 bg-blue-50 text-blue-900",
  danger: "border-red-200 bg-red-50 text-red-900",
};
<div className={cn("rounded-md border p-4", tone[kind])} />
```

Every class name exists whole in source. `bg-${color}-500` is the one construction Tailwind cannot scan; a map or `cva` replaces it.

## `cn()` order

Base classes, then conditionals, then the consumer's `className` last. With the project's `tailwind-merge` helper, later classes win within recognized conflict groups; arbitrary properties against standard utilities are not resolved. Branches are mutually exclusive rather than relying on merge to resolve a conflict.

```tsx
className={cn("inline-flex rounded-md px-3", isActive ? "bg-accent" : "bg-muted", className)}
```

The `shadcn` skill carries the rest: prefer an existing variant over a `className` override, semantic tokens over raw colors, and the merge configuration `cn()` needs when the theme adds custom utilities.

## Three kinds of variable

A variable is declared where its value is decided. Readers consume it and never set it.

- **Runtime value**: a plain `style` prop for an unconditional property on one element, `style={{ height: `${pct}%` }}`. It becomes a variable when a state, breakpoint, or pseudo-element must read it, `style={{ "--hover-color": c }}` with `hover:bg-(--hover-color)`, or when a second element reads it, in which case the variable sits on the nearest shared ancestor and each reader consumes it, `w-(--progress)` or `[width:var(--progress)]`.
- **Layout contract**: one element's geometry that other elements depend on, such as header height, prompt-area height, content max width. The variable goes on the nearest ancestor that contains everyone who needs it, `style={{ "--header-height": "3.5rem" }}`, or as classes when the value changes per breakpoint, `[--header-height:3.5rem] md:[--header-height:4rem]`, or in `:root` when that ancestor is the page. The element it describes reads it, `h-(--header-height)`, and every dependent reads the same variable, `pt-(--header-height)`, so the number exists once. A wrapping layout that changes the geometry overrides it on its own element, `{ "--header-height": "0px" }` when embedded. An element whose size is only known at runtime writes it to the owning ancestor's element through a ref, `layoutRef.current.style.setProperty("--prompt-area-height", `${h}px`)`, and removes it on unmount. `document.documentElement` is the target only when the page itself is the owner. This replaces prop drilling and context for a value that is geometry, not state.
- **Design token** the whole app shares: declare it in `@theme` so Tailwind generates the utility, then use the utility. A token exists when the value has shared ownership, the same test as any shared code. In a shadcn repo the token follows the `shadcn` skill's `:root` plus `@theme inline` form.

## Shared appearance without shared markup

A button look on a `<label>` or an `<a>` is one visual contract on different elements. When the look already has a `cva`, call it: `className={buttonVariants({ variant: "outline" })}`. When it has none, the classes are written on each element; the look becomes a contract when a component owns it. A wrapper component that exists only to hide classes is never the answer.

## Tooling

Class order belongs to the formatter: `prettier-plugin-tailwindcss` with `tailwindFunctions: ["cn", "cva"]`. The rule text carries no ordering convention.

## Sources

Tailwind docs: Styling with utility classes, Managing duplication; Theme variables; Detecting classes in source files. CVA docs: Variants. tailwind-merge: When and how to use it. shadcn skill rules: styling. Kent C. Dodds: AHA Programming. Sandi Metz: The Wrong Abstraction.
