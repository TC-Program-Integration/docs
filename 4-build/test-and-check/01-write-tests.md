---
sidebarTitle: "Write tests"
---

# Test and check: writing tests

> Part 1 of 2 of [Test and check](../test-and-check.md). **For:** writing the tests of a feature or
> a core change. **Result:** tests that run offline in seconds.

Every generated feature already has unit and browser tests. Change them with the feature. The
[worked example](../build-a-feature/04-tests.md) shows a complete set; the short examples below use the project's `quick-task` feature.

## 1. Test the rules

Pure functions, no setup:

```ts
it("rejects a due date in the past", () => {
  expect(validate({ ...valid, dueDate: "2020-01-01" }, new Date("2026-10-08")))
    .toEqual({ field: "dueDate", message: "The due date cannot be in the past." });
});
```

Pass "today" as an argument instead of reading the clock, so the test never goes stale.

## 2. Test the state and the page on the mock

A page that outgrows `useState` keeps its state changes in a plain function, a reducer in
`formState.ts` (see [feature files](../../8-reference/feature-files.md)). Test it like the rules in
section 1: call it with a state and an action, and check the new state. It needs no React.

The page is tested by rendering it and using it the way a user does, with
[React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) and
`userEvent`:

```tsx
const dv = installMockDataverse();                    // Xrm on test/fixtures/mock-data.json
openWithRecord("account", Ids.account);               // as if opened from that record
render(<QuickTaskPage />);                            // React draws it into jsdom
await waitFor(() => expect(screen.getByRole("button", { name: "Save" })).toBeEnabled());   // loaded

const user = userEvent.setup();
await user.type(document.getElementById("subject")!, "Call back");
await user.click(screen.getByRole("button", { name: "Save" }));
await waitFor(() => expect(screen.getByRole("status")).toHaveTextContent("Saved."));

expect(dv.writes()).toEqual([
  expect.objectContaining({ type: "createRecord", table: "task", payload: expect.objectContaining({ subject: "Call back" }) }),
]);
```

Cover, for each page:

- it loads (Save enabled, options present);
- a required field or a rule stops the save, with the message under the field
  (`#<id>-error`) and no writes;
- a valid save writes the expected payload;
- a failed write shows the error and lets the user retry:
  `dv.fail({ createRecord: "Network error" })`.

Habits that keep React tests stable:

| Do | Why |
|---|---|
| Wait for what the user waits for: `await screen.findBy...`, `await waitFor(() => expect(...))` | the page loads and saves asynchronously; a check made too early sees the old screen |
| Find controls by role and name (`getByRole("combobox", { name: "Status" })`) when the label is plain; by id (`document.getElementById("visitStatus")`) when the label has a symbol | the test then fails when a user could not find the control either |
| `await user.click(...)`, never `fireEvent` | `userEvent` fires the same events a browser does (focus, input, change) |
| Render inside `<StrictMode>` in at least one test | it runs effects twice, which finds effects that are not safe to repeat |
| Do not test React itself (that a state change redraws) | test what the user sees and what is written |

Change the data for one test with `installMockDataverse((data) => { ... })`.
All helpers: [mock Dataverse](../../8-reference/mock-dataverse.md#unit-test-helpers).
`test/setup.ts` unmounts what a test rendered, so each test starts on an empty page.

## 3. Test the main path in the browser

```ts
const page = new FeaturePage(browserPage, "quick-task");
await page.open({ table: "account", id: "a1000000-0000-0000-0000-000000000001" });   // waits until Save is enabled
await page.fill("subject", "Call back");
await page.select("priority", "High");
await page.click("save");
await expect(page.field("status")).toHaveText(/Saved/);
expect(await page.writes()).toHaveLength(1);
```

Keep browser tests to the paths a user takes; edge cases belong in unit tests, which are
faster. On a failure, `test/.results/` holds a screenshot and a trace
(`pnpm exec playwright show-trace <trace.zip>`), and `test/.report/index.html` the report.

## 4. Describe what Dataverse must have

`schema.ts` lists the tables, columns, choices, records and relationships the feature relies on
([format](../../8-reference/feature-files.md#schemats)). Keep it in step with `config.ts`;
`pnpm test:dataverse` checks it against a real environment
([read real data](../read-real-data.md#3-keep-checking-with-pnpm-testdataverse)).

## 5. When you fix a bug

Write the test that fails because of the bug first, then fix the code. Never skip or delete a
failing test to get green.

## 6. Coverage

`pnpm test:coverage` runs the unit tests with coverage and fails when it drops below the floor in
`vitest.config.mts` (statements and lines 90 %, functions 90 %, branches 85 %). New code comes with
tests; when coverage grows, raise the floor. After a run, `test/.coverage/index.html` shows which
lines and branches no test reached.

---

**Previous:** [Test and check](../test-and-check.md) · **Next:** [2. The checks](02-the-checks.md)
