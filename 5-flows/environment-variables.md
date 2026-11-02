# Environment variables

> **For:** a flow value that differs between dev, test and prod: a mailbox, a web address, a limit.
> **Result:** a flow that reads the value through a parameter, a solution that creates the variable on
> import, and a value set in each environment. The reference is in [flowkit](../8-reference/flows/flowkit.md#environment-variables).

A Dataverse environment variable is a named setting in the solution. A flow reads it through a
parameter, so the flow's code stays the same in every environment and only the value changes.

## 1. Declare it

Declare each variable once, next to the flow that reads it. The schema name starts with the
publisher's prefix (`prefix` in `flows/solution.json`) and an underscore.

```python
from lib.flowkit import EnvironmentVariable, Flow, chain, compose, http_request_trigger

MAILBOX = EnvironmentVariable(
    schema_name="pidev_SharedMailbox",  # <prefix>_<name>
    display_name="Shared mailbox",  # shown in Power Automate
    default="ops@example.com",  # for development only
    description="Sender of the alert emails.",
)
PORTAL = EnvironmentVariable(schema_name="pidev_PortalUrl", display_name="Portal URL", default="https://contoso.example.com")

flow = Flow(
    name="Show the mailbox",
    key="ShowMailbox",
    trigger=http_request_trigger(),
    actions=chain(
        {
            "Mailbox": compose(MAILBOX.expression),
            "Message": compose(f"Mail from {MAILBOX.interpolated} about {PORTAL.interpolated}"),
        }
    ),
    environment_variables=[MAILBOX, PORTAL],
)
```

`type` is `"String"` by default. `"Number"`, `"Boolean"` and `"JSON"` are also accepted; a `default`
must match the type (a Boolean default is `True` or `False`, a JSON default is the JSON text).

## 2. Use it

| You write | In the flow | Use it for |
|---|---|---|
| `MAILBOX.expression` | `@parameters('Shared mailbox (pidev_SharedMailbox)')` | a whole input, such as an email's `to` |
| `MAILBOX.interpolated` | `@{parameters('Shared mailbox (pidev_SharedMailbox)')}` | inside a longer text, such as an email body |

## 3. Build and import

`pnpm flows:build` writes one `environmentvariabledefinitions/<schema name>/` file per variable into the
solution zip, and the solution lists it as a component. The first import of the zip creates the
variable. A variable used by two flows is written once; declaring it twice with different settings
stops the build, and so does a name without the prefix.

Flows as Code pushes only the flow's `flow.json`. Its **Create new flows** creates a missing variable in the solution too (with the development default), so a new variable comes in with either the zip or that window.

## 4. Set the value in each environment

The default is only for development. After the import, set the value in each environment:

- **In Power Apps:** Solutions, open the solution, Environment variables, and set **Current value**
  (it is saved per environment).
- **At import:** a deployment settings file, given to the import, sets the values in one go:

```json
{
  "EnvironmentVariables": [
    { "SchemaName": "pidev_SharedMailbox", "Value": "ops@contoso.example.com" },
    { "SchemaName": "pidev_PortalUrl", "Value": "https://portal.contoso.example.com" }
  ],
  "ConnectionReferences": []
}
```

The real values are not in the flow's code. Keep each environment's settings file with that
environment's import steps ([Deploy](../6-ship/deploy.md)).

## 5. Check it

- `pnpm flows:build` fails with `parameters('X (Y)'): not declared in the flow's parameters` when a flow
  reads a variable it does not declare.
- A name that does not start with the prefix, or one declared twice with different settings, stops
  the build with a message that names the variable.
- Number, Boolean and JSON variables are declared to the same pattern, but the flow parameter types
  (`Int`, `Bool`, `Object`) are not yet checked against a flow in the dev environment. Check the first
  flow that uses one in the designer after the import.

---

**Back to:** [Build a flow](build-a-flow.md) · [flowkit reference](../8-reference/flows/flowkit.md#environment-variables)
