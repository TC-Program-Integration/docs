# Web Resources Manager

> **Reference.** The team version (2.2026.9.25) of the XrmToolBox Web Resources Manager: the parts the project
> uses. Steps in order: [Web resources and button](../../6-ship/deploy/03-web-resources-and-button.md).
> Its **Help** tab covers everything else.

It reads [`webresources.links.xml`](../configuration.md#webresourceslinksxml), which `pnpm build`
writes at the root of the project.

## Open web resources

| Menu | Opens |
|---|---|
| **Open > Solution...** | the web resources of one solution: what you use most. The last one reopens on connect; **Recent solutions** are one click away. |
| **Open > All web resources of the organization...** | everything you can see (slower) |
| **Open > Local folder...** | files of a folder, not yet in Dataverse: use it to **create** web resources the first time a feature is deployed (pick `dist`, tick the feature's files, create them in your solution) |

## Links

| Action | Does |
|---|---|
| **Link files** (arrow) > **Use a links file...** | pick `webresources.links.xml`: every web resource it lists is linked at once (once per computer) |
| **Link files...** | the links window: **Auto-match from folder...**, link by hand, **Unlink**. Nothing changes until **Save**. |
| **Link files** (arrow) > **Stop using the links file** | back to links kept on this computer only |

New links you make to files inside the repo are added to the links file, and `pnpm build` keeps
them. Links in the file can only point inside the repo (a path such as `../../somewhere` is
ignored); links to files outside it stay on your computer. A web resource whose file is missing
shows in gray italic.

In the links window (**Link files**): **Auto-match from folder...** proposes a file for each web
resource by name (untick what you do not want); double-click a row, or drop a file on it, to link by
hand (dropping several files links each to the web resource of the same name); **Unlink** removes a
link; **Open folder** shows the file in Explorer. Also by right-click: a web resource >
**Link to a local file...**, a folder > **Link to a local folder...**. **Link files** (arrow) >
**Save links to a file in your repo...** creates the links file for the team.

## The explorer

| Color | Means |
|---|---|
| blue | changed, not pushed to the environment |
| red | unsaved edits in the tool's own editor (save with Ctrl+S) |
| gray italic | not linked to a local file |

Hover a web resource for its state and local file. Tick web resources, then use the **Checked** menu
to push, save or delete them together. Opening again with unsaved edits asks first.

## Watching and pushing

- A saved linked file marks its web resource as changed (blue). **Refresh from disk** reads every
  linked file again (after a git pull or a branch switch, or a network drive). A bar names the
  refreshed web resources; ones with unsaved edits in the tool are not overwritten. The
  **Watching N linked file(s)** button shows the folders watched and the last change seen: if a save
  is not picked up, check it first.
- **Push changes (n)**:
  1. runs the build command: the links file's `<BuildCommand>pnpm build</BuildCommand>`, in the
     folder of the links file. It is shown and must be allowed the first time and whenever it
     changes. A failed build stops the push and shows its output. A build is stopped after 10 minutes.
  2. reads every linked file again;
  3. checks whether someone else changed the same web resources in the environment;
  4. opens the push window.

  Unsaved edits in the tool's own editor are left out: save them first. If your command is not in
  the links file, **Settings > Build and Push** has **Build command** and **Build folder** (the
  links file's folder when empty); the links file wins.
- **The push window**: untick what to leave out; **Compare...** (or double-click) shows the
  differences; **Mark as unchanged** forgets a change without pushing it (the content is not
  reverted); **Publish** is ticked by default; **Add to a solution...** also adds them to a
  solution; **Push to** picks the environment. The **Changes to push** panel (View menu) lists the
  same changes and stays open.
- **Changed by someone else**: shown in red and unticked; ticking one asks you to confirm the
  overwrite. **Compare...** opens two tabs: what someone else changed on the environment, and the
  environment against your version. A record only republished with the same content is not reported.
- **Compare** is also in the right-click menu (**Compare with organization**) and in an editor tab
  (**Compare > With the organization version...**). **Settings > Compare Settings** sets your own
  tool (**Tool path**, **Tool arguments**).
- **Push watched files automatically** (Settings) pushes each saved file without the review. It is
  off by default; keep it off.

## Environments

The environment you are connected to is pushed to directly. **Push to > Add an environment...**
pushes to another one: the tool asks once whether it is development or test, and refuses
production and environments labelled PROD in XrmToolBox (shown as *not allowed*). Answers are in
**Settings > Environment types**; remove one to be asked again. Web resources are matched by name on
the other environment: existing ones are updated, missing ones created, then published and added to
a solution if you asked. Pushing elsewhere does not clear them from the changes of your current
environment.

## Editing and testing in the tool

Your own editor is the usual way (**Open in your editor**: the **Editor command** setting, `code` by
default, Notepad if it cannot start). Double-clicking a web resource opens it in a tab; images are
previewed. Editor tab keys:

| Keys | Does |
|---|---|
| Ctrl+S | save in the tool (turns it blue) |
| Ctrl+P | save, push and publish this web resource |
| Ctrl+U | push and publish as last saved |
| Ctrl+F / Ctrl+H / Ctrl+G | find / replace / go to line |
| Ctrl+K then Ctrl+C / Ctrl+U | comment / uncomment lines |
| Ctrl+M then Ctrl+O | fold everything |
| F1 | the help |

Right-click a script > **Open a form using this script...** lists the main and quick create forms
that load it and opens one in the browser as a new record. With **Open the form of a script after
pushing and publishing it** ticked (**Settings > Build and Push**), that form opens after each push
(three forms at most). **Add this script to a form...** is there when none is listed.

## Settings

**View > Settings**, kept per connection on your computer:

| Group | Settings |
|---|---|
| Build and Push | **Build command**, **Build folder**, **Editor command**, **Open the form after push**, **Links file**, **Environment types** |
| Local Sync Settings | **Watch linked files** (on), **Push watched files automatically** (off), **Local folder for webresources** |
| Loading Settings | **Reopen last solution**, prefixes to include or exclude, managed and hidden web resources |

## Safety

The tool uses your XrmToolBox connection and rights and sends no telemetry. Nothing changes without
an explicit push, publish or delete, except with **Push watched files automatically**. The build,
editor and compare commands run on your computer with your rights; a build command from the repo
runs only after you allow it. Web resource names and links from the repo cannot make it read or
write outside their folder, and SVGs that declare a DOCTYPE are not previewed.

## When it does not work

| Problem | Do |
|---|---|
| saved files are not picked up | check the **Watching** button counts your linked files, and that the web resource is linked (not gray italic); **Refresh from disk** always works |
| nothing to push | the files are identical to the environment: compare one |
| the build failed | the window shows the output; fix it and push again (nothing was pushed) |
| an environment is *not allowed* | it is marked production; if wrong, remove it in **Environment types** and pick it again |
| a web resource stays red | it has unsaved edits in the tool: Ctrl+S in its tab |

## Install

`tools\xrmtoolbox\install.cmd` ([XrmToolBox tools](../tools.md#install-or-update-them)). The zip
holds `MscrmTools.WebresourcesManager.dll` and the libraries it needs (`Fizzler.dll`,
`Yahoo.Yui.Compressor.dll`, `Svg.dll`, `EcmaScript.NET.dll`).
