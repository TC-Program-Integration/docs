---
sidebarTitle: "Variables, branches and failure"
---

# Flow patterns: 3. Variables, branches and failure

> **For:** counters, text, parallel branches and failures. **Result:** a flow that reacts to a failure
> on purpose. Part 3 of 6 of [Flow patterns](../flow-patterns.md).

## Variables

Variables hold a value you change as the flow runs. They are declared with `init_variable(name, kind)` at the
**top level only** (never inside a loop or a condition), `kind` being `array`, `string`, `integer`, `float`,
`boolean` or `object`.

| Helper | Does |
|---|---|
| `set_variable(name, value)` | replaces the value |
| `increment_variable(name, value=1)` / `decrement_variable(name, value=1)` | counters (integer or float) |
| `append_to_string(name, value)` | adds to the end of a text |
| `append_to_array(name, value)` | adds one element to an array |

Read one with `@variables('Name')`. `pnpm flows:build` refuses a variable used before it is initialized.
Rounds of a loop that change one variable need `concurrency=1`, or they overwrite each other.

```python
from lib.flowkit import (
    Flow, append_to_string, chain, compose, dataverse, foreach, increment_variable, init_variable, manual_trigger,
)

flow = Flow(
    name="Count and list contacts",
    key="CountContacts",
    trigger=manual_trigger(),
    actions=chain(
        {
            "Initialize_Count": init_variable("Count", "integer"),
            "Initialize_Names": init_variable("Names", "string"),
            "List_contacts": dataverse("ListRecords", {"entityName": "contacts", "$select": "fullname", "$top": 20}),
            "For_each_contact": foreach(
                "@outputs('List_contacts')?['body/value']",
                chain(
                    {
                        "Count_it": increment_variable("Count"),
                        "Add_name": append_to_string("Names", "@{items('For_each_contact')?['fullname']}, "),
                    }
                ),
                concurrency=1,
            ),
            "Show_result": compose("@{variables('Count')} contacts: @{variables('Names')}"),
        }
    ),
)
```

## Parallel branches

`chain()` makes every action wait for the previous one. To run two actions **at the same time**, give both the
same `after`, and let the next action wait for both with `ok("A", "B")`. `chain()` leaves an action that
already has a `runAfter` where it is. `parallel(ok("Start"), {"Branch_A": ..., "Branch_B": ...})` gives the
same `after` to every branch for you; spread it into the chain with `**`.

```python
from lib.flowkit import Flow, chain, compose, manual_trigger, ok, parallel

flow = Flow(
    name="Two things at once",
    key="TwoThings",
    trigger=manual_trigger(),
    actions=chain(
        {
            "Start": compose("go"),
            **parallel(ok("Start"), {"Branch_A": compose("a"), "Branch_B": compose("b")}),
            "Join": compose("done", after=ok("Branch_A", "Branch_B")),
        }
    ),
)
```

## Try, catch, finally

By default a failed action fails the run and everything after it is skipped. To react instead, group the steps
in a **scope** and put a second scope after it that runs only when the first failed. `try_catch(name, attempt,
handle, finally_=None)` builds the three scopes: `Try_<name>`, `Catch_<name>` (runs when the try failed or timed
out) and `Finally_<name>` (runs whatever happened).

```python
from lib.flowkit import (
    Flow, compose, dataverse, dataverse_trigger, query, terminate_failed, try_catch,
)

flow = Flow(
    name="Sync with a safety net",
    key="SyncSafely",
    trigger=dataverse_trigger("contact", "Added"),
    actions=try_catch(
        "Sync",
        attempt={"Create_task": dataverse("CreateRecord", {"entityName": "tasks", "item/subject": "Welcome call"})},
        handle={
            "Find_failed_actions": query("@result('Try_Sync')", "@equals(item()?['status'], 'Failed')"),
            "Explain": compose("@{first(body('Find_failed_actions'))?['error']?['message']}"),
            "Fail_with_reason": terminate_failed("SyncFailed", "@{outputs('Explain')}"),
        },
        finally_={"Log_end": compose("done")},
    ),
)
```

- Inside the catch, `@result('Try_<name>')` lists every action of the try with its `status` and `error`.
  `query` on it finds the failed ones, as above.
- **Handled means succeeded.** If the catch does not end with a `terminate_failed`, the run is marked
  *Succeeded* even though the try failed. Decide which you want.
- Several independent steps where one failure must not stop the others (a loop of writes): use
  `catch(action, into, context)` per step and `terminate_with_list` at the end
  ([flowkit](../../8-reference/flowkit.md#error-handling)). Use `try_catch` for "this whole part".
- A **Terminate** ends the whole run at once, whatever scope it is in; nothing after it runs, including a catch.
- For other statuses use `when(name, "Failed", "Skipped", "TimedOut")` as an action's `after`;
  `when(name)` with no status means "whatever happened".

---

**Previous:** [2. Decisions and loops](02-decisions-and-loops.md) · **Next:** [4. Reliability and cost](04-reliability.md)
