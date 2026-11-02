---
sidebarTitle: "The page component and mock data"
---

# Build a feature: 3. The page component and mock data

> Part 3 of 5 of the worked example in [Build a feature](../build-a-feature.md).

## 6, continued: the component

```tsx
/** The Site visit page: loads the options, validates the form and saves the visit. */
export function SiteVisitPage() {
  const [values, setValues] = useState(
    () =>
      Object.fromEntries(names.map((name) => [name, emptyValue(fieldSpecs[name])])) as Record<FieldName, FieldValue>,
  );
  const [errors, setErrors] = useState<Partial<Record<FieldName, string>>>({});
  const [options, setOptions] = useState<Partial<Record<FieldName, Option[]>>>({});
  const [source, setSource] = useState<SourceRecord>();

  // The reason is asked only for a cancelled visit
  const askReason = needsReason(parseChoice(values.status as string));
  const isVisible = (name: string) => name !== "cancelReason" || askReason;

  const bar = useSaveBar(async (): Promise<SaveResult | false> => {
    if (!source) return false;
    const required = checkRequired(fieldSpecs, values, isVisible);
    if (required.first) {
      setErrors(required.errors);
      focusField(fieldSpecs[required.first]);
      return false;
    }
    const typed = toValues(values);
    const problem = validate(typed);
    if (problem) {
      setErrors({ [problem.field]: problem.message });
      focusField(fieldSpecs[problem.field]);
      return false;
    }
    // If linking the inspectors fails, createThen deletes the visit again: nothing is half-saved.
    const id = await createThen(table, await toPayload(typed, source), (visitId) =>
      associate({ table, id: visitId }, Config.inspectors.relationship, {
        table: Config.inspectors.table,
        ids: typed.inspectors,
      }),
    );
    return { message: "Saved. Close this dialog, or open the new visit.", open: () => openRecord(table, id) };
  });

  // Read the source record and load the options once.
  const { ready, loadFailed } = bar;
  useEffect(() => {
    let current = true;
    const stale = () => !current;
    void (async () => {
      try {
        const record = readSourceRecord();
        const [status, inspectors] = await Promise.all([fieldSpecs.status.load(), fieldSpecs.inspectors.load()]);
        if (stale()) return;
        setSource(record);
        setOptions({ status, inspectors });
        ready();
      } catch (error) {
        if (!stale()) loadFailed(error);
      }
    })();
    return () => {
      current = false;
    };
  }, [ready, loadFailed]);

  const change = (name: string, value: FieldValue) => {
    setValues((previous) => ({ ...previous, [name]: value }));
    setErrors((previous) => ({ ...previous, [name]: undefined }));
  };

  /** Fills the form with sample values (non-production only). */
  const fillTestData = () => {
    const tag = Math.random().toString(36).slice(2, 7).toUpperCase();
    setValues((previous) => ({ ...previous, name: `TEST ${tag}`, status: options.status?.[0]?.value ?? "" }));
  };

  return (
    <FormContext.Provider value={{ specs: fieldSpecs, values, errors, options, locked: {}, isVisible, change }}>
      <Page title="Site visit">
        <TestDataBox onFill={fillTestData} />
        <Section title="Visit">
          <Field name="name" />
          <Row>
            <Field name="visitDate" />
            <Field name="status" />
          </Row>
          <Field name="cancelReason" />
        </Section>
        <Section title="People">
          <Field name="inspectors" />
        </Section>
        <SaveBar bar={bar} />
      </Page>
    </FormContext.Provider>
  );
}
```

Walk through it:

- **State.** `values` holds what the user entered (text as typed, the ticked ids for the
  checkboxes), `errors` the messages under the fields, `options` the loaded lists. Typing calls
  `change`, which updates `values` and clears that field's error.
- **A field asked only sometimes.** `askReason` is computed from the status on every render;
  `isVisible` says `cancelReason` is shown only then, and `<Field>` draws nothing otherwise.
  `checkRequired(fieldSpecs, values, isVisible)` skips fields that are not shown, so the reason is
  required exactly while it is shown. There is no "show" or "hide" call to forget.
- **Loading.** One effect reads the source record and the options when the page opens, then calls
  `ready()` to enable Save, or `loadFailed(error)`, which shows the error and keeps Save disabled.
  The `stale()` check ignores the answer if the page was closed meanwhile.
- **`toPayload`** is the only place values meet column names. Optional values are `null` when empty.
- **Saving.** `useSaveBar` takes the function that means "save": check required fields, then the
  rules, then write. `createThen` creates the visit and runs the follow-up (the N:N link). If the
  follow-up fails, it deletes the visit again and the error says "Nothing was saved", so there is
  never a half-saved record. Returning `false` stops without a message (the errors are already on
  the screen).
- **Layout.** `Page`, `Section` and `Row` are the layout; `Field` draws a field by name; `SaveBar`
  shows the status, Save and Open the record.
- The command (`command/index.ts`) needs no change: it already opens this page for the form's record.

## 7. Mock data for the preview and the tests

The preview and the tests run on a fake Dataverse. Add the new table, the choice, the relationship
and a second user to the data, in **both** `tools/preview/mock-data.json` (for trying things) and
`test/fixtures/mock-data.json` (for the tests). `pnpm dv:pull` writes the table and its choices
for you; add the relationship and the user by hand:

```powershell
pnpm dv:pull pidev_sitevisit --columns pidev_status --name-field pidev_name --into tools/preview/mock-data.json
pnpm dv:pull pidev_sitevisit --columns pidev_status --name-field pidev_name --into test/fixtures/mock-data.json
```

Either way, the data then holds (`dv:pull` also writes `idField`, the default `pidev_sitevisitid`):

```json
"tables": {
  "pidev_sitevisit": { "nameField": "pidev_name", "entitySet": "pidev_sitevisits", "records": [] },
  "systemuser": { "nameField": "fullname", "records": [
    { "systemuserid": "11111111-1111-1111-1111-111111111111", "fullname": "Preview User" },
    { "systemuserid": "11111111-1111-1111-1111-111111111112", "fullname": "Sam Inspector" }
  ] }
},
"choices": {
  "pidev_sitevisit.pidev_status": [[1, "Planned"], [2, "Done"], [3, "Cancelled"]]
},
"relationships": {
  "pidev_sitevisit_inspectors": { "Entity1LogicalName": "pidev_sitevisit", "Entity2LogicalName": "systemuser" }
}
```

(Keep the existing keys; add these ones. The `systemuser` table already has the preview user.)
The format is in [mock Dataverse](../../8-reference/web-resources/mock-dataverse.md#data-file-format). The
preview reloads by itself and shows the page; the log panel lists every write it would make.

---

**Previous:** [2. Rules, fields and the page helpers](02-rules-fields-page.md) · **Next:** [4. Tests](04-tests.md)
