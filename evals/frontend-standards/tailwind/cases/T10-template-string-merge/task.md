# T10 — template-string-merge

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). `src/lib/utils.ts` exports `cn` (clsx + tailwind-merge).

`src/features/settings/components/panel.tsx`

```tsx
type PanelProps = {
  title: string;
  children: React.ReactNode;
  className?: string;
};

export function Panel({ title, children, className }: PanelProps) {
  return (
    <section className={`rounded-lg border p-4 ${className ?? ""}`}>
      <h3 className="mb-3 text-base font-semibold">{title}</h3>
      {children}
    </section>
  );
}
```

`src/features/settings/components/danger-zone.tsx`

```tsx
import { Panel } from "./panel";
import { Button } from "@/components/ui/button";

export function DangerZone() {
  return (
    <Panel title="Danger zone" className="border-red-300 p-6">
      <p className="text-sm text-muted-foreground">
        Deleting the workspace removes all projects and members.
      </p>
      <Button variant="destructive" className="mt-4">
        Delete workspace
      </Button>
    </Panel>
  );
}
```

The red border shows up, but the panel still renders with `p-4` padding instead of the `p-6` passed in, because both classes end up on the element and the stylesheet order decides.

## Change request

Let consumers of `Panel` override any of its default classes, including the padding, by passing `className`.

Make the change and explain in one or two sentences how you structured the classes.
