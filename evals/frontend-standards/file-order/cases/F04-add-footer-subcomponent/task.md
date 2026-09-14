# F04 — add-footer-subcomponent

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/reports/lib/types.ts`

```ts
export type Report = {
  id: string;
  name: string;
  summary: string;
  rows: Array<{ label: string; value: number }>;
  generatedAt: Date;
};
```

`src/features/reports/components/report-panel.tsx`

```tsx
import { Button } from "@/components/ui/button";
import type { Report } from "@/features/reports/lib/types";

type ReportPanelProps = {
  report: Report;
  onRefresh: () => void;
};

export function ReportPanel({ report, onRefresh }: ReportPanelProps) {
  return (
    <section className="rounded-lg border">
      <Header name={report.name} onRefresh={onRefresh} />
      <Body summary={report.summary} rows={report.rows} />
    </section>
  );
}

type HeaderProps = {
  name: string;
  onRefresh: () => void;
};

function Header({ name, onRefresh }: HeaderProps) {
  return (
    <div className="flex items-center justify-between border-b px-4 py-3">
      <h2 className="font-semibold">{name}</h2>
      <Button variant="ghost" size="sm" onClick={onRefresh}>
        Refresh
      </Button>
    </div>
  );
}

function Body({ summary, rows }: Pick<Report, "summary" | "rows">) {
  return (
    <div className="space-y-3 px-4 py-3">
      <p className="text-sm text-muted-foreground">{summary}</p>
      <table className="w-full text-sm">
        <tbody>
          {rows.map((row) => (
            <tr key={row.label}>
              <td>{row.label}</td>
              <td className="text-right tabular-nums">{row.value}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## Change request

Product wants a footer strip under the table: on the left, "Generated <date, locale string>"; on the right, an "Export CSV" button. Add an `onExport: () => void` prop to `ReportPanel` and render the strip as a `Footer` sub-component in the same file. `Footer` takes `generatedAt: Date` and `onExport`; declare a named props type for it, as `Header` has.

Make the change and explain in one or two sentences how you decided where each declaration goes.
