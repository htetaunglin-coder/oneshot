# R04 — toolbar-admin-bar

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/workspace/components/toolbar.tsx`

```tsx
"use client";

import { useState } from "react";
import { Menu as MenuIcon } from "lucide-react";
import { Button } from "@/components/ui/button";
import { NavMenu } from "./nav-menu";

type ToolbarProps = {
  projectName: string;
};

export function Toolbar({ projectName }: ToolbarProps) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="flex items-center gap-2 border-b px-4 py-2">
      <Button
        variant="ghost"
        size="icon"
        aria-expanded={isOpen}
        aria-label="Toggle navigation"
        onClick={() => setIsOpen((prev) => !prev)}
      >
        <MenuIcon />
      </Button>
      <span className="font-medium">{projectName}</span>
      {isOpen && <NavMenu onNavigate={() => setIsOpen(false)} />}
    </div>
  );
}
```

`src/features/workspace/components/admin-bar.tsx`

```tsx
import { Button } from "@/components/ui/button";

export function AdminBar() {
  return (
    <div className="ml-auto flex items-center gap-1">
      <Button variant="outline" size="sm">
        Impersonate
      </Button>
      <Button variant="outline" size="sm">
        Audit log
      </Button>
    </div>
  );
}
```

`src/app/(workspace)/layout.tsx` renders `<Toolbar projectName={project.name} />` and has `session.user.role` (`"admin" | "member"`) in scope.

## Change request

Admins need the `AdminBar` at the right end of the toolbar; members must not see it. Add an `isAdmin: boolean` prop to `Toolbar`, render `AdminBar` when it is true, and pass `isAdmin={session.user.role === "admin"}` from the layout.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
