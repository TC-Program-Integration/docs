# Define the schema in code

> **For:** a project whose tables, columns and choices are written in TypeScript instead of made
> in the maker portal. **Result:** `schema/` describes the tables, `pnpm build` checks it and
> writes `dist/schema.json`, and the Schema as Code tool in XrmToolBox creates or updates them in
> a development environment.

Nobody edits XML. The solution exported from Dataverse still contains XML, but only as the output
that carries the tables to test and prod.

## 1. Start the folder

```powershell
pnpm schema:init --key myproject
```

It reads the publisher and its prefixes from `flows/solution.json` and writes:

| File | Holds |
|---|---|
| `schema/project.ts` | the project's key, a permanent id (never change it), and the solution the tables go into |
| `schema/callback-request.ts` | an example table: the one of the [first-feature tutorial](../3-first-feature/first-feature.md) |
| `schema/index.ts` | the list of tables, and the names the rest of the code imports |

The tables get **a solution of their own**, never the flows' solution: the flows' package is built
without them, and a managed import of it would remove them. The build refuses the flows' name.

## Grouping tables in folders

When a project has more than a handful of tables, group them by business area in subfolders of
`schema/` (for example `simulators/`, `requests/`, `evaluations/`). Keep shared parts in
`schema/shared/`: label helpers, existing tables, global choices, and column groups that several
tables use.

When one table needs several files to stay under the [175-line limit](../8-reference/conventions.md),
give it a folder of its own with an `index.ts` that defines the table and imports its parts:

```text
schema/
├── index.ts               the list of tables
├── project.ts
├── shared/
│   ├── about.ts           label helpers
│   └── choices.ts         global choices used by several tables
├── simulators/
│   └── simulator.ts       one file: the table is defined here
└── requests/
    ├── request/
    │   ├── index.ts       the Request table, importing its parts
    │   ├── intake.ts
    │   └── outcome.ts
    └── qtg-submission.ts  one file: the table is defined here
```

`schema/index.ts` and `schema/project.ts` stay at the top, as `schema:init` writes them. The build
loads `schema/index.ts` and follows its imports, so the other files can move (update their relative
imports). Moving files never changes the built `dist/schema.json`, as long as the table order in
`index.ts` and the column order stay the same. To check a reorganisation, compare the file's hash
before and after it: `Get-FileHash dist\schema.json`.

## 2. Write a table

One file per table (a table in a folder of its own is covered in
[Grouping tables in folders](#grouping-tables-in-folders)), listed in `schema/index.ts`:

```ts
import { choice, date, globalChoice, lookup, table, text } from "@schema-dsl";
import { project } from "./project";
import { Simulator } from "./simulator";

export const RequestType = globalChoice(
  project,
  "nopspi_RequestType",
  { display: { en: "Request type", fr: "Type de demande" }, description: { en: "…", fr: "…" } },
  { New: [0, { en: "New simulator", fr: "Nouveau simulateur" }], Update: [1, { en: "Update", fr: "Mise à jour" }] },
);

export const Request = table("nopspi_Request", {
  display: { en: "Request", fr: "Demande" },
  plural: { en: "Requests", fr: "Demandes" },
  description: { en: "An evaluation request", fr: "Une demande d'évaluation" },
  ownership: "user",
  primary: text("nopspi_Number", { display: { en: "Number", fr: "Numéro" }, description: { en: "…", fr: "…" }, autonumber: "NSEP-{SEQNUM:5}" }),
  columns: {
    type: choice("nopspi_Type", { display: { en: "Type", fr: "Type" }, description: { en: "…", fr: "…" }, set: RequestType }),
    dueDate: date("nopspi_DueDate", { display: { en: "Due date", fr: "Date d'échéance" }, description: { en: "…", fr: "…" } }),
    simulator: lookup("nopspi_SimulatorId", { display: { en: "Simulator", fr: "Simulateur" }, description: { en: "…", fr: "…" }, target: () => Simulator }),
  },
});
```

- **Names** are written once, as schema names (`nopspi_DueDate`); the logical name is their lower case.
- **Every label and description** has English and French: the type requires both.
- **Choice values** are written 0, 1, 2… and stored as the publisher's option prefix × 10,000 + n
  (897630000, 897630001…). Code reads `RequestType.New.value`, never the long number.
- Every builder and option: [Schema DSL](../8-reference/schema-dsl.md).

## 3. Use the names in code

```ts
import { Request, RequestType } from "@schema";

const set = Request.entitySetName;                      // "nopspi_requests"
const lookup = `${Request.columns.simulator.navigationProperty}@odata.bind`;
if (record.nopspi_type === RequestType.New.value) { … }
```

A renamed column is then a type error everywhere it is used, not a bug found in the app.

## 4. Build and check

```powershell
pnpm build            # or pnpm schema:build for the schema alone
```

Each problem is printed with its place, for example
`error tables.nopspi_request.columns.nopspi_phone.maxLength: The maximum length must be a whole number from 1 to 4000 (it is 0).`
Errors stop the build; warnings (such as a French text identical to the English one) don't.

## 5. Push to development

In XrmToolBox, connected to the development environment, open **Schema as Code**, pick the repo
folder, and tick what to create or update. It shows every difference before writing, never touches
production, and deletes only after you confirm. Then check the environment matches the files:

```powershell
pnpm test:dataverse
```

## 6. Carry it to test and prod

Export the tables' solution from development (managed) and import it, tables before flows, as in
[Export, import, go back](../6-ship/deploy/05-export-import.md).

## Good to know

- **Names can't change in Dataverse.** Renaming a column in code means a new column; set
  `previously: "nopspi_OldName"` on it and copy the data before deleting the old one.
- **Tables that point to each other** (or to themselves) need the target's type spelled out, or
  TypeScript can't work out either table: `target: (): Table => Simulator`, or the logical name
  `target: "nopspi_simulator"`.
- **Tables you don't own** (users, contacts, another project's tables) are `existingTable("systemuser")`:
  they can be lookup targets but are never changed.
