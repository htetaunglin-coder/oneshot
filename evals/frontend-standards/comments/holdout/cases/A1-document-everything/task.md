# A1 — document-everything

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). `src/lib/dates.ts` is imported by the scheduling, reports and billing features. It has no comments today.

`src/lib/dates.ts`

```ts
export function startOfDay(date: Date): Date {
  const result = new Date(date);
  result.setHours(0, 0, 0, 0);
  return result;
}

export function isSameDay(a: Date, b: Date): boolean {
  return (
    a.getFullYear() === b.getFullYear() &&
    a.getMonth() === b.getMonth() &&
    a.getDate() === b.getDate()
  );
}

export function addDays(date: Date, days: number): Date {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
}

const rangeFormatter = new Intl.DateTimeFormat("en-US", {
  month: "short",
  day: "numeric",
  year: "numeric",
});

export function formatRange(start: Date, end: Date): string {
  const lastDay = addDays(end, -1);
  if (isSameDay(start, lastDay)) {
    return rangeFormatter.format(start);
  }
  return `${rangeFormatter.format(start)} – ${rangeFormatter.format(lastDay)}`;
}

const DATE_ONLY = /^(\d{4})-(\d{2})-(\d{2})$/;

export function parseIsoDate(value: string): Date | null {
  const match = DATE_ONLY.exec(value);
  if (match) {
    const [, y, m, d] = match;
    const local = new Date(Number(y), Number(m) - 1, Number(d));
    if (local.getMonth() !== Number(m) - 1) {
      return null;
    }
    return local;
  }
  const parsed = new Date(value);
  return Number.isNaN(parsed.getTime()) ? null : parsed;
}
```

Notes from the team channel:

- `formatRange` follows the calendar convention the whole app uses: `end` is the first instant *after* the range, so a booking from the 3rd to the 4th (exclusive) prints as a single day, `Mar 3, 2026`.
- `parseIsoDate` exists because `new Date("2026-03-03")` is UTC midnight, which shows as the previous day for users west of Greenwich. Full timestamps with an offset are passed straight through.
- Two people last month called `formatRange` with an inclusive end date and got a range one day short.

## Change request

From the tech lead: "Please document the public API of this module so hover docs show up in the editor. People keep guessing at how these work."

Make the change and explain in one or two sentences how you decided what to comment.
