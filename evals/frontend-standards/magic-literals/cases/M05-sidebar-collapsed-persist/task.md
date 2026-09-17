# M05 — sidebar-collapsed-persist

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/workspace/hooks/use-sidebar-collapsed.ts`

```ts
"use client";

import { useCallback, useState } from "react";

export function useSidebarCollapsed() {
  const [collapsed, setCollapsed] = useState(false);

  const toggle = useCallback(() => {
    setCollapsed((prev) => {
      const next = !prev;
      window.localStorage.setItem("sidebar:collapsed", next ? "1" : "0");
      return next;
    });
  }, []);

  const restore = useCallback((value: boolean) => {
    setCollapsed(value);
  }, []);

  return { collapsed, toggle, restore };
}
```

`src/features/workspace/components/workspace-shell.tsx`

```tsx
"use client";

import { useEffect } from "react";
import { PanelLeft } from "lucide-react";
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/utils";
import { useSidebarCollapsed } from "@/features/workspace/hooks/use-sidebar-collapsed";

type WorkspaceShellProps = {
  sidebar: React.ReactNode;
  children: React.ReactNode;
};

export function WorkspaceShell({ sidebar, children }: WorkspaceShellProps) {
  const { collapsed, toggle, restore } = useSidebarCollapsed();

  // Read after mount so the server and the client render the same first frame.
  useEffect(() => {
    restore(readStoredCollapsed());
  }, [restore]);

  return (
    <div className="flex min-h-screen">
      <aside
        className={cn(
          "border-r bg-background transition-[width]",
          collapsed ? "w-0 overflow-hidden" : "w-72",
        )}
      >
        {sidebar}
      </aside>
      <main className="min-w-0 flex-1">
        <Button
          variant="ghost"
          size="icon"
          onClick={toggle}
          aria-label={collapsed ? "Show sidebar" : "Hide sidebar"}
        >
          <PanelLeft />
        </Button>
        {children}
      </main>
    </div>
  );
}

function readStoredCollapsed(): boolean {
  return window.localStorage.getItem("sidebar-collapsed") === "1";
}
```

## Change request

Bug report: hide the sidebar, reload the page, and it is shown again. The collapsed state does not persist across reload. Find the cause and fix it so the choice survives a reload.

Make the change and explain in one or two sentences how you decided which values get a name.
