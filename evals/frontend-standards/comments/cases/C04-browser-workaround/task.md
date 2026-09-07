# C04 — browser-workaround

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `members` feature has a dialog with a search input that should be focused when the dialog opens.

`src/features/members/components/add-member-dialog.tsx`

```tsx
"use client";

import { useRef, useState } from "react";
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { MemberSearchResults } from "./member-search-results";

export function AddMemberDialog() {
  const [open, setOpen] = useState(false);
  const [query, setQuery] = useState("");
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Add member</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Add member</DialogTitle>
        </DialogHeader>
        <Input
          ref={inputRef}
          autoFocus
          placeholder="Search by name or email"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
        />
        <MemberSearchResults query={query} onSelect={() => setOpen(false)} />
      </DialogContent>
    </Dialog>
  );
}
```

Bug report from QA (issue #533):

> On iOS Safari 17.4 and 17.5 the search field in "Add member" is not focused when the dialog opens and the keyboard does not appear. Desktop Safari, Chrome and Firefox are fine. Tapping the field works, but the dialog exists to type immediately.
>
> This is the Radix focus-trap timing problem: https://github.com/radix-ui/primitives/issues/2373. Safari discards focus set synchronously while the content's open animation is still running. The workaround confirmed in that thread is to `preventDefault()` in `onOpenAutoFocus` on `DialogContent` and call `.focus()` on the input in a `setTimeout(..., 0)` instead.

## Change request

Fix #533 using the workaround from the issue so the search input is focused on open in iOS Safari without changing behavior in other browsers.

Make the change and explain in one or two sentences how you decided what to comment.
