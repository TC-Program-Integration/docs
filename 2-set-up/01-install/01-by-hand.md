# Install by hand

> **For:** anyone whose `setup.cmd` stopped on a step, or who wants to install or repair one tool
> alone. **Result:** the same tools `setup.cmd` installs, each checked. No administrator rights.

`setup.cmd` does all of this for you ([install](../01-install.md)). Use this page for the step it
could not do, then run `.\setup.cmd` again: it continues where it stopped.

## Before you install anything: check what is already there

A work laptop often has some of these tools already (an earlier project, another team's script,
IT's image). Installing over them is the usual cause of trouble, so look first. In PowerShell:

```powershell
foreach ($tool in "scoop","git","nvm","node","pnpm","uv","python","pac","code","claude") {
  $found = Get-Command $tool -ErrorAction SilentlyContinue
  $where = if ($found) { $found[0].Source } else { "-- not found" }
  "{0,-8} {1}" -f $tool, $where
}
```

| You see | It means | Do |
|---|---|---|
| a path | installed and on the `PATH` | skip its install step; check the version later in section 6 |
| `-- not found` | not installed, **or** installed but not on the `PATH` | look for its folder (table below) before installing |

| Tool | Folder to look for | If the folder exists but the command is not found |
|---|---|---|
| Scoop | `$HOME\scoop` | add `$HOME\scoop\shims` to the user `PATH` (see section 1); do not run the installer again |
| Git | `$HOME\scoop\apps\git`, `C:\Program Files\Git` | add its `cmd` folder to the `PATH`, or `scoop install git` |
| nvm | `$HOME\scoop\apps\nvm`, `$env:NVM_HOME` | not needed; if it blocks tools, `.\setup.cmd -Clean` removes it |
| Node | `$HOME\scoop\apps\nodejs-lts` | must be v24; a second Node (nvm, an old installer) can shadow it: `Get-Command node -All` |
| pnpm | `$HOME\scoop\apps\nodejs-lts`, corepack | `corepack enable pnpm`; `Get-Command pnpm -All` shows every copy, and the first one must run |
| uv, Python | `$HOME\scoop\apps\uv`, `$HOME\.local\bin` | `scoop reset uv`, or add `$HOME\.local\bin` to the `PATH` |
| pac | VS Code extension folder `$HOME\.vscode\extensions\microsoft-isvexptools.powerplatform-vscode-*`, `$HOME\.dotnet\tools` | use VS Code's terminal, or add `$HOME\.dotnet\tools` to the `PATH` |
| VS Code | `$HOME\scoop\apps\vscode` | `scoop reset vscode`; the command is `code` |
| Claude Code CLI | not installed by this project | install it from its official instructions, then `Get-Command claude` |

### Rules that avoid most install problems

1. **Check, then install.** Never run an installer for a tool that `Get-Command` already finds.
2. **"Not recognized" is a `PATH` message**, not "not installed". Check the folder, then call the program by its full path. If that works, fix the `PATH`; do not reinstall.
3. **An installer that stops on an existing folder is protecting it.** Do not delete the folder to make it go away. Look inside, then rename it (`Rename-Item <folder> <folder>.old`) if you really need a clean install.
4. **A new terminal is needed after every install** (or update `$env:Path` as in section 1). Many "it did not install" reports are an old window.
5. **Two copies of one tool shadow each other.** `Get-Command <tool> -All` lists every copy in `PATH` order; the first one runs. Remove the one you did not mean to have (`scoop uninstall <tool>`).
6. **`Set-ExecutionPolicy` only decides which PowerShell scripts may run.** It does not fix a missing `PATH` entry, a non-empty folder or a missing program. Use `-Scope CurrentUser` (or `-Scope Process` for one window); never administrator.
7. **No administrator.** If an instruction says "Run as administrator", stop: there is a user-level way (Scoop, the VS Code extension, a user `PATH` entry), or ask IT.
8. **Corporate proxy or blocked download** (timeouts, certificate or 403 errors): the tool did not install, so the folder is partial. Report the error text to IT, and remove the partial folder only after looking at it.
9. **Change one thing at a time, then re-run the check above.** Pasting a fix for a different problem (a policy change, a reinstall) hides the real cause.
10. **Keep the output.** When you ask for help, paste the failing command and its full message plus the output of the check above.

## 1. Install Scoop

[Scoop](https://scoop.sh) installs command-line tools into your own folder (`C:\Users\<you>\scoop`)
and puts them on your `PATH`, so it works without admin rights.

**Scoop already installed?** If the check above found `scoop`, skip this section. If it did not, but the folder `$HOME\scoop` exists, the installer stops with a message about a non-empty folder. That does not mean Scoop is missing: it means Scoop is installed and only its `shims` folder is missing from the `PATH`:

```powershell
$shims = "$HOME\scoop\shims"
$user = [Environment]::GetEnvironmentVariable("Path", "User")
if (($user -split ";") -notcontains $shims) { [Environment]::SetEnvironmentVariable("Path", "$user;$shims", "User") }
$env:Path += ";$shims"
scoop --version
```

If `scoop` is still not found, `& "$HOME\scoop\shims\scoop.ps1" --version` tells you whether it runs at all. Only when the folder is empty or broken: rename it (`Rename-Item "$HOME\scoop" scoop.old`), then install. Otherwise, in a normal PowerShell window:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

Then Git (Scoop itself needs it to add buckets and to update), and the "extras" bucket for apps
such as VS Code:

```powershell
scoop install git
scoop bucket add extras
```

Open a **new** terminal after each group of installs below so it sees the new `PATH`.

## 2. The tools

| Tool | Needed for | Install |
|---|---|---|
| Node 24 and pnpm | build, preview, tests | [section 3](#3-node-24-and-pnpm) |
| uv and Python | flows, docs site | [section 4](#4-python-with-uv) |
| Power Platform CLI (pac) | reading Dataverse, sign-in | [section 5](#5-power-platform-cli-pac) |
| VS Code | editing | `scoop install vscode` (extras bucket); accept the extensions the folder recommends |
| Microsoft Edge | browser tests | already on Windows |

## 3. Node 24 and pnpm

The project uses Node 24 (written in `.nvmrc` and in `package.json` `engines`). Install exactly that
version with scoop, and hold it so `scoop update` does not move it to the next one:

```powershell
scoop install nodejs-lts@24.21.0
scoop hold nodejs-lts
node --version           # v24.x
```

pnpm comes with Node 24 through corepack, which runs exactly the pnpm version the project pins in
`package.json` (`packageManager`):

```powershell
corepack enable pnpm
pnpm --version
```

If `corepack enable` is refused, `scoop install pnpm` installs pnpm on its own. Do not install
pnpm or corepack with `npm install -g`.

**Already using nvm?** `nvm install 24` and `nvm use 24` work too, but nvm's security check (on some
laptops) blocks any tool it did not install itself, and nvm may pick another Node per folder.
When that happens, `.\setup.cmd -Clean` clears nvm and installs Node 24 the scoop way.

## 4. Python with uv

[uv](https://docs.astral.sh/uv/) installs Python and the Python tools, and creates virtual
environments:

```powershell
scoop install uv
uv python install 3.12           # Python itself; or: scoop install python
```

The project's Python tools are listed in `pyproject.toml` and locked in `uv.lock`: Black, Ruff and
Pyright for `flows/`. One command creates the `.venv` folder and installs them:

```powershell
uv sync
```

There is nothing to activate: the `pnpm` scripts for flows and docs run through `uv run`, which uses
that environment. Run `uv sync` again after pulling a change to `pyproject.toml` or `uv.lock`.

## 5. Power Platform CLI (pac)

pac is how the tools read Dataverse. Without admin rights, install the VS Code extension
**Power Platform Tools** (it includes pac and installs for your user). Its terminal has `pac` on
the `PATH`; for an ordinary terminal, either use VS Code's integrated terminal, or install pac as a
.NET tool (what `setup.cmd` does):

```powershell
scoop install dotnet-sdk
dotnet tool install --global Microsoft.PowerApps.CLI.Tool
```

## 6. Check

Open a **new** terminal and run each of these; every one prints a version (`pac` prints its help):

```powershell
scoop --version; git --version; nvm version; node --version; pnpm --version; uv --version; pac
```

If `python` opens the Microsoft Store, use `py` instead, or turn off the python "App execution
aliases" in Windows Settings. To update everything later: `scoop update *`.

Then run `.\setup.cmd` again: it installs the project's packages and runs the tests. Next,
[sign in with pac](../01-install.md#3-sign-in-to-dataverse-with-pac).
