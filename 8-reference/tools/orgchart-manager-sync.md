# OrgChart Manager Sync

> **Reference.** The OrgChart Manager Sync XrmToolBox tool, version 1.0.0: it sets each employee's
> **Manager** (`parentsystemuserid`) in a development or test environment from the company OrgChart,
> after a read-only preview. It is not part of the web resources, button or flow deploy; it fills a
> user field. Its **Help** tab covers the same ground inside the tool.

It prefers to skip and report over guessing, and it writes nothing it has not shown you first.

## Where the plan comes from

| Input | Button | What it loads |
|---|---|---|
| **A. The OrgChart** | **Crawl OrgChart...** | runs the crawl for you, then loads the saved chart |
| | **Open OrgChart...** | a chart saved earlier by `pnpm orgchart:crawl` (GetChartData JSON, a list of them, or a HAR) |
| **B. A plan file** | **Open plan CSV...** | `orgchart-sync-plan.csv`, made by `pnpm orgchart:plan` ([the command-line tool](orgchart-sync.md)) |

**Crawl OrgChart...** asks for the `orgchart-sync` folder (needs `node` on the PATH and `pnpm install`
done once), the whole chart or one branch (box id or URL), the number of passes, and where to save
the JSON. It runs `node tools\orgchart.mjs crawl ...` as a child process, shows its output in the log
panel and loads the file when it ends. **Stop** ends the crawl. Headless Edge signs in to the OrgChart
with your Windows session; the tool itself reads and passes on no credential, cookie or token. This
is the one network call other than to Dataverse, and the crawl makes it, not the tool.

### Plan CSV format

RFC 4180, UTF-8, a header row, these columns:

`employeeId, employeeName, employeeEmail, managerId, managerName, managerEmail, unit`

`employeeId` and `managerId` are Dataverse `systemuserid` GUIDs; the other columns are for display and
reports. Problems are listed first and those rows are skipped, never fixed silently:

| Problem | Result |
|---|---|
| a bad GUID | row skipped |
| an employee who is their own manager | row skipped |
| an employee on several rows | all of their rows skipped |
| a cycle (found after the preview read, using the current managers of everyone not in the plan) | rows skipped |
| an empty file | nothing to preview |

## Steps

