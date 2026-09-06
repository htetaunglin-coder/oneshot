# Case 01 — small-helper-once

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

The music player feature renders a row per track. The file below is the only place
that formats a duration anywhere in the repo.

### `src/features/player/components/track-row.tsx`

```tsx
"use client";

import { cn } from "@/lib/cn";
import type { Track } from "../api/tracks";

function formatDuration(totalSeconds: number): string {
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;
  return `${minutes}:${seconds}`;
}

type TrackRowProps = {
  track: Track;
  isActive: boolean;
  onSelect: (id: string) => void;
};

export function TrackRow({ track, isActive, onSelect }: TrackRowProps) {
  return (
    <button
      type="button"
      onClick={() => onSelect(track.id)}
      className={cn(
        "flex w-full items-center justify-between rounded px-3 py-2 text-left",
        isActive ? "bg-zinc-800 text-white" : "hover:bg-zinc-900",
      )}
    >
      <span className="truncate">{track.title}</span>
      <span className="tabular-nums text-zinc-400">
        {formatDuration(track.durationSeconds)}
      </span>
    </button>
  );
}
```

## Task

A track of 125 seconds currently renders as `2:5`; it should render as `2:05`
(seconds always two digits). Make that change. Decide where the new/changed code
lives and explain in one or two sentences.
