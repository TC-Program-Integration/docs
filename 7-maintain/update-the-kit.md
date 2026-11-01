# Update the kit

> **For:** a project made from the starter that wants the starter's newer core, UI kit, flowkit,
> tools, tests and shared docs. **Needs:** a newer copy of the starter (a checkout, or its unzipped
> [release](../6-ship/release.md)). **Result:** the new kit in your project, your own changes kept.
> The other direction, a kit fix made in your project: [send kit changes back](send-kit-changes-back.md).

The **kit** is everything projects share with the starter: `src/core`, `src/ui`, `flows/lib/flowkit`,
`tools/`, the tests of those, the configs, the workflows and the shared docs. The starter's
`kit.json` lists them (`include`, `exclude`). The kit files live in your project like any other
file: you can read and change them. Your `kit.json` remembers the version of each file you got
(`files`), so an update knows what you changed.

## Check what you changed

```powershell
pnpm kit:status
```

```text
Kit 1.4.0 from pp-starter-react. 2 kit file(s) differ from it:
  changed  src/ui/fields/TextField.tsx
  deleted  docs/7-maintain/write-the-docs.md
  to merge tools/build.mjs.kit-new
```

`changed` and `deleted` are your edits since the last update; `to merge` is a `.kit-new` file still
waiting for you (below). In the starter itself it only counts the kit's files.

## Update

```powershell
pnpm kit:update ..\pp-starter-react --dry-run   # what would happen; writes nothing
pnpm kit:update ..\pp-starter-react             # do it
pnpm install
pnpm check
```

When the starter's updater (`tools/kit.mjs`, `tools/kit/`) is newer than yours, the update runs the
starter's, so its improvements apply to this update already: it prints
`kit: the starter's updater is newer`. A project whose updater predates this (kit 1.5.7 and
earlier) runs the starter's once by hand:

```powershell
$env:KIT_PROJECT = "."; node ..\pp-starter-react\tools\kit.mjs update ..\pp-starter-react; Remove-Item env:KIT_PROJECT
```

It prints one line per file that changes, then a count per action. For each kit file:

| Your copy | The kit's new version | Action | What happens |
|---|---|---|---|
| the same as the new version | | (none) | nothing |
| unchanged | changed | `update` | replaced by the new version |
| changed by you | unchanged | `keep` | yours is kept |
| changed by you | changed | `conflict` | **yours is kept**; the kit's is written next to it as `<file>.kit-new` |
| missing, never installed | new in the kit | `add` | added |
| deleted by you | still in the kit | `keep-deleted` | left deleted (add it to `exclude` to stop the note) |
| unchanged | removed from the kit | `remove` | deleted |
| changed by you | removed from the kit | `keep-removed` | kept |

Line endings do not count as a change: a Windows checkout compares like a Linux one.

Then `package.json` gets the scripts and packages the kit's tools need (the starter's `kit.json`
lists them under `packageJson`): new ones are added and versions updated, each printed as
`package.json scripts.release added` or `devDependencies.vitest ^4 -> ^5`. Your own scripts and
packages are not touched, and nothing is removed.

The docs menu (`navigation` in `docs/docs.json`) is yours, but the kit's pages must be in it. A
kit page that is in your project and not in your menu is put next to the page or group it follows in
the starter's menu. A group your menu lacks (for example `Choice values`, with a new page) is created
at its place in the starter's menu, with its pages. A page the kit removed is taken out. Each change
is printed as `docs menu 4-build/define-the-schema added` or `docs menu Choice values group added`.
A page or group with no neighbour in your menu is printed with "add it to the menu by hand": add it
yourself, the update leaves it out. Your own pages and groups are not touched. Last,
`kit.json` records the new version.

## Merge a .kit-new file

Open the two files side by side (VS Code: select both, **Compare Selected**), bring the kit's
changes into your file, then delete the `.kit-new` file. `pnpm kit:status` lists the ones left;
`pnpm check` then tells you if the merge broke something.

## Files your project owns

Some kit files every project rewrites (a docs page about your own app, a test helper with your own
methods). List them under `exclude` in your `kit.json`, and updates leave them alone:

```json
"exclude": ["docs/6-ship/deploy.md", "test/e2e/helpers/featurePage.ts"]
```

A whole folder ends with `/`, for example `"docs/8-reference/"`. Your `exclude` is kept by every
update.

## A project with no kit.json

The first update has no record of what you installed. Without help, every kit file that differs from
the new starter looks changed by you, and gets a `.kit-new`. Give it the starter your project was made
from with `--base`, so the files you never changed are simply updated:

1. If `pnpm kit:update` does not exist yet, copy `tools/kit.mjs`, `tools/kit/` and `tools/release/`
   from the new starter.
2. Optionally create `kit.json` with your own `exclude` list and `"files": {}`.
3. Run the update with both starters:

```powershell
node tools/kit.mjs update ..\pp-starter-react-new --base ..\pp-starter-react-old --dry-run
node tools/kit.mjs update ..\pp-starter-react-new --base ..\pp-starter-react-old
```

`--base` is used only when `kit.json` has no `files` yet; after that, `kit.json` is the base.

## After an update

- A docs page printed with "add it to the menu by hand" needs a line in `navigation` in
  `docs/docs.json` (`pnpm test` reports pages not in the menu).
- Read the starter's release notes for anything you must do yourself.
- Commit the update on its own, so it is easy to review and to undo.

## In the starter

The starter is where the kit is made. Its `kit.json` has `include`, `exclude` and `packageJson`
and no `files`. Change kit files there; a new tool, script or package the kit uses goes in
`kit.json` too. Run `pnpm check`, then [make a release](../6-ship/release.md). A kit fix made in a
project arrives as a pull request ([send kit changes back](send-kit-changes-back.md)): review it here.
