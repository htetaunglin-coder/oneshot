# F07 — page-metadata-subcomponent

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/projects/lib/types.ts`

```ts
export type Project = {
  id: string;
  name: string;
  ownerName: string;
  createdAt: Date;
};
```

`src/features/projects/lib/queries.ts`

```ts
import { db } from "@/lib/db";
import type { Project } from "./types";

export async function getProject(id: string): Promise<Project | null> {
  return db.project.findUnique({ where: { id } });
}
```

`src/app/(app)/projects/[projectId]/page.tsx`

```tsx
import { notFound } from "next/navigation";
import { getProject } from "@/features/projects/lib/queries";
import { ProjectBoard } from "@/features/projects/components/project-board";

type PageProps = {
  params: Promise<{ projectId: string }>;
};

export default async function ProjectPage({ params }: PageProps) {
  const { projectId } = await params;
  const project = await getProject(projectId);

  if (!project) {
    notFound();
  }

  return (
    <section className="space-y-4">
      <h1 className="text-xl font-semibold">{project.name}</h1>
      <ProjectBoard project={project} />
    </section>
  );
}
```

Other pages in `src/app/(app)/` set a static tab title via the App Router `metadata` API.

## Change request

Two small things for this page. First, the browser tab still says the app name; set the tab title to "Project · Acme" (a static title is fine; per-project titles are a later ticket). Second, under the `<h1>`, show a one-line strip "Owned by <ownerName> · Created <createdAt as locale date>" in `text-sm text-muted-foreground`. That strip is page-only chrome, so keep it as a private `ProjectMeta` piece inside `page.tsx` rather than adding it to the feature folder.

Make the change and explain in one or two sentences how you decided where each declaration goes.
