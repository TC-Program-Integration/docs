# Send kit changes back to the starter

> **For:** a project made from the starter where you changed a kit file (flowkit, `src/core`, a UI
> field, a tool, a shared docs page) that every project should have. **Needs:** `git`, and the
> [GitHub CLI](https://cli.github.com) (`gh`, signed in with `gh auth login`) to open the pull
> request for you. **Result:** a pull request on the starter; once merged, every project gets the
> change with [`pnpm kit:update`](update-the-kit.md).

Kit files are made in the starter, so a fix made in a project has to get there. `pnpm kit:push` does
the copying; you review and merge. Nothing reaches the starter's `main` without a pull request.

## 1. Keep project-only files out first

`kit.json` lists the kit with `include`. A file of your own inside an included folder (a tool for your
app, a query, mock data in `tools/`) looks like a kit change and would be sent to the starter. List
it under `exclude` before the first push:

```json
"exclude": ["tools/my-app-report.mjs", "tools/queries/my-app/", "tools/preview/mock-data.json"]
```

A whole folder ends with `/`. Check with the dry run below: every line it prints must be something
every project should have.

## 2. Send the changes as a pull request

Set where the starter lives, once, in your `kit.json` (or the environment variable `KIT_STARTER_REPO`):

```json
"starterRepo": "https://github.com/<owner>/Power-Platform-Starter.git"
```

```powershell
pnpm kit:push --pr --dry-run   # what would go on the pull request; writes and pushes nothing
pnpm kit:push --pr             # branch kit-push/<your project folder> + pull request
```

It clones the starter into a temp folder, puts your changed kit files on the branch
`kit-push/<project folder>`, pushes that branch and opens the pull request with `gh`. Without `gh` it
tells you to open the pull request by hand. Running it again force-updates that one branch, so each
project has one open pull request, always current. Your own branches, and any local copy of the
starter, are not touched. For each file:

| Your file | The starter's file | Action | What happens |
|---|---|---|---|
| changed or new | as installed (or missing) | `push` / `push-new` | goes on the pull request |
| changed or new | also changed since you installed | `push-conflict` | **left out**: `pnpm kit:update` first, merge, push again |
| deleted | | `push-deleted` | reported only; nothing is deleted in the starter |
| the same, or unchanged | | (none) | nothing |

Then, in the starter: review the pull request, run `pnpm check`, merge and
[release](../6-ship/release.md). In each project run `pnpm kit:update` (a project that sent a change
gets `kit.json` re-synced by it).

## 3. Do it on every push (optional)

The `pre-push` git hook can run the same thing. Put this in the project's `package.json`:

```json
"simple-git-hooks": { "pre-commit": "pnpm exec lint-staged", "pre-push": "node tools/kit.mjs hook && pnpm check" }
```

and in `kit.json` `"starterRepo": "..."` and `"autopush": "pr"` (or the environment variables
`KIT_STARTER_REPO` and `KIT_AUTOPUSH=pr`). Install it once with `pnpm exec simple-git-hooks`.

<Warning>
  With `"autopush": "pr"` every `git push` that carries kit changes creates or updates a branch on the
  starter's repository, and running `node tools/kit.mjs hook` by hand does the same. Do step 1 first.
  To stop it, remove `autopush` from `kit.json`.
</Warning>

Without `autopush` the hook only **lists** what the starter lacks. It never fails your push and prints
nothing when there is no `starterRepo` (a teammate's machine, CI).

## Without GitHub: copy into a local checkout

With a clone of the starter next to your project, copy into it instead of opening a pull request:

```powershell
pnpm kit:push ..\pp-starter-react --dry-run   # what would be copied
pnpm kit:push ..\pp-starter-react             # copy the files into that folder
```

A file the starter changed too is not overwritten: yours is written next to it as `<file>.push-new`,
to merge by hand and delete. Nothing is committed: review `git diff` in the starter, run
`pnpm check`, commit and release. The hook does this on every push with `"starter": "../pp-starter-react"`
(or `KIT_STARTER`) and `"autopush": true` (or `KIT_AUTOPUSH=1`).

## If it does not work

| Symptom | Fix |
|---|---|
| "open the pull request ... (the gh CLI did not do it)" | the branch is pushed; open the pull request on the starter's GitHub page, or install `gh` and run `gh auth login` |
| The push lists files that are only yours | add them to `exclude` in `kit.json` (step 1) |
| `push-conflict` on a file | the starter changed it too: `pnpm kit:update`, merge the `.kit-new` file, push again |
| A `kit-push/...` branch you do not want | delete it on GitHub, or `git push <starter url> --delete kit-push/<project folder>` |
| `kit:push` says the project has no `files` in `kit.json` | this is the starter itself, or a project that never ran `kit:update` once |

The settings are in [configuration](../8-reference/configuration.md#kitjson), the commands in
[commands](../8-reference/commands.md).
