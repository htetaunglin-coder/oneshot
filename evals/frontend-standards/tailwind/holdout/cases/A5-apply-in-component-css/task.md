# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui`, `cn` is exported from `src/lib/utils`, theme and global CSS live in `src/app/globals.css`. CSS modules are supported by the Next config and `@apply` works inside them because `globals.css` does `@import "tailwindcss"`.

### `src/features/admin/components/data-table.module.css`

```css
@reference "@/app/globals.css";

.cell {
  @apply px-3 py-2 text-sm border-b;
}

.header-cell {
  @apply px-3 py-2 text-xs font-medium uppercase border-b;
}
```

### `src/features/admin/components/data-table.tsx`

```tsx
import styles from "./data-table.module.css";

export type Column<T> = {
  key: keyof T & string;
  label: string;
  align?: "left" | "right";
};

type DataTableProps<T extends { id: string }> = {
  columns: Column<T>[];
  rows: T[];
};

export function DataTable<T extends { id: string }>({ columns, rows }: DataTableProps<T>) {
  return (
    <div className="overflow-x-auto rounded-md border">
      <table className="w-full text-left">
        <thead className="bg-muted/50">
          <tr>
            {columns.map((col) => (
              <th
                key={col.key}
                scope="col"
                className={styles["header-cell"]}
                style={{ textAlign: col.align ?? "left" }}
              >
                {col.label}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {rows.map((row) => (
            <tr key={row.id} className="hover:bg-muted/30">
              {columns.map((col) => (
                <td key={col.key} className={styles.cell} style={{ textAlign: col.align ?? "left" }}>
                  {String(row[col.key] ?? "")}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

Nothing else imports `data-table.module.css`. `DataTable` is used on two admin pages.

## Change request

Long values in the body cells wrap and make rows uneven. Add `whitespace-nowrap` to the body cells (header cells too, so the columns stay aligned).

Make the change and explain in one or two sentences how you structured the classes.
