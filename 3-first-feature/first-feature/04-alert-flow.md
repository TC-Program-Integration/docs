---
sidebarTitle: "The alert flow"
---

# Your first feature: 4. The alert flow

> Part 4 of 6 of [Your first feature](../first-feature.md). Step 7.

Most real tasks have a web resource **and** a flow. Here the flow reacts to what your dialog creates:
**when an Urgent callback request is created, email the owner of its contact.** You write it in Python
with `flowkit`, like every flow in this project, and prove it with tests that need no Power Automate.

Read first: [Build a flow](../../5-flows/build-a-flow.md) (the whole path, a few minutes) and, while writing,
[Flow patterns](../../5-flows/flow-patterns.md) (each building block with a snippet) and
[Write the steps of a flow](../../5-flows/build-a-flow/01-write-the-steps.md) (the craft).

## The brief

| # | The flow must |
|---|---|
| F1 | start when a **Callback request is added**, and only when its priority is **Urgent**. The filter is in the **trigger**, so other requests never start a run. |
| F2 | read the **contact** the request is linked to, then the contact's **owner** (a system user) |
| F3 | email the owner through Outlook: subject `Urgent callback: <topic>`, importance High, a body with the contact's name, the topic, the phone, the preferred date as `yyyy-MM-dd` (or "none") and the notes |
| F4 | when the owner has **no email address**, end the run as **Failed** with a clear message and the code `NoOwnerEmail` |
| F5 | when any other step fails, end the run as **Failed** with the error text (a try/catch with scopes) |
| F6 | keep every Dataverse name in `settings.py`, in a folder `flows/callback-alert/` (`flow.py` and `settings.py`) |

Flow name: *Urgent callback alert*. Key: `CallbackAlert`.

## Steps

1. **Start from an example.** Copy `flows/examples/intake-webhook/` to `flows/callback-alert/` (it shows
   `try_catch`, `switch` and `response`) and read `flows/examples/overdue-tasks/flow.py` (it shows
   Dataverse reads, a condition and an Outlook email). Delete what you do not need.
2. **Write the settings.** Table, columns, the priority value, the lookup column and the entity set
   names. A lookup column is read from a record as `_<column>_value`: look at the real names with
   `pnpm dv:q` when you have the table in dev; until then use the names from [the brief](01-brief.md).
3. **The trigger** (F1): `dataverse_trigger(table, "Added", filter_expression="...")`. The table is the
   **logical** name; the filter is an OData condition on the column.
4. **The reads** (F2): two `dataverse("GetItem", {...})` actions. `entityName` is the entity **set** name;
   `recordId` reads the lookup from the trigger: `@triggerOutputs()?['body/<column>']`; the second read
   uses what the first returned: `@outputs('Get_contact')?['body/<column>']`.
5. **The decision and the email** (F3, F4): `condition(...)` on whether the owner's email is empty; the
   email is `send_email(...)` (or `connector("shared_office365", "SendEmailV2", {...})`); the other branch is
   `terminate_failed(code, message)`.
6. **The failure path** (F5): wrap steps 4 and 5 in `try_catch(...)`; its `handle` part ends with a
   Terminate. [Flow patterns](../../5-flows/flow-patterns/03-variables-and-failure.md#try-catch-finally) explains the pattern.
7. **Build it often:** `pnpm flows:build` formats, lints, type-checks and **checks** the flow, and
   writes `flows/out/CallbackAlert/flow.json`. Read that file once: it is what Power Automate stores.

## Tests

`flows/tests/test_callback_alert.py`, with `unittest` as in
[Rules and testing](../../5-flows/build-a-flow/03-rules-and-testing.md#test-it): load the flow with
`load_flow` and look at its JSON. At least five tests:

| Test | Proves |
|---|---|
| the flow passes `check(flow.definition())` | no wrong reference or variable |
| the trigger watches your table, for added rows, with the Urgent filter | F1 |
| the owner is read after the contact, and the condition after the owner | F2: the order of the steps |
| the email goes to the owner's address, with High importance and the topic in the subject | F3 |
| the no-email branch ends the run Failed with `NoOwnerEmail` | F4 |
| the catch scope runs when the try scope fails or times out, and ends with a Terminate | F5 |

Run them with `pnpm test:flows`. `pnpm check:flows` adds formatting, the linter and the type check.

**What these tests cannot prove:** that Power Automate accepts the connector's parameter names, and that
the email arrives. That is the dev test in step 9: build the flow once in the designer if a name is in
doubt, and copy the names from its **Peek code** ([Flow patterns](../../5-flows/flow-patterns/05-connectors.md#find-the-json-of-any-action)).

**Next:** [Check it and show it](05-check-and-show.md).

---

**Previous:** [3. Test it and build the component](03-tests-and-component.md)
