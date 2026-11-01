# Write the docs

> **For:** reading these docs as a website and changing them. **Result:** a page that builds, with
> working links, in the menu.

The docs are the `.md` pages in `docs/`, shown as a website by [Holocron](https://holocron.so), an
open-source docs site that runs on Vite. They stay on your laptop: nothing publishes them online.

| Command | What it does |
|---|---|
| `pnpm docs:serve` | the site at `http://localhost:5173`; a saved page shows at once |
| `pnpm docs:build` | builds the site into `site/` and fails on a page that does not parse or a link to a missing page; `pnpm check` runs it |

The switch at the top right picks a light, dark or system theme (or press **d**). On a wide screen
the whole site grows, as the browser's zoom would make it (`docs/style.css`).

`test/unit/docs-site.test.ts` (part of `pnpm test`) checks the rest: every page is in the menu, every
menu entry has a page, and every `#heading` in a link exists.

## The files

| File | What it is |
|---|---|
| `docs/docs.json` | the site's name, logo, colours and menu (`navigation`) |
| `docs/index.md` | the home page |
| `docs/style.css` | styles added to the whole site |
| `docs/theme/` | our additions to Holocron: the diagram zoom, and the AI chat's local model (`theme/chat/`) |
| `docs/public/` | images served as they are (`logo.svg`, `favicon.svg`) |
| `docs/vite.config.mts`, `docs/tsconfig.json` | the Vite settings that start Holocron; TypeScript for `theme/` |

## Write a page

- Start with `# Title`, then a `> **For:** ... **Result:** ...` line. The title is the page's name
  in the menu; a short `sidebarTitle` at the top of the file shortens it there.
- A page has no line limit, but one page covers one subject. Split a long guide as `<guide>.md` plus
  `<guide>/01-name.md` pages.
- Add every new page to `navigation` in `docs/docs.json`, without `.md`
  (`"7-maintain/write-the-docs"`).
- Link with relative paths to `.md` files, as in any Markdown: `[commands](../8-reference/commands.md)`.
- Explain each job in one place and link to it from the others.

Pages are MDX: Markdown that can also hold components. Two characters therefore mean something
outside code: `<` starts a component and `{` starts code. In text, write them as `\<` and `\{`, or
put them in backticks. A web address goes in a link, `[https://example.com](https://example.com)`.

A note or a warning box:

```md
<Note title="Prod">

The same path, with the prod environment.

</Note>
```

`<Warning>`, `<Tip>` and `<Info>` work the same way. Keep the blank lines inside the box.

## Diagrams

Write a diagram as a fenced block with the language `mermaid`. `pnpm docs:build` does not check what
is inside, so a syntax error only shows on the page (the diagram's text instead of the drawing):
look at the page with `pnpm docs:serve`, or paste the block into
[https://mermaid.live](https://mermaid.live), before you commit. Avoid `<` and `>` in labels (use words), and
break lines in labels with `<br/>`.

A diagram fits the width of the page. Click it (or press Enter on it) to open it full screen:
drag to move it, scroll or use **+** and **-** to zoom, **Fit** to see all of it, **Esc** to close.
The zoom view is `docs/theme/` (it wraps Holocron's own diagram); if a Holocron update breaks it,
`pnpm docs:build` or the zoom button stops working, so look at a diagram after updating Holocron.

## The AI chat

Holocron's **Ask AI** chat normally sends questions to holocron.so. Here,
`docs/theme/chat/plugin.mts` answers them on the docs server itself with a model in Ollama
([Ask the docs](../2-set-up/03-ask-the-docs.md)), so they stay on the laptop:

- `"assistant": {"enabled": true}` in `docs/docs.json` shows the chat; `false` hides it.
  `DOCS_AI_CHAT=off` hides it for one server or build (`theme/chat/switch.mts`): the
  [published docs](publish-the-docs.md) are built that way.
- Holocron sends the question and every page; `theme/chat/search.mts` picks the matching sections,
  `theme/chat/prompt.mts` writes the rules and the question, `theme/chat/providers.mts` picks who
  answers (`DOCS_CHAT_PROVIDER`): `ollama.mts` (default, local), `claude.mts` or `openai.mts` (API key,
  or the read-only login of Claude Code / Codex from `credentials.mts`), over `sse.mts`.
- The answer only uses what a page says, so a clear page with the right words in its headings is
  what makes a good answer.

- The answer appears as the model writes it. Holocron's chat shows text only once a block is
  finished, so `patches/@holocron.so__vite@0.37.1.patch` (listed in `pnpm-workspace.yaml`) makes it
  show the text so far, every 50 ms. `theme/chat/pace.mts` sends it a word at a time at an even
  pace, and `theme/chat/chat.css` adds the cursor and the fade-in.

It also hooks into Holocron (the address its chat calls, and that patch), so ask a question after
updating Holocron. pnpm refuses to install a Holocron version the patch does not fit: redo the
patch with `pnpm patch @holocron.so/vite` (it changes `src/lib/chat-stream.ts` and
`src/chat/chat-submit.ts`), or drop it and answers appear in one piece.
