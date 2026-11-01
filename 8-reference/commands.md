# Commands

> **Reference.** Every command of the project, its options and what it needs. Run them from the
> project folder. Which command to run when is in the guides; this page has the exact forms.

`[x]` is optional, `<x>` is a value you give. "Needs" lists what must be installed besides Node and
pnpm.

## Set up

| Command | Does | Needs |
|---|---|---|
| `setup.cmd` | One-time laptop setup. Installs only what is missing (Scoop, PowerShell 7, Git, gh, Node, pnpm, uv, Python, pac, XrmToolBox), then the packages and the Python tools, Ollama and the docs' AI chat model, team XrmToolBox plugins, then runs the offline tests. Safe to run again: what works is left alone. Starts `setup.ps1` with the execution policy bypassed for that window only, explains each failure, and writes `setup.log`. | Windows |
| `pnpm xrm` | launches XrmToolBox after checking for and updating the team's plugins | Windows |
| `setup.cmd -SkipBrowser` | the same, without downloading Chromium; Edge runs the browser tests | Windows |
| `setup.cmd -SkipDocsChat` | the same, without Ollama and the AI chat model ([Ask the docs](../2-set-up/03-ask-the-docs.md)) | Windows |
| `setup.cmd -Clean` | when Node or pnpm are mixed up (not Node 24, pnpm does not run, nvm blocking tools), clears nvm and scoop's Node and pnpm without asking, then installs Node 24 and pnpm with scoop. Without `-Clean`, setup lists the problems and asks first. | Windows |
| `setup.cmd -Doctor` | checks the laptop and the project without installing or changing anything; writes `doctor-report.txt` to send to whoever helps you | Windows |
| `tools\xrmtoolbox\install.cmd [-Force] [-Zip <zip>] [-PluginsFolder <dir>]` | installs or updates the team's [XrmToolBox tools](tools.md#install-or-update-them). Close XrmToolBox first. | Windows |
| `pnpm install` | installs the packages and the git hooks (`prepare`) | |
| `uv sync` | creates `.venv` with the Python tools from `pyproject.toml` and `uv.lock`: Black, Ruff and Pyright for `flows/` | uv |

## Write code

| Command | Does | Needs |
|---|---|---|
| `pnpm new:feature <name> ["Title"]` | creates `src/features/<name>/` from `tools/templates/feature/`, its tests `test/unit/<name>.test.tsx` and `test/e2e/<name>.spec.ts`, and adds it to `features` in `webresources.config.mjs`. `<name>` is kebab-case (`site-visit`). It does not add a button. | |
| `pnpm dev` | starts together, in one terminal: the preview (`http://localhost:5600`) and the docs website (`http://localhost:5173`). Each output line is marked with its name. Ctrl+C stops both. Run the unit tests with `pnpm test` (or `pnpm test:watch` in its own terminal). | uv |
| `pnpm dv:preview [feature] [--port <n>] [--data <file>]` | the preview alone: every feature on mock data, rebuilt and reloaded on save; `/?feature=<name>` switches. Default port 5600, default data `tools/preview/mock-data.json`. Writes are logged, never sent. | |
| `pnpm watch` | rebuilds `dist/` on save, without format, lint or type checks | |
| `pnpm first-feature:check [name]` | checks the [first-feature exercise](../3-first-feature/first-feature.md): the files are your own work, then format, lint, types, tests with coverage, the feature's browser test, the flow checks, `flows:build` and the build pass. Prints `READY` or what is left. Default name `callback-request`. | Edge, uv |

## Schema

Tables, columns, choices, relationships and keys written in TypeScript: [Define the schema](../4-build/define-the-schema.md).

