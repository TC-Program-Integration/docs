---
sidebarTitle: "Reliability and cost"
---

# Flow patterns: 4. Reliability and cost

> **For:** a flow that will run unattended. **Result:** a flow that does not loop on itself, survives
> running twice, stays cheap and keeps secrets out of the run history. Part 4 of 6 of [Flow patterns](../flow-patterns.md).

A flow runs unwatched, possibly thousands of times, on data other people change. These are the
failures that appear after the first week, and what stops them.

## A flow that triggers itself

A flow triggered when a row of X is **modified**, that **updates** a row of X, starts itself again,
and again, until the platform throttles it. Use both defences:

1. **`filter_columns`** on the trigger: it fires only when one of those columns changed. A flow that
   writes other columns does not wake itself.
2. **A condition on the state**, so a second run finds nothing to do (the status is already "Done").

```python
from lib.flowkit import Flow, chain, condition, compose, dataverse, dataverse_trigger

flow = Flow(
    name="Mark as reviewed",
    key="MarkReviewed",
    trigger=dataverse_trigger("task", "Modified", filter_columns=["prioritycode"], filter_expression="prioritycode eq 2"),
    actions=chain(
        {
            "Check_not_done": condition(
                {"and": [{"not": {"equals": ["@triggerOutputs()?['body/statecode']", 1]}}]},
                then={
                    "Add_note": dataverse(
                        "UpdateRecord",
                        {
                            "entityName": "tasks",
                            "recordId": "@triggerOutputs()?['body/activityid']",
                            "item/description": "Reviewed",  # a column the trigger does not watch
                        },
                    )
                },
                otherwise={"Skip": compose("already done")},
            ),
        }
    ),
)
```

## Do the same thing twice, safely

Triggers can fire twice, and you will re-run a failed run by hand. A flow that **creates** something should
first look whether it is already there:

```python
from lib.flowkit import Flow, chain, condition, dataverse, dataverse_trigger

flow = Flow(
    name="One welcome task per contact",
    key="WelcomeTask",
    trigger=dataverse_trigger("contact", "Added"),
    actions=chain(
        {
            "Find_existing_task": dataverse(
                "ListRecords",
                {
                    "entityName": "tasks",
                    "$select": "activityid",
                    "$filter": "_regardingobjectid_value eq @{triggerOutputs()?['body/contactid']} and subject eq 'Welcome call'",
                    "$top": 1,
                },
            ),
            "If_none_yet": condition(
                {"and": [{"equals": ["@length(outputs('Find_existing_task')?['body/value'])", 0]}]},
                then={"Create_task": dataverse("CreateRecord", {"entityName": "tasks", "item/subject": "Welcome call"})},
            ),
        }
    ),
)
```

An **update** of a value is naturally safe to repeat. An **increment**, an **email**, a **payment** is not:
ask yourself "what if this ran twice?" for each.

## Cost and speed

| Do | Because |
|---|---|
| filter in the **trigger** (`filter_expression`, `filter_columns`) | a run that never starts costs nothing |
| `$select` only the columns you use, `$top` only the rows you need | less to transfer and to show in the history |
| `$filter` in the query, not `query()` after reading everything | the server does it |
| `$expand` related rows instead of one `GetItem` per row | one call instead of many |
| `query`/`select` instead of a loop that only filters or maps | one action instead of thousands |
| `foreach` with the default concurrency when rounds are independent | up to 20 at a time |
| `concurrency=1` when rounds share a variable or must keep their order | otherwise they overwrite each other |
| `delay` and `do_until` only when nothing can trigger the flow instead | a waiting run is still a run |
| `with_options(action, retries=2, timeout="PT5M")` on a call that can fail for a moment | a short outage does not fail the run, and a call that hangs ends |

## Limits to know

| Limit | What it means for you |
|---|---|
| A run lasts at most **30 days** | an approval that waits longer ends the run as timed out |
| A **synchronous** flow (one that answers a caller with `response`) must answer within about **2 minutes** | do the slow part after the response, or answer first and report later |
| Run history is kept for **28 days** | write what you need to keep into Dataverse |
| Each connection has **request limits** per period; a licence decides how many | a loop of thousands of calls can be throttled: keep `concurrency` modest and the filter tight |
| An array has a size limit (on the order of **100,000** elements) | select and filter first |

Microsoft sets these numbers and changes them. Check the current ones in the Power Platform
documentation ("requests limits and allocations", "limits of automated, scheduled and instant flows")
before you design around one.

## Secrets and data in the run history

The run history shows every action's inputs and outputs to anyone who can open the run. Keep secrets out:

- never write a password or key into `headers`, `uri` or `body`; keep it in an **environment variable** or
  a **connection** (the connector stores it for you);
- for an action whose inputs or outputs are sensitive, turn on secure inputs and outputs, which hides them
  in the history: `with_options(action, secure=True)`, or `runtimeConfiguration={"secureData": {"properties":
  ["inputs", "outputs"]}}` on the action (`connector(...)` and `http(...)` accept it as an extra argument);
- a flow runs with its **connections' identity**, not the person who caused the run. Use a service or team
  account for connections, so the flow does not stop when a person leaves, and so you know whose rights it has.

## Testing and debugging

Unit tests that need no Power Automate, and how to read a failed run, are in
[Rules and testing](../build-a-flow/03-rules-and-testing.md#test-it).

---

**Previous:** [3. Variables, branches and failure](03-variables-and-failure.md) · **Next:** [5. Connectors](05-connectors.md)
