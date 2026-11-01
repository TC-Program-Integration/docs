# Configuration

> **Reference.** Every configuration file, its exact format, and the files the build writes for
> the deploy tools.

| File | You change it | Holds |
|---|---|---|
| [`webresources.config.mjs`](#webresourcesconfigmjs) | per project | publisher prefix, features, command buttons |
| [`flows/solution.json`](#flowssolutionjson) | per project | the solution and publisher the flows are built for |
| [`kit.json`](#kitjson) | rarely | what the shared kit is, and the kit version installed |
| [`webresources.links.xml`](#webresourceslinksxml) | no: `pnpm build` writes it | built files linked to web resources, for the Web Resources Manager |
| [`dist/deploy.json`](#distdeployjson) | no: `pnpm build` writes it | web resources and buttons, for `dv:verify` and Command Buttons |
| [`flows/out/flows.json`](#flowsoutflowsjson) | no: `pnpm flows:build` writes it | each flow's key, name and id, for `dv:verify` |
| [`flows.links.json`](#flowslinksjson) | through the Flows as Code tool | flow files linked to flows |
| [`tools/xrmtoolbox/tools.json`](#toolsxrmtoolboxtoolsjson) | when a tool moves | the team's XrmToolBox tools |
| `tools/preview/mock-data.json` | freely | data the preview serves ([format](mock-dataverse.md#data-file-format)) |
| `test/fixtures/mock-data.json` | with the tests | data the unit and browser tests rely on |
| `tools/queries/*.xml` | add your own | FetchXML for `pnpm dv:q` |
| `package.json` | name, description, version | scripts, `react`, `react-dom` and `chart.js` (bundled into the pages that use them), dev dependencies; `engines` and `.nvmrc` say Node 24; `.npmrc` turns off pnpm's update notice (the version is pinned) |
| `tsconfig.json` | rarely | TypeScript for `src/`: browser and Xrm types, strict, `jsx: react-jsx`, the [import aliases](#import-aliases) |
| `test/tsconfig.json` | rarely | extends it for tests and configs, with Node types |
| `eslint.config.mjs` | rarely | lint rules: typescript-eslint, React hooks, accessibility, doc comments |
| `vitest.config.mts`, `vitest.dataverse.config.mts` | rarely | unit tests: jsdom, `test/unit/**/*.test.{ts,tsx}`, `test/setup.ts`, the coverage floor; Dataverse checks: Node, `test/dataverse/**/*.test.ts`, long timeouts |
| `playwright.config.ts` | rarely | browser tests: Edge, preview on port 5655 with the test fixture, traces on failure |
| `pyproject.toml`, `uv.lock` | rarely | Python tools and their settings (Black, Ruff, Pyright) |
| `docs/docs.json`, `docs/vite.config.mts`, `docs/style.css` | adding a page | the docs site: name, logo and menu; how Vite starts Holocron; site-wide styles ([Write the docs](../7-maintain/write-the-docs.md)) |
| `pnpm-workspace.yaml`, `.vscode/extensions.json`, `.vscode/settings.json`, `.gitignore` | rarely | esbuild's install script allowed, the docs site's unneeded ones skipped; recommended VS Code extensions; editor settings (format on save, the `.venv` interpreter, `python.languageServer` off so the Pyright extension is the only Python language server); the Highlight extension's rules that colour Power Automate expressions (`@...` and `@{...}`) inside Python strings, so they stand out from the string text; what git leaves out |

## webresources.config.mjs

```js
export default {
  prefix: "pidev",                 // publisher prefix: 2 to 8 lowercase letters or digits
  features: ["quick-task"],        // folders of src/features/ to build and preview
  buttons: {                       // optional: command bar buttons, by feature
    "quick-task": { label: "Quick task", table: "account" },
    // the same command on several tables: a list
    // "site-visit": [{ label: "Site visit", table: "account" }, { label: "Site visit", table: "contact", location: "grid" }],
  },
};
```

| Key | Meaning |
|---|---|
| `prefix` | the publisher prefix of the solution the web resources go in. Starts every web resource name (`<prefix>_<feature>_<file>`) and is injected into code as `WEB_RESOURCE_PREFIX`. |
| `features` | features to build and preview, in order (the preview opens the first). `pnpm new:feature` adds to it. |
| `buttons` | optional. Maps a feature to one button, or to a list of buttons when the same command goes on several tables. |

Fields of a button (types in `webresources.config.d.mts`):

| Field | Required | Meaning |
|---|---|---|
| `label` | yes | the button text |
| `table` | yes | logical name of the table whose command bar gets the button |
| `location` | no | `"form"` (default) or `"grid"` (the main grid) |
| `function` | no | the function exported by the feature's `command/index.ts`; default `"open"` |

`pnpm build` checks each button: the feature is in `features`, `command.js` exports the function,
`label` and `table` are set. A wrong entry is printed as a warning and left out of
`dist/deploy.json`. The button's unique name is made from the parts, lowercase letters and digits
only: `<prefix>_<feature>_<function>_<table>` (`pidev_quicktask_open_account`).

## flows/solution.json

```json
{
  "uniqueName": "MyProjectFlows", "displayName": "My project flows",
  "publisher": "MyPublisher", "publisherDisplayName": "My publisher",
  "prefix": "pidev", "optionPrefix": 10000, "version": "1.0.0.0"
}
```

`publisher` is the publisher's **unique** name (letters, digits, `_`; find it with
`pnpm dv:q tools/queries/publishers.xml`). `uniqueName` follows the same rule. `prefix` is 2 to 8
lowercase letters or digits. `version` is `a.b.c.d`; raise it before each import to production.
`flows/build-all.py` reads every key; its [options](commands.md#flows) override them.

The top-level object is the **home** solution. Two optional keys add solutions, so one flow can
belong to several of them:

- `flows`: the flow folders the home solution ships (the folder of `flows/<folder>/flow.py`).
  Absent: every flow found.
- `solutions`: a list of extra solutions. Each has `uniqueName`, `displayName`, `version` and
  `flows` (its folders). `publisher`, `publisherDisplayName`, `prefix` and `optionPrefix` are
  inherited from the top level unless the entry sets its own.

```json
{
  "uniqueName": "MyProjectFlows", "displayName": "My project flows",
  "publisher": "MyPublisher", "prefix": "pidev", "optionPrefix": 10000, "version": "1.0.0.0",
  "flows": ["intake-webhook", "new-submission", "overdue-tasks"],
  "solutions": [
    {
      "uniqueName": "MyProjectReminders", "displayName": "My project reminders", "version": "1.0.0.0",
      "flows": ["overdue-tasks"]
    }
  ]
}
```

Here `overdue-tasks` is in both solutions: `MyProjectFlows` and `MyProjectReminders` each get a
zip with the flow. Such a flow needs a fixed `workflow_id` in its `flow.py`; build-all stops
without one. Errors stop the build with the reason: an unknown folder, two solutions with one
`uniqueName`, an extra solution without `flows`. A prefix that differs from the home's is a
warning. [One flow in several solutions](../6-ship/deploy.md#one-flow-in-several-solutions)
explains the rules.

## kit.json

| Key | In the starter | In a project |
|---|---|---|
| `include` | files and folders (ending in `/`) that are the kit | copied from the starter at each update |
| `exclude` | kit files projects do not take | **your own list**, kept by every update |
| `packageJson` | the `scripts`, `dependencies` and `devDependencies` a project takes | |
| `source`, `version`, `files` | | written by `kit:update`: the starter, its version, and a hash of each kit file as installed |
| `starterRepo`, `starter`, `autopush` | | by hand, kept by every update: the starter's git URL, or a local checkout; `autopush` `"pr"` makes the `pre-push` hook open a pull request, `true` copies into the checkout. [Send kit changes back](../7-maintain/send-kit-changes-back.md), [update the kit](../7-maintain/update-the-kit.md) |

## Files the build writes

### webresources.links.xml

At the project root. The Web Resources Manager reads it ([Web Resources Manager](tools/web-resources-manager.md)).

```xml
<?xml version="1.0"?>
<WebResourcesManagerLinks>
  <BuildCommand>pnpm build</BuildCommand>
  <Folders />
  <Links>
    <Link Name="pidev_quick_task_command.js" Path="dist/pidev_quick_task_command.js" />
    <Link Name="pidev_quick_task_page.css" Path="dist/pidev_quick_task_page.css" />
  </Links>
</WebResourcesManagerLinks>
```

`Path` is relative to the file. Links sorted by name. Each build rewrites the links of the built
files and keeps `<Folders>` and any other link added in the tool. Commit it.

### dist/deploy.json

```json
{
  "prefix": "pidev",
  "webResources": [
    { "name": "pidev_quick_task_command.js", "file": "dist/pidev_quick_task_command.js", "type": "js" }
  ],
  "commands": [{
    "feature": "quick-task", "uniqueName": "pidev_quicktask_open_account", "label": "Quick task",
    "table": "account", "location": "form", "library": "pidev_quick_task_command.js",
    "function": "QuickTask.open", "parameters": ["PrimaryControl"]
  }]
}
```

`type` is `js`, `html` or `css`. `function` is `<Global>.<function>`, the global being the feature
name in PascalCase. `parameters` is always `["PrimaryControl"]`. `icon`, `tooltip` and `sequence`
are written only when the button sets them in `webresources.config.mjs`. How the tool uses each
field: [Command Buttons](tools/command-buttons.md).

### flows/out/flows.json

Written by `pnpm flows:build`: a list of `{ "key", "name", "workflowId" }`, one per flow.

### dist/schema.json

Written by `pnpm build` and `pnpm schema:build` when the project has a `schema/` folder
([Define the schema](../4-build/define-the-schema.md)); read by the Schema as Code XrmToolBox tool.
Never edited by hand. Its format is fixed by `test/fixtures/schema/minimal.json` and `full.json`
(the tool's checks use the same files, guarded by the hash in `test/fixtures/schema/HASH`).

```json
{ "format": "schema-as-code", "formatVersion": 1, "generatedBy": "power-platform-starter 1.6.0",
  "project": { "key": "nsep", "id": "6f1c…" },
  "publisher": { "uniqueName": "Program_Integration", "prefix": "nopspi", "optionPrefix": 89763 },
  "solution": { "uniqueName": "NSEP", "display": { "1033": "NSEP", "1036": "PNES" } },
  "languages": [1033, 1036],
  "existing": { "tables": [{ "logicalName": "systemuser", "primaryId": "systemuserid", "entitySetName": "systemusers" }], "choices": [] },
  "choices": [{ "schemaName": "nopspi_RequestType", "logicalName": "nopspi_requesttype", "display": {…}, "description": {…},
               "options": [{ "value": 897630000, "label": {…} }] }],
  "tables": [{ "schemaName": "nopspi_Request", "logicalName": "nopspi_request", "entitySetName": "nopspi_requests",
               "primaryId": "nopspi_requestid", "ownership": "user", "primary": {…}, "columns": [{…}], "status": {…}, "keys": [{…}], … }],
  "relationships": [{ "kind": "oneToMany", "schemaName": "…", "referenced": "…", "referencing": "…", "lookup": {…}, "cascade": "removeLink" },
                    { "kind": "customer", … }, { "kind": "manyToMany", "schemaName": "…", "intersect": "…", "tableA": "…", "tableB": "…" }] }
```

- Labels are keyed by language code (`"1033"`, `"1036"`); option values are the stored ones.
- Tables, columns, choices, relationships and keys are sorted by name; options and status reasons
  keep their written order (their display order). There are no timestamps: the file only changes
  when the schema does.
- Lookup and customer columns appear only under `relationships`, with their column definition,
  because Dataverse creates the column with its relationship.
- A tool that doesn't know `formatVersion` refuses the file.

## flows.links.json

Written by `pnpm flows:build` and by the Flows as Code tool, in the folder you pick for flow files
(the project's `flows` folder). Commit it.

The build adds each flow it built (`id` is the workflow id, `name`, `file`, and `description` when the
flow has one) and updates the entry with the same id. Entries for other flows, extra keys and a
`buildCommand` you set are kept as they are; the build sets `buildCommand` to `pnpm flows:build` only
when the file has none. Entries stay sorted by name. A file that is not valid JSON is left alone, and
the build prints a warning and still succeeds. The build writes the file only with the default output
folder (`flows/out`): with `--out` the paths would be wrong, so nothing is written. The build writes it
only when its content changes.

```json
{
  "buildCommand": "pnpm flows:build",
  "flows": [{ "file": "out/OverdueTasks/flow.json", "id": "9b898176-5724-5bb8-8793-4f4962f3f13e", "name": "Overdue task reminder" }]
}
```

`file` is relative to the links file. `description` (optional) is given to a flow that
[Create new flows](tools/flows-as-code.md#create-new-flows) creates from the link. `buildCommand`
(optional) runs in that folder before each push, once you allow it. See
[Flows as Code](tools/flows-as-code.md).

## tools/xrmtoolbox/tools.json

`{ "tools": [{ "name": "Flows as Code", "repo": "TC-Program-Integration/FlowsAsCode", "files": ["FlowsAsCode.dll"] }] }`:
`repo` is the GitHub repository whose latest release holds the tool's zip; `files` are the DLLs the
installer copies from it. The list: [XrmToolBox tools](tools.md).

## Import aliases

Code imports the shared folders by name, not by a chain of `../`:

```ts
import { createThen, openRecord } from "@core";       // src/core
import { checkRequired, Field, text } from "@ui";     // src/ui
import type { FeatureSchema } from "@core/schema";    // a file inside one of them
import { table, text } from "@schema-dsl";            // src/schema-dsl: the schema builders (kit)
import { CallbackRequest } from "@schema";             // schema/index.ts: the project's own tables
```

Set in four places, so editor, build, preview and tests agree: `paths` in `tsconfig.json`,
`aliasesFor()` in `tools/features.mjs`, `resolve.alias` in both vitest configs. A new alias goes in all four.

## Where the prefix appears

It must match the publisher of the solution. It appears in:

- web resource names: `<prefix>_<feature>_<file>` and button unique names (`webresources.config.mjs`);
- custom table and column names in `config.ts`, written in full (`pidev_address`);
- connection reference names of flows: `<prefix>_<FlowKey><Connector>` (`flows/solution.json`).