| Command | Does | Needs |
|---|---|---|
| `pnpm schema:init [--key <key>]` | creates `schema/` from `tools/templates/schema/`: `project.ts` (prefix from `flows/solution.json`, a new project id, a solution of its own), one example table and `index.ts`. Refuses when `schema/` exists. `<key>` defaults to the project folder's name. | |
| `pnpm schema:build [--no-type-check]` | type-checks, checks `schema/` and writes `dist/schema.json` for the Schema as Code tool. Prints each problem as `error path: message` or `warning path: message`; errors stop it. Does nothing in a project without `schema/`. Unlike `pnpm build` it doesn't clear `dist/`. | |

`pnpm build` also writes `dist/schema.json`, after the web resources.

## Check

| Command | Does | Needs |
|---|---|---|
| `pnpm check` | every check CI runs except the browser tests: `check:web`, `check:flows`, `docs:build`. The pre-push hook runs it. | uv |
| `pnpm check:web` | `format:check`, `lint`, `type-check`, `test:coverage` | |
| `pnpm check:flows` | `format:flows:check`, `lint:flows`, `type-check:flows`, `test:flows` | uv |
| `pnpm format` / `pnpm format:check` | Prettier (`.prettierrc.json`, width 120) rewrites / only checks the web code, tools and config files | |
| `pnpm lint` / `pnpm lint --fix` | ESLint (`eslint.config.mjs`) on `src/`, `test/` and `docs/theme/` `.ts`/`.tsx`: strict type-aware rules, React hooks rules, JSX accessibility, a doc comment on every function, class, method, interface and type in `src/`. `pnpm lint --fix` fixes what it can. | |
| `pnpm type-check` | TypeScript on `src/` (browser types), then on `test/` and the configs (Node types), then on the docs site's `docs/theme/` | |
| `pnpm test` | unit and component tests (`test/unit/**/*.test.{ts,tsx}`) | |
| `pnpm test:watch` | the same, rerun on save | |
| `pnpm test:coverage` | unit tests with V8 coverage; fails below the floor in `vitest.config.mts`; report in `test/.coverage/` | |
| `pnpm test:e2e` | browser tests (`test/e2e/`) on a preview with `test/fixtures/mock-data.json`, port 5655 | Edge |
| `pnpm check:all` | `check` and the browser tests (`test:e2e`): everything CI runs | Edge, uv |
| `pnpm docs:serve` / `pnpm docs:build` | these docs as a website at `http://localhost:5173` / built into `site/`, failing on a page that does not parse or a broken link ([Write the docs](../7-maintain/write-the-docs.md)) | |
| `pnpm docs:webresources` | the docs built as Dataverse web resources into `pages-site/dist-docs/`, with `webresources.docs.links.xml`, without the AI chat ([Publish the docs](../7-maintain/publish-the-docs.md)) | |
| `pnpm docs:pages` / `pnpm docs:pages:upload` | the same pages prerendered into `pages-site/` (what `docs:webresources` builds on); `docs:pages:upload` uploads them to a Power Pages code site with `pac pages upload-code-site`, if your org has one | pac |
| `pnpm docs:model` | makes the AI chat's model, `holocron-docs`, from `qwen3.5:2b` ([Ask the docs](../2-set-up/03-ask-the-docs.md)) | Ollama |

Git hooks (installed by `pnpm install`): **pre-commit** runs ESLint `--fix` and Prettier on the
staged web files, Black and Ruff `--fix` on staged Python; **pre-push** runs `pnpm check`. The
checks are explained in [The checks](../4-build/test-and-check/02-the-checks.md).

## Build

| Command | Does | Needs |
|---|---|---|
| `pnpm build` | runs `format`, `lint`, `type-check` (an error stops it), then builds every feature of `webresources.config.mjs` into `dist/<prefix>_<feature>_` (`command.js`, `page.js`, `page.html`, `page.css`). Prints each web resource name and each command library's `Library` and `Function`. Ends by writing [`webresources.links.xml` and `dist/deploy.json`](configuration.md#files-the-build-writes) and warning about any wrong `buttons` entry. | |
| `pnpm build --minify` | the same with minified scripts: `page.js` drops from about 730 KB to about 240 KB for the example feature. Stack traces are harder to read. | |

## Dataverse (pac, read-only)

