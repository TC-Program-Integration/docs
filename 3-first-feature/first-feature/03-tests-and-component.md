---
sidebarTitle: "Test it and build the component"
---

# Your first feature: 3. Test it and build the component

> Part 3 of 6 of [Your first feature](../first-feature.md). Steps 5 and 6.

## Step 5. Tests

The scaffold created `test/unit/callback-request.test.tsx` and `test/e2e/callback-request.spec.ts`
for a task. Rewrite them for a callback request. [Write tests](../../4-build/test-and-check/01-write-tests.md) explains the tools.

**Rules, tested directly** (no page, no mock): one test per line, named as a sentence.

| Test | Why it matters |
|---|---|
| a topic of 100 characters is accepted, 101 is not | the boundary, both sides |
| today is accepted as the preferred date, yesterday is not | pass `today` in; never depend on the real clock |
| an Urgent request without a phone is rejected, with a valid phone it is accepted | rule 3 |
| other priorities do not ask for a phone | rule 4 |
| a phone with 6 digits is rejected, with 7 accepted; letters are rejected | the phone check alone |
| notes of 500 characters are accepted, 501 are not | the boundary |

**The page, with the mock Dataverse** (`render`, `userEvent`): the helpers are described in
[Mock Dataverse](../../8-reference/mock-dataverse.md).

| Test | Proves |
|---|---|
| saving without a topic shows "Enter topic." and writes nothing | the required check, no write |
| the phone field appears for Urgent and disappears for another priority | `isVisible` |
| Urgent without a phone shows the phone message | a hidden-then-shown field is required |
| notes over 500 characters turn the counter red and block saving | the component and the rule together |
| a valid request creates the record, with your column names and the contact link | the payload |
| when the create call fails, the error shows and Save works again | rule 7 |

**One browser test** in `test/e2e/callback-request.spec.ts`: open the page from a contact, choose
*Urgent*, fill the topic and phone, save, and read the payload from `page.writes()`. Delete the
scaffold's other tests or rewrite them. `pnpm test:e2e` runs it (it uses Microsoft Edge).

**A test that cannot fail is not a test.** After writing one, break the code on purpose (change `100`
to `101`) and check that the test goes red; then put it back.

## Step 6. The `CharCounter` component

Follow [Add a UI component](../../4-build/add-a-ui-component.md), which builds two other components step by
step. Yours:

1. `src/ui/char-counter/char-counter.css`: the look. Class names start with `pp-`, colours come from
   `var(--pp-*)`. Muted text normally; the error colour when over the limit.
2. `src/ui/char-counter/CharCounter.tsx`: props `value: string` and `max: number`, both documented.
   Shows `5 / 500`; over the limit it adds the words "too long" and the over-the-limit style. It
   announces the change to screen readers (`aria-live="polite"`).
3. Export it from `src/ui/index.ts`.
4. Use it in the page under the notes field: `<CharCounter value=... max=... />`. The limit is the
   same number the rule uses: export it from `rules.ts` instead of writing 500 twice.
5. `test/unit/ui/char-counter.test.tsx`: shows the length and the limit; plain at exactly the limit;
   red and "too long" one character over.
6. Add a row for it to [UI components](../../8-reference/ui-components.md) in the documentation, as
   the add-a-component guide says.

`pnpm lint` enforces the conventions (a doc comment on every export, no inline `style`, no HTML in
strings). Fix what it reports; the messages say what to do.

**Next:** [The alert flow](04-alert-flow.md).

---

**Previous:** [2. Build it](02-build.md)
