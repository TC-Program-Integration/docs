# Read real data

> **For:** looking at a real table before you write code, and keeping your code in step with it.
> **Needs:** pac signed in to the dev environment.

Three commands read Dataverse. All of them go through pac with your own sign-in, and all of them
only **read**: nothing here can change data or customizations
([how the tools reach Dataverse](../9-help/access-and-security.md)).

| Command | Use it to |
|---|---|
| `pnpm dv:q` | ask any question in FetchXML and see the rows |
| `pnpm dv:pull` | get a table's real names, choices and a few rows into `config.ts` and the mock data |
| `pnpm test:dataverse` | check that the environment still has everything each feature's `schema.ts` lists |

Each one takes another environment than pac's current one: `--env https://<org>.crm3.dynamics.com`
for `dv:q` and `dv:pull`, `$env:DATAVERSE_URL="https://<org>.crm3.dynamics.com"` for
`test:dataverse`.

## 1. Ask a question with pnpm dv:q

```powershell
pnpm dv:q tools/queries/my-user.xml
```

| Query file in `tools/queries/` | Shows |
|---|---|
| `my-user.xml` | your own user record: a quick check that pac is connected to the right org |
| `table-names.xml` | a table's logical name, entity set name and type code |
| `choice-options.xml` | the values and labels of a choice column |
| `recent-records.xml` | the latest rows of a table |
| `publishers.xml` | publishers with their unique name and prefixes |

Each file starts with a comment saying what to change. For your own query, copy the closest file,
change the entity and attributes, and run it:

```xml
<fetch count="10">
  <entity name="task">                       <!-- logical name, lowercase -->
    <attribute name="subject" />
    <attribute name="prioritycode" />
    <filter>
      <condition attribute="statecode" operator="eq" value="0" />
    </filter>
    <order attribute="createdon" descending="true" />
  </entity>
</fetch>
```

A short query can also be passed inline:

```powershell
pnpm dv:q "<fetch count='5'><entity name='account'><attribute name='name'/></entity></fetch>"
```

- Use `count`, not `top`: pac pages results itself (the tool converts `top` for you).
- Joins use `<link-entity>`; FetchXML can also aggregate. A view's FetchXML is a good start: in
  the classic Advanced Find, **Download FetchXML** gives it.
- The last query and its output are saved in `tools/.out/last-query.txt`.
- Choices come back as numbers and lookups as ids, as the API stores them; the app shows labels
  and names. Compare with the same row in the app when in doubt.

When the output shows a case your code must handle, add it to the mock data and to a test.

## 2. Pull a table into config and mock data

`pnpm dv:pull` reads a table's names so you do not type them by hand. Name the table (logical
name, lower case) and the columns your feature writes:

```powershell
pnpm dv:pull pidev_sitevisit --columns pidev_name,pidev_visitdate,pidev_status
```

It prints a `target` block to paste into `config.ts`, with the entity set as a comment, and the
choice columns it found:

```ts
  target: {
    table: "pidev_sitevisit", // Web API entity set: pidev_sitevisits
    columns: {
      name: "pidev_name",
      visitdate: "pidev_visitdate",
      status: "pidev_status",
    },
  },
```

Rename the keys as you like (`visitDate`); the values are the real column names.

| Option | Does |
|---|---|
| `--into tools/preview/mock-data.json` | adds the table (entity set, id column, rows) and its choice options to that mock data file. Run it again with `test/fixtures/mock-data.json` for the tests |
| `--name-field <column>` | the table's primary name column, written as `nameField` in the mock data |
| `--id-field <column>` | the id column; the default is `<table>id` (activities such as `task` use `activityid`) |
| `--records 3` | also takes 3 real rows |
| `--env <org url>` | another environment |

The result is also saved to `tools/.out/pull-<table>.json`.

<Warning title="Real rows are real data">

`--records` copies rows from the environment. Before you commit the mock data, replace names,
emails, phone numbers and anything else personal with made-up values.

</Warning>

`dv:pull` does not read relationships: the navigation property of a lookup (for `@odata.bind`)
and N:N relationship names come from Power Apps, table > **Relationships**. The mock data format
is in [Mock Dataverse](../8-reference/mock-dataverse.md#data-file-format).

## 3. Keep checking with pnpm test:dataverse

Each feature's `schema.ts` lists the tables, columns, choice values, records and N:N
relationships it relies on ([format](../8-reference/feature-files.md#schemats)).
`pnpm test:dataverse` checks every one of them against the environment:

```powershell
pnpm test:dataverse                                                        # pac's active environment
$env:DATAVERSE_URL="https://<org>.crm3.dynamics.com"; pnpm test:dataverse  # another one
```

A failure names the table and every missing column, value or label. Run it after you change
`config.ts`, and before you deploy, against each environment you deploy to. If pac is not signed
in, it says so and stops: sign in with `pac auth create --environment <org url>`.

**Next:** [Build a feature](build-a-feature.md)
