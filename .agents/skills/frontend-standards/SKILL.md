---
name: frontend-standards
description: Opinionated house rules for React, Next.js, and Tailwind code in a feature-based folder structure, covering what the Vercel skills leave out. Colocation — where a component, type, helper, or test lives, and when to extract or share it. Tailwind — when a class list becomes a map, cva, a token, or a component. Use when writing or reviewing frontend components, hooks, feature folders, or className props, and when deciding whether to extract, promote, or share code or styles.
---

# Frontend Standards

House rules that complement the Vercel skills. Each rule names the Vercel rule IDs it touches, with a one-line meaning, so it reads without them installed.

Precedence, in order:

1. The repository's own standards (`AGENTS.md`, `CLAUDE.md`, and whatever they point at). These rules fill the room the repository leaves. On a conflict, follow the repository and say the conflict exists.
2. Vercel `react-best-practices` for measured performance, `composition-patterns` for component APIs, the `nextjs` skill for framework mechanics, the `shadcn` skill for component-library conventions. Read the rule the task touches, not the whole skill.
3. These rules for default code shape.

## Rules

| Rule | One line |
|---|---|
| [colocation](rules/colocation.md) | Narrowest scope that matches ownership today. Extract on a named problem, share on shared ownership. |
| [tailwind](rules/tailwind.md) | Classes stay on the element. Repetition is evidence, not a command. `cn` → map → `cva` by count. |
