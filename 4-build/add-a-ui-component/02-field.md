---
sidebarTitle: "A field"
---

# Add a UI component: 2. A field

> Part 2 of 2 of [Add a UI component](../add-a-ui-component.md). **Example:** `PhoneField`, a
> phone number input.

A field has a label, a hint, an error message and a value. The label, hint and error come from
`FieldShell`; you only supply the control. The page keeps the text as typed, and `digitsOf` turns
it into digits only when the value is used, so `(613) 555-0100` is saved as `6135550100`.

**1. The component**, `src/ui/phone-field/PhoneField.tsx`:

```tsx
/**
 * @file PhoneField.tsx
 * @description A phone number field. The form holds the text as typed; digitsOf() gives the
 * digits only, so "(613) 555-0100" is saved as "6135550100". Label, hint and error come from FieldShell.
 */

import { controlA11y, FieldShell } from "../field/FieldShell";

/** The digits of a phone number as typed: "(613) 555-0100" → "6135550100". */
export function digitsOf(text: string): string {
  return text.replace(/\D/g, "");
}

/** The props of a phone field. */
export interface PhoneFieldProps {
  id: string;
  label: string;
  value: string;
  required?: boolean;
  hint?: string;
  error?: string;
  onChange: (value: string) => void;
}

/** A telephone input inside the shared field shell. */
export function PhoneField({ id, label, value, required, hint, error, onChange }: PhoneFieldProps) {
  return (
    <FieldShell id={id} label={label} required={required} hint={hint} error={error}>
      <input
        id={id}
        type="tel"
        value={value}
        required={required}
        {...controlA11y(id, hint, error)}
        onChange={(event) => {
          onChange(event.target.value);
        }}
      />
    </FieldShell>
  );
}
```

There is no `.css` here because the field's look is the shared `field/field.css`, which
`FieldShell` imports. Add a `.css` (and import it) only when the component needs rules of its own.

`FieldShell` takes the field's `id`, `label`, `required`, `hint` and `error`. `controlA11y` gives
the control the `aria-describedby` (the hint, then the error) and `aria-invalid` that a screen
reader needs. The control's own `id` must be the same `id`, so the label points at it.

**2. Export it**:

```ts
export { digitsOf, PhoneField, type PhoneFieldProps } from "./phone-field/PhoneField";
```

**3. Use it.** A component that is not one of the built-in field kinds is used directly in the
page, with the page's state:

```tsx
const [phone, setPhone] = useState("");
// ...
<PhoneField id="phone" label="Phone" hint="Digits only are saved." value={phone} required onChange={setPhone} />
// in the payload:  [columns.phone]: digitsOf(phone),
```

(To make it a field kind that `<Field name="phone" />` can draw, add a spec kind in
`form/types.ts`, a builder in `form/specs.ts` and a `case` in `form/Field.tsx`. Do that only when
several pages need it.)

**4. Test it**, in `test/unit/ui/phone-field.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { useState } from "react";
import { describe, expect, it } from "vitest";
import { digitsOf, PhoneField } from "@ui";

describe("PhoneField", () => {
  /** A phone field with its own state, as a page would hold it. */
  function Harness() {
    const [value, setValue] = useState("");
    return <PhoneField id="phone" label="Phone" hint="Digits only are saved." value={value} onChange={setValue} />;
  }

  it("keeps what the user typed, and digitsOf gives the digits", async () => {
    render(<Harness />);
    await userEvent.type(screen.getByRole("textbox", { name: "Phone" }), "(613) 555-0100");
    expect(screen.getByRole("textbox")).toHaveValue("(613) 555-0100");
    expect(screen.getByRole("textbox")).toHaveAccessibleDescription("Digits only are saved.");
    expect(digitsOf("(613) 555-0100")).toBe("6135550100");
    expect(digitsOf("n/a")).toBe("");
  });
});
```

## A field that loads data

A field with options from Dataverse (like `choice`) is drawn by `SelectField` from a spec whose
`load()` reads the options. The page loads them once and passes them down as `options`; while
they are `undefined` the select is disabled and says "Loading...". Copy `SelectField.tsx` and
the `choice` builder in `form/specs.ts` as the starting point.

Then run the checks in [the steps](../add-a-ui-component.md#the-steps) and add a row to
[UI components](../../8-reference/web-resources/ui-components.md).

**Next:** [Build a feature](../build-a-feature.md) · [UI components reference](../../8-reference/web-resources/ui-components.md)

---

**Previous:** [1. A display component](01-display-component.md)
