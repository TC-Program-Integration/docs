# Mock Dataverse

> **Reference.** The fake Xrm used by the preview and the tests: its data format, what it
> supports, and the test helpers built on it.

## Files

| File | Role |
|---|---|
| `tools/preview/mock-xrm.js` | the fake `Xrm` object (its row lookup and filtering are in `mock-query.js`) |
| `tools/preview/mock-metadata.mjs` | answers the Web API metadata requests (`EntityDefinitions`, choices, relationships) |
| `tools/preview/mock-data.json` | data for the preview; edit freely |
| `test/fixtures/mock-data.json` | data for the tests; change only together with the tests |
| `test/helpers/mockDataverse.ts` | unit test helpers |
| `test/e2e/helpers/featurePage.ts` | browser test helper |

## Data file format

```json
{
  "user": { "id": "11111111-1111-1111-1111-111111111111", "fullname": "Preview User" },
  "sources": ["account", "contact"],
  "tables": {
    "account": {
      "nameField": "name",
      "records": [
        { "accountid": "a1000000-0000-0000-0000-000000000001", "name": "Air Test Ltd." },
        { "accountid": "a1000000-0000-0000-0000-000000000009", "name": "Closed", "statecode": 1 }
      ]
    },
    "task": { "nameField": "subject", "idField": "activityid", "entitySet": "tasks", "records": [] }
  },
  "choices": {
    "task.prioritycode": [[0, "Low"], [1, "Normal"], [2, "High"]],
    "isactive": "boolean"
  },
  "relationships": {
    "prefix_table1_table2": {
      "Entity1LogicalName": "prefix_table1", "Entity2LogicalName": "prefix_table2",
      "Entity1NavigationPropertyName": "prefix_table1_table2s", "Entity2NavigationPropertyName": "prefix_table2_table1s"
    }
  }
}
```

| Key | Meaning |
|---|---|
| `user` | the signed-in user (also needs a `systemuser` record to read its columns); an optional `roles` (`[{ "id", "name" }]`) is what `currentUserRoles()` returns, none by default |
| `previewUsers` | optional: users the preview bar's **Preview as** list can switch to (`[{ "id", "fullname" }]`); switching reopens the page as that user |
| `sources` | tables the preview offers to open a page from |
| `tables.<t>.nameField` | primary name column |
| `tables.<t>.idField` | primary key, default `<t>id` |
| `tables.<t>.entitySet` | entity set name, default `<t>s` |
| `tables.<t>.records` | rows; a lookup is `{ "id": ..., "name": ... }`; a row is active unless `"statecode": 1` |
| `choices` | `"table.column"` (or `"column"` for every table): `[[value, "Label"], ...]`, or `"boolean"` for Yes/No |
| `relationships` | N:N definitions returned by `Metadata.manyToMany`: both logical names and both navigation property names (a definition without them is incomplete) |

In the preview bar, **No source record** (or `&hub=1` in the address) opens the page without a
source record, as a navigation page is opened from the app's menu.

## What the fake supports

| Area | Supported |
|---|---|
| `Xrm.WebApi` | `retrieveRecord`, `retrieveMultipleRecords`, `createRecord`, `updateRecord`, `deleteRecord`, `online.execute` (Associate, Disassociate) |
| Query options | `$select`, `$top`, `$orderby` (one column, `asc`/`desc`), `$filter` with `eq` / `ne` conditions joined by `and` |
| FetchXML (`?fetchXml=`) | `attribute`, `order`, `count` / `page` paging (with `moreRecords`), inner and outer `link-entity` joins, nested `and` / `or` filters with `eq`, `ne`, `null`, `not-null` conditions; anything else fails instead of returning unfiltered rows |
| `Xrm.Utility` | `getGlobalContext` (client URL, org name `local-preview`, user), `getEntityMetadata` |
| `Xrm.Navigation` | `openAlertDialog`, `openConfirmDialog`, `openErrorDialog`, `openForm`, `navigateTo` |
| Metadata requests | attribute type, choice and Yes/No options, N:N relationship definitions |

Not simulated: other filter operators (they are **ignored**, so a filter on `contains` returns
every row), security roles, plugins, business rules, calculated columns, server validation.
Writes are never saved: reads keep returning the file's data.

## Recorded writes and alerts

Creates, updates, deletes, Associate/Disassociate, `openForm` and `navigateTo` are recorded in
order as `{ type, table, id, payload }`. Alerts and error dialogs are recorded as text. The
preview shows both in its log panel and terminal.

To make an operation fail, set a failure by operation name (`createRecord`, `updateRecord`,
`deleteRecord`, `Associate`, ...) with the helpers below.

## Unit test helpers

From `test/helpers/mockDataverse.ts`:

| Helper | Does |
|---|---|
| `installMockDataverse(edit?)` | installs `window.Xrm` on a fresh copy of the test fixture; `edit(data)` can change it first. Returns `dv`. |
| `dv.writes()` | the recorded writes |
| `dv.alerts()` | the alerts and error dialogs shown |
| `dv.fail({ createRecord: "Network error" })` | makes those operations throw |
| `dv.data` | the data in use |
| `openWithRecord(table, id)` | sets the URL a dialog receives from `openRecordDialog`; call it before `render(<Page />)` |
| `loadFixture()` | a fresh copy of the fixture data |
| `Ids` | record ids in the fixture (`user`, `account`, `otherAccount`, `closedAccount`, `contact`) |

To act on a rendered page use React Testing Library (`render`, `screen`, `userEvent`); see [write tests](../4-build/test-and-check/01-write-tests.md#2-test-the-state-and-the-page-on-the-mock).

## Browser test helper

`new FeaturePage(page, "<feature>")` from `test/e2e/helpers/featurePage.ts`:

| Method | Does |
|---|---|
| `open({ table, id })` | opens the feature in the preview for that record and waits until Save is enabled |
| `field(id)` | a locator inside the page's iframe |
| `fill(id, text)`, `select(id, label)`, `click(id)` | user actions |
| `writes()`, `alerts()` | what the page wrote and showed |

Browser dialogs (alerts, confirms) are accepted automatically.