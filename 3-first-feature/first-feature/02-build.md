---
sidebarTitle: "Build it"
---

# Your first feature: 2. Build it

> Part 2 of 6 of [Your first feature](../first-feature.md). Steps 1 to 4.

## Step 1. Setup is green

```powershell
setup.cmd -Doctor
```

It checks the tools without changing anything and writes `doctor-report.txt`. No **FAIL** line
means you can start; otherwise run `setup.cmd` again, or send the report to your mentor.

## Step 2. Create the feature

```powershell
pnpm new:feature callback-request "Callback request"
pnpm test
pnpm dev
```

The scaffold is a working "create a task" dialog, already added to `features` in
`webresources.config.mjs`. Open `http://localhost:5600/?feature=callback-request` and pick a
**contact** as the source record. Open `src/features/callback-request/` and read each file once
before changing anything: [feature files](../../8-reference/feature-files.md) says what every file
is for.

**Check yourself:** which file would you open to change a column name? Which one to change a
message? Which one to change the layout? (`config.ts`, `rules.ts`, `CallbackRequestPage.tsx`.)

## Step 3. Names, mock data and the button

Dataverse names live in `config.ts` and nowhere else.

1. **`config.ts`**: set the table and the five columns from [the brief](01-brief.md#where-the-data-goes).
   `sourceBinding` returns the navigation property that links the new record to the contact. Keep the
   value of the *Urgent* option here too (a named constant), not as a bare `2` inside the rules.
   The table does not exist in dev yet, so you write these names from the brief; `pnpm dv:pull`
   confirms them in step 9.
2. **`schema.ts`**: it is built from `config.ts`. It lists the table and columns, and the choice column
   the page reads options from. Check that it matches what you wrote.
3. **Mock data**: the preview and the tests use fake data, not Dataverse. Add your table to **both**
   `tools/preview/mock-data.json` and `test/fixtures/mock-data.json`: its name column, its id column,
   its entity set and no records, and the options of the priority column. The format is in
   [Mock Dataverse](../../8-reference/mock-dataverse.md#data-file-format).
4. **The button**: in `webresources.config.mjs`, add
   `"callback-request": { label: "Callback request", table: "contact" }` to `buttons`
   ([how](../../4-build/build-a-feature.md#6-describe-the-command-button)). The build turns it into
   the button you create in dev in step 9.
5. Reload the preview. If the priority list says "Loading..." forever, the mock choice key
   (`"<table>.<column>"`) does not match `config.ts`.

**Check yourself:** `pnpm test` still runs. Do the scaffold's own tests now fail? That is expected:
they still test a *task*. You rewrite them in [step 5](03-tests-and-component.md).

## Step 4. The form, the rules and the page

Work in this order; the preview shows each step working before the next.

1. **`page/fields.ts`**: describe the five fields with the builders (`text`, `choice`, `date`). Ids:
   `topic`, `priority`, `preferredDate`, `phone`, `notes`. The notes field has `rows` but no
   `maxLength` (see rule 5). The phone is `required`; the page decides when it counts.
2. **`logic/rules.ts`**: the values interface, then `validate(values, today)` returning the first
   problem as `{ field, message }`, or `null`. Pass today's date in as a parameter, so a test can pin
   it. Put the phone check in its own small exported function; it is easy to test alone.
3. **`page/CallbackRequestPage.tsx`**:
   - `toValues` and `toPayload` map the form to the rules' values and to the columns;
   - one `isVisible(name)` function says whether a field is shown. Give it to **both**
     `FormContext` (then `<Field name="phone" />` draws nothing while it is hidden) and
     `checkRequired`, so a hidden field is never required
     ([Build a feature, step 4](../../4-build/build-a-feature.md#4-build-the-form));
   - lay the fields out with `Page`, `Row` and `Field`;
   - update the test-data helper and the page title.
4. Try every rule in the preview. Watch the log panel: the payload must have your column names and
   the `@odata.bind` link.

**Check yourself:**

- Choose *Urgent*, leave the phone empty, save: which message, on which field, and where is the cursor?
- Choose *Urgent*, then *Low*: is the phone field gone? Save with the phone empty: does it save?
- Type a past date and a 101-character topic: do both messages appear, one at a time?

Stuck? [Mistakes people make](../../4-build/build-a-feature.md#mistakes-people-make) lists the
usual causes.

**Next:** [Test it and build the component](03-tests-and-component.md).

---

**Previous:** [1. The brief](01-brief.md)
