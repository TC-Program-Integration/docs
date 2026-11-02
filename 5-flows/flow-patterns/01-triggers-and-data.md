---
sidebarTitle: "Triggers and data"
---

# Flow patterns: 1. Triggers and Dataverse data

> **For:** starting a flow and reading or writing Dataverse rows. **Result:** the right trigger and
> data actions. Part 1 of 6 of [Flow patterns](../flow-patterns.md).

## Triggers

A flow has exactly one trigger. It decides **when a run starts** and what the first actions can read.

| Helper | Starts when | The first actions read it with |
|---|---|---|
| `recurrence(frequency, interval, time_zone, hours, minutes, week_days)` | the clock says so | nothing: ask Dataverse for what you need |
| `dataverse_trigger(table, message, scope, filter_columns, filter_expression)` | a row is added, changed or deleted | `@triggerOutputs()?['body/<column>']` |
| `manual_trigger({"Name": "string"})` | a person presses a button | `@triggerBody()?['Name']` |
| `http_request_trigger(schema)` | another system calls its web address | `@triggerBody()?['<property>']` |
| `sharepoint_trigger(site, list_id, message)`, `outlook_trigger(folder)` | a SharePoint item is created or modified, an email arrives ([example](05-connectors.md#helpers-for-common-connectors)) | `@triggerOutputs()?['body/<column>']` |

`trigger_options(trigger, conditions=..., split_on=..., concurrency=...)` adds start conditions to any of them
([flowkit options](../../8-reference/flows/flowkit-options.md#trigger-options)).

**A schedule.** `time_zone` takes Windows time zone names (`"Romance Standard Time"`, `"Pacific Standard Time"`);
the default is Eastern. `hours` and `minutes` pin the time of a daily or weekly run.

```python
from lib.flowkit import Flow, chain, compose, recurrence

flow = Flow(
    name="Monday digest",
    key="MondayDigest",
    trigger=recurrence("Week", 1, time_zone="Romance Standard Time", hours=[8], minutes=[0], week_days=["Monday"]),
    actions=chain({"Say_hello": compose("@{utcNow()}")}),
)
```

**A row change, but only some rows.** Filter in the trigger, not in a condition after it: a filtered-out row
starts no run at all, so the run history and the request quota only count real work.

```python
from lib.flowkit import Flow, chain, compose, dataverse_trigger

flow = Flow(
    name="Escalated case",
    key="EscalatedCase",
    trigger=dataverse_trigger(
        "incident",
        "Modified",
        filter_columns=["prioritycode"],  # only a change of this column starts a run
        filter_expression="prioritycode eq 1",  # and only when it is now High
    ),
    actions=chain({"Note_case": compose("@{triggerOutputs()?['body/title']}")}),
)
```

`message` is `Added`, `Modified`, `AddedOrModified` or `Deleted`; `scope` is who owns the rows you want
(`User`, `BusinessUnit`, `ParentChildBusinessUnits`, `Organization`). `filter_columns` matters most:
without it, **every** update of the row starts a run, including the ones your own flow makes (see
[Reliability](04-reliability.md#a-flow-that-triggers-itself)).

**A button and a web address.** `manual_trigger` takes typed inputs (`"string"`, `"number"`, `"boolean"`).
`http_request_trigger` takes a JSON schema of the body, and the flow must answer with `response(...)`
(see [Web services](06-teams-and-web.md#web-services)).

## Reading Dataverse

Dataverse actions take the table's **entity set name** (`contacts`, `tasks`; the plural), not its logical name.
The trigger takes the **logical** name (`contact`). Look both up with `pnpm dv:q` ([query Dataverse](../../4-build/read-real-data.md)).

| Operation | Parameters | Result |
|---|---|---|
| `ListRecords` | `entityName`, `$select`, `$filter`, `$orderby`, `$top`, `$expand` | `@outputs('Name')?['body/value']` is the array of rows |
| `GetItem` | `entityName`, `recordId`, `$select`, `$expand` | `@outputs('Name')?['body/<column>']` is one value |

```python
from lib.flowkit import Flow, chain, compose, dataverse, foreach, manual_trigger

flow = Flow(
    name="Open tasks of an account",
    key="OpenTasks",
    trigger=manual_trigger({"AccountId": "string"}),
    actions=chain(
        {
            "List_open_tasks": dataverse(
                "ListRecords",
                {
                    "entityName": "tasks",
                    "$select": "subject,scheduledend,prioritycode",
                    "$filter": "_regardingobjectid_value eq @{triggerBody()?['AccountId']} and statecode eq 0",
                    "$orderby": "scheduledend asc",
                    "$top": 50,
                    "$expand": "owninguser($select=fullname,internalemailaddress)",
                },
            ),
            "For_each_task": foreach(
                "@outputs('List_open_tasks')?['body/value']",
                chain({"Note_task": compose("@{items('For_each_task')?['subject']}")}),
            ),
        }
    ),
)
```

What to remember when reading:

- **Lookups** come back as `_<column>_value` holding the id (`_regardingobjectid_value`, `_ownerid_value`). In a
  `$filter` the id is written without quotes: `_ownerid_value eq <guid>`.
- **Choice columns** come back as the option **number**. The label is in an annotation:
  `@{items('Loop')?['prioritycode@OData.Community.Display.V1.FormattedValue']}`.
- **Related rows**: `$expand` reads them in the same call (`owninguser($select=fullname)` above), so you do
  not need one `GetItem` per row. A lookup you already hold the id of is a `GetItem`.
- **A text value in a filter** is quoted, and a quote inside it is doubled: use
  `odata_quote(expression)` ([flowkit](../../8-reference/flows/flowkit.md#expressions)).
- **A date in a filter** is an ISO text without quotes: `scheduledend lt @{utcNow()}`.
- **More than one page of rows**: a list returns up to 5000 rows. To read all of a larger table,
  switch on pagination, which the helper does not set: pass it on the action itself.

```python
from lib.flowkit import connector

list_all = connector(
    "shared_commondataserviceforapps",
    "ListRecords",
    {"entityName": "contacts", "$select": "fullname"},
    runtimeConfiguration={"paginationPolicy": {"minimumItemCount": 20000}},
)
```

SharePoint lists are read with `sharepoint_items(site, list_id)`, which already pages.

## Writing Dataverse

| Operation | Parameters |
|---|---|
| `CreateRecord` | `entityName`, `item/<column>`: value, `item/<NavigationProperty>@odata.bind`: `/<entityset>(<id>)` for a lookup |
| `UpdateOnlyRecord` | `entityName`, `recordId`, `item/<column>`: value. Fails when the row does not exist. |
| `UpdateRecord` | the same parameters, but it **creates** the row when it does not exist (Upsert a row) |
| `DeleteRecord` | `entityName`, `recordId` |

`add_row`, `update_row`, `delete_row` and `list_rows` write these actions for you
([flowkit](../../8-reference/flows/flowkit.md#connector-actions)); `update_row` is `UpdateOnlyRecord`.

```python
from lib.flowkit import Flow, chain, dataverse, dataverse_trigger

flow = Flow(
    name="Follow-up task",
    key="FollowUpTask",
    trigger=dataverse_trigger("pidev_callbackrequest", "Added"),
    actions=chain(
        {
            "Create_follow_up": dataverse(
                "CreateRecord",
                {
                    "entityName": "tasks",
                    "item/subject": "Call back: @{triggerOutputs()?['body/pidev_topic']}",
                    "item/scheduledend": "@{addDays(utcNow(), 1)}",
                    "item/regardingobjectid_contact_task@odata.bind": "/contacts(@{triggerOutputs()?['body/_pidev_contact_value']})",
                },
            ),
        }
    ),
)
```

- A lookup is set **by name of its navigation property**, which is case-sensitive and often the schema name
  (`pidev_Contact`). A wrong letter case is the most common reason a create fails with "undeclared property".
- To clear a column, write the literal `None` (JSON `null`), never an empty text.
- To change a row by a rule the flow itself caused, read [A flow that triggers itself](04-reliability.md#a-flow-that-triggers-itself).

---

**Previous:** [Flow patterns](../flow-patterns.md) · **Next:** [2. Decisions and loops](02-decisions-and-loops.md)
