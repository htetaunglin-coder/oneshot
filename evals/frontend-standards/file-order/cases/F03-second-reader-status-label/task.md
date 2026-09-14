# F03 — second-reader-status-label

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/tasks/lib/types.ts`

```ts
export type TaskStatus = "todo" | "in_progress" | "blocked" | "done";

export type Task = {
  id: string;
  title: string;
  status: TaskStatus;
};
```

`src/features/tasks/components/task-card.tsx`

```tsx
import { cn } from "@/lib/utils";
import type { Task, TaskStatus } from "@/features/tasks/lib/types";

type TaskCardProps = {
  task: Task;
  onSelect: (id: string) => void;
};

export function TaskCard({ task, onSelect }: TaskCardProps) {
  return (
    <button
      type="button"
      onClick={() => onSelect(task.id)}
      className="flex w-full items-center justify-between rounded-md border p-3 text-left hover:bg-accent"
    >
      <span className="truncate">{task.title}</span>
      <StatusBadge status={task.status} />
    </button>
  );
}

const STATUS_LABEL: Record<TaskStatus, string> = {
  todo: "To do",
  in_progress: "In progress",
  blocked: "Blocked",
  done: "Done",
};

function StatusBadge({ status }: { status: TaskStatus }) {
  return (
    <span
      className={cn(
        "rounded-full px-2 py-0.5 text-xs",
        status === "done" && "bg-green-100 text-green-800",
        status === "blocked" && "bg-red-100 text-red-800",
        status === "in_progress" && "bg-blue-100 text-blue-800",
        status === "todo" && "bg-muted text-muted-foreground",
      )}
    >
      {STATUS_LABEL[status]}
    </span>
  );
}
```

## Change request

An accessibility audit found that screen readers announce the card as just the title and then a second, disconnected "In progress" chunk. Give the `<button>` an `aria-label` of the form `"<title>, <status>"` using the same human wording the badge shows (never the raw enum value), and mark the badge `aria-hidden` so the status is not read twice.

Make the change and explain in one or two sentences how you decided where each declaration goes.
