# M03 — sidebar-width-change

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/workspace/components/workspace-shell.tsx`

```tsx
"use client";

import { useState } from "react";
import { PanelLeft } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";

type WorkspaceShellProps = {
  sidebar: React.ReactNode;
  children: React.ReactNode;
};

export function WorkspaceShell({ sidebar, children }: WorkspaceShellProps) {
  const [open, setOpen] = useState(true);

  return (
    <div className="flex min-h-screen overflow-x-hidden">
      <aside
        className="flex shrink-0 flex-col border-r bg-background transition-[margin]"
        style={{ width: 320, marginLeft: open ? 0 : -320 }}
      >
        <div className="flex-1 overflow-y-auto">{sidebar}</div>
        <QuickNote />
      </aside>
      <main className="min-w-0 flex-1">
        <Button
          variant="ghost"
          size="icon"
          onClick={() => setOpen((prev) => !prev)}
          aria-label={open ? "Hide sidebar" : "Show sidebar"}
        >
          <PanelLeft />
        </Button>
        {children}
      </main>
    </div>
  );
}

function QuickNote() {
  const [note, setNote] = useState("");

  return (
    <div className="border-t p-3">
      <label htmlFor="quick-note" className="text-xs text-muted-foreground">
        Quick note
      </label>
      <Textarea
        id="quick-note"
        value={note}
        onChange={(event) => setNote(event.target.value)}
        maxLength={320}
        rows={3}
        placeholder="Scratch space, not saved"
      />
    </div>
  );
}
```

## Change request

Design widened the sidebar: it is now 360px wide. It must still move fully out of view when hidden. Nothing else about the shell changes.

Make the change and explain in one or two sentences how you decided which values get a name.
