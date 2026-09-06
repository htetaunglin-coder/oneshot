# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui`, `cn` is exported from `src/lib/utils`, theme lives in `src/app/globals.css`.

### `src/features/tags/components/tag-pill.tsx`

```tsx
import { cn } from "@/lib/utils";

type TagPillProps = {
  /** Display label, already trimmed by the API. */
  label: string;
  /** Hex color chosen by the user when they created the tag, e.g. "#0ea5e9". */
  color: string;
  selected?: boolean;
  onToggle?: () => void;
  className?: string;
};

export function TagPill({
  label,
  color,
  selected = false,
  onToggle,
  className,
}: TagPillProps) {
  return (
    <button
      type="button"
      aria-pressed={selected}
      onClick={onToggle}
      style={{ backgroundColor: color }}
      className={cn(
        "inline-flex h-7 items-center rounded-full px-3 text-xs font-medium text-white",
        "transition-colors outline-none",
        selected && "ring-2 ring-offset-2 ring-offset-background",
        className,
      )}
    >
      {label}
    </button>
  );
}
```

### `src/features/tags/components/tag-list.tsx`

```tsx
import { TagPill } from "./tag-pill";
import type { Tag } from "../types";

type TagListProps = {
  tags: Tag[];
  selectedIds: Set<string>;
  onToggle: (id: string) => void;
};

export function TagList({ tags, selectedIds, onToggle }: TagListProps) {
  return (
    <ul className="flex flex-wrap gap-1.5">
      {tags.map((tag) => (
        <li key={tag.id}>
          <TagPill
            label={tag.name}
            color={tag.color}
            selected={selectedIds.has(tag.id)}
            onToggle={() => onToggle(tag.id)}
          />
        </li>
      ))}
    </ul>
  );
}
```

## Request

Design wants two interaction states on `TagPill`. On hover, the background should be the same tag color at 80% opacity. On `focus-visible`, the pill should show a ring in that tag color (keep the existing `ring-2` and offset behaviour for the selected state, and use the tag color for the ring when focused). Nothing else in the codebase uses the tag color; it is only ever painted on this one button.

Make the change and explain in one or two sentences how you structured the classes.
