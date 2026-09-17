# S3 — third-role-badge

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The community feature renders a comment header with the author's name, a role badge, and a timestamp. The badge line is one short expression and has never been touched since the file was created.

`src/features/community/components/comment-header.tsx`

```tsx
import { formatRelative } from "@/lib/format";
import { AdminBadge } from "@/features/community/components/badges/admin-badge";
import { GuestBadge } from "@/features/community/components/badges/guest-badge";
import { UserBadge } from "@/features/community/components/badges/user-badge";
import type { CommentAuthor } from "@/features/community/lib/types";

type CommentHeaderProps = {
  author: CommentAuthor | null;
  createdAt: string;
};

export function CommentHeader({ author, createdAt }: CommentHeaderProps) {
  return (
    <div className="flex items-center gap-2 text-sm">
      <span className="font-medium">{author?.displayName ?? "Guest"}</span>
      {author ? (author.isAdmin ? <AdminBadge /> : <UserBadge />) : <GuestBadge />}
      <time dateTime={createdAt} className="text-muted-foreground">
        {formatRelative(createdAt)}
      </time>
    </div>
  );
}
```

`CommentAuthor` in `src/features/community/lib/types.ts`:

```ts
export type CommentAuthor = {
  id: string;
  displayName: string;
  isAdmin: boolean;
  isModerator: boolean;
};
```

`src/features/community/components/badges/moderator-badge.tsx` already exists and exports `ModeratorBadge`, styled like the others.

## Change request

"Show the `ModeratorBadge` for moderators. An admin who is also a moderator shows the admin badge only. Guests and plain users are unchanged. The badge line is tiny, so keep it small."

Make the change and explain in one or two sentences how you shaped the conditional rendering.
