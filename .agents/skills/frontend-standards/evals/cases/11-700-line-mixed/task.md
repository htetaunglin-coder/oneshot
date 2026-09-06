# Case 11 — 700-line-mixed

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/features/profile/components/profile-page.tsx` — 700 lines

Outline, top to bottom:

| Lines | Section |
| --- | --- |
| 1–50 | imports, `ProfilePageProps`, `Profile` type import |
| 51–300 | `ProfileHeader`, `ProfileBio`, `ProfileStats`, avatar upload — 250 lines |
| 301–450 | CSV export: `escapeCsvCell`, `rowsToCsv`, `profileToRows`, `downloadBlob` — 150 lines of pure functions, no JSX |
| 451–650 | `NotificationPreferencesForm` — 200 lines: its own `useForm`, zod schema, submit handler, toggles |
| 651–700 | `ProfilePage` default export composing the above; renders an "Export CSV" button that calls `downloadBlob(rowsToCsv(profileToRows(profile)), "profile.csv")` |

Excerpt of the CSV section:

```ts
function escapeCsvCell(value: string | number | boolean | null): string {
  if (value == null) return "";
  const s = String(value);
  return /[",\n]/.test(s) ? `"${s.replace(/"/g, '""')}"` : s;
}

function rowsToCsv(rows: Array<Array<string | number | boolean | null>>): string {
  return rows.map((r) => r.map(escapeCsvCell).join(",")).join("\n");
}

function profileToRows(profile: Profile) {
  return [
    ["field", "value"],
    ["id", profile.id],
    ["name", profile.name],
    ["email", profile.email],
    ["joined", profile.joinedAt],
    ["public", profile.isPublic],
  ];
}

function downloadBlob(text: string, filename: string, mime = "text/csv") {
  const url = URL.createObjectURL(new Blob([text], { type: mime }));
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
```

Excerpt of the preferences section:

```tsx
const prefsSchema = z.object({
  emailDigest: z.boolean(),
  mentions: z.boolean(),
  marketing: z.boolean(),
});

function NotificationPreferencesForm({ initial }: { initial: z.infer<typeof prefsSchema> }) {
  const form = useForm({ resolver: zodResolver(prefsSchema), defaultValues: initial });
  /* ~180 lines of toggles, submit, toast */
}
```

Nothing outside `profile-page.tsx` uses any of these pieces.

## Task

Add a "Download as JSON" button next to "Export CSV" that downloads the same
fields as `profile.json`. Make the change. Decide where the new/changed code lives
and explain in one or two sentences.
