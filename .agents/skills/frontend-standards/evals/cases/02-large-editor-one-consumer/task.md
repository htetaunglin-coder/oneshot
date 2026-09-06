# Case 02 — large-editor-one-consumer

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/features/notes/components/note-page.tsx` — about 900 lines

Outline of the file, top to bottom:

| Lines | Section |
| --- | --- |
| 1–40 | imports, `NotePageProps`, small local types |
| 41–120 | `NoteHeader` (title input, last-saved indicator, publish menu) |
| 121–470 | rich-text editor: `EditorState` reducer, `useEditorKeyboard`, `EditorToolbar`, `EditorSurface`, `applyMark`, `toggleBlock` |
| 471–620 | `NoteSidebar` (backlinks, tags, attachments) |
| 621–780 | `NoteHistoryDrawer` (version list, diff viewer) |
| 781–900 | `NotePage` default export composing the above |

The editor block (lines 121–470) owns its own state and helpers and does not read
anything from the header, sidebar, or history drawer except the `noteId` prop it is
given. Nothing outside this file imports any of its pieces; `note-page.tsx` is the
only consumer. Representative excerpt from the editor block:

```tsx
type Mark = "bold" | "italic" | "code";

type EditorState = {
  doc: Block[];
  selection: Range | null;
  activeMarks: Set<Mark>;
};

function editorReducer(state: EditorState, action: EditorAction): EditorState {
  switch (action.type) {
    case "toggle-mark":
      return { ...state, doc: applyMark(state.doc, state.selection, action.mark) };
    case "toggle-block":
      return { ...state, doc: toggleBlock(state.doc, state.selection, action.kind) };
    case "set-selection":
      return { ...state, selection: action.range };
    default:
      return state;
  }
}

function EditorToolbar({ state, dispatch }: { state: EditorState; dispatch: Dispatch }) {
  const marks: Mark[] = ["bold", "italic", "code"];
  return (
    <div role="toolbar" className="flex gap-1 border-b border-zinc-800 p-1">
      {marks.map((mark) => (
        <ToolbarButton
          key={mark}
          label={mark}
          pressed={state.activeMarks.has(mark)}
          onClick={() => dispatch({ type: "toggle-mark", mark })}
        />
      ))}
    </div>
  );
}

function useEditorKeyboard(dispatch: Dispatch) {
  useEffect(() => {
    function onKeyDown(e: KeyboardEvent) {
      if (!(e.metaKey || e.ctrlKey)) return;
      if (e.key === "b") dispatch({ type: "toggle-mark", mark: "bold" });
      if (e.key === "i") dispatch({ type: "toggle-mark", mark: "italic" });
      if (e.key === "e") dispatch({ type: "toggle-mark", mark: "code" });
    }
    window.addEventListener("keydown", onKeyDown);
    return () => window.removeEventListener("keydown", onKeyDown);
  }, [dispatch]);
}
```

## Task

Add a strikethrough button to the editor toolbar (with `Cmd/Ctrl+Shift+X` as the
keyboard shortcut). Decide where the new/changed code lives and explain in one or
two sentences.
