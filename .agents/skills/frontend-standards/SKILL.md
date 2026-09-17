---
name: frontend-standards
description: Opinionated house rules for React, Next.js, and Tailwind code in a feature-based folder structure, covering what the Vercel skills leave out. Colocation — where a component, type, helper, or test lives, and when to extract or share it. Tailwind — when a class list becomes a map, cva, a token, or a component. Comments — what a comment carries that code cannot, when an export gets JSDoc, the TODO form, and simple-English wording. File order — the slot for each constant, type, component, sub-component, and helper inside a file. Magic literals — when a number or string takes a name, how it is named, and `as const` unions over enums. Conditional rendering — the ladder from early return to sub-component to IIFE, and when a ternary nests. Use when writing or reviewing frontend components, hooks, feature folders, className props, comments, JSDoc, the layout of a file, a literal number or string, or a conditional branch in JSX, and when deciding whether to extract, promote, or share code or styles.
---

# Frontend Standards

House rules that complement the Vercel skills. Each rule names the Vercel rule IDs it touches, with a one-line meaning, so it reads without them installed.

Precedence, in order:

1. The repository's own standards (`AGENTS.md`, `CLAUDE.md`, and whatever they point at). These rules fill the room the repository leaves. On a conflict with the repository or with a Vercel rule, follow the higher item and say the conflict exists.
2. Vercel `react-best-practices` for measured performance, `composition-patterns` for component APIs, the `nextjs` skill for framework mechanics, the `shadcn` skill for component-library conventions. Read the rule the task touches, not the whole skill.
3. These rules for default code shape.

## Rules

| Rule | One line |
|---|---|
| [colocation](rules/colocation.md) | Narrowest scope that matches ownership today. Extract on a named problem, share on shared ownership. |
| [tailwind](rules/tailwind.md) | Classes stay on the element. Repetition is evidence, not a command. `cn` → map → `cva` by count. |
| [comments](rules/comments.md) | Code says the what and the how first. A comment is the residue: why, why-not, warning, contract, coupling, reference. |
| [file-order](rules/file-order.md) | Newspaper order: imports, shared constants and types, main export, sub-components, helpers. A part's own constant sits above it. `function` declarations keep it top-down. |
| [magic-literals](rules/magic-literals.md) | A literal stays inline when its meaning is on the line. It takes a name when a reader must decode it or a second reader appears. `as const` unions, no `enum`. |
| [conditional-render](rules/conditional-render.md) | Lowest rung that holds the branch: early return → ternary → map → sub-component → variable before JSX → IIFE last. A ternary never nests. |
