# Schema DSL

> **Reference.** Every builder of `@schema-dsl` (`src/schema-dsl/`) and its options. How to use them:
> [Define the schema in code](../4-build/define-the-schema.md). The file they produce:
> [dist/schema.json](configuration.md#distschemajson).

Every label is `{ en, fr }`; both are required. Every column takes `display` and `description` plus
the options of its kind.

## Project and schema

| Builder | Notes |
|---|---|
| `defineProject({ key, id, publisherFrom, solution, languages? })` | `publisherFrom` is `flows/solution.json` (publisher, prefix, option prefix). `solution` is `{ uniqueName, display }`, not the flows' solution. `id` is a GUID made by `pnpm schema:init`: never change it. `languages` defaults to `{ en: 1033, fr: 1036 }`. |
| `defineSchema(project, { tables, choices?, manyToMany?, existing? })` | The default export of `schema/index.ts`. Global choices and existing tables used by columns are found by themselves. `tables: []` is allowed and warns: pushing it deletes everything the solution owns. |

## Tables

`table("nopspi_Request", options)` returns a typed table: `Request.columns.<code>.logicalName`,
`Request.entitySetName`, `Request.primaryId`, `Request.state.Active` (0), `Request.status.Draft`.

| Option | Default | Notes |
|---|---|---|
| `display`, `plural`, `description` | | labels |
| `ownership` | | `"user"` or `"organization"`; can't change later |
| `primary` | | a `text()` column: the name shown in lists and lookups |
| `columns` | | `{ code: column }`; the code name is what the rest of the code uses |
| `keys` | none | `{ code: { display, columns: ["primary", "code"…] } }`; 1–16 columns, at most 10 keys, text, whole number, decimal, date, lookup or choice columns, about 900 bytes |
| `status` | none | `statusReasons(…)`: Status Reasons added to the built-in Active and Inactive |
| `audit` | `true` | |
| `notes`, `activities` | `false` | can't be turned off once on |
| `quickCreate` | `true` | |
| `changeTracking`, `duplicateDetection` | `false` | |
| `entitySetName` | English plural of the logical name | the Web API name of the records |
| `previously` | | the old schema name, when this table replaces a renamed one |

## Columns

Common options: `display`, `description`, `required` (`"none"`, `"recommended"`, `"required"`,
enforced by apps), `audit` (default `true`), `searchable` (default `true`, except for file and image columns), `previously`.

**Searchable.** `searchable` decides whether the column is offered in Advanced find and column filters. Dataverse never allows file or image columns there, so `file` and `image` columns default to `searchable: false`, and the serialized JSON says `"searchable": false` for them. Setting `searchable: true` on a file or image column is an error (`File and image columns can't be searchable in Dataverse: remove searchable or set it to false.`), because Dataverse would ignore it and the Schema as Code plugin would offer the same change on every scan. Every other column type is searchable unless you set `searchable: false`.

| Builder | Options |
|---|---|
| `text` | `maxLength` 1–4,000 (100); `format` `"text"`, `"email"`, `"url"`, `"phone"`, `"textArea"`; `autonumber` such as `"REQ-{SEQNUM:5}"`: one `{SEQNUM:n}`, plus `{RANDSTRING:1-6}` and `{DATETIMEUTC:format}` |
| `memo` | `maxLength` up to 1,048,576 (2,000) |
| `wholeNumber` | `min`, `max` |
| `bigNumber` | none |
| `decimal` | `precision` 0–10 (2), `min`, `max` |
| `money` | `precision` 0–4 (2), `min`, `max`; Dataverse adds the currency and exchange rate columns |
| `float` | `precision` 0–5 (2), `min`, `max` |
| `yesNo` | `yes`, `no` (labels; Yes/Oui, No/Non), `default` (`false`) |
| `date` | `behavior` `"dateOnly"` (default), `"userLocal"`, `"timeZoneIndependent"`; `format` `"date"` or `"dateTime"` (a date-only column always shows a date) |
| `choice`, `multiChoice` | `set`: a `globalChoice`, a `localChoice` or an `existingChoice`; `default`: a stored value (`Set.Key.value`) |
| `lookup` | `target`: `() => Table`, an `existingTable(…)`, or a logical name; `cascade` `"removeLink"` (default), `"restrict"` or `"parental"`; `relationship` (default `<prefix>_<target>_<table>_<column>`); `menu` (label of the related list) |
| `customer` | an account or a contact; `relationships: { account?, contact? }` |
| `file` | `maxSizeKb` up to 10,485,760 (32,768); not searchable (see Searchable above) |
| `image` | `maxSizeKb` up to 30,720 (10,240); `primary`: the table's picture; not searchable (see Searchable above) |

Lookups and customers also give `navigationProperty` (for `@odata.bind`) and `webApiValue` (`_<name>_value`).

## Choices and status reasons

| Builder | Notes |
|---|---|
| `globalChoice(project, "nopspi_RequestType", { display, description, values? }, { Key: [n, label, description?] })` | Shared by any column. `Set.Key.value` is the stored value. |
| `localChoice(project, { Key: [n, label] }, { values? })` | Belongs to one column. |
| `existingChoice("nopspi_status")` | A global choice the schema uses but doesn't own: never changed. |
| `statusReasons(project, { active?, inactive?, stateLabels?, values? })` | `active`/`inactive` are `{ Key: [n, label] }`; `stateLabels` renames the Active and Inactive states. Keys can't be `Active` or `Inactive`, values can't be 1 or 2. |

`values`: `"prefixed"` (default) stores the publisher's option prefix × 10,000 + n, with n 0–9,999;
`"raw"` stores n as written. Keys are PascalCase. A value can't change once created.

## Relationships and existing tables

| Builder | Notes |
|---|---|
| `manyToMany("nopspi_request_inspector", tableA, tableB, { intersect?, menuA?, menuB? })` | `intersect` defaults to the schema name in lower case; a table can be linked to itself. |
| `existingTable("systemuser", { primaryId?, entitySetName? })` | A lookup target the schema never changes. |

## Other functions

| Function | Does |
|---|---|
| `validate(schema)` | every problem, as `{ severity, path, message }`; used by the build |
| `serialize(schema, generatedBy)` | the `dist/schema.json` object; used by the build |
| `choiceOptions(set)` | a set's options in their written order |
