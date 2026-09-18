---
title: Magic literals, when a number or string takes a name
impact: MEDIUM
tags: magic-numbers, constants, naming, enums, as-const, string-unions
---

# Magic literals: when a number or string takes a name

A literal stays inline when its meaning is on the line. It takes a name when a reader must decode it, or when a second reader appears.

## The gate

| Stays inline | Takes a name |
|---|---|
| `0`, `1`, and `-1` from `indexOf`: loop start, step, empty check, not found; an array index, `rows[0]`, `pair[1]`. This row wins over the duplicate row below | A value a reader must decode: `86400`, `0.0825`, `25_000`, `300_000` |
| A unit conversion or an identity in its formula: `hourlyRate * 8`, `radius * 2`, `cents / 100` | A limit or a rule the business owns: a page size, a retry count, an approval threshold. A rate, a fee, or a contract multiplier, `amount * 0.029 + 30`, `subtotal * 1.2` |
| A one-off duration whose units are on the line, `5 * 60 * 1000`, or that is one unit, `1000`, when the duration is a mechanism, not a rule; `300_000` is decoded and is written as units or named | The same literal, with the same meaning, on two lines, in one file or two. Same digits with a different meaning are two literals |
| A library constant, `Math.PI`, `Number.MAX_SAFE_INTEGER`; a protocol code the reader knows, `404`, `429`; a value the platform or a library defines, as an argument, `"en-US"`, `{ behavior: "smooth" }`, or in a comparison, `event.key === "Enter"`, `type === "checkbox"`, `NODE_ENV === "production"`; the empty string | A string the code defines and then compares, switches on, sends, or stores: a status, an event name, a storage key, a query param. A key segment read once inside a query factory stays inline; the factory is the name |
| A default used once, as a parameter or a `??` fallback, when the parameter name carries its meaning, `delay = 300`; when the code later compares against it, it is a sentinel and takes a name | A value another file reads. A default that is a limit enforced elsewhere too, a page size the server clamps, `pageSize = 25` |
| A fixture input in a test whose name carries the meaning; an expected output the reader cannot decode is written as the formula that produces it, `2.5 * RATE_PER_KG` | A number in JSX that is a design decision: `<Image width={640}>` when `640` is the card width elsewhere too. A duration that is a rule, a session warning at five minutes, is a business limit and takes a name |

The name says what the value means, `MAX_RETRIES`, and the unit when there is one, `TIMEOUT_MS`, `THRESHOLD_CENTS`; a name that restates the value, `THREE`, fails the gate.

```ts
// Incorrect: decoded by the reader, and the same 250 lives in the approval email
if (order.totalCents < 25_000) skipApproval();

// Correct
const APPROVAL_THRESHOLD_CENTS = 25_000;
if (order.totalCents < APPROVAL_THRESHOLD_CENTS) skipApproval();
```

A literal on a line the change touches takes its name in the same change. A literal that fails the gate elsewhere in the function the change edits is named too, since the function is the unit of the change; one elsewhere in the file stays and the reply names it, the same focused-change rule as the colocation file size limit.

## Naming

`CONSTANT_CASE` is for constant data at module level: a primitive, an array, a plain object, `PAGE_SIZE`, `STATUS_LABEL`, `SORT_OPTIONS`. A value produced by a call keeps its conventional `camelCase` name, `styles = cva(...)`, `ThemeContext = createContext(...)`, `dollarFormat = new Intl.NumberFormat(...)`. A `const` inside a function is `camelCase`, whatever it holds, except a component picked from a map, `const View = STATUS_VIEW[status]`, which is `PascalCase` so JSX reads it as a component. A module-level object that is mutated is not a constant and is `camelCase`. An existing file's `camelCase` constants are not a precedent, even when the request asks for consistency with the file: the new one is `CONSTANT_CASE`, the old ones convert when the change touches them, and otherwise the reply names them.

## Strings with a fixed set of values

A string that can take one of a fixed set of values is an `as const` array with a derived union, so the values exist once at runtime and once in the type:

```ts
const SORT_ORDERS = ["asc", "desc"] as const;
type SortOrder = (typeof SORT_ORDERS)[number];
```

A literal compared against a value typed by the union stays inline, `invoice.status === "paid"`, because the union is the name and catches the typo; a check against a subset of the union is a function, `isPaidOrSent(invoice)`. A request for an `enum` is met with this form and one line saying why. A set the code never iterates or validates is a plain union type. A subset of an existing union is typed against the parent, `PREMIUM_TIERS: readonly CustomerTier[]`, with no second union. A file that holds one concept, its array, its type, and its guard, `lib/invoice-status.ts`, is an owner, not a bag. `enum` is not used: it is not erasable syntax (TypeScript 5.8 `erasableSyntaxOnly`, Node type stripping), and the `as const` form covers every case.

A label keyed by a value is a map beside its reader, `STATUS_LABEL[status]`. A string that is rendered and also compared, a sentinel such as `"all"`, follows the compare: it takes a name.

## Where the constant lives

The file-order rule gives the slot: the top section when the main export or two parts read it, directly above its one reader otherwise (see [file-order](file-order.md)). The colocation rule decides when it leaves the file: shared ownership, never a count (see [colocation](colocation.md)). When two files in one feature read a constant, it lives in the file that enforces or sends the value, and the other file imports it; when no single file does, or the owner is a `"use server"` file, which exports only async functions, it lives in a feature `lib` module for that concept and both import it. An `as const` array is a runtime value, so it lives in that owning module and the type derives beside it; a `types.ts` holds no runtime value. A file that collects unrelated constants is a bag, not an owner.

A number inside a Tailwind class, `w-[347px]`, is the same literal in class form: an arbitrary value once, a `@theme` token when the value is app-wide. A feature-owned value read by a JSX prop and by a class has the JS constant as its source; the class reads a variable set from it on the nearest ancestor, `style={{ "--card-image-width": `${CARD_IMAGE_WIDTH_PX}px` } as React.CSSProperties}` with `max-w-(--card-image-width)` (see [tailwind](tailwind.md)).

## Tooling

No linter carries this gate. ESLint `no-magic-numbers` is frozen; it allows JSX numbers, variable initializers, and object values by default and reports every other literal in an expression or a call, so `radius * 2` and `5 * 60 * 1000` fail under any option set. Biome `style/noMagicNumbers` has no options and ignores `2`, `10`, `24`, `60`, wider than this rule. None covers strings. The gate is review work; a linter, when a repository wants one, runs at warning level. Checked 2026-09 against ESLint 9.39, Biome 2.1.

## Sources

Martin Fowler: Refactoring, Replace Magic Literal. Robert C. Martin: Clean Code, ch. 17 G25 Replace Magic Numbers with Named Constants. Steve McConnell: Code Complete, ch. 12.1 Numbers in General. Google TypeScript Style Guide: Identifiers, Enums. Airbnb JavaScript Style Guide: Naming Conventions 23.10. TypeScript Handbook: Enums, Objects vs Enums; TypeScript 5.8 release notes, `--erasableSyntaxOnly`. Matt Pocock: enums vs `as const`. ESLint, typescript-eslint, Biome: `no-magic-numbers` rule docs. Tailwind docs: arbitrary values, theme variables.
