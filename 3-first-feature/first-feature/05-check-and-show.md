---
sidebarTitle: "Check it and show it"
---

# Your first feature: 5. Check it and show it

> Part 5 of 6 of [Your first feature](../first-feature.md). Steps 8 and 9.

## Step 8. Run the check

```powershell
pnpm first-feature:check
```

(`pnpm first-feature:check some-other-name` checks a feature with a different name; the flow folder is
always `flows/callback-alert/`.) It does two things, in order.

**1. Your files.** It looks at the files and prints `ok` or `TODO` with what to do:

```text
  ok   src/features/callback-request/ has all its files
  TODO page/fields.ts is your own
         -> describe the exercise's fields
```

It cannot judge your code. It notices what is missing or still the scaffold:

| It looks for | Step |
|---|---|
| all the feature's files, and the feature in `features` of `webresources.config.mjs` | 2 |
| `config.ts` pointing at a table other than the scaffold's; that table in both mock data files | 3 |
| `rules.ts`, `fields.ts` and the page changed from the scaffold by more than a few lines of code | 4 |
| at least 5 more unit tests than the scaffold has; a browser test of your own | 5 |
| `CharCounter` in `src/ui/char-counter/`, exported, used by the page, with its own test | 6 |
| `flows/callback-alert/flow.py` with the trigger on your table and its filter, the two reads, the email and a try/catch; at least 5 tests in `flows/tests/test_callback_alert.py` | 7 |

Fix the TODO lines first.

**2. Quality checks**, only when the files are all `ok`: formatting, the linter, the type check, the unit
tests with their coverage floor, the browser test of your feature, the flow checks and tests, the flow
build, and the web resources build (which must produce your page in `dist/`). A failed one shows the
end of its output; fix it and run the check again.

It ends with one line:

```text
READY. 31 checks passed.
```

or `NOT READY: 2 of 31 checks left.` READY means "nothing is missing or broken". It does not mean
"good": that is the review. Copy the whole output for your mentor.

## Step 9. Review, then dev

1. **Review.** Your mentor reads the code with you ([what they look at](06-for-the-mentor.md)). Expect
   comments; fixing them is part of the exercise. Run the check again after each fix.
2. **Deploy to dev.** Follow [Deploy your first solution](../../6-ship/deploy/01-first-time.md): it
   uses this exercise as its example. With your mentor you create the *Callback request* table as
   [the brief](01-brief.md#where-the-data-goes) says, push the web resources with the links file
   that `pnpm build` wrote, create the button from `dist/deploy.json`, and push the flow.
3. **Compare the names.** Now that the table exists, `pnpm dv:pull pidev_callbackrequest --columns
   pidev_topic,pidev_priority,pidev_preferreddate,pidev_phone,pidev_notes` shows the real names and
   options, and `pnpm test:dataverse` checks your `schema.ts`. If a real name differs from your
   guess (a lookup's navigation property, a column), change `config.ts` and
   `flows/callback-alert/settings.py`, not the code.
4. **Try it for real** on a contact: one normal request, one Urgent request with a phone, one that a
   rule must refuse. Open the saved records in the app and compare them with your payload.
   `pnpm dv:verify` confirms the web resources, the button and the flow are there and the flow is
   on; the **Runs** tab of Flows as Code shows that the Urgent request started one run, the others
   none (if the tab stays empty, the flow's run history in Power Automate shows the same). Check
   that the owner got the email.

You are done when the check says READY, the review is finished, and the three requests above work
in dev, flow included. Tell your mentor; then your first real task is next.

## When it does not work

| Symptom | Look at |
|---|---|
| The check says a file is "not your own" but you changed it | it counts new lines of code (not comments); a real change of rules, fields or page is more than a few lines |
| The linter says a doc comment is missing | every exported function, interface and type needs one |
| The tests pass but the browser test fails | the preview shows a field the test does not fill, or the test uses an id that changed |
| The coverage check fails | a branch of your rules or page is not tested: open `test/.coverage/index.html` after `pnpm test:coverage` |
| `pnpm flows:build` lists problems | an action waits on one outside its scope, or an expression names an action that does not exist: [rules and testing](../../5-flows/build-a-flow/03-rules-and-testing.md) |
| The build warns about `buttons.callback-request` | the button's `label` or `table` is missing, or the feature is not in `features` |
| Saving in dev fails with a "navigation property" error | the lookup's name or its letter case: [mistakes people make](../../4-build/build-a-feature.md#mistakes-people-make) |
| The button or the dialog does not work in dev | [Troubleshooting: the button or the dialog](../../9-help/troubleshooting.md#the-button-or-the-dialog) |
| The flow does not start, or its run fails | [When a flow fails](../../6-ship/when-a-flow-fails.md) |
| Anything else with setup or tools | [Troubleshooting](../../9-help/troubleshooting.md) |

---

**Previous:** [4. The alert flow](04-alert-flow.md)
