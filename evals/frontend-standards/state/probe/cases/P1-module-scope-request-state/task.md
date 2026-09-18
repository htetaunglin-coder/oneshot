# P1 — module-scope-request-state

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`. Shared client values use Zustand. Signed-in pages live under `src/app/(app)/`; that group's layout reads the session on the server and seeds a small session store so client components anywhere below it can read the current org without threading it through props.

`src/lib/auth.ts` exports `getSession(): Promise<Session | null>` (reads the auth cookie, returns `null` when signed out):

```ts
export type Session = {
  userId: string;
  orgId: string;
  orgName: string;
  plan: "free" | "team" | "enterprise";
};
```

`src/lib/session/store.ts`

```ts
"use client";

import { create } from "zustand";

export type SessionState = {
  orgId: string | null;
  orgName: string | null;
};

const initialState: SessionState = {
  orgId: null,
  orgName: null,
};

export const useSessionStore = create<SessionState>()(() => initialState);

export const selectOrgId = (state: SessionState) => state.orgId;
export const selectOrgName = (state: SessionState) => state.orgName;

export function resetSessionStore() {
  useSessionStore.setState(initialState);
}
```

`src/components/session-hydrator.tsx`

```tsx
"use client";

import { useRef } from "react";
import { useSessionStore } from "@/lib/session/store";

type SessionHydratorProps = {
  orgId: string;
  orgName: string;
};

// Renders nothing. Mounted once in the (app) layout so the store is filled
// before the first page under it renders. Features read it through the
// selectors in store.ts instead of receiving the session as a prop.
export function SessionHydrator({ orgId, orgName }: SessionHydratorProps) {
  const seeded = useRef(false);

  if (!seeded.current) {
    useSessionStore.setState({ orgId, orgName });
    seeded.current = true;
  }

  return null;
}
```

`src/app/(app)/layout.tsx`

```tsx
import type { ReactNode } from "react";
import { redirect } from "next/navigation";
import { AppSidebar } from "@/components/app-sidebar";
import { SidebarInset, SidebarProvider } from "@/components/ui/sidebar";
import { SessionHydrator } from "@/components/session-hydrator";
import { getSession } from "@/lib/auth";

export default async function AppLayout({ children }: { children: ReactNode }) {
  const session = await getSession();
  if (!session) redirect("/login");

  return (
    <SidebarProvider>
      <SessionHydrator orgId={session.orgId} orgName={session.orgName} />
      <AppSidebar />
      <SidebarInset>
        <main className="flex flex-1 flex-col gap-6 p-6">{children}</main>
      </SidebarInset>
    </SidebarProvider>
  );
}
```

Two other client components already read the store: `src/lib/session/components/org-switcher.tsx` (sidebar header) uses `useSessionStore(selectOrgName)`, and `src/features/billing/components/invoice-list.tsx` uses `useSessionStore(selectOrgId)` to build its query key.

`src/features/reports/components/export-button.tsx`

```tsx
"use client";

import { useState } from "react";
import { Download } from "lucide-react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { exportReport } from "@/features/reports/api";
import { selectOrgId, useSessionStore } from "@/lib/session/store";

type ExportButtonProps = {
  reportId: string;
};

export function ExportButton({ reportId }: ExportButtonProps) {
  const orgId = useSessionStore(selectOrgId);
  const [pending, setPending] = useState(false);

  async function handleClick() {
    if (!orgId) return;
    setPending(true);
    try {
      const { url } = await exportReport({ orgId, reportId });
      window.open(url, "_blank", "noopener");
    } catch {
      toast.error("Export failed. Try again.");
    } finally {
      setPending(false);
    }
  }

  return (
    <Button variant="outline" size="sm" onClick={handleClick} disabled={pending || !orgId}>
      <Download className="size-4" />
      Export CSV
    </Button>
  );
}
```

`ExportButton` is rendered by `ReportHeader` (`src/features/reports/components/report-header.tsx`, a client component that receives `report: Report`), which the server page `src/app/(app)/reports/[id]/page.tsx` renders after loading the report.

## Change request

"CSV export becomes a paid feature in the next release. `getSession()` already returns `plan` next to `orgId`, and we keep `orgId` in the session store, so add `plan` next to it and hydrate it from the (app) layout the same way. Then in `ExportButton`: when the plan is `free`, render the button disabled and wrap it in a `Tooltip` (`Tooltip`, `TooltipTrigger`, `TooltipContent` from `@/components/ui/tooltip`) that reads "Upgrade to export". `team` and `enterprise` keep the current behaviour."

Make the change and explain in one or two sentences how you decided where the value lives.
