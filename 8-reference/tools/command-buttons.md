# Command Buttons

> **Reference.** The Command Buttons XrmToolBox tool, version 1.0.1: it creates or updates the
> command bar button of each feature from [`dist/deploy.json`](../configuration.md#distdeployjson),
> instead of adding it by hand in the command designer. Steps in order:
> [Web resources and button](../../6-ship/deploy/03-web-resources-and-button.md).

## What you pick

| Setting | Remembered |
|---|---|
| the **project folder** (the tool reads `dist/deploy.json` in it; picking `dist` itself works too) | yes |
| the **model-driven app** the buttons go to | per environment |
| the **unmanaged solution** new buttons are added to | per environment |

Pick them with the toolbar buttons **Repo folder...**, **App...** and **Solution...**. Select a
command to see its fields, and what differs, in the bottom pane. Tick the ones to publish: missing
and different ones are ticked for you.

Run `pnpm build` first: the buttons come from `buttons` in
[`webresources.config.mjs`](../configuration.md#webresourcesconfigmjs). Push the web resources
first too (Web Resources Manager): this tool never uploads web resources.

## States

Each command of `deploy.json` shows one state:

| State | Means | What to do |
|---|---|---|
| *Missing* | no button with this unique name in the app | **Publish selected** creates it |
| *Up to date* | the button matches `deploy.json` | nothing |
| *Different* | a field differs; each one is listed, old -> new | **Publish selected** updates it |
| *Web resource missing* | the `library` web resource is not in the environment | push the web resources, then reload (F5) |
| *Table missing* | no table has that logical name | fix `table` in `webresources.config.mjs` |
| *Problem in deploy.json* | the entry cannot be used: a field is missing or malformed, two commands share a unique name, or a parameter has no known format | fix the config, `pnpm build`, reload |
| *Blocked* | the button is managed here, or the unique name is already used by another app's command or by a table or global command | for a managed button, change it where it is developed; for a name, change `function` or `table` (the name is made from them) |

An entry with a problem is listed in red with the reason and is never published; the other entries
still work. What is compared: name, label, table, location, button type, action (JavaScript),
library, function, parameters, hidden, and `icon`, `tooltip` and `sequence` when `deploy.json`
gives them.

## Publish selected

1. Shows exactly what will be created or changed, field by field.
2. Reads the environment again: a button changed since the preview is skipped.
3. Creates new buttons **in your solution**; updates only the fields that differ; adds an updated
   button that is not in the solution yet to it. A failure on one button does not stop the others.
4. Publishes the app once. Reload the app in the browser (Ctrl+F5) to see the buttons.

The results window says what happened to each button. It **never deletes** a button; to remove one,
use the command designer. Development and test environments only: you are asked once per
environment; production, and environments labelled PROD in XrmToolBox, are refused. To be asked
again, use **Settings > Forget environment answers**. Reading `deploy.json` and the states works
everywhere; nothing is written until you publish.

## How a button is written

Each command becomes an `appaction` record of the app:

| `deploy.json` | Button |
|---|---|
| `uniqueName` | its unique name (set on create) and name |
| `label` | the button label |
| `table` | the table it shows on |
| `location` | `form`: the form's command bar; `grid`: the main grid's |
| `library`, `function` | the JavaScript web resource and the function the button runs |
| `parameters` | the parameters passed to the function: `PrimaryControl` (the form) or `SelectedControl` (the grid) |
| `sequence` (optional) | its place in the command bar. Without it, a new button gets 100 and a place you set in the command designer is kept. |
| `icon`, `tooltip` (optional) | font icon; tooltip title and description |

Set `sequence`, `icon` and `tooltip` on the button in `webresources.config.mjs` to have them in
`deploy.json`. Without `sequence`, move the button in the command designer and the tool keeps that place.

## The parameter format (unverified)

Microsoft does not document how the command designer stores a button's parameters: a JSON list with
one item per parameter. The tool writes `PrimaryControl` as `{"type":5}` and `SelectedControl` as
`{"type":12}`, the way community tools report the designer's values; this is not yet confirmed on a
real environment. A raw item such as `{"type":5}` in `deploy.json` is stored as is. If the button
runs but your function gets no form:

1. Make one button by hand in the command designer that passes **PrimaryControl**.
2. In the tool, **Settings > Copy parameter format from a command...**, pick that button, and say
   it stands for `PrimaryControl`.
3. **Publish selected** again: that exact format is used from then on, for that exact parameter list.

Any other parameter name is refused until a format is copied for it. Copied formats are kept on
your computer and can be forgotten from the same Settings menu.

## Unique names

The `uniqueName` is used as is. If a command of **another** app already uses it, the button is
*Blocked*: Dataverse unique names cannot be shared, so change `function` or `table`.

## Safety

The tool uses your XrmToolBox connection and your rights, talks only to the connected environment,
and only reads `dist/deploy.json`: it runs nothing from the repo. Its settings stay on your
computer (repo folder, environment answers, app and solution, copied parameter formats).

## Keys

F5 reads `deploy.json` and the environment again; F1 opens the help.

## Install

`tools\xrmtoolbox\install.cmd` ([XrmToolBox tools](../tools.md#install-or-update-them)); the zip
holds `CommandButtons.dll`.
