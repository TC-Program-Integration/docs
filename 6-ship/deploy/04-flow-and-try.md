---
sidebarTitle: "Flow, then try it"
---

# Deploy, the first time: flow, then try it

> **For:** steps 4 and 5 of [the first time](01-first-time.md). **Result:** the *Callback alert*
> flow is in dev and on, `pnpm dv:verify` says everything is there, and the three test requests
> behave.

## 4. The flow

A flow reaches Dataverse in two ways. You use the first once, the second every day after.

| | First time | Every time after |
|---|---|---|
| What | **create** the flow from its file with [Flows as Code](../../8-reference/tools/flows-as-code.md) (**More > Create new flows...**). The solution zip that `pnpm flows:build` makes is the alternative | **push** `flow.json` with Flows as Code |
| Why | Flows as Code creates a flow only from a linked file; the window creates the connection references and environment variables it needs | fast: no export or import |
| Creates | the flow, off | nothing; it replaces the flow's steps |

### Check the settings and build

`flows/solution.json` holds the project's real `publisher` (its unique name, from step 1), `prefix`,
and a `uniqueName` for the flows solution. `flows/callback-alert/settings.py` holds the real names
from step 2 (table set name `pidev_callbackrequests`, the lookup `_pidev_contact_value`).

```powershell
pnpm flows:build
```

It formats, lints and type-checks the Python, checks every flow (a mistake stops the build with the
reason), and writes:

```text
flows/out/CallbackAlert/flow.json          the flow, to read or to push
flows/out/flows.json                       key, name and id of each flow, for dv:verify
flows/out/MyProjectFlows_1_0_0_0.zip       the solution to import once
```

Its last lines list the **connection references** each flow needs (here a Dataverse one and an
Outlook one).

### Link it in Flows as Code

The links file names the flow before Flows as Code can create it.

1. If the flows solution is not in dev yet, create it: Power Apps (dev) > **Solutions > New
   solution**, with the `displayName`, `uniqueName`, publisher and `version` of
   `flows/solution.json`.
2. `pnpm flows:build` adds the flow to `flows/flows.links.json`: `id` is the `workflowId` in
   `flows/out/flows.json`, `name`, and `file` is `out/CallbackAlert/flow.json`. It also sets
   `buildCommand` to `pnpm flows:build`. Check the entry is there, and allow the build command once:
   every push then rebuilds first.
3. In XrmToolBox, connect to dev and open **Flows as Code**. Answer *dev* when it asks. Load the
   flows solution (**Load flows > Flows of a solution...**) and set **Folder...** to the project's
   `flows` folder.

### Create it (first time only)

1. **More > Create new flows...**. The window lists the flows of the links file that dev does not
   have yet. Tick the flow and create it. The window creates the missing connection references and
   environment variables first, then the flow. It is created **off**.
2. Before you turn the flow on, open the solution in Power Apps (dev). Open each new connection
   reference and pick a connection signed in as **you** (or the project's service account).

**Or: import the zip.** Power Apps (dev) > **Solutions > Import solution > Browse** > the zip >
**Next**. The wizard asks for a connection for each connection reference (one to Dataverse, one to
Outlook, signed in as **you**): pick or create them, then **Import**. The flow comes in inside a new
solution, probably **Off**. Then link it as above, using **Link to an existing file** >
`flows/out/CallbackAlert/flow.json`. If your project keeps the flows in the same solution as the
table, add the flow there too: in *Callback request* > **Add existing > Automation > Cloud flow**.

### Look at it, turn it on

Open the flow in Power Automate. You should see the shape you wrote: a trigger, two *Get a row*, a
condition and an email inside a `Try` scope, with a `Catch` scope beside it. A red **warning** on a
step (for example "connection invalid") is a connection problem, not a code problem: open the step
and pick a valid connection. Then turn the flow **On**.

From now on a change is: edit `flow.py`, **Push changes**
([the deploy loop](../deploy.md#4-push-the-flows)). A change made in the designer must be copied
back to `flow.py`, or the next push overwrites it.

## 5. Try it in dev

First check every piece is there:

```powershell
pnpm dv:verify
```

Every line should say `ok`: the four web resources, the button, and the flow (exists and is on).
A `NO` names what is missing ([the deploy loop](../deploy.md#5-verify)).

Then do these three on a real contact, with your mentor.

| Try | Expect |
|---|---|
| **A normal request** (priority Normal) | the dialog saves; a *Callback request* record has every field as you typed it; **no flow run** |
| **An Urgent request with a phone** | the record exists; one run, **Succeeded**; the contact's owner gets the email (check their inbox, or *Sent items* of the connection's account) |
| **A request a rule must refuse** (Urgent with no phone) | the dialog shows the rule's message and **nothing** is saved |

Where to look:

- **The record:** the app > Callback requests > open it, and compare each field with what you typed.
- **The run:** the **Runs** tab of Flows as Code lists each run, its status and its error;
  double-click opens it in Power Automate, where every step shows its inputs and outputs
  ([when a flow fails](../when-a-flow-fails.md)).
- **No run at all:** the flow is Off, or the trigger's filter did not match (Urgent is the number
  `2`; check the choice values from step 2).

Fix, rebuild, push, and repeat. When all three behave, the work is in dev.

**Next:** [Export, import, go back](05-export-import.md).
