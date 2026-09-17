# N10 — second-role-check

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The workspace feature shows members and a settings navigation. The API already returns an `admin` role, but the UI only ever treated `owner` as privileged.

`src/features/workspace/lib/types.ts`

```ts
export type WorkspaceRole = "owner" | "admin" | "member";

export type WorkspaceMember = {
  id: string;
  name: string;
  role: WorkspaceRole;
};
```

`src/features/workspace/components/member-actions.tsx`

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { useRemoveMember } from "@/features/workspace/hooks/use-remove-member";
import type { WorkspaceMember } from "@/features/workspace/lib/types";

type Props = { viewer: WorkspaceMember; member: WorkspaceMember };

export function MemberActions({ viewer, member }: Props) {
  const removeMember = useRemoveMember();

  if (viewer.role === "owner" && viewer.id !== member.id) {
    return (
      <Button variant="destructive" size="sm" onClick={() => removeMember(member.id)}>
        Remove
      </Button>
    );
  }
  return null;
}
```

`src/features/workspace/components/settings-nav.tsx`

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";
import { cn } from "@/lib/utils";
import type { WorkspaceMember } from "@/features/workspace/lib/types";

type Props = { workspaceId: string; viewer: WorkspaceMember };

export function SettingsNav({ workspaceId, viewer }: Props) {
  const pathname = usePathname();
  const base = `/workspaces/${workspaceId}/settings`;
  const tabs = [
    { href: base, label: "General" },
    { href: `${base}/members`, label: "Members" },
  ];

  return (
    <nav className="flex gap-2">
      {tabs.map((tab) => (
        <Link
          key={tab.href}
          href={tab.href}
          className={cn("rounded-md px-3 py-1.5 text-sm", pathname === tab.href && "bg-muted")}
        >
          {tab.label}
        </Link>
      ))}
    </nav>
  );
}
```

## Change request

"Admins get the same powers as owners in the workspace UI. Two things: (1) admins can remove members too, and (2) add a Billing tab at `${base}/billing` to the settings nav that only owners and admins can see. Members see neither."

Make the change and explain in one or two sentences how you decided which values get a name.
