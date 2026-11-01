# Conventions

> **Reference.** The rules code in this repository follows. Reviews check against this page.

## Dataverse names

- **One place per name.** Tables, columns, web resource names and choice values live in the
  feature's `config.ts`. Code reads them from there; `schema.ts` is built from there.
- **Never hard-code choice labels or options.** Load them with `Metadata.columnOptions`. When
  code must compare against a value or label, list it in `schema.ts` (`values`, `labels`) so
  `pnpm test:dataverse` catches a change.
- **Entity set names come from metadata** (`Metadata.recordPath`), not from adding an "s".
  Custom tables often have odd plurals (`pidev_addresses`).
- **Lookups are read as `_<column>_value`** (`readLookup`) **and written with the navigation
  property** (`<Nav>@odata.bind`). The navigation property is case-sensitive and often the
  schema name (`pidev_Manager`, not `pidev_manager`); confirm it in the table's
  relationships.
- **Code uses logical names**, never display names.
- **Escape every value in a filter** with `odataString`.
- Check real data with `pnpm dv:q` before writing logic that depends on it, and add those
  cases to the mock data.

## Names in code

| Thing | Form | Example |
|---|---|---|
| Feature folder (kebab); web resource name uses underscores | kebab-case folder | `site-visit` → `pidev_site_visit_page.html` |
| Command global | PascalCase of the feature | `SiteVisit.open` |
| Page component, config | `<Name>Page`, `<Name>Config` | `SiteVisitPage` |
| Component files, hooks | `PascalCase.tsx` for a component, `useThing.ts` for a hook | `SaveBar.tsx`, `useSaveBar.ts` |
| Flow folder / key | kebab-case / PascalCase | `flows/overdue-tasks`, `OverdueTasks` |
| Flow actions | `Verb_object`, underscores | `Find_user`, `Add_row` |

## Code

- TypeScript, strict. `pnpm format:check`, `pnpm lint`, `pnpm type-check` and `pnpm test:coverage` must pass (`pnpm check` runs them all); `pnpm build` formats, lints and type-checks first. See [Quality checks](../4-build/test-and-check/02-the-checks.md).
- Python in `flows/` is formatted with Black, linted with Ruff and passes Pyright strict: no `Any`, no `# type: ignore`, `# pyright:` or `# noqa` comments. Flow data is typed with `Json`, `Action` and `Actions` from flowkit. Dependencies are in `pyproject.toml` (managed with uv).
- No rule is silenced with a comment (`eslint-disable`, `@ts-ignore`). `unknown` and narrowing replace `any`; `catch (error)` reads the message with `errorMessage(error)` from the core.
- **No file is longer than 175 lines** (code, tests, tools and flows; documentation pages, data files such as JSON, lock files and generated output are not counted). A long file is hard to read, most of all when many functions refer to each other. When a file grows, move a coherent part into its own module and import it: a page of fields into one file per topic, a long test into one file per behaviour with shared helpers beside it. `test/unit/file-length.test.ts` fails the build otherwise. Documentation pages have no limit, but a page that covers two subjects is better as two.
- Every function, class, method, interface and type alias in `src/` has a doc comment, and every module, class, function and method in `flows/` has a docstring. ESLint and Ruff fail the build when one is missing.
- File header: `@file` and `@description` saying what the file is for. Doc comments on
  exported functions say **when** to use them, not how they work line by line.
- Errors reach the user in plain words (the save bar's `error(...)`, or an alert) and the
  details go to `console.error` with the module name.
- Saving: `createThen` for anything with follow-up writes. Never leave a half-saved record.
- Test helpers in pages (fill test data) only show when `isNonProduction()`.
- Markup follows what the components do: a `<label>` for every input, `<button type="button">`,
  errors linked with `aria-describedby`. `eslint-plugin-jsx-a11y` checks the rest.
- React: function components only, one component per file (small helpers may share it), props as a
  documented interface. State that two components need lives in their parent. Derive values from
  state while rendering instead of copying them into more state.
- Effects are for talking to the outside (Dataverse, the browser). An effect that awaits ignores a
  stale answer (`stale()` in its cleanup) and lists every value it uses in its dependency array;
  `react-hooks/exhaustive-deps` fails the build otherwise.

## The core

- A feature reaches Dataverse only through `src/core`. No `Xrm.` or `fetch` in features.
- The core never names a project table.
- Forms are built from `src/ui` components; the core does not import React.
- A helper or piece of UI used by two features moves into the core, with a doc comment saying
  when to use it, a unit test, an export from its `index.ts`, and a line in
  [core API](core-api.md) or [UI components](ui-components.md).
- Markup is JSX and styles are in `.css` files next to the component that uses them: one folder per UI component (`src/ui/<name>/`) or page part. No HTML in strings, no `innerHTML`, no `dangerouslySetInnerHTML`, no `style={...}`, no elements created by hand (`test/unit/conventions.test.ts` checks).
- Import the core and the UI kit through the aliases `@core` and `@ui` (see [Configuration](configuration.md#import-aliases)), not through `../`. Files inside a folder still import their neighbours with `./`.
- A feature never imports another feature; a shared piece moves to `src/ui` or `src/core`.
- Pure logic goes in the feature's `logic/` folder, e.g. `logic/rules.ts` (no React, no DOM, no Xrm). Loading and state changes that need no screen (`load.ts`, `formState.ts`) are plain TypeScript too, so they are tested without rendering.

## Tests

- Every feature has unit tests (rules, state, page) and browser tests (the main paths). The scaffold
  writes both; keep them passing as you change the feature.
- `test/fixtures/mock-data.json` is changed together with the tests that rely on it. For trying
  things by hand, edit `tools/preview/mock-data.json` instead.
- Page tests render the page and act like a user (Testing Library); they do not look at React's internals.
- A bug fix comes with a test that failed before it.
- Never skip or delete a failing test to get green; fix the code or the test's expectation.

## Flows

- One flow per folder, `flows/<name>/flow.py`, built with flowkit; no hand-edited JSON.
- Every write is followed by its `catch()`, and the flow ends with `terminate_with_list()` when
  anything failed, so the run shows Failed with every problem.
- Rows removed at the source are deactivated, not deleted.
- Changes made in the designer are copied back into `flow.py`; the next push overwrites them otherwise.

## Git

- Small commits, one change each, message in the imperative ("Add the site visit dialog").
- A change to a command, setting or function updates its page in `docs/8-reference/` in the same change.
- `pnpm check` passes before pushing (the pre-push hook runs it), plus `pnpm test:e2e` when a page
  changed. CI runs the same checks.
- `dist/` and `flows/out/` are build output and are not committed. `webresources.links.xml` and
  `flows/flows.links.json` are committed: the team shares the links.

## Access

- Dataverse is reached only through pac, read-only, from the project's tools. The rules:
  [access and security](../9-help/access-and-security.md).
- The XrmToolBox tools push to development and test environments only; production gets changes by
  solution import ([XrmToolBox tools](tools.md#safety-rules-they-share)).
