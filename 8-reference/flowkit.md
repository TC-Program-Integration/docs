# flowkit

> **Reference.** Every helper in `flows/lib/flowkit/` ([where each one lives](flowkit-layout.md)). A flow file, `flows/<name>/flow.py`, imports
> what it uses (`from lib.flowkit import Flow, chain, recurrence`) and defines `flow = Flow(...)`.
> How to write one: [Build a flow](../5-flows/build-a-flow.md).

The solution's XML is in `flows/templates/*.xml` (filled with `${placeholders}`), not in Python;
edit those files to change what the package contains.

Every action helper returns a plain dict in Power Automate's definition format. Helpers that
take `after` set the action's `runAfter`; leave it out inside `chain()`.

## Flow

```python
Flow(name, key, trigger, actions, description="", workflow_id=None, connection_label=None)
```

| Field | Meaning |
|---|---|
| `name` | display name in Power Automate |
| `key` | PascalCase id; connection references are named `<prefix>_<key><Connector>` |
| `trigger` | one trigger helper's result |
| `actions` | usually `chain({...})` |
| `description` | shown in the solution |
| `workflow_id` | fixed GUID; by default derived from the solution and `key`, so re-imports update the same flow |
| `connection_label` | connection reference display names are `<label> - <Connector>`; default `name` |
| `environment_variables` | the [environment variables](#environment-variables) the actions read; each is a parameter of the flow. Default: none. |
| `connection_references` | a dict from connector name (the connection id, e.g. `shared_sharepointonline`) to the logical name of an existing connection reference. That connector uses the given name instead of the generated `<prefix>_<key><Connector>`, so an already-deployed flow keeps its connection references. Default: none (every connector is generated). |

## Triggers

| Helper | Trigger |
|---|---|
| `recurrence(frequency="Hour", interval=1, time_zone="Eastern Standard Time", hours=None, minutes=None, week_days=None)` | a schedule. `frequency`: Minute, Hour, Day, Week, Month. `hours`/`minutes` pin the time; `week_days` (`["Monday", ...]`) the days for Week. |
| `dataverse_trigger(table, message="Added", scope="Organization", filter_columns=None, filter_expression=None)` | a row changes. `table`: logical name. `message`: Added, Modified, AddedOrModified, Deleted. `scope`: User, BusinessUnit, ParentChildBusinessUnits, Organization. |
| `manual_trigger(inputs=None)` | started by hand; `inputs` is `{"name": "string" \| "number" \| "boolean"}` |
| `http_request_trigger(schema=None, method="POST")` | another system calls the flow's web address (premium). Answer with `response(...)`. `schema` describes the body. |
| `sharepoint_trigger(site, list_id, message="Created")` | polls (every minute) for a SharePoint item created (`"Created"`) or created or modified (`"Modified"`) |
| `outlook_trigger(folder="Inbox", with_attachment=False)` | polls for a new email (V3) in `folder` |
| `trigger_options(trigger, conditions=None, split_on=None, concurrency=None)` | adds options to a trigger helper's result: [see flowkit options](flowkit-options.md#trigger-options) |

## Ordering

| Helper | Gives |
|---|---|
| `chain({name: action, ...})` | each action runs after the previous one succeeds. An action that already has `runAfter` (a `catch`, a parallel branch) keeps it and does not move the chain. |
| `ok(*names)` | `runAfter`: after these succeeded |
| `failed(name)` | `runAfter`: only when this failed |
| `when(name, *statuses)` | `runAfter`: after `name` ended with one of the statuses (`Succeeded`, `Failed`, `Skipped`, `TimedOut`); with none, whatever happened |

## Connector actions

| Helper | Action |
|---|---|
| `connector(connection, operation, parameters, after=None, **extra)` | any connector, e.g. `connector("shared_office365", "SendEmailV2", {...})` |
| `dataverse(operation, parameters, after=None)` | Dataverse: `ListRecords`, `GetItem`, `CreateRecord`, `UpdateOnlyRecord` (Update a row), `UpdateRecord` (Upsert a row), `DeleteRecord`. `entityName` is the **entity set name**; columns are `item/<column>`; lookups `item/<NavigationProperty>@odata.bind` with `/<entityset>(<id>)`. |
| `list_rows(entity, select=None, filter_=None, top=None, order_by=None, expand=None, fetch_xml=None, after=None)` | Dataverse List rows. `entity` is the entity set name; `select` a comma-separated list of columns; `filter_` an OData filter; `expand` an OData `$expand`; `fetch_xml` a FetchXML query. |
| `add_row(entity, columns)` / `update_row(entity, record_id, columns)` / `delete_row(entity, record_id)` | Dataverse Add / Update / Delete a row. `columns` is `{column: value}` (the `item/` prefix is added). `update_row` fails when the row does not exist (`UpdateOnlyRecord`). |
| `perform_action(name, inputs=None)` | Dataverse: perform the unbound action `name`; `inputs` is `{parameter: value}` |
| `sharepoint_items(site, list_id, after=None, minimum_items=5000)` | all items of a SharePoint list, paged |
| `send_email(to, subject, body, importance="Normal")` | Outlook Send an email (V2). `to`: addresses separated by `;`. `body`: HTML. `importance`: Low, Normal, High. |
| `post_teams_message(team_id, channel_id, message)` | Teams: post `message` (HTML) in a channel as Flow bot |
| `get_user(user)` / `get_manager(user)` | Office 365 Users: profile / manager of `user` (email or id) |

**Checked against Microsoft's connector reference:** the operation ids and parameter keys of all helpers above,
except the two items below. **Not documented there, so confirm them with Peek code before first use:** the Teams
channel keys (`body/recipient/groupId`, `body/recipient/channelId`, `body/messageBody`) and the `item/` prefix of
`perform_action` inputs. `UpdateRecord` creates the row when it is missing; the older examples use it to update.

Connectors with a short name in connection reference names (`CONNECTORS`):
Dataverse, SharePoint, Outlook (`shared_office365`), Teams, Office365Users, Approvals. Others
use the connector id without `shared_`.

## Built-in actions

| Helper | Action |
|---|---|
| `compose(value)` | Compose |
| `init_variable(name, kind="array", value=None)` | Initialize variable; `kind`: array, string, integer, float, boolean, object |
| `set_variable(name, value)` / `append_to_array(name, value)` | Set variable / Append to array variable |
| `foreach(items, actions, concurrency=None)` | Apply to each; `concurrency=1` runs items one at a time |
| `condition(expression, then, otherwise=None, after=None)` | Condition; `expression` like `{"and": [{"greater": ["@length(...)", 0]}]}`, or built with [`and_`, `equals`...](flowkit-options.md#condition-builders) |
| `query(source, where)` | Filter array (`where` uses `item()`) |
| `select(source, value)` | Select |
| `terminate_failed(code, message)` | Terminate with status Failed; `message` becomes the run's error. `message` cannot use `variables()` (the run fails with "The following functions could not be evaluated: variables"). |
| `terminate_succeeded()` / `terminate_cancelled()` | Terminate with status Succeeded (an early exit, the actions that have not run are skipped) / Cancelled |
| `increment_variable(name, value=1)` / `decrement_variable(name, value=1)` | Increment / Decrement variable (integer or float) |
| `append_to_string(name, value)` | Append to string variable |
| `terminate_with_list(variable, code, name="Report_problems")` | Two actions, `<name>_message` and `<name>`: joins the array variable's entries (`a \| b`) in a Compose, then terminates Failed with them as the error. Use this to report what `catch()` collected. |

## Structure and data

| Helper | Action |
|---|---|
| `parallel(after, branches)` | branches that start together once `after` is satisfied (`after` is `ok("A")`, or `{}` for the start of the flow). Spread it into `chain()`: `**parallel(ok("A"), {"B": ..., "C": ...})`; the next action waits for both with `after=ok("B", "C")`. |
| `scope(actions, after=None)` | Scope: a group that succeeds or fails as one |
| `try_catch(name, attempt, handle, finally_=None)` | the three scopes `Try_<name>`, `Catch_<name>` (runs when the try failed or timed out) and `Finally_<name>` (always). Read the failure with `@result('Try_<name>')`. |
| `switch(expression, cases, default=None)` | Switch; `cases` maps a value to its actions (named `Case_1`, `Case_2`...). Values keep their type: a Choice column needs integer keys. |
| `do_until(expression, actions, count=60, timeout="PT1H")` | Do until; both limits are always set |
| `delay(count, unit="Minute")` / `delay_until(timestamp)` | Delay / Delay until |
| `parse_json(content, schema)` | Parse JSON |
| `table(source, columns, format_="HTML")` | Create HTML table / CSV table; `columns` is `{header: expression}` |
| `http(method, uri, headers=None, body=None, queries=None, **extra)` | HTTP (premium). Extra keywords go on the action, e.g. `runtimeConfiguration`. |
| `response(status_code=200, body=None, headers=None)` | Response to an `http_request_trigger` flow |
| `respond_to_powerapp(outputs, types=None)` | Respond to a Power App or flow; `outputs` is `{name: value}`, `types` the type of each (default string). Its JSON shape is from a community example, not Microsoft's reference: check it with Peek code. |

These are explained, with whole flows, in [Flow patterns](../5-flows/flow-patterns.md).

## Options and conditions

Retry, time limit, hidden data (`with_options`), trigger conditions (`trigger_options`) and the condition builders
(`and_`, `equals`, ...): see [flowkit options](flowkit-options.md).

## Error handling

`catch(action, into, context="")` returns `{"Record_error_<action>": ...}`: an action that runs
only when `action` fails and appends `<context><action> failed: <error message>` to the array
variable `into`. Spread it after the action:

```python
"Add_row": dataverse("CreateRecord", {...}),
**catch("Add_row", "Problems", "Item @{items('For_each')?['ID']}: "),
```

End the flow with a `condition` whose `then` is `terminate_with_list("Problems", "MyCode")` when the variable is not empty.

## Expressions

`odata_quote(expression)` gives `'@{replace(<expression>, '''', '''''')}'`: a value inside an
OData filter with its quotes escaped. The expressions themselves are Power Automate's: see
[Flow expressions](flow-expressions.md).

## Checks

`check(definition)` returns a sorted list of problems (`flows/lib/flowkit/check/`);
`Solution.build` raises `FlowCheckError` with them and `pnpm flows:build` stops. Each rule catches
a mistake Power Automate would only report at save or run time:

| Rule | Message starts with |
|---|---|
| a `runAfter` naming an action that is not in the same scope (top level, scope, loop, condition branch, switch case) | `<path>: runAfter '<x>' is not in the same scope` |
| a scope where every action has `runAfter`, so none can start | `no action starts the scope` |
| `outputs()`, `body()`, `items()`, `actions()` or `result()` naming an action or trigger that does not exist | `outputs('<x>'): no action or trigger has that name` |
| `outputs()`, `body()`, `actions()` or `result()` in an action naming an action of the flow that does not run before it: not on its `runAfter` path (through the actions before it, and the ones they contain), nor on the path of the container it is in. Triggers are always allowed; a name the flow does not have is reported by the row above | `<path> reads "<x>", which does not run before it` |
| `variables()` naming a variable no Initialize variable creates | `variables('<x>'): not initialized` |
| Set, Append to array, Append to string, Increment or Decrement on a variable never initialized | `SetVariable '<x>': not initialized` |
| `variables()` inside a Terminate, which cannot evaluate it | `a Terminate cannot evaluate variables()` |
| a `null()` call: there is no such function, write the literal `null` | `null(): there is no null() function` |
| a function Power Automate does not have (`toTitle()`, `sum()`; names ignore case) | `<name>(): not a Power Automate expression function` |
| a quote that is not closed in an expression | `a quote is not closed` |
| `(` and `)`, or `[` and `]`, that do not match in an expression | `'(' and ')' do not match` |
| an Initialize variable inside a scope, loop or condition | `Initialize variable must be at the top level` |
| the same variable initialized twice | `<action>: variable '<x>' is initialized more than once` |
| `parameters('<x>')` (an environment variable) that the flow does not declare | `parameters('<x>'): not declared in the flow's parameters` |
| `items('<Loop>')` used outside that for each | `items('<Loop>'): used outside the '<Loop>' loop` |

Expressions are every string that starts with `@` (not `@@`) and every `@{...}` inside a string.
The known functions are in `flows/lib/flowkit/check/functions.py`; if Power Automate adds one, add it
there. How to fix each problem: [Build a flow](../5-flows/build-a-flow.md#3-build-and-check).

## Environment variables

```python
EnvironmentVariable(schema_name, display_name, type="String", default=None, description="")
```

| Field or attribute | Meaning |
|---|---|
| `schema_name` | `<prefix>_<name>` in a solution (the prefix is the publisher's, in `flows/solution.json`); letters, digits and `_` |
| `display_name` | the name in Power Automate; no single quote |
| `type` | `"String"` (default), `"Number"`, `"Boolean"`, `"JSON"`: the Dataverse type codes 100000000 to 100000003 |
| `default` | the development value: `str`, `int`, `bool`, or the JSON text for `"JSON"`; must match `type`. `None`: no default |
| `description` | shown in the flow's parameter metadata and in the solution |
| `.parameter_name` | `<display_name> (<schema_name>)`, the parameter's name |
| `.expression` | `@parameters('<parameter_name>')`: a whole input |
| `.interpolated` | `@{parameters('<parameter_name>')}`: inside a text |

The flow's `parameters` gets one entry per variable, next to `$connections` and `$authentication`:
`{"defaultValue": ..., "type": ..., "metadata": {"schemaName": ..., "description": ...}}`. The type is
`String`, `Int` (Number), `Bool` (Boolean) or `Object` (JSON). Number, Boolean and JSON are not yet
checked against a flow in dev. The `Solution` writes `environmentvariabledefinitions/<schema name>/environmentvariabledefinition.xml`
and a `RootComponent` of type 380 for each variable any flow declares; the same schema name declared
twice with other settings raises `ValueError`. How to use it: [Environment variables](../5-flows/environment-variables.md).

## Choice values

The number behind a choice label, looked up at run time from the `stringmap` table by the English label. How to use
it: [Choice values](../5-flows/choice-values.md).

| Helper | Gives |
|---|---|
| `choice_lookup(columns, name="Choice_values", language=1033)` | three actions in a chain: `List_<name>` (the options of the columns in `language`, from stringmaps), `Select_<name>` (one text `"<table>\|<column>\|<label>":<number>` per option) and `<name>` (the options as one object). `columns` is a list of `(table, column)` logical names (`ChoiceColumn`). |
| `require_choice_values(keys, name="Choice_values")` | `If_<name>_missing`: when a key `(table, column, label)` is not in the lookup (null), ends the run as Failed with the code `ChoiceValueMissing` and a message listing the missing keys |
| `choice_value_expr(table, column, label, name="Choice_values")` | `@outputs('<name>')?['<table>\|<column>\|<label>']`: the number, as a whole value |
| `choice_value_text(table, column, label, name="Choice_values")` | `@{outputs(...)}`: the number, inside a text |
| `choice_value(table, column, label, name="Choice_values")` | the expression without the `@`: `outputs('<name>')?['<table>\|<column>\|<label>']` |

Table and column are logical names (letters, digits and `_`); `name` is a letter-led name. Labels are escaped
(an apostrophe is doubled). The lookup has no check rule of its own: `check()` covers the expressions it builds.

## Solution

`flows/build-all.py` (`pnpm flows:build`) wraps the flows in a `Solution` filled from
`flows/solution.json`:

```python
Solution(unique_name, display_name, publisher, prefix, option_prefix,
         publisher_display_name=None, version="1.0.0.0", flows=[...]).build(out_dir)
```

`build` validates the names (publisher and solution unique names of letters, digits and `_`,
prefix of 2 to 8 lowercase letters or digits, version `a.b.c.d`, distinct flow keys), checks every
flow, then writes `<out>/<key>/flow.json`, `<out>/flows.json` and `<out>/<UniqueName>_<version>.zip`
(dots in the version become `_`): an unmanaged solution with the flows and their connection
references. The zip is for the first import of a flow; after that the
[Flows as Code tool](tools/flows-as-code.md) pushes `flow.json`. Options: [commands](commands.md#flows).

With several solutions, build-all calls the functions below instead. A flow can be in more than one
solution: it is checked and written once, and each solution's zip holds its own flows.

```python
settings = load_settings(Path("flows/solution.json"))         # ProjectSettings: the home and the extra solutions
solutions, warnings = solutions_for(settings, flows, known)   # flows: {folder: Flow}; known: every folder
for solution, zip_path in write_project(solutions, Path("flows/out")):
    print(zip_path)
```

| Function | Does |
|---|---|
| `load_settings(path_or_dict)` | reads `flows/solution.json` into `ProjectSettings` (`home`, `extra`). Raises `ValueError` for a missing key, an extra solution without `flows`, or two solutions with one name. |
| `override_home(settings, **changes)` | a copy where the home solution takes the given values (None keeps the file's); build-all uses it for the command-line options. |
| `solutions_for(settings, flows, known)` | the `Solution` objects, home first, with their flows, and the warnings. Raises `ValueError` for an unknown folder, or a flow in two solutions without a fixed `workflow_id`. A solution with no loaded flow comes back empty. |
| `write_project(solutions, out_dir, only=None)` | checks every flow once, writes each `<out>/<key>/flow.json` once and `<out>/flows.json`, then one zip per solution. `only` writes one solution. Returns `(solution, zip path)` pairs. |

Each `Solution` has `home_unique_name` and `home_prefix` (None on the home itself). A flow's id is
derived from the home's unique name, so it is the same in every solution; the flow's JSON uses the
home's prefix (`flow_prefix`). [One flow in several solutions](../6-ship/deploy.md#one-flow-in-several-solutions)
explains the rules.
