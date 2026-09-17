# N2 — constants-bag

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui) with a feature-based layout. The repo has a shared values file that several features already import from.

`src/lib/constants.ts`

```ts
export const APP_NAME = "Northwind Admin";
export const MAX_UPLOAD_MB = 25;
export const DEFAULT_LOCALE = "en-US";
```

`src/features/uploads/components/upload-dropzone.tsx` imports `MAX_UPLOAD_MB` from `@/lib/constants`; `src/app/layout.tsx` imports `APP_NAME` and `DEFAULT_LOCALE` from the same file.

The members feature currently loads every member in one request.

`src/features/members/hooks/use-members.ts`

```ts
"use client";

import { useQuery } from "@tanstack/react-query";
import { fetchMembers } from "@/features/members/lib/fetch-members";

export function useMembers(orgId: string) {
  return useQuery({
    queryKey: ["members", orgId],
    queryFn: () => fetchMembers({ orgId }),
  });
}
```

`src/features/members/lib/fetch-members.ts`

```ts
import type { Member } from "@/features/members/lib/types";

type FetchMembersInput = {
  orgId: string;
  page?: number;
  pageSize?: number;
};

type MembersPage = {
  items: Member[];
  total: number;
};

export async function fetchMembers(input: FetchMembersInput): Promise<MembersPage> {
  const params = new URLSearchParams({ orgId: input.orgId });
  if (input.page !== undefined) params.set("page", String(input.page));
  if (input.pageSize !== undefined) params.set("pageSize", String(input.pageSize));
  const res = await fetch(`/api/members?${params}`);
  if (!res.ok) throw new Error("Failed to load members");
  return res.json();
}
```

`src/features/members/components/members-table.tsx`

```tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { useMembers } from "@/features/members/hooks/use-members";
import { MemberRow } from "@/features/members/components/member-row";

type MembersTableProps = {
  orgId: string;
};

export function MembersTable({ orgId }: MembersTableProps) {
  const [page, setPage] = useState(1);
  const { data, isPending } = useMembers(orgId);

  if (isPending || !data) return <p className="text-sm text-muted-foreground">Loading…</p>;

  return (
    <div className="space-y-3">
      <table className="w-full text-sm">
        <tbody>
          {data.items.map((member) => (
            <MemberRow key={member.id} member={member} />
          ))}
        </tbody>
      </table>
      <footer className="flex items-center justify-between text-sm text-muted-foreground">
        <span>{data.total} members</span>
        <div className="flex gap-2">
          <Button variant="outline" size="sm" disabled={page === 1} onClick={() => setPage((p) => p - 1)}>
            Previous
          </Button>
          <Button variant="outline" size="sm" onClick={() => setPage((p) => p + 1)}>
            Next
          </Button>
        </div>
      </footer>
    </div>
  );
}
```

## Change request

"Paginate the members list at 25 per page. The hook should pass the page and page size through to `fetchMembers`, the footer should read `Showing 1–25 of 140` style text, and the Next button should disable on the last page."

Make the change and explain in one or two sentences how you decided which values get a name.
