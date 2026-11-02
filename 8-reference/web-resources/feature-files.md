# Feature files

> **Reference.** What each file of `src/features/<name>/` contains. The `site-visit`
> scaffold (`pnpm new:feature`) is the example used below.

| Path | Bundled as | Required |
|---|---|---|
| `config.ts` | (imported) | yes |
| `schema.ts` | (imported by the Dataverse checks) | yes |
| `command/index.ts` | `<prefix>_<name>_command.js`, functions on the global `<Name>` | if there is a command or form script |
| `page/index.tsx` | `<prefix>_<name>_page.js`, with React | if there is a page |
| `page/<Name>Page.tsx` | (imported by `page/index.tsx`) | if there is a page |
| `page/fields.ts` | (imported by the page) | if there is a form |
| `page/page.html` | `<prefix>_<name>_page.html` | if there is a page |
| `page/page.css` | `<prefix>_<name>_page.css`, after the CSS of the components the page uses | optional |
| `page/*.ts(x)` | (imported by the page) | as the page grows: a `useThing.ts` hook, `load.ts`, `formState.ts`, `testData.ts`, components in sub-folders |
| `logic/rules.ts` | (imported by the page) | if there is a page |
| `test/unit/<name>.test.tsx` | (not bundled) | yes: the scaffold writes it |
| `test/e2e/<name>.spec.ts` | (not bundled) | yes: the scaffold writes it |

Folders are only for source: each web resource is a single flat file named
`<prefix>_<name>_page.html`, `<prefix>_<name>_page.js`, `<prefix>_<name>_page.css` and
`<prefix>_<name>_command.js`. Add more plain-logic files to `logic/` and more page
parts (extra components, helpers) to `page/` as the feature grows.

`<Name>` is the feature name in PascalCase (`site-visit` → `SiteVisit`). Files a feature does
not have are skipped by the build. The feature is listed in `features` of
[`webresources.config.mjs`](../project/configuration.md#webresourcesconfigmjs); its command bar button, if
any, in `buttons` there.

## config.ts

Every Dataverse name the feature uses, in one object. Code, schema and tests read names from
here, never from string literals elsewhere.

```ts
export const SiteVisitConfig = {
  pageWebResource: `${WEB_RESOURCE_PREFIX}_site_visit_page.html`,   // prefix injected by the build
  dialogTitle: "Site visit",
  target: {
    table: "task",                                                    // logical name
    columns: { subject: "subject", description: "description", dueDate: "scheduledend", priority: "prioritycode" },
  },
  // The lookup to the source record, as "<NavigationProperty>@odata.bind"
  sourceBinding: (sourceTable: string) => `regardingobjectid_${sourceTable}_task@odata.bind`,
} as const;
```

## schema.ts

What the feature expects to find in Dataverse; `pnpm test:dataverse` checks it. Its type is
`FeatureSchema` from `@core/schema`. The example uses every key; the scaffold writes only
`tables` and `choices`.

```ts
export const schema: FeatureSchema = {
  feature: "site-visit",
  tables: [{ table: "task", columns: ["subject", "description", "scheduledend", "prioritycode"] }],
  choices: [{ table: "task", column: "prioritycode", values: [2], labels: ["High"] }],
  records: [{ table: "pidev_visittype", nameColumn: "pidev_name", contains: "Site inspection" }],
  relationships: [{ schemaName: "pidev_sitevisit_inspector" }],
};
```

| Key | Checked |
|---|---|
| `tables` | the table exists and has every column (all missing columns are listed) |
| `choices` | the column has options; `values` the code compares against exist; `labels` the code compares against exist |
| `records` | a record whose `nameColumn` contains `contains` exists (for records the code finds by name) |
| `relationships` | the N:N relationship's intersect table exists |

Build `tables` and `choices` from `config.ts` so the two never disagree.

## command/index.ts

Exported functions become `<Name>.<function>` in the app.

```ts
export function open(primaryControl: Xrm.FormContext): Promise<void> {
  return openRecordDialog(primaryControl, Config.pageWebResource, { title: Config.dialogTitle, width: 50, height: 60 });
}
```

The button runs it as Library `<prefix>_<name>_command.js`, Function `<Name>.open`, parameter
`PrimaryControl`: the [Command Buttons tool](../tools/command-buttons.md) writes that from `buttons`
in `webresources.config.mjs`. Form event handlers are exported the same way.

## fields.ts

The form as data, with the builders of the [UI kit](ui-components.md):

```ts
export const fieldSpecs = {
  subject: text({ id: "subject", label: "Subject", required: true, maxLength: 200 }),
  priority: choice({ id: "priority", label: "Priority", table, column: columns.priority, placeholder: "Select a priority" }),
};
export type FieldName = keyof typeof fieldSpecs;
```

The page draws a field with `<Field name="subject" />`; the required check reads the same specs.

## &lt;Name&gt;Page.tsx

The page component. In a small feature it holds everything; when it grows, move the behaviour
into a `use<Name>.ts` hook and the state changes into a `formState.ts`, as a larger page would.

| Piece | Job |
|---|---|
| state: `values`, `errors`, `options`, `source` | what the user entered, the messages, the loaded lists, the record the dialog was opened from |
| `useSaveBar(onSave)` | the Save button's states; `onSave` is the save function below |
| the loading effect | reads the source record and the options once, then `bar.ready()` (or `bar.loadFailed(error)`) |
| `isVisible(name)` | which fields are shown; a field that is not shown is not required |
| `toValues` / `toPayload` | the typed values for the rules; values mapped to the columns in `config.ts` |
| the save function | `checkRequired`, then `rules.validate`, then `createThen` with the follow-up writes |
| `fillTestData()` | sample values for the test-data button (non-production only) |
| the returned JSX | `FormContext.Provider`, `Page`, `Section`, `Row`, `Field`, `SaveBar` |

## logic/rules.ts

The form's values as an interface and its validation as pure functions. `validate(values,
today)` returns `{ field, message }` for the first problem, or `null`. No React, no DOM, no Xrm, so tests
call it directly. Required fields are checked by `checkRequired`, not here.

## page/index.tsx, page.html, page.css

- `page/index.tsx` is one line, `mountPage(<YourPage />)` (from `@ui`): once the document has loaded it
  creates the React root in `#root` and renders the page inside `<StrictMode>`.
- `page.html` is a shell: it references `page.css` and `page.js` by their plain names (the build
  rewrites these to the files' full `<prefix>_<name>_*` names so they resolve in Dataverse) and has
  `<div id="root">`, which React fills.
- `page.css` holds only styles specific to this page; each component's styles are in its own folder (`src/ui/<name>/<name>.css`), and the build puts the ones the page uses in front of this file. Markup is JSX and styles belong in `.css` files, never in strings or `style` props.
