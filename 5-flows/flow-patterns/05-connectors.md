---
sidebarTitle: "Connectors"
---

# Flow patterns: 5. Connectors

> **For:** using Outlook or any other connector. **Result:** a `connector(...)` action with the right
> names. Part 5 of 6 of [Flow patterns](../flow-patterns.md).

## Any connector with `connector(...)`

Some connectors have their own helpers ([the list below](#helpers-for-common-connectors)). **Every other
action** (Approvals, OneDrive, Excel, Planner, a custom connector, or an Outlook action the helpers do not
cover) is written with the same helper:

```python
connector("shared_office365", "SendEmailV2", {"emailMessage/To": "...", "emailMessage/Subject": "..."})
```

Three things identify an action: the **connector id** (`shared_office365`), the **operation id**
(`SendEmailV2`) and its **parameters** (a dictionary). Extra keyword arguments go on the action itself
(`runtimeConfiguration=...`), and `after=` sets `runAfter` as for every helper.

### Find the JSON of any action

You do not need to guess names. Build the action once in the designer, then copy its JSON:

1. In Power Automate, create a scratch flow, add the action, fill the fields with example values.
2. On the action, open **… → Peek code**.
3. Copy three things from it into `connector(...)`: `host.connectionName` (the connector id),
   `host.operationId`, and the keys and values under `parameters`.
4. Replace the example values with expressions, and delete the scratch flow.

A dynamic value in the designer (`Topic`) is an expression in the JSON (`@{triggerOutputs()?['body/pidev_topic']}`):
the peek shows which. For a whole flow that already exists, **export the solution** and read
`Workflows/<name>.json` inside it: the same JSON for every action.

Each connector in a flow gets a **connection reference** named `<prefix>_<FlowKey><Connector>`.
flowkit adds them and the build prints their names. At the first import, you set a connection on
each ([Deploy, step 4](../../6-ship/deploy/04-flow-and-try.md#4-the-flow)).

### Helpers for common connectors

| Helper | Does |
|---|---|
| `send_email(to, subject, body, importance="Normal")` | Outlook: send an email. `to`: addresses separated by `;`. `body`: HTML. |
| `post_teams_message(team_id, channel_id, message)` | Teams: post `message` (HTML) in a channel as Flow bot |
| `get_user(user)` / `get_manager(user)` | Office 365 Users: the profile / the manager of `user` (an email address or id) |
| `sharepoint_trigger(site, list_id, message)` / `outlook_trigger(folder)` | start the flow when a SharePoint item is created or modified / an email arrives |
| `list_rows`, `add_row`, `update_row`, `delete_row`, `perform_action` | Dataverse rows and actions, without the operation ids |

All of them are in the [flowkit reference](../../8-reference/flowkit.md#connector-actions), with their arguments.
The example reads a new SharePoint item, finds its author's manager and emails them. The trigger starts a run only
when the item has an author (`trigger_options`); a failed email is tried again twice (`with_options`).

```python
from lib.flowkit import (
    Flow, chain, get_manager, send_email, sharepoint_trigger, trigger_options, with_options,
)

flow = Flow(
    name="Tell the manager",
    key="TellManager",
    trigger=trigger_options(
        sharepoint_trigger("https://contoso.sharepoint.com/sites/team", "00000000-0000-0000-0000-000000000000"),
        conditions=["@not(empty(triggerOutputs()?['body/Author/Email']))"],
    ),
    actions=chain(
        {
            "Get_manager": get_manager("@triggerOutputs()?['body/Author/Email']"),
            "Email_manager": with_options(
                send_email(
                    "@outputs('Get_manager')?['body/mail']",
                    "New item: @{triggerOutputs()?['body/Title']}",
                    "<p>A new item was added.</p>",
                ),
                retries=2,
            ),
        }
    ),
)
```

The site address and list id are placeholders. `get_manager` reports "No manager found" when the person has no
manager in Microsoft Entra ID; when that can happen, build the flow so that one missing manager does not stop it
(`flows/examples/new-submission/` is a complete example).

### Email with `connector(...)`

`send_email` covers the common email. When you need a field it does not have (a CC, a sender), write the action
with `connector(...)`:

```python
from lib.flowkit import Flow, chain, connector, manual_trigger

flow = Flow(
    name="Send a note",
    key="SendNote",
    trigger=manual_trigger({"To": "string", "Text": "string"}),
    actions=chain(
        {
            "Send_email": connector(
                "shared_office365",
                "SendEmailV2",
                {
                    "emailMessage/To": "@triggerBody()?['To']",
                    "emailMessage/Subject": "A note for you",
                    "emailMessage/Body": "<p>@{triggerBody()?['Text']}</p>",
                    "emailMessage/Importance": "Normal",
                },
            ),
        }
    ),
)
```

The body is HTML. The mail is sent **as the connection's owner**, so use a shared mailbox or a service
account, and tell recipients who it comes from.

### A table in a message

`table(source, {header: expression})` makes an HTML (or CSV) table from an array, ready for an email body:

```python
from lib.flowkit import Flow, chain, connector, dataverse, manual_trigger, table

flow = Flow(
    name="Email the open tasks",
    key="MailOpenTasks",
    trigger=manual_trigger({"To": "string"}),
    actions=chain(
        {
            "List_tasks": dataverse(
                "ListRecords", {"entityName": "tasks", "$select": "subject,scheduledend", "$filter": "statecode eq 0", "$top": 25}
            ),
            "Make_table": table(
                "@outputs('List_tasks')?['body/value']",
                {"Subject": "@item()?['subject']", "Due": "@formatDateTime(item()?['scheduledend'], 'yyyy-MM-dd')"},
            ),
            "Send_email": connector(
                "shared_office365",
                "SendEmailV2",
                {
                    "emailMessage/To": "@triggerBody()?['To']",
                    "emailMessage/Subject": "Open tasks",
                    "emailMessage/Body": "@{body('Make_table')}",
                },
            ),
        }
    ),
)
```

---

**Previous:** [4. Reliability and cost](04-reliability.md) · **Next:** [6. Teams, approvals and web services](06-teams-and-web.md)
