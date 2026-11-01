---
sidebarTitle: "Solution and table"
---

# Deploy, the first time: solution and table

> **For:** step 1 and 2 of [the first time](01-first-time.md). **Result:** a solution with the
> project's publisher, and the *Callback request* table inside it, checked from your laptop.

## 1. The solution

Every piece you create gets the publisher's prefix. Use the project's publisher, never the default
one ("Default Publisher", prefix `new`), or your names will not match the code.

1. Find it: `pnpm dv:q tools/queries/publishers.xml`. Its *unique name* and *prefix* go in
   `flows/solution.json` (`publisher`, `prefix`). If they are still the placeholders
   (`MyPublisher`, `pidev`), ask your mentor for the real ones and write them there.
2. In Power Apps: **Solutions > New solution**. Display name *Callback request*, name
   `CallbackRequest`, the **project's** publisher, version `1.0.0.0`. **Create**.

From now on, work **inside this solution** (click it in the list). What you create from inside it is
added to it; a table created outside it has to be added later, by hand.

## Or: the solution and the table as code

Instead of step 2 of the solution and all of section 2, the [Schema as Code](../../8-reference/tools/schema-as-code.md)
tool can make them from code. The publisher (step 1) must exist either way.

1. `pnpm schema:init`: writes `schema/` with this exact table, the choice numbers 0, 1, 2 and the
   French labels ([Define the schema](../../4-build/define-the-schema.md)).
2. `pnpm build`: writes `dist/schema.json`.
3. In XrmToolBox, open **Schema as Code** on the development environment, **Repo folder...**, then
   **Push ticked**. It creates the solution, the table, its columns and the choice, and publishes.

The lookup's navigation property is then `CallbackRequest.columns.contact.navigationProperty` in code: no
copying from the portal. Skip to [Check the table](#check-the-table-from-your-laptop).

## 2. The table

[The brief](../../3-first-feature/first-feature/01-brief.md#where-the-data-goes) gives the exact
names. Inside the solution: **New > Table > Table** (not "Table from external data").

| Setting | Value |
|---|---|
| Display name | `Callback request` |
| Plural display name | `Callback requests` |
| Schema name (logical) | `pidev_callbackrequest` (the prefix is added; check the full name under **Advanced options**) |
| Primary column | display name `Topic`, schema name `pidev_topic` |

Then **New > Column** for each of the others, *inside this table*:

| Display name | Schema name | Type | Notes |
|---|---|---|---|
| Preferred date | `pidev_preferreddate` | Date only | Behavior **Date only**, not "Date and time" |
| Priority | `pidev_priority` | Choice | a **new local** choice: Low, Normal, Urgent. Set the numbers (below) |
| Phone | `pidev_phone` | Text | format Phone |
| Notes | `pidev_notes` | Text, multiple lines | maximum length 500 or more |
| Contact | `pidev_contact` | Lookup | to the **Contact** table; keep the relationship's defaults |

**Choice numbers.** Dataverse gives a new option a long number such as `100000002`, not `2`. The code
and the flow use `0`, `1`, `2`. Edit each option and set its **value**: Low 0, Normal 1, Urgent 2. Do
this before any record exists; changing it later does not change saved records.

**The lookup's name.** The page sets the link with the relationship's *navigation property*
(`pidev_Contact`), and **the letter case matters**. After saving, open the table > **Relationships** >
the one to Contact, and copy its *Lookup column* and *Relationship name* into `config.ts` if they
differ from your guess. The flow reads the same link as `_pidev_contact_value`.

**Save the table**, then **Publish** it (the button on the table's page).

## Check the table from your laptop

```powershell
pnpm test:dataverse
```

It compares what the code uses (the feature's `schema.ts`) with the real
environment and names any table or column it did not find: fix the name in Dataverse or in the
code and run it again. It only reads; it cannot damage anything.

**Next:** [Web resources and button](03-web-resources-and-button.md).
