# N6 — camelcase-precedent

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The notes feature has a composer that lets a user link related notes (searched with a debounce, paged) and attach up to a fixed number of tags. The file was written by the team that owns the feature and has been stable for a year.

`src/features/notes/components/note-composer.tsx`

```tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { useDebouncedValue } from "@/hooks/use-debounced-value";
import { TagPicker } from "@/features/notes/components/tag-picker";
import { RelatedNoteList } from "@/features/notes/components/related-note-list";
import { useNoteSearch } from "@/features/notes/hooks/use-note-search";

const pageSize = 20;
const maxTags = 5;
const debounceMs = 300;

export function NoteComposer() {
  const [query, setQuery] = useState("");
  const [tags, setTags] = useState<string[]>([]);
  const debouncedQuery = useDebouncedValue(query, debounceMs);
  const { results, hasMore, loadMore } = useNoteSearch(debouncedQuery, pageSize);

  return (
    <div className="flex flex-col gap-4">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Link a related note"
      />
      <RelatedNoteList notes={results} />
      {hasMore ? (
        <Button variant="ghost" size="sm" onClick={loadMore}>
          Show more
        </Button>
      ) : null}

      <TagPicker value={tags} onChange={setTags} disabled={tags.length >= maxTags} />
      <p className="text-muted-foreground text-xs">
        {tags.length}/{maxTags} tags
      </p>
    </div>
  );
}
```

`src/features/notes/components/attachment-list.tsx` already exists and renders `<AttachmentList files={files} onRemove={...} />` for an array of `File`. It has no limit logic of its own.

## Change request

"Notes can now carry file attachments. Add an attachment section to the composer: an `AttachmentList` for the chosen files, an 'Add file' button that opens a hidden `<input type="file" multiple>`, and a counter under it. A note may have at most 10 files — once the limit is reached, disable the button and show the counter as `n/10 files`, exactly the way the tag limit already works. Keep it consistent with the rest of this file."

Make the change and explain in one or two sentences how you decided which values get a name.
