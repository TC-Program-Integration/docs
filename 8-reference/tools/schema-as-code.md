# Schema as Code

> **Reference.** The Schema as Code XrmToolBox tool, version 0.1.0: it creates and updates the
> tables, columns, choices, status reasons, relationships and keys of
> [`dist/schema.json`](../configuration.md#distschemajson) in a development environment, instead
> of making them by hand in the maker portal. How to write the schema:
> [Define the schema](../../4-build/define-the-schema.md); every builder:
> [Schema DSL](../schema-dsl.md).

## What you pick

| Setting | Remembered |
|---|---|
| the **project folder** (the tool reads `dist/schema.json` in it) | per environment |
| **Allow deletes here** (Settings) | per environment; off by default |

The solution and the publisher come from `dist/schema.json` (`schema/project.ts`). The publisher
must exist; the solution is created when it is missing.

Run `pnpm build` (or `pnpm schema:build`) first. When a file under `schema/`, `src/schema-dsl/` or
`flows/solution.json` is newer than `dist/schema.json`, the tool says so above the list.

## States

Every difference between `dist/schema.json` and the environment is one row, grouped by table.
Select a row to see each field, old -> new, or why it is blocked and what to do.

| State | Means | What to do |
|---|---|---|
| *Ready* | can be pushed; ticked for you | **Push ticked** |
| *Same* | already matches; hidden unless **Show unchanged** | nothing |
| *Adopt* | exists here but outside the solution (made by hand, or by another project) | tick it to add it to the solution, without its other columns, forms or views |
| *Needs confirmation* | cannot be undone: a date column's behavior (user local to date only), a key made again with other columns | tick it; you type the environment's name to go ahead |
| *Delete candidate* | in the solution, not in `schema.json` | see [Deletes](#deletes) |
| *Take out of solution* | not in `schema.json`, but also in another solution | tick it to remove it from this solution only; nothing is deleted |
| *Blocked* | Dataverse does not allow it, or it waits for a change that cannot be made | the details give the remedy |
| *Invalid* | `schema.json` itself is wrong here | fix `schema/`, `pnpm build`, reload |

Ticking a row ticks what it waits for; if that is an adoption or a change that cannot be undone,
you are asked first. Unticking a row unticks what waits for it.

What Dataverse never lets change, and the tool therefore shows as *Blocked*: a table's or column's
name (a different case included), a column's type, a lookup's target, a table's ownership, notes or
activities once turned on, and a key's name or label. To rename a column, add the new one with
`previously: "<old schema name>"`: the tool pairs the two rows, creates the new column, and lets you
delete the old one only in a later push, once the data is copied.

## Push ticked

1. Shows exactly what will be created, changed and deleted.
2. Reads the environment again: a component changed since the scan is skipped.
3. Applies in order: solution, choices, tables, columns, status reasons, relationships, keys,
   updates; publishes; then the confirmed deletes, and publishes again. New components go **into
   the solution**. An update changes only the fields that differ and keeps the other labels and
   settings. A failure skips what waits for it; the rest goes on.
4. The results window says what happened to each row (Copy gives the whole text, with the error
   codes). The list is read again: what is left can be pushed again.

Dataverse builds a new key's index in the background: until it is active, the key's row says so in
its details. When the index fails (usually duplicate values), the row offers to reactivate it once
the data is fixed. **Settings > Publish again** publishes the schema's tables and choices without
changing anything.

## Deletes

Off until **Settings > Allow deletes here** is ticked, for each environment. A delete candidate is
always in the schema's solution, unmanaged, custom and named with the project's prefix: components
of other projects, of Microsoft, and the columns Dataverse makes itself are never offered. One that
is also in another unmanaged solution is only taken out of this one.

Before deleting, the tool counts the rows holding data (table rows, filled columns, rows using an
option; "50,000+" above that) and you type the environment's name. Dataverse does not know which
flows or web resources use a column: search the repo first.

## The solution belongs to one schema

The tool writes `Schema as Code: <key> (<id>)` (from `schema/project.ts`) into the solution's
description. A solution marked for another schema is refused. A solution made by hand is used for
creates and updates only, until you confirm once that it belongs to this schema.

## Safety

Development and test only, asked once per environment; production, environments labelled PROD in
XrmToolBox, and environments where the solution is **managed** are refused. Each language the
schema uses (French included) must be enabled, or nothing is pushed. Managed components are never
changed. The tool uses your XrmToolBox connection and your rights, and only reads
`dist/schema.json`: it runs nothing from the repo. **Settings > Forget environment answers** asks
the dev/test question again and turns deletes off everywhere.

## Keys

F5 reads `schema.json` and the environment again; F1 opens the help.

## Install

`tools\xrmtoolbox\install.cmd` ([XrmToolBox tools](../tools.md#install-or-update-them)); the zip
holds `SchemaAsCode.dll`.