1. Connect to a **dev or test** environment. Load the plan (table above).
2. **Preview** (read-only). Chart input: the tool reads the enabled internal users of the
   environment, matches people by name, finds each person's supervisor by walking up the units and
   builds the plan. Each row gets one class ([below](#row-classes)). A warning shows when an unexpected
   class takes more than 25% of the plan. The plan is built per environment: reconnecting means
   previewing again.
3. Read the counts and the table. Filter it, select a row for its details, untick rows you do not want written.
4. **Apply** is enabled only after a preview of the same file, connection and checkbox. It asks dev or
   test (once), then shows the environment, the answer and the row count: "This writes to Dataverse."
5. Save the results and issues if you need them ([output files](#output-files)).

Running the same file again is safe: rows already correct are not written again.

**Only set empty managers** is on by default: *Will change* rows are listed but not written. Turning
it off asks for a second confirmation.

## Row classes

Every row is in exactly one class.

| Class | Means | Written |
|---|---|---|
| *Will set* | the employee has no manager yet | yes |
| *Will change* | the employee has a different manager | only when **Only set empty managers** is off |
| *Already correct* | the manager is already the planned one | no |
| *Employee not found* / *Employee disabled* | no usable user for the employee | no |
| *Manager not found* / *Manager disabled* | no usable user for the manager | no |
| *Issue* | a bad GUID, a self-manager, a duplicate employee or a cycle | no, listed |

## How people are matched (chart input)

A person is accepted only when **exactly one** user fits; the rest are never guessed. Accents, case and
punctuation are ignored. Rules, in order:

1. First and last name.
2. If several users share it, the middle name or initial decides, when both sides have one.
3. Otherwise the same words in any order across first, middle and last name.
4. Otherwise the first word of the first name and the last word of the last name.

People accepted by rules 2 to 4 are listed as `RelaxedMatch` in the review list so you can check
them. Looser overlaps (a second surname, for example) are only suggested as *Name differs slightly*
in **Resolve unmatched...**. People with no match, several matches, or an unmatched manager are
listed, never guessed: **Save reports > People to review...** exports them. People at the top of the chart have no
manager to set and are not listed.

### Resolve unmatched...

1. **Preview** first; the summary says how many people could not be matched.
2. **Resolve unmatched...** lists them with the reason (no user, several share the name, or the name
   differs slightly).
3. Narrow the list by reason (no user, several users share the name, name differs slightly, not chosen
   yet, chosen), by unit, or by words in the name or unit.
4. Select a person, search the Dataverse users on the right (or tick **Only suggested users (same or
   similar name)**; those users are marked `*` and listed first), then **Use this user**.
5. **OK**: the preview runs again with your choices. They last until you load a different chart.

## The screen

- A **banner** under the toolbar says the next step. It is red for a production-looking environment
  and amber when something needs attention.
- The **toolbar** follows the order of work: **1 Get the plan**, **2 Check**, **3 Write**. Every button
  has a tooltip.
- **Show / Unit / Search** filter the table by class (with counts), unit, and words in a name, email or unit.
- The **Write** column ticks the rows Apply writes; only rows that can be written have a tick box.
  **Tick shown** and **Untick shown** change every visible row, so you can apply one unit, or a few
  people, first. Unticked rows are reported as *Skipped*.
- The **details panel** explains the selected row: ids, current and planned manager, why it is in its
  class, and what Apply will do with it.
- The **log** records each step. It never holds secrets.

## How it writes

Rows are sent as `UpdateRequest`s that set only `parentsystemuserid`, in `ExecuteMultipleRequest`
batches of at most 100 (`ContinueOnError`, `ReturnResponses`). **Stop** finishes the batch in progress
and stops. A batch that fails with a temporary fault is retried once; validation faults are not.

| Setting (menu **Settings**) | Default | Meaning |
|---|---|---|
| Parallel batches | 1 | 1 to 4 batches written at once; each uses its own clone of the connection. 1 is the safest. |

## Output files

| Menu **Save reports** | File |
|---|---|
| **Result of the last apply...** | `timestamp, employeeId, employeeName, managerId, managerName, previousManagerId, outcome, message`; `outcome` is `Updated`, `Skipped` (including rows you unticked), `Failed` or `NotAttempted` |
| **Issues found in the plan...** | the issue list: row, reason, values |
| **People to review...** | chart input: `kind, detail` with `RootUser`, `MissingManager`, `RelaxedMatch` and `Unmatched` rows |

<Warning title="Real names">

The plan, the saved chart and every export hold real names, email addresses and ids. Save them only
where they may be kept, and do not share them. The tool shows this notice each time you save.

</Warning>

## Safety

- **Development and test only.** Before the first write you answer **Dev** or **Test** for the
  connected environment (kept for the session, per connection). There is no production choice.
- If the organization name or URL contains `prod`, `production` or `prd`, **Apply** stays disabled.
  Preview still works.
- It uses only the XrmToolBox connection: no sign-in cache, token file or PAC profile is read, no
  password is asked, and it talks only to the connected organization.
- It never deletes anything; it only sets the manager.

More on the shared rules: [XrmToolBox tools](overview.md#safety-rules-they-share).

## Install

Unlike the other three tools, this one is not in
[`tools/xrmtoolbox/tools.json`](../project/configuration.md#toolsxrmtoolboxtoolsjson), so `install.cmd` does
not fetch it. Close XrmToolBox, then copy `OrgChartManagerSync.dll` into its Plugins folder
([find it](../../2-set-up/01-install/02-xrmtoolbox-tools.md#1-find-the-plugins-folder)), unblock it
(`Unblock-File`), and start XrmToolBox. It shows as **OrgChart Manager Sync** in the **Tools** list.

Source: the plugin is in the private GitHub repo `OrgChartManagerSync`; the crawl and plan command-line
tool it can start is in `orgchart-sync`. Version 1.0.0 is released there as
`OrgChartManagerSync-v1.0.0.zip` (the DLL, built by CI): download it from the release page, unzip it and
copy the DLL as above. A release is made by pushing a tag such as `v1.1.0`.

To build it from source (from `XrmTools\`):

```powershell
.\build-all.ps1 -Only OrgChartManagerSync            # writes out\OrgChartManagerSync\
.\build-all.ps1 -Only OrgChartManagerSync -Deploy    # also copies it to the Plugins folder (XrmToolBox closed)
```
