---
sidebarTitle: "The brief"
---

# Your first feature: 1. The brief

> Part 1 of 6 of [Your first feature](../first-feature.md).

## What the user sees

A command button **Callback request** on a contact's form opens a dialog titled **Callback request**.
The user fills it in and saves; the dialog says it is saved and offers to open the new record.

| Field | Kind | Required | Notes |
|---|---|---|---|
| Topic | text | yes | up to 100 characters |
| Priority | choice | no | Low, Normal, Urgent |
| Preferred date | date | no | |
| Phone | text | **only** for an Urgent request | shown only when the priority is Urgent |
| Notes | text area, 4 rows | no | up to 500 characters; a character counter is shown under it |

## The rules

1. The topic is required and is at most 100 characters.
2. The preferred date may be today or later, never in the past.
3. When the priority is **Urgent**, the phone is required and must look like a phone number: only
   digits, spaces and `+ - ( )`, with at least 7 digits.
4. For any other priority (or none) the phone field is **not shown** and is not required.
5. The notes are at most 500 characters. The text area must still let the user type past 500, so the
   counter can show the problem; the rule reports it when saving.
6. A rule's message appears under the field it is about, and the cursor moves to that field.
7. If saving fails, the dialog shows the error and the user can try again.

## The flow

When an **Urgent** callback request is created, a flow emails the **owner of the contact** it is
linked to: the contact's name, the topic, the phone, the preferred date and the notes. If the owner has
no email address, or anything else fails, the run ends as Failed with a clear message. The full list
of requirements (F1 to F6) is in [4. The alert flow](04-alert-flow.md).

## Where the data goes

The page creates one record in a **new table**. The names below use the prefix `pidev`; if your
project's publisher prefix is different, your mentor will tell you which to use instead.

| Part | Name |
|---|---|
| Table (logical name) | `pidev_callbackrequest`, display name *Callback request* |
| Entity set | `pidev_callbackrequests` |
| Primary name column | `pidev_topic` |
| Other columns | `pidev_preferreddate` (date only), `pidev_priority` (choice), `pidev_phone`, `pidev_notes` |
| Priority options | Low = 0, Normal = 1, Urgent = 2 |
| Link to the contact | a lookup to *contact*, set with the navigation property `pidev_Contact` |

The link is set when the record is created, with `"pidev_Contact@odata.bind": "/contacts(<id>)"`. The
letter case of a navigation property matters. In the dev environment the real name may differ; look at
the table's relationships in Power Apps, and change only `config.ts`.

## The small UI component

`CharCounter` shows `123 / 500` under a text field and turns red, with the words "too long", when the
text is longer than the limit. It takes two props, the text and the maximum, and knows nothing about
notes or Dataverse. It belongs to the shared kit in `src/ui/char-counter/`, because any page could
use it.

## Done means

- every rule above works in the preview;
- the unit tests cover each rule at its boundary (100 and 101 characters, today and yesterday, 6 and 7
  digits, 500 and 501 characters) and the page's behaviour;
- one browser test follows an Urgent request from opening to saved;
- `CharCounter` has its own test;
- the alert flow builds, passes its tests, and does what F1 to F6 say;
- `pnpm first-feature:check` prints **READY**;
- your mentor has reviewed it, and it works in the dev environment, flow included.

**Next:** [Build it](02-build.md).

---

**Previous:** [Your first feature](../first-feature.md)
