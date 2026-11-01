---
sidebarTitle: "Export, import, go back"
---

# Deploy: export, import, go back

> **For:** step 6 of [the first time](01-first-time.md), and step 7 of every
> [deploy](../deploy.md#7-prod). **Result:** the solution, tried in dev, runs in test or prod, and
> you know how to go back.

Test and prod never receive hand edits. They receive **the solution**, exported from dev and imported
there, so what runs in test is exactly what you tried in dev.

## Export

1. **Raise the version.** In dev, open the solution > **Solution details** (or *Edit*) and set the
   version, for example `1.0.0.1`. Do the same in `flows/solution.json` (`version`) so the next
   `pnpm flows:build` zip has the same number. A solution cannot be imported over a *higher or equal*
   version of itself as an upgrade: the number is how Dataverse knows it is newer.
2. **Publish all customizations** (an unpublished change is not exported).
3. **Export** > the wizard first runs **Check** (look at its warnings; the missing-dependency ones
   matter) > **Next**.
4. **Managed or unmanaged?**

| | Managed | Unmanaged |
|---|---|---|
| Use for | test and prod | moving work between dev environments |
| Editable there | no, locked | yes |
| Remove later | delete the solution and its pieces go | pieces stay behind |

   Ask your mentor what the project uses; the default is **managed**.
5. **Export** and download the zip. Name it after the version, e.g. `CallbackRequest_1_0_0_1_managed.zip`.

If the flows are in a second solution, export it too. Import order matters (below).

## Import in test

1. Power Apps > switch to the **test** environment (top right; read its name twice).
2. **Solutions > Import solution** > the zip. Import the solution with the **table first**, then the
   flows solution, because the flow needs the table to exist.
3. **Connections.** The wizard asks for a connection for each connection reference. Create them for
   **test**, signed in with the account the flow should run as in test (often a service account, ask
   your mentor). Never reuse your personal account in prod.
4. **Import.** Wait for the green bar. A red one has a **Download log** link: read it, then see
   [troubleshooting](../../9-help/troubleshooting.md#import-and-solutions).
5. **Turn the flow on.** Imported flows often arrive **Off**.
6. **Publish** if it asks, then open the app and press Ctrl+F5.
7. `pnpm dv:verify --env https://<test-org>.crm3.dynamics.com` checks every web resource, button
   and flow arrived, and that the flows are on.

### The smoke test

A quick proof that the deployment works, with a record you then delete:

1. Open a test contact, press the button, save a Normal request: the record exists, **no flow run**.
2. Save an Urgent request: one run, Succeeded, the email arrives.
3. Delete the two test records.

Write the result (and the version) in your task. If one step fails, [go back](#go-back); do not "fix it
quickly in test".

<Note title="Prod">

The path is the same with the **prod** environment, a mentor or release manager, and often a
pipeline instead of your hands. First run `$env:DATAVERSE_URL="https://<prod-org>.crm3.dynamics.com"; pnpm test:dataverse`
to see that prod has what the code uses.

</Note>

## Go back

You cannot press undo on an import, so decide *before* you import:

| Situation | What to do |
|---|---|
| Broken after a **first** import (nothing existed before) | **delete the managed solution** in that environment: all its pieces go with it, **including the data of its tables** |
| Broken after an **upgrade** | ask your mentor. Dataverse will not install an older version over a newer one, and deleting a managed solution deletes its tables' data, so the usual way out is a new version with the fix (or the previous code under a higher number) |
| A flow only is wrong | turn that flow **Off** in the environment, fix in dev, export a new version |
| Data was created by the broken version | records are not removed by deleting code; list and fix them with the owner |

Always keep the zip of every version you imported. It is your way back.

**Next:** [when a flow fails](../when-a-flow-fails.md), or [make a release](../release.md) to
hand the project over.
