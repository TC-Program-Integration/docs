---
sidebarTitle: "A display component"
---

# Add a UI component: 1. A display component

> Part 1 of 2 of [Add a UI component](../add-a-ui-component.md). **Example:** `Notice`, a
> highlighted message to put above or between fields.

**1. The look**, `src/ui/notice/notice.css`:

```css
.pp-notice {
  display: flex;
  gap: 8px;
  padding: 8px 12px;
  border-left: 4px solid var(--pp-accent);
  background: color-mix(in srgb, var(--pp-accent) 8%, var(--pp-surface));
  color: var(--pp-text);
}
.pp-notice.pp-warning {
  border-left-color: var(--pp-error);
  background: color-mix(in srgb, var(--pp-error) 8%, var(--pp-surface));
}
```

**2. The component**, `src/ui/notice/Notice.tsx`:

```tsx
/**
 * @file Notice.tsx
 * @description A highlighted message above or between fields.
 */

import type { ReactNode } from "react";
import "../tokens.css";
import "./notice.css";

/** The props of a notice. */
export interface NoticeProps {
  title: string;
  /** The red look; the default is the blue, informational one. */
  warning?: boolean;
  children: ReactNode;
}

/** A message box with a bold title and the message as its children. */
export function Notice({ title, warning, children }: NoticeProps) {
  return (
    <div className={warning ? "pp-notice pp-warning" : "pp-notice"} role="status">
      <strong>{title}</strong>
      <span>{children}</span>
    </div>
  );
}
```

Three things to notice: the `.tsx` imports the `.css` (that is how the build knows to ship it),
the props are an exported, documented interface, and the message is `children`, so it can hold
more than text.

**3. Export it**, in `src/ui/index.ts`:

```ts
export { Notice, type NoticeProps } from "./notice/Notice";
```

**4. Use it** in a page, wherever a section or row would go:

```tsx
<Notice title="Heads up">Offices are closed on Monday.</Notice>
```

**5. Test it**, in `test/unit/ui/notice.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import { Notice } from "@ui";

describe("Notice", () => {
  it("shows a title and a message, red when it is a warning", () => {
    const { rerender } = render(<Notice title="Heads up">Offices are closed on Monday.</Notice>);
    expect(screen.getByRole("status")).toHaveTextContent("Heads upOffices are closed on Monday.");
    expect(screen.getByRole("status")).not.toHaveClass("pp-warning");
    rerender(
      <Notice title="Stop" warning>
        This cannot be undone.
      </Notice>,
    );
    expect(screen.getByRole("status")).toHaveClass("pp-warning");
  });
});
```

**6. Document it**: add a row to [UI components](../../8-reference/web-resources/ui-components.md).

---

**Previous:** [Add a UI component](../add-a-ui-component.md) · **Next:** [2. A field](02-field.md)
