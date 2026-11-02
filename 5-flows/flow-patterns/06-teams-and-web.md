---
sidebarTitle: "Teams, approvals and web services"
---

# Flow patterns: 6. Teams, approvals and web services

> **For:** Teams, approvals, HTTP calls, and what flowkit does not cover. **Result:** the last
> blocks. Part 6 of 6 of [Flow patterns](../flow-patterns.md).

## Teams and approvals

The shapes below are what the designer produces. Confirm the names with **Peek code** before first use:
connectors change their parameters from time to time. For a plain channel message, `post_teams_message(team_id,
channel_id, message)` writes the first action for you; approvals have no helper.

```python
from lib.flowkit import Flow, chain, connector, manual_trigger

flow = Flow(
    name="Ask a manager",
    key="AskManager",
    trigger=manual_trigger({"Manager": "string", "Request": "string"}),
    actions=chain(
        {
            "Tell_the_channel": connector(
                "shared_teams",
                "PostMessageToConversation",
                {
                    "poster": "Flow bot",
                    "location": "Channel",
                    "body/recipient/groupId": "00000000-0000-0000-0000-000000000000",
                    "body/recipient/channelId": "19:example@thread.tacv2",
                    "body/messageBody": "<p>@{triggerBody()?['Request']}</p>",
                },
            ),
            "Wait_for_approval": connector(
                "shared_approvals",
                "StartAndWaitForAnApproval",
                {
                    "approvalType": "Basic",
                    "ApprovalCreationInput/title": "@{triggerBody()?['Request']}",
                    "ApprovalCreationInput/assignedTo": "@triggerBody()?['Manager']",
                    "ApprovalCreationInput/details": "Please approve or reject.",
                },
            ),
        }
    ),
)
```

An approval's answer is in `@outputs('Wait_for_approval')?['body/outcome']` (`Approve` or `Reject`); decide on
it with a `condition` or `switch`. The ids in the Teams action are placeholders: take the real ones from the
designer. An approval waits up to 30 days ([limits](04-reliability.md#limits-to-know)).

## Web services

Flows can call any web address, and any system can call a flow. Both are **premium** features: everyone who
runs or owns the flow needs a licence that includes them. Ask before you build on them.

**Calling out: `http` then `parse_json`.** A response arrives as text; `parse_json` gives later actions
named values to read.

```python
from lib.flowkit import Flow, chain, compose, http, manual_trigger, parse_json

flow = Flow(
    name="Look up a postcode",
    key="PostcodeLookup",
    trigger=manual_trigger({"Postcode": "string"}),
    actions=chain(
        {
            "Call_service": http("GET", "https://api.example.org/postcodes/@{triggerBody()?['Postcode']}"),
            "Read_answer": parse_json(
                "@{body('Call_service')}",
                {"type": "object", "properties": {"city": {"type": "string"}, "region": {"type": "string"}}},
            ),
            "Show_city": compose("@{body('Read_answer')?['city']}"),
        }
    ),
)
```

Check the status before using the body when the service can fail: a non-2xx answer fails the `http` action,
so wrap it in a [`try_catch`](03-variables-and-failure.md#try-catch-finally).

**Being called: `http_request_trigger` and `response`.** The trigger gives the flow a web address (created when
the flow is saved; treat it as a secret, because anyone who has it can run the flow). The flow **must** answer,
or the caller waits until it times out: put a `response` at the end of the success path and one in the catch.
`flows/examples/intake-webhook/` is the complete example.

Do not put secrets in `headers`, `uri` or `body`: they would be stored in the flow and shown in the run
history. Use a connection or an environment variable, and secure inputs and outputs
([Reliability](04-reliability.md#secrets-and-data-in-the-run-history)).

## What flowkit does not cover

| You need | Do |
|---|---|
| a child flow (call another flow and wait) | no helper: build the called flow with a `manual_trigger` or `http_request_trigger`; call it with `http`, or build the call in the designer first and read its JSON from Peek code |
| an action no helper exists for | `connector(...)` with the JSON from Peek code, as above |
| a trigger no helper exists for (Teams, Approvals) | the trigger is a dictionary `{"Name": {...}}`: write it by hand from Peek code and pass it as `trigger=` |
| a Power Apps button | `manual_trigger` takes its inputs (`string`, `number`, `boolean`); the app calls the flow from its own formulas, and the flow can answer with `respond_to_powerapp(...)` |

The helpers that do exist (triggers for SharePoint and Outlook, Outlook, Teams and Office 365 Users actions, Dataverse
rows, condition builders, retry and other options) are in the [flowkit reference](../../8-reference/flows/flowkit.md).

When you add a helper to `flows/lib/flowkit/` for something your team uses often, give it a test in
the matching folder of `flows/tests/` (`actions/`, `connectors/` or `triggers/`) and a row in the
[flowkit reference](../../8-reference/flows/flowkit.md). The layout is in [flowkit layout](../../8-reference/flows/flowkit-layout.md).

---

**Previous:** [5. Connectors](05-connectors.md) · **Back to:** [Build a flow](../build-a-flow.md)
