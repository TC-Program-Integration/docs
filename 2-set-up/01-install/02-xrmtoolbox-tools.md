---
sidebarTitle: XrmToolBox tools
---

# Set up the XrmToolBox tools

> **For:** installing, updating or repairing the team's four XrmToolBox tools. **Result:** they
> show in XrmToolBox's **Tools** list, connected to dev and pointed at the project.

The team's tools are not in XrmToolBox's Tool Library. They are DLL files that you put in
XrmToolBox's **Plugins folder**; XrmToolBox loads every tool it finds there when it starts.

| Tool | Release page | Files that go in the Plugins folder |
|---|---|---|
| Web Resources Manager | [TC-Program-Integration/WebResourcesManager](https://github.com/TC-Program-Integration/WebResourcesManager/releases/latest) | `MscrmTools.WebresourcesManager.dll`, `Fizzler.dll`, `Yahoo.Yui.Compressor.dll`, `Svg.dll`, `EcmaScript.NET.dll` |
| Command Buttons | [TC-Program-Integration/CommandButtons](https://github.com/TC-Program-Integration/CommandButtons/releases/latest) | `CommandButtons.dll` |
| Flows as Code | [TC-Program-Integration/FlowsAsCode](https://github.com/TC-Program-Integration/FlowsAsCode/releases/latest) | `FlowsAsCode.dll` |
| Schema as Code | [TC-Program-Integration/SchemaAsCode](https://github.com/TC-Program-Integration/SchemaAsCode/releases/latest) | `SchemaAsCode.dll` |

The list is `tools/xrmtoolbox/tools.json` ([configuration](../../8-reference/configuration.md#toolsxrmtoolboxtoolsjson)).

## 1. Find the Plugins folder

Start `XrmToolBox.exe` once and close it: the first start creates the folder.

| XrmToolBox | Plugins folder |
|---|---|
| installed the usual way | `%AppData%\MscrmTools\XrmToolBox\Plugins` |
| portable (settings kept next to the program) | the `Plugins` folder next to `XrmToolBox.exe` |

Paste `%AppData%\MscrmTools\XrmToolBox` in the File Explorer address bar to check: if it holds a
`Plugins` folder, that is the one.

## 2. Put the tools there

Close XrmToolBox completely first: it locks the DLLs it has loaded.

**With the script** (what [Install](../01-install.md#2-install-xrmtoolbox-and-the-teams-tools) does),
from the project folder:

```powershell
tools\xrmtoolbox\install.cmd                                       # latest releases
tools\xrmtoolbox\install.cmd -PluginsFolder D:\XrmToolBox\Plugins  # a portable XrmToolBox
tools\xrmtoolbox\install.cmd -Zip C:\Downloads\CommandButtons.zip  # a zip you were sent
```

**By hand**, when the script cannot run:

1. Download the zip of each tool from its release page (table above).
2. Unblock each zip: right-click > **Properties** > tick **Unblock** > **OK**, or
   `Unblock-File C:\Downloads\*.zip`. Windows blocks downloaded files, and XrmToolBox then skips
   the DLLs without a message.
3. Copy the DLLs straight into the Plugins folder, not into a subfolder.
4. Delete older copies the browser renamed, such as `FlowsAsCode (1).dll`: two copies of one tool
   stop it from loading.

To update a tool, do the same with the new release; the files are replaced.

## 3. Check and connect

1. Start XrmToolBox. The three tools are in the **Tools** list (search their name).
2. Connect to **dev** with your own work account (**Connect** at the top).
3. Open each tool once. The first time it meets an environment, it asks whether it is development
   or test; production is refused.

## 4. Point each tool at the project

Each tool remembers its settings per computer, so you do this once.

| Tool | First time | Page |
|---|---|---|
| Web Resources Manager | **Open > Solution...** (your solution), then **Link files** (arrow) > **Use a links file...** > the project's `webresources.links.xml` | [Web Resources Manager](../../8-reference/tools/web-resources-manager.md) |
| Command Buttons | the project folder (or its `dist` folder), the app and the solution | [Command Buttons](../../8-reference/tools/command-buttons.md) |
| Flows as Code | **Load flows > Flows of a solution...**, then **Folder...** > the project's `flows` folder | [Flows as Code](../../8-reference/tools/flows-as-code.md) |

Run `pnpm build` before: it writes `webresources.links.xml` and `dist/deploy.json`, which the first
two read.

## A tool does not show

| Check | Fix |
|---|---|
| the DLLs are in a subfolder of Plugins | move them to the Plugins folder itself |
| the DLLs are still blocked (right-click > **Properties** shows **Unblock**) | unblock them, or run `install.cmd`, which does it |
| a renamed copy is there (`(1).dll`) | delete it |
| XrmToolBox was open while you copied | close it, copy again, start it |
| it is the wrong Plugins folder | check step 1; a portable XrmToolBox ignores `%AppData%` |
