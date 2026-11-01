# Troubleshooting

> **For:** when something does not work. **Result:** you know which tool shows the cause, and the
> usual fix. Find your symptom; error texts are shortened, so search for a distinctive part of yours.

Most problems are shown by one tool. Run it before you guess:

| Where it breaks | Run | It shows |
|---|---|---|
| your laptop, the install, the pac sign-in | `.\setup.cmd -Doctor` | every tool and its version, the usual blockers, the sign-in; writes `doctor-report.txt` to send. Changes nothing |
| the code, before any deploy | `pnpm check` | format, lint, types, tests, flow checks, docs build |
| a flow, before any push | `pnpm flows:build` | the flow's mistakes, by name, before anything reaches Dataverse |
| table and column names | `pnpm test:dataverse` | every name the code uses that the environment does not have |
| what is in the environment | `pnpm dv:verify` | each web resource, button and flow: missing, off, or ok |
| someone's access | `pnpm dv:access [email]` | user enabled, roles (direct or by team), publisher, solution |
| a flow that ran | the **Runs** tab of Flows as Code | each run, its status and its error ([when a flow fails](../6-ship/when-a-flow-fails.md)) |
| a page in the app | the browser's dev tools (F12) | the console error and the files loaded |

When you ask for help, send the **exact message**, the **environment** and **what you clicked**.

## My change is not in the app

| Symptom | Shows the cause | Fix |
|---|---|---|
| The change does nothing | `pnpm dv:verify` | missing: push it with the Web Resources Manager. Everything ok: **Publish all customizations**, then Ctrl+F5. F12 > Sources shows which file the browser has |
| No button | `pnpm dv:verify` | missing: **Publish selected** in Command Buttons. Present: publish the app, Ctrl+F5, open a *saved* record (a form button hides on new ones) |
| Command Buttons shows *Web resource missing* | the tool | push the web resources first |
| Command Buttons shows *Blocked* | the tool | another app's command uses that unique name; ask who owns it |
| A tool refuses the environment | the tool | the XrmToolBox tools push to dev and test only; prod gets the [solution](../6-ship/deploy/05-export-import.md) |

## The button or the dialog

