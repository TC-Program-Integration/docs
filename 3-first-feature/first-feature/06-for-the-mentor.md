---
sidebarTitle: "For the mentor"
---

# Your first feature: for the mentor

> Part 6 of 6 of [Your first feature](../first-feature.md). **For:** whoever reviews the exercise.

The check proves that nothing is missing and everything passes. Your review decides whether the
student understood what they built. About an hour: read the code with them, ask four questions,
watch the dev test, flow included.

## Read the code

| Look at | Good |
|---|---|
| `config.ts` | every Dataverse name is here; the Urgent value is a named constant; no names elsewhere |
| `logic/rules.ts` | pure functions; today's date is a parameter; messages say what to do; the limits are exported and used by the page |
| `page/fields.ts` | specs only, no logic; `notes` has no `maxLength` (rule 5); ids are the ones the tests use |
| `CallbackRequestPage.tsx` | one `isVisible` used by the layout and by `checkRequired`; `toPayload` is the only place columns are mapped; no `style={...}`, no HTML strings |
| unit tests | boundaries on both sides; names read as sentences; no test repeats the implementation |
| the browser test | follows one user journey; reads the payload, not only the screen |
| `flows/callback-alert/` | the filter is in the trigger; names only in `settings.py`; a condition, not a hope, for the missing email; every step inside the try scope; the catch ends with a Terminate |
| `flows/tests/test_callback_alert.py` | order, trigger filter, email fields and both failure paths are asserted; it does not just re-read the flow's own constants |
| `CharCounter` | knows nothing about notes or Dataverse; documented props; announced to screen readers; its own test |

Ask the student to break one rule on purpose (change `100` to `101`) and show which test fails. If
none does, the tests are not testing that rule.

## Four questions

1. Why is `validate` in its own file and not inside the page? *(It is pure, so it is tested without
   a page or Dataverse; the page only draws.)*
2. An Urgent request is saved, then the create call fails halfway. What does the user see, and what is
   left in Dataverse? *(The error and a Save button that works; `createThen` removes a half-saved record.)*
3. Why is the Urgent filter in the flow's trigger and not in a condition after it? *(A condition inside the flow still starts a run for every request, which then ends early; the trigger filter starts no run at all, so the run history and the quota only show real work.)*
4. Why is the phone not required when it is hidden? *(Required is checked only for visible fields, so
   hidden and required can never disagree.)*

## The dev test

- the flow is in the project's solution with its two connection references (Dataverse, Outlook) pointing at a connection that belongs to a service or team account, not the student;
- an Urgent request starts exactly one run, which emails the contact's owner; a Normal request starts none;
- a contact whose owner has no email gives a Failed run with the message `NoOwnerEmail`;
- the table exists in the project's solution, with the names in `config.ts`; `pnpm test:dataverse` passes;
- the web resources were pushed with the links file and the button created from `dist/deploy.json`,
  not typed by hand; `pnpm dv:verify` passes;
- the command button is on the contact form and opens the dialog;
- a normal request, an Urgent request with a phone, and a refused one (no phone; past date) behave as
  the brief says, and the saved records are linked to the contact.

## Sign-off

READY output pasted, review comments fixed, dev test done. Record the date and the reviewer in
whatever your team uses for onboarding. The student's feature is an exercise, not part of the
project: do not merge it. Keep the branch for reference, or delete it; on a shared branch, remove
`callback-request` from `features` and `buttons` in `webresources.config.mjs`, its folders in
`src/features/`, `test/` and `flows/`, and `src/ui/char-counter/`.

---

**Previous:** [5. Check it and show it](05-check-and-show.md)
