# flowkit options

> **Reference.** Run options for actions and triggers, and the condition builders, of `flows/lib/flowkit/`.
> The other helpers: [flowkit](flowkit.md).

## Action options

`with_options(action, retries=None, retry_interval="PT20S", timeout=None, secure=False, tracked=None)` adds
options to any action and returns it:

| Argument | Effect |
|---|---|
| `retries` | how many times a failed call is tried again, `retry_interval` apart (ISO 8601: `PT20S` is 20 seconds, `PT1M` a minute). `retries=0` turns retrying off. Only for actions that call a service (a connector, `http`). |
| `timeout` | the longest the action may take (ISO 8601: `PT10M`); after that it ends as timed out |
| `secure` | hides the action's inputs and outputs in the run history |
| `tracked` | `{name: expression}`: values shown in the run history for the action |

```python
send_email("a@contoso.com", "Subject", "<p>Body</p>")                  # without options
with_options(send_email("a@contoso.com", "Subject", "<p>Body</p>"), retries=2, secure=True)
```

## Trigger options

`trigger_options(trigger, conditions=None, split_on=None, concurrency=None)` adds options to the result of a
trigger helper and returns it:

| Argument | Effect |
|---|---|
| `conditions` | a list of expressions that must **all** be true for a run to start; a run that does not start costs nothing |
| `split_on` | an expression that gives an array: one run starts per element |
| `concurrency` | how many runs may be active at once (`1`: one at a time) |

```python
trigger_options(
    sharepoint_trigger("https://contoso.sharepoint.com/sites/team", "<list id>"),
    conditions=["@not(empty(triggerOutputs()?['body/Author/Email']))"],
    concurrency=1,
)
```

## Condition builders

They build the `expression` of `condition(...)` without writing the nesting by hand.

| Helper | Gives |
|---|---|
| `and_(*terms)` / `or_(*terms)` / `not_(term)` | all / any / the opposite of the terms |
| `equals(a, b)`, `greater(a, b)`, `greater_or_equals(a, b)`, `less(a, b)`, `less_or_equals(a, b)` | a comparison of two values |
| `contains(collection, value)` | the text, array or object holds `value` |
| `is_empty("@expression")` | the expression gives an empty value or null |

```python
condition(and_(equals("@variables('Status')", "Open"), not_(is_empty("@outputs('Get_owner')?['body/mail']"))), then)
```
