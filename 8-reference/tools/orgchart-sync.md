# OrgChart Sync (command line)

> **Reference.** The `orgchart-sync` command-line tool: it crawls the company OrgChart, matches its
> people to Dataverse users and writes the manager plan. It never writes to Dataverse. The
> [OrgChart Manager Sync](orgchart-manager-sync.md) XrmToolBox tool applies the plan, and can also
> start the crawl and build the plan itself.

You need this tool only to run the steps from a terminal, or to make `orgchart-sync-plan.csv` for the
plugin's **Open plan CSV...** button. The plugin does not need it to preview or apply a chart you
saved, and it starts the crawl through this tool.

## Set up

| You need | For |
|---|---|
| Node 24 and `pnpm install` once in the folder | every command |
| Microsoft Edge and a Windows sign-in to the OrgChart | the crawl (headless Edge signs in with your Windows session) |
| The Power Platform CLI, signed in with `pac auth create` | reading Dataverse users (`match`, `plan`, `all`) |

Dataverse is reached only through `pac org fetch`, read-only, as you
([access and security](../../9-help/access-and-security.md)). The tool reads no sign-in cache and
handles no token, cookie or password. The crawl reuses the Edge session only to ask the OrgChart for
its data.

## Commands

Run them from the `orgchart-sync` folder, which is its own repository and not part of this project, so
its `pnpm orgchart:<name>` scripts are not in this project's `package.json`. `<source>` is a saved chart file (GetChartData JSON, a list of
them, or a HAR), an OrgChart URL, or a box id.

| Command | What it does |
|---|---|
| `pnpm orgchart` (or `orgchart:all`) | prints the hierarchy overview, then writes the plan |
| `pnpm orgchart:crawl <source> [--root] [--passes N] [--out file]` | crawls a branch (or the whole chart) and saves it |
| `pnpm orgchart:flatten <source> [--csv file]` | counts units, positions and people; `--csv` writes the flat rows, one JSON object per line |
| `pnpm orgchart:match <source> [--env url] [--refresh]` | matches people to enabled Dataverse users by name, writes `orgchart-match.json` |
| `pnpm orgchart:nesting <source> [--depth N] [--person name] [--summary] [--json file]` | prints the hierarchy tree, one person's chain of command, or only the totals |
| `pnpm orgchart:tree <source>` | the same as `nesting` |
| `pnpm orgchart:plan <source> [--env url] [--refresh]` | writes the manager plan for the plugin |

### Options

| Option | Default | Meaning |
|---|---|---|
| `--root` | off | start from the top of the chart (box `HQ--00001`) |
| `--passes N` | 10 | rounds of child units to expand during a crawl |
| `--out file` (or `--json file`) | `tools/.out/orgchart-full.json` | where `crawl` saves the chart |
| `--env url` | the active `pac` environment | the Dataverse environment to read users from |
| `--refresh` | off | read the users again instead of using the cached list |
| `--depth N`, `--person name`, `--summary` | whole tree | limit what `nesting` prints |
| `--crawl` (or `-r`) | off | crawl even when a saved chart exists |
| `--pw` (or `--browser`) | off | load a URL in headless Edge instead of a plain request |
| `--cookie value` | none | a cookie for a plain request (also read from the `ORGCHART_COOKIE` variable) |

Environment variables: `PW_CHANNEL` picks the browser for the crawl (default `msedge`) and
`PAC_CONCURRENCY` sets how many `pac` calls run at once (default 6; 1 runs them one at a time).

### Which chart is used

- A saved file is read as it is. A URL is fetched; an OrgChart URL goes through headless Edge.
- A box id (anything with a `-` that is not a file) is crawled, so is `crawl`, `--crawl` and `-r`.
- With no source: `--root` means `HQ--00001`; otherwise `tools/.out/tc-full.json` is used when it exists
  and `--crawl` is not given; otherwise the crawl starts at `HQ--00001`.

## How the plan is built

1. The chart is flattened to one row per position, with each unit indexed by its box id.
2. People are matched to enabled internal Dataverse users (not application users) by **first and last
   name**: accents, case and punctuation ignored, middle names ignored. A person is accepted only when
   **exactly one** user has that name. The plugin adds relaxed middle-name rules on top of this
   ([how people are matched](orgchart-manager-sync.md#how-people-are-matched-chart-input)).
3. Each person's supervisor is found by walking up the units: the manager of their own unit (another
   person than themselves), otherwise the manager of the unit above, and so on. A unit's own manager
   reports to the unit above.
4. A person goes into one list:

| List | Meaning |
|---|---|
| `ready` | the person and the supervisor each match exactly one user: a plan row |
| `rootUsers` | matched, but nobody above them: no manager to set |
| `missingManager` | matched, but the supervisor matches no user, or several |
| `unmatched` | no user, or several users, with that name |

Reading the users takes four `pac org fetch` calls, one per group of last-name letters (A to F, G to L,
M to R, S to Z), so a large result does not overflow `pac`. The list is cached for 24 hours.

## Output files

Written under `tools/.out/`, which git ignores:

| File | Holds |
|---|---|
| `orgchart-sync-plan.csv` | the plan for the plugin: `employeeId, employeeName, employeeEmail, managerId, managerName, managerEmail, unit`, one row per `ready` entry |
| `orgchart-sync-plan.json` | the same rows, plus `rootUsers`, `missingManager` and `unmatched` for review |
| `orgchart-match.json` | the match report of `orgchart:match` |
| `orgchart-full.json` | the crawled chart, when `--out` is not given |
| `dataverse-users.json` | the cached list of users |

<Warning title="Real names">

Every file in `tools/.out/` holds real names and email addresses. Do not commit or share them.

</Warning>

## Check it

`pnpm test` runs the unit tests of the crawler, loader, nesting and plan builder. The plan commands print
a summary (people in the chart, ready, at the top, missing manager, unmatched) and the line
`No changes made to Dataverse`.

Source: the private GitHub repo `orgchart-sync`. Apply the plan with
[OrgChart Manager Sync](orgchart-manager-sync.md).
