# Expected decisions — tailwind class organization

| Case | Expected decision |
| --- | --- |
| T01 repeated-utilities | Write `flex items-center justify-between gap-2` inline on the new `filter-bar.tsx` element. No shared layout component, no exported/shared constant. Repetition of a short utility string across siblings is fine. |
| T02 hoisted-constant | Delete `CARD_CLASSES`; put the classes directly on the `<div>` including `hover:shadow-md transition-shadow`. A single-use constant is removed, not extended. |
| T03 one-axis-three-states | Replace the ternary with a local object map keyed by `kind` holding complete class strings, e.g. `const tone = { info: "...", danger: "...", warning: "..." }`, applied with `cn(base, tone[kind], className)`. Do NOT introduce cva — one axis, three options. Icon may use a parallel map. |
| T04 four-plus-options | Now two axes and five named options in total (3 variants + 2 sizes): convert to `cva` with `variants: { variant: {...}, size: { sm, md } }`, `defaultVariants: { variant: "default", size: "md" }`, and props typed via `VariantProps<typeof badgeVariants>`. Export `badgeVariants`. Existing `<Badge variant="secondary">` still compiles. |
| T05 dynamic-class-name | Remove the template-literal `bg-${color}-500`. Use a map of whole class names, e.g. `{ red: "bg-red-500", amber: "bg-amber-500", green: "bg-green-500", emerald: "bg-emerald-500" }`, so every class exists whole in source and survives the production build. |
| T07 runtime-value | Set the runtime value once on the container as a CSS variable: `style={{ "--progress": `${clamped}%` }}`. Fill consumes it with `w-(--progress)` (or `[width:var(--progress)]`), label with `left-(--progress)` (or `[left:var(--progress)]`). Neither child carries its own inline width/left. |
| T08 repeated-hex-color | Add `--color-brand: #4f46e5;` inside `@theme` in `src/app/globals.css`. Replace every `bg-[#5b4bff]`, `text-[#5b4bff]`, `hover:bg-[#5b4bff]/90` with `bg-brand`, `text-brand`, `hover:bg-brand/90`. No arbitrary hex values remain in components. |
| T09 button-look-on-link | Import `buttonVariants` from `@/components/ui/button` and use `className={buttonVariants({ variant: "outline" })}` on the `<Link>`. Copied class string removed. No wrapper component, no `asChild` rewrite needed, no `<a>` inside `<Button>`. |
| T10 template-string-merge | Replace the template literal with `cn("rounded-lg border p-4", className)`, `className` last so tailwind-merge lets `p-6` win. Template string removed; no `?? ""`. |

T06 removed 2026-09-07: the spacing rule it tested was cut as a styling technique, not organization.
