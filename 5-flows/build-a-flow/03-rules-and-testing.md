---
sidebarTitle: "Rules and testing"
---

# Build a flow: Rules and testing

> **For:** choosing each action and getting it right. **Result:** a flow that fails clearly when data
> is bad, and tests that need no Power Automate. Part 3 of 3 of [Build a flow](../build-a-flow.md).

## Choosing the action

| You want to | Use | Notes |
|---|---|---|
| Read rows | `dataverse("ListRecords", {...})` | `entityName` is the entity **set** name (the plural); `$select` only what you use; `odata_quote()` around any text from the flow in a `$filter` |
| Read one row by id | `dataverse("GetItem", {...})` | |
| Create or change a row | `dataverse("CreateRecord" / "UpdateOnlyRecord", {...})`, or `add_row` / `update_row` | columns are `"item/<column>"`; a lookup is `"item/<NavProperty>@odata.bind": "/<set>(<id>)"`. `UpdateRecord` also creates a missing row. |
| Name an intermediate result | `compose(value)` | read it later with `outputs('Name')` |
| Keep only some items of a list | `query(source, where)` | `where` uses `item()` |
| Reshape each item of a list | `select(source, value)` | |
| Do something for each item | `foreach(items, chain({...}), concurrency=1)` | `concurrency=1` when the loop changes a shared variable |
| Branch | `condition(expression, then=..., otherwise=...)` | |
| Collect problems | `init_variable` + `catch(...)` | see the rules below |
| End the run as failed | `terminate_with_list(variable, code)` | puts every collected problem in the run's error |
| Read SharePoint | `sharepoint_items(site, list_id)` | |
| Send an email, post in Teams | `send_email(...)`, `post_teams_message(...)` | [Helpers for common connectors](../flow-patterns/05-connectors.md#helpers-for-common-connectors) |
| Any other connector | `connector("shared_approvals", "CreateAnApproval", {...})` | take the operation id from **Peek code** ([Connectors](../flow-patterns/05-connectors.md#find-the-json-of-any-action)) |

Each of these is shown in a whole flow in [Flow patterns](../flow-patterns.md).

## The rules

- **One bad row must not stop the others.** After a write inside a loop, add
  `**catch("Action", VARIABLE, context)`. It adds an action that runs only if the write fails and
  appends the error to the array variable. End the flow with `terminate_with_list`: the run fails
  once, and its error lists every problem. Without this you only see "BadRequest".
- **Initialize variables once, at the top level**, before any loop or condition:
  `init_variable("Problems", "array")`. Power Automate refuses them anywhere else.
- **A Terminate cannot read `variables()`.** `terminate_with_list` joins the list in a Compose first.
- **There is no `null()` function.** Write the literal `null` inside an expression.
- **Wrap text from the flow in `odata_quote()`** inside an OData `$filter`. It escapes single quotes
  (`O'Brien`).
- **Date-only columns are calendar days, not moments.** Format them with
  `formatDateTime(x, 'yyyy-MM-dd')`; do not pass them through `convertFromUtc`.
- **Lookup navigation properties are case-sensitive** (`pidev_Account`, not `pidev_account`).
- **Filter in Dataverse, not in the flow.** `$filter` and `$select` on the list action are faster
  and cheaper than reading everything and using `query`.
- **Do not edit the generated JSON.** Change the Python and rebuild.
- **Connections:** every connector the flow uses needs a connection reference in the target
  solution. The build prints their names.

`pnpm flows:build` enforces the ones it can ([the list](../build-a-flow.md#3-build-and-check)).
The rest show up only in a run, so read the [reliability patterns](../flow-patterns/04-reliability.md)
before a flow goes to prod.

## Test it

`pnpm test:flows` runs every `test_*.py` under `flows/` (the kit's in `flows/tests/`, the project's next to its flows): the kit's own tests, the snippets of
[Flow patterns](../flow-patterns.md), and yours. When you change flowkit (`flows/lib/flowkit/`),
these tests also check that the examples still pass `check()`. A test is found only in a folder Python can
import: one with an `__init__.py` and no dash in its name, so not in a `flows/<my-flow>/` folder itself;
`commands.test.ts` fails when a test file would be skipped.

When your flow has logic worth pinning, add a test in `flows/tests/`, named `test_<your flow>.py`. Load the
flow and assert what matters: that it passes `check()`, the trigger and its filter, the order of steps, the fields
of an email, the failure path. `FLOWS` (from `tests.helpers`) is the `flows/` folder. The tests are run from
`flows/`, so `lib.flowkit` and `tests.helpers` import without any setup.

```python
"""Tests for the my-flow flow (flows/tests/test_my_flow.py)."""

import unittest

from lib.flowkit import check, load_flow
from tests.helpers import FLOWS


class MyFlowTest(unittest.TestCase):
    """my-flow builds and is wired as intended."""

    def test_passes_the_checks(self):
        """check() finds nothing."""
        flow = load_flow(FLOWS / "my-flow" / "flow.py")
        self.assertEqual(check(flow.definition()), [])
```
[The first-feature exercise](../../3-first-feature/first-feature/04-alert-flow.md#tests) has a fuller set.

## Debug a run

- **In dev**, cause the trigger on purpose (create a record, press the button), then read the run in
  the Runs tab ([step 5](../build-a-flow.md#5-see-the-run-and-its-error)).
- **A wrong value:** add a temporary `compose(...)` of the expression, run again, read its output,
  then remove it.
- **A failed run in prod:** find the first red action and read its error. The cause is usually data
  your dev records never had (an empty lookup, a missing email). Make the flow say so clearly with
  [try and catch](../flow-patterns/03-variables-and-failure.md#try-catch-finally).

---

**Previous:** [A complete example](02-complete-example.md) · **Next:** [Flow patterns](../flow-patterns.md)
