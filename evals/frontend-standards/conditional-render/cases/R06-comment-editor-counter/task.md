# R06 — comment-editor-counter

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/comments/lib/types.ts`

```ts
export type Comment = {
  id: string;
  authorName: string;
  body: string;
  createdAt: string;
};
```

`src/features/comments/components/comment-item.tsx`

```tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import type { Comment } from "../lib/types";

type CommentItemProps = {
  comment: Comment;
  canEdit: boolean;
  onSave: (id: string, body: string) => Promise<void>;
};

export function CommentItem({ comment, canEdit, onSave }: CommentItemProps) {
  const [isEditing, setIsEditing] = useState(false);

  const Editor = () => {
    const [draft, setDraft] = useState(comment.body);
    const [isSaving, setIsSaving] = useState(false);

    async function handleSave() {
      setIsSaving(true);
      await onSave(comment.id, draft);
      setIsSaving(false);
      setIsEditing(false);
    }

    return (
      <div className="flex flex-col gap-2">
        <Textarea
          value={draft}
          onChange={(event) => setDraft(event.target.value)}
          rows={3}
          aria-label="Edit comment"
        />
        <div className="flex gap-2">
          <Button size="sm" onClick={handleSave} disabled={isSaving}>
            Save
          </Button>
          <Button size="sm" variant="ghost" onClick={() => setIsEditing(false)}>
            Cancel
          </Button>
        </div>
      </div>
    );
  };

  return (
    <article className="rounded-lg border p-3">
      <header className="mb-2 flex items-center justify-between">
        <span className="text-sm font-medium">{comment.authorName}</span>
        {canEdit && !isEditing ? (
          <Button size="sm" variant="ghost" onClick={() => setIsEditing(true)}>
            Edit
          </Button>
        ) : null}
      </header>
      {isEditing ? (
        <Editor />
      ) : (
        <p className="whitespace-pre-wrap text-sm">{comment.body}</p>
      )}
    </article>
  );
}
```

## Change request

The API rejects comment bodies longer than 500 characters, and users only find out after clicking Save. Add a live counter under the textarea in the edit view that reads `{length} / 500`; when the draft is over the limit, the counter uses `text-destructive` and the Save button is disabled. The read view does not change.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
