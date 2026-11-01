---
sidebarTitle: "A complete example"
---

# Build a flow: A complete example

> **For:** seeing every file of a flow split into modules. **Result:** a pattern to copy.
> Part 2 of 3 of [Build a flow](../build-a-flow.md). The same files are in `flows/examples/stale-visits/`.

A daily flow that closes Planned site visits older than 30 days and fails the run with the list of
visits it could not close. Table and column names are examples.

`settings.py`

```python
"""Where the flow reads and writes. Change a table or a day count here only."""

VISIT_SET = "pidev_sitevisits"  # entity SET name (the plural), not the logical name
STALE_AFTER_DAYS = 30  # a Planned visit older than this is closed
PROBLEMS = "Problems"  # name of the array variable that collects what went wrong
```

`expressions.py`

```python
"""Action names and the expressions the steps share."""

from .settings import PROBLEMS, STALE_AFTER_DAYS

# Action names: each step is declared under one of these, and expressions refer to it by it.
LIST_STALE = "List_stale_visits"
FOR_EACH_VISIT = "For_each_stale_visit"

visit = f"items('{FOR_EACH_VISIT}')"  # the visit the loop is on
stale_rows = f"outputs('{LIST_STALE}')?['body/value']"
cutoff = f"formatDateTime(addDays(utcNow(), -{STALE_AFTER_DAYS}), 'yyyy-MM-dd')"
context = f"Visit @{{{visit}?['pidev_name']}}: "  # prefix for an error recorded by catch()
problems = f"variables('{PROBLEMS}')"
```

`steps/find.py`

```python
"""Step 1: the Planned visits that are older than the cutoff."""

from lib.flowkit import dataverse, init_variable

from ..expressions import cutoff
from ..settings import PROBLEMS, VISIT_SET

initialize_problems = init_variable(PROBLEMS, "array")  # variables are initialized at the top level only

list_stale_visits = dataverse(
    "ListRecords",
    {
        "entityName": VISIT_SET,
        "$select": "pidev_sitevisitid,pidev_name",
        "$filter": f"pidev_status eq 1 and pidev_visitdate lt @{{{cutoff}}}",  # 1 = Planned
    },
)
```

`steps/close.py`

```python
"""Step 2: close each stale visit. A failed write is recorded and the loop carries on."""

from lib.flowkit import catch, chain, dataverse, foreach

from ..expressions import context, stale_rows, visit
from ..settings import PROBLEMS, VISIT_SET

close_visit = dataverse(
    "UpdateRecord",
    {
        "entityName": VISIT_SET,
        "recordId": f"@{visit}?['pidev_sitevisitid']",
        "item/pidev_status": 3,  # 3 = Closed
    },
)

# concurrency=1 because every iteration may append to the shared Problems variable.
for_each_visit = foreach(
    f"@{stale_rows}",
    chain({"Close_visit": close_visit, **catch("Close_visit", PROBLEMS, context)}),
    concurrency=1,
)
```

`steps/report.py`

```python
"""Step 3: fail the run, listing every problem, when there was one."""

from lib.flowkit import condition, terminate_with_list

from ..expressions import problems
from ..settings import PROBLEMS

if_any_problems = condition(
    {"and": [{"greater": [f"@length({problems})", 0]}]},
    then=terminate_with_list(PROBLEMS, "StaleVisitsFailed", "Report_problems"),
)
```

`flow.py`

```python
"""Closes every Planned site visit older than 30 days, once a day."""

from lib.flowkit import Flow, chain, recurrence

from .expressions import FOR_EACH_VISIT, LIST_STALE
from .steps.close import for_each_visit
from .steps.find import initialize_problems, list_stale_visits
from .steps.report import if_any_problems

flow = Flow(
    name="Close stale site visits",
    key="StaleVisits",
    description="Closes Planned site visits older than 30 days.",
    trigger=recurrence("Day", 1, hours=[6]),
    actions=chain(
        {
            "Initialize_Problems": initialize_problems,
            LIST_STALE: list_stale_visits,
            FOR_EACH_VISIT: for_each_visit,
            "If_any_problems": if_any_problems,
        }
    ),
)
```

The example type-checks under Pyright strict, is formatted by Black and passes `check()`. It has not
been run in Power Automate.

---

**Previous:** [Write the steps](01-write-the-steps.md) · **Next:** [Rules and testing](03-rules-and-testing.md)
