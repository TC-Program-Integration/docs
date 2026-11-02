# Choice values

> **For:** a flow that reads or writes a choice column (a status, a type, a category) and must not depend on its numbers.
> **Result:** a flow that looks each number up at run time by its English label, fails with a clear message when a
> label is missing, and reads the numbers from the lookup. The reference is in [flowkit](../8-reference/flows/flowkit.md#choice-values).

A choice column stores a number and shows a label. The numbers are set when the option is created, so they can
differ between environments. A flow that writes `2` for "Inactive" works in one environment and writes the wrong
status in another. The flow reads the number from Dataverse's `stringmap` table instead, keyed by the English label.

## 1. Look the numbers up

Add `choice_lookup` at the start of the flow, with each choice column the flow uses as `(table, column)`:

```python
from lib.flowkit import Flow, chain, choice_lookup, choice_value_expr, compose, dataverse_trigger, require_choice_values

STATUS_KEYS = [("account", "statuscode", "Active"), ("account", "statuscode", "Inactive")]

flow = Flow(
    name="Account status message",
    key="AccountStatusMessage",
    trigger=dataverse_trigger("account", message="Modified"),
    actions=chain(
        {
            **choice_lookup([("account", "statuscode")]),
            **require_choice_values(STATUS_KEYS),
            "Active_status": compose(choice_value_expr("account", "statuscode", "Active")),
        }
    ),
)
```

`choice_lookup` adds three actions: `List_Choice_values` (the options of the columns, in English),
`Select_Choice_values` (one text per option) and `Choice_values` (the options as one object). The name is
`Choice_values` by default; pass `name="Status"` to use `List_Status`, `Select_Status` and `Status`, for a
second lookup in the same flow.

## 2. Require the values

`require_choice_values` goes right after the lookup and lists every key the flow uses, as
`(table, column, label)`. When one of them is not in the lookup, the run ends as Failed with the code
`ChoiceValueMissing` and a message that names the missing keys, such as
`Choice values not found in stringmap: account|statuscode|Inactive; `. The run stops there, so no action
uses a missing number.

## 3. Use the number

| You write | In the flow | Use it for |
|---|---|---|
| `choice_value_expr("account", "statuscode", "Active")` | `@outputs('Choice_values')?['account\|statuscode\|Active']` | a whole input, such as a row's column |
| `choice_value_text("account", "statuscode", "Active")` | `@{outputs('Choice_values')?['account\|statuscode\|Active']}` | inside a longer text |

The label is the option's English label as it is in the schema (Power Apps: the table's column, the option's
label). It is the key of the lookup, so write it exactly as the label. Labels with an apostrophe are escaped for you.

`statecode` is the exception: Dataverse defines its numbers (`0` is Active, `1` is Inactive) the same in every
environment, so a flow may write them directly. Every other choice column goes through the lookup.

## 4. Trigger conditions cannot use the lookup

A trigger's conditions are checked before the flow's actions run, so the lookup is not there yet. A trigger
condition that compares a choice column with a number stays a number. For a label, move the check into the
flow's body: a condition after the lookup, with a Terminate Succeeded to stop early, as in
[Flow patterns](flow-patterns.md).

## 5. Check it

- `choice_lookup` and `require_choice_values` raise an error when a table or column is not a logical name, when
  `name` does not start with a letter, or when no column or key is given. `pnpm flows:build` runs them.
- In the run history, `List_Choice_values` lists the options of the columns. A run that fails with
  `ChoiceValueMissing` names the key that is not in the lookup: check its spelling against the schema.
- The lookup uses the table's logical name. The Dataverse Web API accepts `objecttypecode eq '<table logical name>'`
  on `stringmaps` and returns the same logical name. Checked read-only on 2026-10-27 against a dev environment:
  `stringmaps?$filter=objecttypecode eq 'account' and attributename eq 'statuscode' and langid eq 1033` returned
  Active (1) and Inactive (2). Only FetchXML needs the numeric type code; the Dataverse connector's List rows uses
  the Web API. The connector run itself is confirmed on the first flow run: check the rows of `List_Choice_values`
  then.

---

**Back to:** [Build a flow](build-a-flow.md) · [flowkit reference](../8-reference/flows/flowkit.md#choice-values)
