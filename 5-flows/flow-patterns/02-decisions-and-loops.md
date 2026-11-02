---
sidebarTitle: "Decisions and loops"
---

# Flow patterns: 2. Decisions and loops

> **For:** branching, repeating and waiting. **Result:** the right `condition`, `switch`, loop or
> wait. Part 2 of 6 of [Flow patterns](../flow-patterns.md).

## Decisions

**Two branches: `condition`.** `expression` is a dictionary with `and`, `or` or `not` around comparisons:
`equals`, `greater`, `greaterOrEquals`, `less`, `lessOrEquals`, `contains`, `startsWith`, `endsWith`. Each takes
a list of two values: an expression (`"@..."`) and what to compare it with.

```python
from lib.flowkit import Flow, chain, compose, condition, dataverse, dataverse_trigger

owner = "outputs('Get_owner')?['body/internalemailaddress']"

flow = Flow(
    name="Owner with an email",
    key="OwnerWithEmail",
    trigger=dataverse_trigger("contact", "Added"),
    actions=chain(
        {
            "Get_owner": dataverse(
                "GetItem",
                {"entityName": "systemusers", "recordId": "@triggerOutputs()?['body/_ownerid_value']"},
            ),
            "Check_email": condition(
                {"and": [{"not": {"equals": [f"@empty({owner})", True]}}]},
                then={"Use_email": compose(f"@{{{owner}}}")},
                otherwise={"Say_missing": compose("No email address")},
            ),
        }
    ),
)
```

The nesting can also be written with builders: `and_`, `or_`, `not_`, `equals`, `greater`, `less`, `contains`,
`is_empty`. The condition above is `and_(not_(is_empty("@" + owner)))`
([flowkit options](../../8-reference/flows/flowkit-options.md#condition-builders)).

A condition's `then` and `otherwise` are their own scopes: an action inside one cannot be waited on by an
action outside, and `chain()` orders each side separately.

**Several ways: `switch`.** One value, one branch per possible value, and a `default`. Compare with the
value's **type**: a Choice column is a number, so the keys are integers.

```python
from lib.flowkit import Flow, chain, compose, dataverse_trigger, switch

flow = Flow(
    name="Route by priority",
    key="RouteByPriority",
    trigger=dataverse_trigger("pidev_callbackrequest", "Added"),
    actions=chain(
        {
            "Route": switch(
                "@triggerOutputs()?['body/pidev_priority']",
                cases={
                    0: {"Note_low": compose("low")},
                    2: {"Note_urgent": compose("urgent")},
                },
                default={"Note_normal": compose("normal")},
            ),
        }
    ),
)
```

## Loops

**`foreach(items, actions, concurrency=None)`** runs the actions once per element of an array; inside, the element
is `items('<the loop's name>')`. Loops run their rounds **in parallel** by default (up to 20 at once). Set
`concurrency=1` whenever rounds share a variable or must keep their order; the order of the array is only
kept then.

**`query` and `select`** work on a whole array without a loop: `query(source, "@equals(item()?['statecode'], 0)")`
keeps some elements; `select(source, {"id": "@item()?['id']"})` reshapes each. Prefer them to a loop that only
filters or maps. Inside them the element is `item()`.

**`do_until(expression, actions, count=60, timeout="PT1H")`** repeats until the expression is true. Use it to
**wait for something** (a status, an answer). Always set both limits low: a loop that never succeeds runs
until a limit stops it, and you pay for every round.

```python
from lib.flowkit import (
    Flow, chain, condition, dataverse, delay, do_until, init_variable, manual_trigger, set_variable, terminate_failed,
)

flow = Flow(
    name="Wait for approval",
    key="WaitForApproval",
    trigger=manual_trigger({"RecordId": "string"}),
    actions=chain(
        {
            "Initialize_Approved": init_variable("Approved", "boolean"),
            "Poll_until_approved": do_until(
                "@equals(variables('Approved'), true)",
                {
                    "Read_status": dataverse("GetItem", {"entityName": "tasks", "recordId": "@triggerBody()?['RecordId']"}),
                    "Remember_status": set_variable(
                        "Approved", "@equals(outputs('Read_status')?['body/prioritycode'], 2)"
                    ),
                    "Wait_a_minute": delay(1, "Minute"),
                },
                count=30,
                timeout="PT45M",
            ),
            "Check_outcome": condition(
                {"and": [{"equals": ["@variables('Approved')", False]}]},
                then={"Give_up": terminate_failed("NoAnswer", "Nobody answered within 30 minutes.")},
            ),
        }
    ),
)
```

To wait for a date, use `delay_until("2026-12-01T09:00:00Z")`; for a fixed pause, `delay(5, "Minute")`.
A waiting run still counts as running: when you can, let a trigger (the row changes) start the flow
instead of polling.

---

**Previous:** [1. Triggers and Dataverse data](01-triggers-and-data.md) · **Next:** [3. Variables, branches and failure](03-variables-and-failure.md)
