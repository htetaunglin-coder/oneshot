# Case 12 — tiny-fix-in-large-file

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/components/decorations
src/lib
src/hooks
```

### `src/components/decorations/profile.tsx` — 980 lines

A single decorative illustration rendered on the About page: one `<svg>` built from
hand-written paths, split into small named sub-parts, a few of which carry a CSS
keyframe animation. Outline:

| Lines | Section |
| --- | --- |
| 1–30 | imports, `ProfileDecorationProps` |
| 31–120 | `<Background/>` gradient defs and backdrop paths |
| 121–260 | `<Face/>` |
| 261–330 | `<HatBrim/>` |
| 331–420 | `<HatCrown/>` |
| 421–560 | `<Collar/>`, `<Shoulders/>` |
| 561–700 | `<Glasses/>` with `animate-blink` |
| 701–860 | `<FloatingSparkles/>` with `animate-float` |
| 861–980 | `ProfileDecoration` export composing the sub-parts |

Excerpt of `<HatBrim/>`:

```tsx
function HatBrim() {
  return (
    <g id="hat-brim">
      <path
        d="M112 214c38-22 118-22 156 0 12 7 12 21 0 28-38 22-118 22-156 0-12-7-12-21 0-28z"
        fill="#1c1c1e"
        stroke="#333"
        strokeWidth={2}
      />
      <path
        d="M130 226c30-14 90-14 120 0"
        fill="none"
        stroke="#333"
        strokeWidth={1.5}
        strokeLinecap="round"
      />
    </g>
  );
}
```

`ProfileDecoration` is imported by `src/app/about/page.tsx` only. The sub-parts
are not exported.

## Task

Design asked for the hat-brim outline to use the zinc-700 tone: change the stroke
colour of both paths in `<HatBrim/>` from `#333` to `#3f3f46`. Make the change.
Decide where the new/changed code lives and explain in one or two sentences.
