# Case 04 — three-lookalikes-diverge

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

Three features each turn a string into a URL-safe token. Each has its own rules,
set by the team that owns that feature.

### `src/features/blog/lib/slugify-title.ts`

```ts
export function slugifyTitle(title: string): string {
  return title
    .trim()
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, "")
    .replace(/\s+/g, "-")
    .replace(/-+/g, "-")
    .slice(0, 80);
}
```

### `src/features/tags/lib/slugify-tag.ts`

```ts
export function slugifyTag(raw: string): string {
  return raw
    .trim()
    .replace(/^#+/, "")
    .toLowerCase()
    .replace(/[^a-z0-9-]/g, "-")
    .replace(/-+/g, "-")
    .replace(/^-|-$/g, "");
}
```

### `src/features/profiles/lib/slugify-username.ts`

```ts
export function slugifyUsername(raw: string): string {
  return raw
    .trim()
    .replace(/[^A-Za-z0-9_]/g, "")
    .replace(/_+/g, "_")
    .slice(0, 30);
}
```

Usage: `slugifyTitle` in `features/blog/components/post-form.tsx`;
`slugifyTag` in `features/tags/components/tag-input.tsx`;
`slugifyUsername` in `features/profiles/components/handle-field.tsx`.

## Task

Blog post titles that contain emoji (for example `"Ship it 🚀 today"`) should drop
the emoji before the rest of the transform runs, so the result is `ship-it-today`.
Tags and usernames are not part of this request. Make the change. Decide where the
new/changed code lives and explain in one or two sentences.
