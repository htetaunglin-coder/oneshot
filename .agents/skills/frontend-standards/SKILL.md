---
name: frontend-standards
description: Opinionated house rules for React, Next.js, and Tailwind code that the Vercel skills leave out. Currently colocation — where a component, type, helper, or test lives, and when to extract or share it. Use when writing or reviewing frontend components, hooks, or feature folders, and when deciding whether to extract, promote, or share code.
---

# Frontend Standards

House rules that complement the Vercel skills. Each rule names the Vercel rule IDs it touches, with a one-line meaning, so it reads without them installed.

Precedence, in order:

1. The repository's own standards (`AGENTS.md`, `CLAUDE.md`, `agent_docs/`). These rules fill the room the repository leaves. On a conflict, follow the repository and say the conflict exists.
2. Vercel `react-best-practices` for measured performance, `composition-patterns` for component APIs, the `nextjs` skill for framework mechanics. Read the rule the task touches, not the whole skill.
3. These rules for default code shape.

## Rules

| Rule | One line |
|---|---|
| [colocation](rules/colocation.md) | Narrowest scope that matches ownership today. Extract on a named problem, share on shared ownership. |
