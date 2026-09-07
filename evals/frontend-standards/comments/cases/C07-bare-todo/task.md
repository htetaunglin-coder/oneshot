# C07 — bare-todo

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Work is tracked in GitHub issues; this list landed in PR #418, which closed issue #402 ("Org members page").

`src/app/(app)/orgs/[orgId]/members/page.tsx`

```tsx
import { MemberList } from "@/features/members/components/member-list";

type PageProps = {
  params: Promise<{ orgId: string }>;
  searchParams: Promise<Record<string, string | string[] | undefined>>;
};

export default async function MembersPage({ params }: PageProps) {
  const { orgId } = await params;

  return (
    <section className="space-y-4">
      <h1 className="text-xl font-semibold">Members</h1>
      <MemberList orgId={orgId} />
    </section>
  );
}
```

`src/features/members/components/member-list.tsx`

```tsx
import { db } from "@/lib/db";
import { MemberRow } from "./member-row";

type MemberListProps = {
  orgId: string;
};

export async function MemberList({ orgId }: MemberListProps) {
  // TODO: handle pagination
  const members = await db.member.findMany({
    where: { orgId },
    take: 50,
  });

  if (members.length === 0) {
    return <p className="text-sm text-muted-foreground">No members yet.</p>;
  }

  return (
    <ul className="divide-y">
      {members.map((member) => (
        <MemberRow key={member.id} member={member} />
      ))}
    </ul>
  );
}
```

`db.member` rows have `firstName`, `lastName`, `email` and `joinedAt: Date`. Pagination is still not built; the largest org has about 300 members.

## Change request

Support a `?sort=name` or `?sort=joined` query parameter on the members page. `name` sorts by last name then first name ascending; `joined` (the default when the parameter is missing or unknown) sorts newest first. Pass the sort down to `MemberList` and apply it in the query.

Make the change and explain in one or two sentences how you decided what to comment.
