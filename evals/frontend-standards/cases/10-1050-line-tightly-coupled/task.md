# Case 10 — 1050-line-tightly-coupled

## Scenario

React + TypeScript + Next.js App Router + Tailwind. Layout:

```
src/features/<name>/{components,lib,api,hooks}
src/components/ui
src/lib
src/hooks
```

### `src/features/chat/lib/parse-incomplete-markdown.ts` — 1,050 lines

A streaming-markdown parser: it is fed a partial assistant message and returns a
block tree, treating unterminated constructs as still open. Every function takes
and mutates one `Cursor` (position, pending-token stack, open-fence info); the
functions call each other recursively. Outline:

| Lines | Section |
| --- | --- |
| 1–60 | types: `Cursor`, `Block`, `Inline`, `Fence` |
| 61–140 | `createCursor`, `peek`, `advance`, `pushPending`, `popPending` |
| 141–330 | `parseBlocks` — top-level loop, dispatches by line prefix |
| 331–420 | `parseCodeFence` |
| 421–560 | `parseList` / `parseListItem` (recursive into `parseBlocks`) |
| 561–640 | `parseBlockquote` (recursive into `parseBlocks`) |
| 641–900 | `parseInline`, `parseEmphasis`, `parseLink`, `parseInlineCode` |
| 901–1050 | `finalize` — closes open constructs, rebuilds from `pending` stack |

Excerpt from `parseCodeFence`:

```ts
function parseCodeFence(c: Cursor): Block {
  const fence = readFenceOpen(c); // consumes ``` or ~~~ and info string
  pushPending(c, { kind: "fence", fence });
  const lines: string[] = [];
  while (!atEnd(c)) {
    const line = readLine(c);
    if (isFenceClose(line, fence)) {
      popPending(c);
      return { type: "code", lang: fence.info, text: lines.join("\n"), open: false };
    }
    lines.push(line);
  }
  // Reached end of input without a closing fence.
  return { type: "code", lang: fence.info, text: lines.join("\n"), open: true };
}
```

Bug: `readLine` on the last iteration reads to end of input, so the fence is not
marked open until *after* the following paragraph text has been consumed into
`lines`. `atEnd(c)` and `readLine(c)` are used by every other block parser, so their
behaviour is relied on throughout the file. There is a test file at
`src/features/chat/lib/parse-incomplete-markdown.test.ts` (~400 lines).

## Task

Fix the bug so that an unclosed code fence does not swallow the paragraph that
follows it in the next streamed chunk; the fence should be returned with
`open: true` and the paragraph parsed as its own block. Make the change. Decide
where the new/changed code lives and explain in one or two sentences.
