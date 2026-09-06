# Case 09 — 1200-line-screen-with-editor

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/features/docs/components/doc-screen.tsx` — 1,200 lines

Outline, top to bottom:

| Lines | Section |
| --- | --- |
| 1–70 | imports, `DocScreenProps`, local types |
| 71–150 | `DocHeader` (breadcrumb, title, presence avatars) — 80 lines |
| 151–350 | `SidebarTree` (folder tree, drag-and-drop reorder) — 200 lines |
| 351–850 | editor: `useDocEditor` hook, `editorReducer`, `EditorToolbar`, `EditorCanvas`, `useAutosave`, selection helpers — 500 lines |
| 851–1100 | `CommentsPanel` (thread list, composer, resolve) — 250 lines |
| 1101–1200 | `DocFooter` and `DocScreen` default export — 100 lines |

The editor section holds all of its own state through `useDocEditor` and takes only
`docId` and `initialContent` from `DocScreen`. The other sections do not reach into
editor state; they communicate through props on `DocScreen`. Only `DocScreen` uses
any of these pieces.

Excerpt from the editor section:

```tsx
type DocEditorState = {
  content: Node[];
  selection: Selection | null;
  dirty: boolean;
};

export function useDocEditor(docId: string, initialContent: Node[]) {
  const [state, dispatch] = useReducer(editorReducer, {
    content: initialContent,
    selection: null,
    dirty: false,
  });
  useAutosave(docId, state.content, state.dirty);
  return { state, dispatch };
}

function EditorToolbar({ dispatch }: { dispatch: Dispatch<EditorAction> }) {
  return (
    <div role="toolbar" className="flex items-center gap-1 border-b px-2 py-1">
      <ToolbarButton label="Bold" onClick={() => dispatch({ type: "mark", mark: "bold" })} />
      <ToolbarButton label="Italic" onClick={() => dispatch({ type: "mark", mark: "italic" })} />
      <ToolbarButton label="Heading" onClick={() => dispatch({ type: "block", kind: "h2" })} />
    </div>
  );
}

function EditorCanvas({ state, dispatch }: { state: DocEditorState; dispatch: Dispatch<EditorAction> }) {
  /* ~180 lines: contentEditable surface, selection sync, paste handling */
}
```

## Task

Show a live word count (for example `1,204 words`) at the right end of the editor
toolbar, computed from the editor's current content. Make the change. Decide where
the new/changed code lives and explain in one or two sentences.
