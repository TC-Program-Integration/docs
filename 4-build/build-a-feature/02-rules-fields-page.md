---
sidebarTitle: "Rules, fields and the page helpers"
---

# Build a feature: 2. Rules, fields and the page helpers

> Part 2 of 5 of the worked example in [Build a feature](../build-a-feature.md).

## 4. The rules: `logic/rules.ts`

Validation that needs no screen and no Dataverse is a pure function, so it is tested directly.

```ts
/**
 * @file rules.ts
 * @description The Site visit form's values and validation. Pure functions with no React,
 * DOM or Dataverse access, so they are tested directly. Required fields are checked by the
 * page (checkRequired); these are the other rules.
 */

import { SiteVisitConfig as Config } from "../config";

/** The values the Site visit form collects. */
export interface SiteVisitValues {
  name: string;
  /** yyyy-mm-dd, or null when not set. */
  visitDate: string | null;
  /** A status option value, or null when not chosen. */
  status: number | null;
  cancelReason: string;
  /** System user ids. */
  inspectors: string[];
}

/** A rule that failed, and the field to show it on. */
export interface Problem {
  field: "name" | "visitDate" | "cancelReason";
  message: string;
}

/** True when the form must ask why the visit was cancelled. */
export function needsReason(status: number | null): boolean {
  return status === Config.statusCancelled;
}

/** Returns the first problem with the values, or null when they can be saved. */
export function validate(values: SiteVisitValues, today: Date = new Date()): Problem | null {
  if (values.name.length > 100) return { field: "name", message: "The name must be 100 characters or fewer." };
  if (!needsReason(values.status) && values.visitDate && values.visitDate < today.toISOString().slice(0, 10)) {
    return { field: "visitDate", message: "The visit date cannot be in the past." };
  }
  return null;
}
```

Required fields are not here: `checkRequired` handles them, from the specs in the next step.

## 5. The fields: `page/fields.ts`

Each field is described once, as data. The page draws it by name and the required check reads the
same description.

```ts
/**
 * @file fields.ts
 * @description The Site visit form's fields as data. The page draws them by name
 * (<Field name="..." />) and the required check reads the same specs.
 */

import { checkboxes, choice, date, text } from "@ui";
import { SiteVisitConfig as Config } from "../config";

const { table, columns } = Config.target;

export const fieldSpecs = {
  name: text({ id: "name", label: "Visit name", required: true, maxLength: 100 }),
  visitDate: date({ id: "visitDate", label: "Visit date", required: true }),
  status: choice({ id: "visitStatus", label: "Status", table, column: columns.status, required: true }),
  // Asked only for a cancelled visit: required, but not shown until then.
  cancelReason: text({
    id: "cancelReason",
    label: "Why was it cancelled?",
    rows: 3,
    required: true,
    requiredMessage: "Enter why it was cancelled.",
  }),
  inspectors: checkboxes({
    id: "inspectors",
    label: "Inspectors",
    hint: "Select everyone who takes part.",
    table: Config.inspectors.table,
  }),
};

/** The name of a field of the form. */
export type FieldName = keyof typeof fieldSpecs;
```

- The key (`name`, `visitDate`...) is how the rest of the code refers to the field; the `id` is
  the HTML id, which tests and error messages use.
- **Ids must be unique on the page**, and four are taken by the kit itself: `status`, `save`,
  `openRecord` and `fillTestData`. That is why the status *field* here has the id `visitStatus`.
- `cancelReason` is `required: true`. It is asked only for a cancelled visit, which the page
  decides (next step): a field that is not shown is not required.

## 6. The page: `page/SiteVisitPage.tsx`

```tsx
/**
 * @file SiteVisitPage.tsx
 * @description The Site visit page: draws the form with the UI kit, validates it and saves the
 * visit with its inspectors. Fields are in fields.ts, Dataverse names in config.ts, rules in rules.ts.
 */

import { useEffect, useState } from "react";
import { associate, createThen, Metadata, openRecord, readSourceRecord, type SourceRecord } from "@core";
import {
  checkRequired,
  emptyValue,
  Field,
  type FieldValue,
  focusField,
  FormContext,
  type Option,
  Page,
  parseChoice,
  parseDate,
  Row,
  SaveBar,
  type SaveResult,
  Section,
  TestDataBox,
  useSaveBar,
} from "@ui";
import { SiteVisitConfig as Config } from "../config";
import { needsReason, type SiteVisitValues, validate } from "../logic/rules";
import { type FieldName, fieldSpecs } from "./fields";

const { table, columns } = Config.target;
const names = Object.keys(fieldSpecs) as FieldName[];

/** Reads the form into the values the rules and the payload use. */
function toValues(values: Record<FieldName, FieldValue>): SiteVisitValues {
  return {
    name: (values.name as string).trim(),
    visitDate: parseDate(values.visitDate as string),
    status: parseChoice(values.status as string),
    cancelReason: (values.cancelReason as string).trim(),
    inspectors: values.inspectors as string[],
  };
}

/** Maps the values to the target table's columns. */
async function toPayload(values: SiteVisitValues, source: SourceRecord): Promise<Record<string, unknown>> {
  return {
    [columns.name]: values.name,
    [columns.visitDate]: values.visitDate,
    [columns.status]: values.status,
    [columns.cancelReason]: needsReason(values.status) ? values.cancelReason : null,
    [Config.sourceBinding]: await Metadata.recordPath(source.table, source.id),
  };
}

```

---

**Previous:** [1. Scaffold, config and schema](01-scaffold-and-config.md) · **Next:** [3. The page component and mock data](03-page-and-mock-data.md)
