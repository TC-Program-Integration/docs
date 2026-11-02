---
sidebarTitle: "Core API"
---

# Core API (`src/core`)

> **Reference.** Every function of the shared core. Import from `"@core"`.
> Each has a unit test in `test/unit/core/`.

## Xrm and ids (`xrm.ts`)

| Function | Use |
|---|---|
| `getXrm()` | The client API, from the page or its parent frame (dialogs). When neither has a native Xrm but the page has `GetGlobalContext` (a navigation page loaded with `ClientGlobalContext.js.aspx`), it returns the standalone client described below instead. Throws a clear message otherwise. |
| `cleanGuid(id)` | Drops `{}` and lowercases, so ids from forms, lookups and the Web API compare equal. |
| `sameGuid(a, b)` | Compares two ids whatever their braces or case. |

## Web API values (`webApi.ts`)

| Function | Use |
|---|---|
| `DataRecord` | The type of a record from the Web API: `Record<string, unknown>`. |
| `readLookup(record, "ownerid")` | `{ id, name }` of a lookup from a retrieved record (select `_ownerid_value`), or `null`. |
| `textOf(record, property)` | A text property, `""` when it is missing or not text. |
| `lookupValueProperty("ownerid")` | `"_ownerid_value"`, the property to `$select`. |
| `formattedValue(record, property)` | The display text of a choice, lookup or date. |
| `bindPath(entitySet, id)` | `"/accounts(<id>)"` for `<nav>@odata.bind`. Prefer `Metadata.recordPath`, which finds the entity set. |
| `odataString(value)` | A value quoted and escaped for `$filter`. Always use it for user input. |
| `getJson<T>(path, version?)` | GET a path under `/api/data/<version>/` that Xrm.WebApi does not wrap (metadata). Default version `WEB_API_VERSION` (`"v9.2"`). Throws `WebApiError`, which has the HTTP `status`. |

## Metadata (`metadata.ts`), cached per page

| Function | Use |
|---|---|
| `Metadata.entity(table)` | Entity set name, primary id and name columns. |
| `Metadata.recordPath(table, id)` | The `@odata.bind` value with the real entity set name (`task` → `tasks`, custom plurals...). |
| `Metadata.columnOptions(table, column)` | Options of a Choice, Status or Yes/No column, in the column's order. |
| `Metadata.recordOptions(table, filter?)` | Active records as options (id, primary name), sorted by name. |
| `Metadata.manyToMany(schemaName)` | An N:N relationship's definition, or `null` when it does not exist. It has both tables' logical names and both navigation property names (`Entity1NavigationPropertyName`, `Entity2NavigationPropertyName`), the names used in `$expand`. |
| `Metadata.clearCache()` | Forgets what was cached (tests, or after a metadata change in the same session). |

## Dialogs and navigation (`navigation.ts`)

| Function | Use |
|---|---|
| `openRecordDialog(formContext, webResource, { title, width, height, extra, unsavedMessage })` | From a command: opens the page as a centered modal for the form's record, `width` x `height` percent of the window (default 60 x 70). Asks to save a new record first (`unsavedMessage`, default "Save the record first."). Resolves when the dialog closes. |
| `openDialog(webResource, { title, width, height, extra })` | From a command with no record, e.g. a grid button that acts on every row: opens the page as a centered modal with only the `extra` values. The page reads them with `readDialogData()`; `readSourceRecord()` throws there. Resolves when the dialog closes. |
| `readSourceRecord()` | In the page: the `{ table, id }` it was opened for. |
| `readDialogData()` | In the page: all values passed, including `extra`, as `URLSearchParams`. |
| `openRecord(table, id)` | Opens a record's form (e.g. the one just created). |

## Writing (`records.ts`)

| Function | Use |
|---|---|
| `createThen(table, payload, finish)` | Creates a record, runs the follow-up writes in `finish(id)`, and returns the new id. If `finish` fails, deletes the record again and throws; the message says whether the delete worked. |
| `associate({ table, id }, relationship, { table, ids })` | Links records through an N:N relationship (schema name) in one request. Does nothing for an empty `ids`. |

## Errors (`errors.ts`)

