# F06 — arrow-file-add-helper

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`. This file was written by a contractor last year.

`src/features/schedule/lib/types.ts`

```ts
export type Shift = {
  id: string;
  employeeName: string;
  startsAt: Date;
  endsAt: Date;
};
```

`src/features/schedule/components/shift-list.tsx`

```tsx
import type { Shift } from "@/features/schedule/lib/types";

type ShiftListProps = {
  shifts: Shift[];
};

export const ShiftList = ({ shifts }: ShiftListProps) => {
  if (shifts.length === 0) {
    return <p className="text-sm text-muted-foreground">No shifts scheduled.</p>;
  }

  return (
    <ul className="divide-y">
      {shifts.map((shift) => (
        <ShiftRow key={shift.id} shift={shift} />
      ))}
    </ul>
  );
};

const ShiftRow = ({ shift }: { shift: Shift }) => {
  const start = shift.startsAt.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
  const end = shift.endsAt.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });

  return (
    <li className="flex items-center justify-between py-2 text-sm">
      <span>{shift.employeeName}</span>
      <span className="tabular-nums text-muted-foreground">
        {start} – {end}
      </span>
    </li>
  );
};
```

## Change request

Managers see a week at a time and cannot tell where one day ends. Group the shifts by calendar day of `startsAt` and render an `<h3>` with the day (e.g. "Mon 14 Sep") above each day's rows. Put the grouping in a `groupByDay(shifts: Shift[]): Array<{ day: string; shifts: Shift[] }>` helper in the same file, keeping shifts in the sequence they arrive within a day. `ShiftRow` is unchanged.

Make the change and explain in one or two sentences how you decided where each declaration goes.