| Symptom | Shows the cause | Fix |
|---|---|---|
| Button does nothing, console: `QuickTask is not defined` | F12 console | the library must be `<prefix>_<feature>_command.js` and the function as `pnpm build` prints it (`QuickTask.open`); then publish |
| Dialog blank or 404 | `pnpm dv:verify` | a web resource is missing or misnamed: names are exactly as `pnpm build` prints them, prefix included |
| Dialog loads without styles or script | `pnpm dv:verify` | `page.html` references its `page.js` / `page.css` by full name (`<prefix>_<feature>_page.js`); upload all three under exactly the names `pnpm build` prints |
| Dialog blank, console: `page.html has no #root element.` | F12 console | put `<div id="root">` back in `page.html` |
| "This page must be opened from a record" | the button | the record is new (save it first), or the button does not pass PrimaryControl: in Command Buttons, **Settings > Copy parameter format from a command...** ([tools](../8-reference/tools.md#command-buttons-deployjson)) |
| "The Power Apps client API (Xrm) is not available" | | the page was opened outside the app: open it from the app, or use `pnpm dv:preview` |
| "Save the record first." | | expected on a new record; hide the command on create forms if needed |
| Where is my `console.log`? | F12 console | it runs in the dialog's iframe: pick that frame in the console's context list |
| Stack traces point into a huge `page.js` | | debug with `pnpm dv:preview`, where source maps are inline |

## Saving or reading fails

| Message (or part) | Shows the cause | Fix |
|---|---|---|
| `Resource not found for the segment 'pidev_address'` | `pnpm test:dataverse` | the logical name was used where the entity set name is needed: `Metadata.recordPath`, or `pnpm dv:pull` ([read real data](../4-build/read-real-data.md)) |
| `An undeclared property 'pidev_manager'` / `does not have a property named` on `@odata.bind` | the relationship's page | the navigation property's name or letter case is wrong: use the schema name (`pidev_Manager`) in `config.ts` |
| `Could not find a property named 'pidev_manager'` on read | | a lookup is read as `_pidev_manager_value`: `readLookup(record, "pidev_manager")` |
| A choice saved as `100000002` | `pnpm dv:pull <table>` | the option values were not set: Low 0, Normal 1, Urgent 2; fix existing records |
| `principal user ... is missing prv<Action><Table> privilege` | `pnpm dv:access [email]` | the role lacks that privilege: ask for the role change. Test with a normal user, never work around it |
| "You do not have permission" | `pnpm dv:access [email]` | ask your mentor for the right role on the table |
| `A record with matching key values already exists` | | an alternate key or a duplicate: look up the existing row first |
| `0x80040217 ... Does Not Exist` | `pnpm dv:q` | a wrong id, or a row the user cannot see: check sharing and ownership |
| HTTP 429 / "Number of requests exceeded" | | service protection limits: one query with `$filter`, not one per row |

## A flow does not run or fails

| Symptom | Shows the cause | Fix |
|---|---|---|
| `pnpm flows:build` stops: "not a Power Automate expression function", brackets that "do not match", `items('X'): used outside the 'X' loop`, "not initialized" | `pnpm flows:build` | the message names the expression; fix `flow.py` ([what the build checks](../5-flows/build-a-flow/03-rules-and-testing.md)) |
| Push refused: "Unable to parse template language expression" | `pnpm flows:build` | rebuild: the build catches unknown functions and unbalanced brackets |
| No run at all | `pnpm dv:verify` | the flow is off, or the trigger filter does not match |
| A run failed | the **Runs** tab | [when a flow fails](../6-ship/when-a-flow-fails.md) lists the usual errors |
| The Runs tab is empty | the tab's message | no runs in the period, no read access to the Flow Run table, or runs older than 28 days: use Power Automate's run history |
| Create new flows leaves a flow gray | the window's **Ready** column | a missing file, a file with errors, or a flow with its id outside the solution; the window creates the connection references and environment variables itself: [create it](../6-ship/deploy/04-flow-and-try.md#create-it-first-time-only) |

## Import and solutions

| Message says | Shows the cause | Fix |
|---|---|---|
| "Invalid character specified for publisher unique name" | `pnpm dv:q tools/queries/publishers.xml` | the display name was used: put its `uniquename` in `flows/solution.json` |
| missing dependency | the import log | import the solution that holds it first (the table before the flow), or add it to yours |
| the version is not newer | | raise the version in dev (and `flows/solution.json`), export again |
| a connection cannot be created | `pnpm dv:access` | you may lack the right in that environment: ask your mentor |
| anything else | **Download log** | find the first line with "error" and send it |
| The flow is off after import | `pnpm dv:verify --env <url>` | set its connections, then turn it on |

## Setup, pac and sign-in

| Symptom | Shows the cause | Fix |
|---|---|---|
| Not sure what is wrong with your setup | `.\setup.cmd -Doctor` | send `doctor-report.txt` |
| `scoop`, `git`, `node`, `pnpm`, `uv`, `pac` or `code` is not recognized | `.\setup.cmd -Doctor` | open a **new** terminal; then the check at the top of [install and sign in](../2-set-up/01-install.md) and fix the `PATH` |
| Scoop installer stops: the folder exists and is not empty | | Scoop is already installed: do not delete the folder; add `$HOME\scoop\shims` to the `PATH` |
| `claude` is not recognized | | the Claude Code CLI is installed separately (not by Scoop): its official instructions, then `Get-Command claude` |
| `node --version` is not v24 | `Get-Command node -All` | `.\setup.cmd -Clean` clears the other Node and installs Node 24 |
| `pnpm` is not recognized after installing Node | | `corepack enable pnpm`, then a new terminal |
| Setup sits on step 4 (pnpm) with no output | | corepack is asking to download pnpm and the question is hidden: press Ctrl+C, run `pnpm install` once and answer Y, then run `setup.cmd` again (an updated setup no longer asks) |
| "NVM blocked package-manager execution" (NVM4306), or "pnpm is not installed for this Node.js version" | `.\setup.cmd -Doctor` | nvm blocks tools it did not install, or nvm runs another Node. `.\setup.cmd -Clean` clears nvm and installs Node 24 and pnpm with scoop |
| Works now, broken in a new terminal | `.\setup.cmd -Doctor` (the "new terminal" line) | `.\setup.cmd -Clean` |
| "running scripts is disabled" | | `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` (not administrator); it fixes only this message |
| An install times out, or a certificate or 403 error | | a proxy blocks the download: send IT the full message; look inside the partial folder before removing it |
| `python` opens the Microsoft Store | | use `py`, or turn off the python App execution aliases in Windows settings |
| "You don't have access to this" on sign-in | | Conditional Access blocked that method: use `pac auth create`, nothing else ([access and security](access-and-security.md)) |
| A `dv:` command or `test:dataverse` says pac is not signed in | `pac org who` | `pac auth create --environment <url>`, or `pac auth select` the right profile |
| `dv:q` says entity not found | | `entity name=` is the logical name (lowercase), not the display name |
| Many "file is being used by another process" errors | | `$env:PAC_CONCURRENCY=2; pnpm test:dataverse` |
| Browser tests cannot find Edge | | `PW_CHANNEL=chrome`, or `PW_EXECUTABLE_PATH` ([commands](../8-reference/commands.md#environment-variables)) |

## React, the preview and tests

| Symptom | Shows the cause | Fix |
|---|---|---|
| Console: `process is not defined` | F12 console | a library reads `process.env`: add the variable to `define` in `tools/features.mjs` |
| Console: `Invalid hook call` | `pnpm why react` | two copies of React, or a hook called outside a component |
| Lint: `React Hook useEffect has a missing dependency` | `pnpm lint` | add it, or move the value into the effect; do not silence the rule |
| Lint: `Cannot access variable before it is declared` / `Calling setState synchronously within an effect` | `pnpm lint` | a React Compiler rule: compute the value while rendering, or pass it as an argument |
| A field shows the old value | | keep anything the screen shows in `useState` / `useReducer`, not a variable or a ref |
| The preview loads twice | | `<StrictMode>` runs effects twice in development; production does not |
| A filter returns every row in the preview | | the mock only knows `eq` / `ne` ([mock Dataverse](../8-reference/mock-dataverse.md#what-the-fake-supports)); test that in dev |
| "The mock data has no table ..." | | add the table to the mock data ([format](../8-reference/mock-dataverse.md#data-file-format)), or `pnpm dv:pull <table> --into` |
| "The preview does not mock ..." | | add the request to `tools/preview/mock-metadata.mjs` |
| Test warning: `not wrapped in act(...)` | | `await` the user action; wait with `waitFor` or `findBy...` |
| Test: `Unable to find an accessible element with the role ...` | `screen.debug()` | the control's name differs (a `*` in the label, a hidden field), or find it by `#id` |
| A test passes alone but fails with others | | change data only through `installMockDataverse(edit)`, which copies the fixture |
| The docs show `Parse failure` or `There is a new version of the pre-bundle` | | the docs server was rebuilding its packages: stop it (Ctrl+C), delete `node_modules\.vite`, run `pnpm docs:serve` again |
| The docs chat says Ollama is not running, or has no model | | [Ask the docs](../2-set-up/03-ask-the-docs.md#when-it-does-not-answer) |

## The kit, `kit:push` and the pre-push hook

| Symptom | Shows the cause | Fix |
|---|---|---|
| `kit:push` lists files that are only yours (a tool or query of your app) | `pnpm kit:push --pr --dry-run` | add them to `exclude` in `kit.json` before pushing |
| "the gh CLI did not do it" | `gh auth status` | the branch is pushed: open the pull request on GitHub, or install `gh` and run `gh auth login` |
| `push-conflict` | | the starter changed the file too: `pnpm kit:update`, merge the `.kit-new` file, push again |
| Every `git push` creates a `kit-push/...` branch | `autopush` in `kit.json` | remove `autopush` to only list changes; delete the branch with `git push <starter url> --delete kit-push/<project folder>` |
| The pre-push hook did nothing | | needs `starterRepo` (or `starter`) in `kit.json`; run `pnpm exec simple-git-hooks` after editing `package.json` |

[Send kit changes back](../7-maintain/send-kit-changes-back.md).

Not here? Write down the message, the environment and what you clicked, then ask. When you find
the answer, add a row.