| Function | Use |
|---|---|
| `errorMessage(error)` | The text of anything thrown: an error's `message`, a string as it is, anything else as JSON. Use it in every `catch (error)`. |

## The user and the environment (`user.ts`, `environment.ts`)

| Function | Use |
|---|---|
| `currentUser()` | `{ id, name }` from the client context, no request. |
| `currentUserRoles()` | `[{ id, name }]`: the user's security roles (including the roles from their teams), from the client context, no request. |
| `currentUserHasRole(...names)` | `true` when the user has at least one of the named roles (case-insensitive). Use it in a command's enable rule to show a button to some roles only; the table's privileges stay the real access control. |
| `currentUserRecord(columns)` | Columns of the user's own systemuser record, or `{}` when they cannot read it. |
| `isNonProduction(rules?)` | True in a non-production org, by whole words of the org host and unique name (`contoso-dev2` matches, `devonshire` does not), and on `localhost` (the preview). Fails closed: any error means production. Use it to show test helpers only where safe. |
| `defaultEnvironmentRules` | The words it looks for: `dev`, `test`, `tst`, `uat`, `qa`, `sandbox`, `sbx`, `preview`; hosts `localhost`, `127.0.0.1`. Pass your own `{ keywords, hostNames }` to change them. |

## FeatureSchema (`schema.ts`)

The type of a feature's `schema.ts`; see [feature files](feature-files.md#schemats).

## Standalone client (`standaloneClient.ts`)

`getXrm()` returns this client for a navigation page with no parent Xrm. It sends same-origin Web
API requests as the signed-in user, and implements only these operations (anything else is not
there):

| Operation | Use |
|---|---|
| `WebApi.retrieveMultipleRecords(table, query, maxPageSize)` | `{ entities, nextLink, fetchXmlPagingCookie, moreRecords }`: the paging values come from the FetchXML response (`moreRecords`, the paging cookie) or OData (`nextLink`). |
| `WebApi.retrieveRecord(table, id, query)` | One record by id. |
| `WebApi.createRecord(table, body)` | Creates a record and returns `{ id, entityType }`. |
| `WebApi.deleteRecord(table, id)` | Deletes a record. |
| `WebApi.online.execute` | The `Associate` operation only, for an N:N link (`$ref`). Any other operation throws. |
| `Utility.getEntityMetadata(table)` | The entity set name and the primary id and name columns, cached per page. |
| `Utility.lookupObjects({ entityTypes })` | Opens the record picker for the first table; resolves with the chosen record, or `[]`. |
| `Utility.getGlobalContext()` | The context the client was built from. |

It rejects a table name that is not letters, digits and `_`, and a record id that is not a GUID
(braces are removed first), before any request.

## Standalone lookup (`standaloneLookup.ts`)

| Export | Use |
|---|---|
| `openStandaloneLookup(options)` | Dispatches a `standalone-record-lookup` event for the mounted host and resolves with the chosen `Xrm.LookupValue[]` (`[]` when cancelled or when a newer lookup replaces it). Rejects with "The record picker is not available. Refresh the page." when no host answers. |
| `StandaloneLookup` | The event's detail: `table`, `info`, `base`, `request` and `resolve(records)`. |
| `standaloneLookupEvent` | The event name, `"standalone-record-lookup"`. |

The host that answers the event is `StandaloneLookupHost` in [UI components](ui-components.md#standalone-pages),
which `mountPage` always mounts.

## XML (`xml.ts`)

| Export | Use |
|---|---|
| `XmlNode` | A structured element: `{ name, attributes?, children? }`, where an attribute is a string or a number. |
| `serializeXml(root)` | Serializes an `XmlNode` tree to a string, with the attribute values escaped. Use it for FetchXML instead of string concatenation. |

## Adding to the core

`serializeXml(root)` serializes structured FetchXML with escaped attributes. Standalone pages
use `openStandaloneLookup(options)` to request a single-record selection from the React lookup
host installed by `mountPage`; the core contains no React components.

Rules in [conventions](../project/conventions.md#the-core): no project table names, a doc comment saying
when to use it, a unit test, exported from `index.ts`, and a line on this page.
The core knows nothing about React or the DOM; the form components are in [UI components](ui-components.md).
