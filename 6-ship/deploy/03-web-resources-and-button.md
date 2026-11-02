---
sidebarTitle: "Web resources and button"
---

# Deploy, the first time: web resources and button

> **For:** step 3 of [the first time](01-first-time.md). **Result:** the four web resources of
> the feature exist in your solution, published, and the contact form has a *New callback
> request* button.

## Build

```powershell
pnpm build
```

The build prints every file to create, then the entry point for the button:

```text
  pidev_callback_request_command.js
  pidev_callback_request_page.js
  pidev_callback_request_page.html
  pidev_callback_request_page.css

Command entry points (command designer: Library and Function):
  Library  pidev_callback_request_command.js
  Function CallbackRequest.open
```

It also writes `webresources.links.xml` and `dist/deploy.json`, which the two tools below read.

## Create the web resources

They do not exist yet, so the Web Resources Manager creates them from the `dist` folder once.

1. In XrmToolBox, connect to **dev** and open the **Web Resources Manager**. The first time, it
   asks whether this environment is dev or test; it refuses production.
2. **Open > Local folder...** and pick the project's `dist` folder.
3. Tick the four `callback-request` files and create them **in your solution**
   (*Callback request*). Their names must be exactly the printed paths, prefix included.
4. **Link files** > **Use a links file...** > `webresources.links.xml`. Every web resource it
   lists is now linked to its file (once per computer).
5. **Push changes**: it runs `pnpm build`, shows what changed, pushes and publishes.

From now on, step 5 is all you do ([the deploy loop](../deploy.md#2-push-the-web-resources)).

<Warning title="The name must match exactly">

The command opens `pidev_callback_request_page.html`, and that page references
`pidev_callback_request_page.js` and `pidev_callback_request_page.css` by their full names
(the build writes those names into the HTML). One letter wrong and the dialog opens blank.

</Warning>

<Note title="By hand, in Power Apps">

Inside your solution, for **each** printed file: **New > More > Web resource**. **Name**: the
printed name exactly (if the box already shows `pidev_`, type only `callback_request_page.js`).
**Type**: `.js` Script (JScript), `.html` Webpage (HTML), `.css` Style Sheet (CSS). **File**:
from `dist/`. Save. Then **Publish all customizations**. Doing it once by hand shows you what a
web resource is; the tool is faster and makes no typing mistakes.

</Note>

## Add the command button

`pnpm new:feature` adds the feature to `webresources.config.mjs` but not its button. Add it under
`buttons`, keyed by the feature's name:

```js
buttons: {
  "callback-request": { label: "New callback request", table: "contact" },
},
```

`location` is `"form"` (the default) or `"grid"`; `function` is the exported function to run
(default `open`). Run `pnpm build` again: it checks the function exists in `command.js` and writes
the button to `dist/deploy.json`.

1. In XrmToolBox open **Command Buttons**.
2. Pick the project folder, the model-driven app that shows contacts, and your **unmanaged**
   solution. All three are remembered.
3. The button shows *Missing*. Other states: *Web resource missing* (create the web resources
   first), *Table missing*, *Problem in deploy.json*, *Blocked* (another app's command uses that
   name) ([all states](../../8-reference/tools/overview.md#command-buttons-deployjson)).
4. **Publish selected**. It shows what it will write, creates the button in your solution and
   publishes the app.

**Check:** open a saved contact in the app, press **Ctrl+F5** (so the browser forgets the old
version), and the button is on the command bar. Click it: the dialog opens.

<Note title="Not yet tried on a real environment">

How Dataverse stores the button's PrimaryControl parameter is not documented. If the button
runs but the dialog says it must be opened from a record, make one button by hand in the
command designer (below), then in Command Buttons use **Settings > Copy parameter format from
a command...** and publish again.

</Note>

<Note title="By hand, in the command designer">

In the app designer, open the **Contact** table's command bar (*Edit command bar*, **Main
form**). **New > Command**, label `New callback request`. **Action**: Run JavaScript.
**Library**: `pidev_callback_request_command.js`. **Function**: `CallbackRequest.open`.
**Parameter 1**: PrimaryControl. **Visibility**: on a saved record. **Save and publish** the
command bar, then the app.

</Note>

**Next:** [Flow, then try it](04-flow-and-try.md).
