# B3 — second-reader-left-in-place

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The support feature renders ticket cards in a list. The file is small and has not been touched in months; the request is a one-line visual tweak from design.

`src/features/tickets/components/ticket-card.tsx`

```tsx
import { cn } from "@/lib/utils";
import type { Ticket, TicketPriority } from "@/features/tickets/lib/types";

type TicketCardProps = {
  ticket: Ticket;
  onOpen: (id: string) => void;
};

export function TicketCard({ ticket, onOpen }: TicketCardProps) {
  return (
    <button
      type="button"
      onClick={() => onOpen(ticket.id)}
      className="flex w-full items-start justify-between rounded-lg border bg-card p-4 text-left hover:bg-accent"
    >
      <div>
        <p className="font-medium">{ticket.subject}</p>
        <p className="text-sm text-muted-foreground">{ticket.requester}</p>
      </div>
      <Pill priority={ticket.priority} />
    </button>
  );
}

const TONE: Record<TicketPriority, string> = {
  low: "border-slate-300 text-slate-700",
  medium: "border-amber-400 text-amber-800",
  high: "border-red-500 text-red-800",
};

function Pill({ priority }: { priority: TicketPriority }) {
  return (
    <span className={cn("rounded-full border px-2 py-0.5 text-xs capitalize", TONE[priority])}>
      {priority}
    </span>
  );
}
```

## Change request

"Design wants the card itself to carry the priority colour, not only the pill: give the `<button>` a `border-l-4` whose colour matches the pill for that priority. Smallest diff that does it."

Make the change and explain in one or two sentences how you decided where each declaration goes.