These run FetchXML through `pac org fetch` with the pac profile you are signed in with. They only
read and never handle a password. Rules: [access and security](../9-help/access-and-security.md).
`--env <url>` uses another environment than pac's active one.

| Command | Does | Needs |
|---|---|---|
| `pnpm dv:q <file.xml \| "<fetch>…</fetch>"> [--env <url>]` | runs a query and prints the rows; also saves them to `tools/.out/last-query.txt`. A `top` attribute is turned into `count`. Ready-made queries: `tools/queries/*.xml`. | pac |
| `pnpm dv:pull <table> --columns a,b [--records <n>] [--into <mock.json>] [--name-field <c>] [--id-field <c>] [--env <url>]` | reads the table's entity set and the options of its choice columns, prints a `config.ts` block, saves `tools/.out/pull-<table>.json`. `--records` also takes n rows (check them for personal data). `--into` adds the table and choices to that mock data file. `--id-field` defaults to `<table>id` (activities such as `task` use `activityid`). | pac |
| `pnpm dv:access [email] [--roles "A,B"] [--env <url>]` | checks a user (default: you): exists, enabled, has the roles directly or through a team (default `System Customizer`), and the project's publisher and solution exist | pac |
| `pnpm dv:verify [--env <url>]` | checks that every web resource and button of `dist/deploy.json` and every flow of `flows/out/flows.json` exists, and that the flows are on. Exits 1 when something is missing. Cannot see whether a web resource was published. See [Deploy: verify](../6-ship/deploy.md#5-verify). | pac |
| `pnpm test:dataverse` | checks every feature's `schema.ts` against the environment | pac |

## Flows

| Command | Does | Needs |
|---|---|---|
| `pnpm flows:build` | runs `format:flows`, `lint:flows`, `type-check:flows`, then `flows/build-all.py`: checks every flow ([checks](flowkit.md#checks)) and writes `flows/out/<key>/flow.json` (once per flow), `flows/out/flows.json` and one `flows/out/<Solution>_<version>.zip` per solution in [`flows/solution.json`](configuration.md#flowssolutionjson), and adds the built flows to [`flows/flows.links.json`](configuration.md#flowslinksjson) (only with the default output folder). A problem stops it. | uv |
| `pnpm test:flows` | flowkit and flow tests (`flows/tests/`, in folders like `flows/lib/flowkit/`, and a project's own tests next to its flows) | uv |
| `pnpm format:flows` / `pnpm format:flows:check` | Black (width 120) formats / only checks `flows/` | uv |
| `pnpm lint:flows` / `pnpm lint:flows --fix` | Ruff on `flows/`: imports, likely bugs, a docstring on every module, class, function and method | uv |
| `pnpm type-check:flows` | Pyright strict on `flows/` | uv |

`uv run python flows/build-all.py` takes these options (defaults from
[`flows/solution.json`](configuration.md#flowssolutionjson)):

| Option | Default |
|---|---|
| `<folder or flow.py> ...` | every `flows/<name>/flow.py` except `lib`, `examples`, `out`. The solutions then hold only these flows; a solution left empty is skipped with a message |
| `--only-solution <uniquename>` | every solution. Builds only that solution's zip, and the `flow.json` files and `flows.json` entries of its flows |
| `--publisher <uniquename>` / `--publisher-name <text>` | `publisher` / `publisherDisplayName` of the home solution |
| `--prefix <prefix>` / `--option-prefix <n>` | `prefix` / `optionPrefix` of the home solution |
| `--solution <uniquename>` / `--solution-name <text>` | `uniqueName` / `displayName` of the home solution |
| `--version <a.b.c.d>` | `version` of the home solution |
| `--out <folder>` | `flows/out` |

The overrides change the home solution only; extra solutions keep their values from
`flows/solution.json`. `--only-solution` is the solution filter; `--solution` renames the home one.
Example: `uv run python flows/build-all.py --only-solution MyProjectReminders`.

## Ship

| Command | Does | Needs |
|---|---|---|
| `pnpm version patch\|minor\|major` | sets the version in `package.json` and makes a `v<version>` git tag. First runs the upgrade check below; a failing check stops it. | git |
| `release:upgrade-check` (starter only, not sent to projects) | simulates the upgrade a project makes: copies the previous release (highest `v*` tag), records its kit as a project does, updates it from this starter's working tree, then installs and runs `pnpm check` there. Prints a line per step; takes several minutes. `--keep` leaves its temporary folder. | git, uv, Edge |
| `pnpm release` | copies the project to a clean folder, then there: `pnpm install --frozen-lockfile`, `pnpm check`, the browser tests, the build. If all pass, writes `release/<name>-<version>.zip` and `release/<name>-<version>.md` (the notes). Nothing is zipped when a check fails. | uv, Edge |
| `pnpm release --skip-e2e` | the same without the browser tests | uv |
| `pnpm release --no-check` | zip only; the notes say it was not checked | |

Pushing a `v*` tag (`git push --follow-tags`) runs `.github/workflows/release.yml`: it checks the
tag matches `package.json`, runs `pnpm release` and attaches the zip and notes to a GitHub
Release. See [Make a release](../6-ship/release.md).

## Kit

| Command | Does |
|---|---|
| `pnpm kit:status` | which kit files you changed, and merges left to do |
| `pnpm kit:push <starter folder> [--dry-run]` | in a project: copies the kit files you changed or added into a starter checkout, without overwriting a file the starter changed too (yours goes to `<file>.push-new`). Commits nothing. See [Send kit changes back](../7-maintain/send-kit-changes-back.md). |
| `pnpm kit:push [<starter git url>] --pr [--dry-run]` | like `kit:push`, but puts the changes on the branch `kit-push/<project folder>` of the starter's git repo and opens a pull request (needs `starterRepo` in `kit.json`, or env `KIT_STARTER_REPO`, when no URL is given). Run it with `--dry-run` first: only files of every project belong in the kit (`exclude` the rest). [Send kit changes back](../7-maintain/send-kit-changes-back.md). |
| `node tools/kit.mjs hook` | the `pre-push` hook, not run by hand: lists kit changes the starter lacks, and with `autopush` sends them (`"pr"`: a pull request, `true`: a local checkout). Running it by hand does the same. |
| `pnpm kit:update <starter folder> [--dry-run] [--base <older starter folder>]` | takes the kit from a newer starter (a checkout or an unzipped release) without losing your changes. `--dry-run` shows what would happen. `--base`, first update only: the starter the project was made from. See [Update the kit](../7-maintain/update-the-kit.md). |

## Environment variables

| Variable | Read by | Effect |
|---|---|---|
| `DATAVERSE_URL` | `test:dataverse` | environment to check instead of pac's active one |
| `PAC_CONCURRENCY` | every pac tool | pac processes at once (default 6); set `1` if pac reports locked files |
| `PW_CHANNEL` | `test:e2e` | browser channel instead of Edge (`msedge`), e.g. `chrome` |
| `PW_EXECUTABLE_PATH` | `test:e2e` | path to any Chromium-based browser |
| `CI` | `test:e2e` | when set, always starts a fresh preview server |
| `KIT_STARTER_REPO`, `KIT_STARTER`, `KIT_AUTOPUSH` | `kit:push`, the `pre-push` hook | the starter's git URL, a local starter checkout, and `pr` / `1` to send on every push; override `kit.json` ([Send kit changes back](../7-maintain/send-kit-changes-back.md)) |

In PowerShell: `$env:DATAVERSE_URL="https://<org>.crm3.dynamics.com"; pnpm test:dataverse`.

## pac commands you will use

| Command | Does |
|---|---|
| `pac auth create --environment <url>` | signs in to an environment |
| `pac auth list` / `pac auth select --index <n>` / `pac auth delete --index <n>` | lists, picks or removes sign-in profiles |
| `pac org who` | shows the active user and environment |
