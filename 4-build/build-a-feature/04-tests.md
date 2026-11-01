---
sidebarTitle: "Tests"
---

# Build a feature: 4. Tests

> Part 4 of 5 of the worked example in [Build a feature](../build-a-feature.md).

## 8. Tests

Unit tests (`test/unit/site-visit.test.tsx`) cover, in this order of value: the rules, the page's
behaviour (what shows, what is refused, what is written), and the failure path. The page tests
use React Testing Library: they render the page and use it the way a user does.

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { beforeEach, describe, expect, it } from "vitest";
import { Metadata } from "@core";
import { validate } from "../../src/features/site-visit/logic/rules";
import { SiteVisitPage } from "../../src/features/site-visit/page/SiteVisitPage";
import { schema } from "../../src/features/site-visit/schema";
import { Ids, installMockDataverse, type MockDataverse, openWithRecord } from "../helpers/mockDataverse";

const valid = { name: "Hangar 3", visitDate: null, status: 1, cancelReason: "", inspectors: [] };

describe("site-visit rules", () => {
  it("accepts a name and a status alone", () => {
    expect(validate(valid)).toBeNull();
  });

  it("rejects a visit date in the past", () => {
    expect(validate({ ...valid, visitDate: "2020-01-01" }, new Date("2026-10-08"))?.field).toBe("visitDate");
  });

  it("lets a cancelled visit keep a past date", () => {
    expect(validate({ ...valid, status: 3, visitDate: "2020-01-01" }, new Date("2026-10-08"))).toBeNull();
  });
});

describe("site-visit schema", () => {
  beforeEach(() => installMockDataverse());

  it("has the Cancelled option and the inspectors relationship in the mock data", async () => {
    const options = await Metadata.columnOptions("pidev_sitevisit", "pidev_status");
    expect(options.map((option) => Number(option.value))).toContain(3);
    await expect(Metadata.manyToMany(schema.relationships![0].schemaName)).resolves.toBeTruthy();
  });
});

describe("site-visit page", () => {
  let dv: MockDataverse;
  const field = (id: string) => document.getElementById(id) as HTMLInputElement & HTMLSelectElement;
  const save = () => screen.getByRole("button", { name: "Save" });
  const status = () => screen.getByRole("status");

  /** Fills the fields every visit needs. */
  async function fillVisit(user: ReturnType<typeof userEvent.setup>, statusValue: string) {
    await user.type(field("name"), "Hangar 3");
    await user.type(field("visitDate"), "2030-01-01");
    await user.selectOptions(field("visitStatus"), statusValue);
  }

  beforeEach(async () => {
    dv = installMockDataverse();
    openWithRecord("account", Ids.account);
    render(<SiteVisitPage />);
    await waitFor(() => expect(save()).toBeEnabled());
  });

  it("asks for the reason only when the status is Cancelled", async () => {
    const user = userEvent.setup();
    expect(field("cancelReason")).toBeNull();
    await user.selectOptions(field("visitStatus"), "3");
    expect(field("cancelReason")).toBeInTheDocument();
  });

  it("does not save a cancelled visit without a reason", async () => {
    const user = userEvent.setup();
    await fillVisit(user, "3");
    await user.click(save());
    await waitFor(() => expect(field("cancelReason-error")).toHaveTextContent("Enter why it was cancelled."));
    expect(dv.writes()).toEqual([]);
  });

  it("creates the visit linked to the account, then links the inspectors", async () => {
    const user = userEvent.setup();
    await fillVisit(user, "1");
    await user.click(screen.getByRole("checkbox", { name: "Sam Inspector" }));
    await user.click(save());
    await waitFor(() => expect(status()).toHaveTextContent("Saved."));
    expect(dv.writes().map((write) => write.type)).toEqual(["createRecord", "Associate"]);
    expect(dv.writes()[0].payload).toMatchObject({
      pidev_name: "Hangar 3",
      pidev_status: 1,
      "pidev_Account@odata.bind": `/accounts(${Ids.account})`,
    });
  });

  it("deletes the visit again when linking the inspectors fails", async () => {
    const user = userEvent.setup();
    dv.fail({ Associate: "Link failed" });
    await fillVisit(user, "1");
    await user.click(screen.getByRole("checkbox", { name: "Sam Inspector" }));
    await user.click(save());
    await waitFor(() => expect(status()).toHaveTextContent("Nothing was saved"));
    expect(dv.writes().map((write) => write.type)).toEqual(["createRecord", "Associate", "deleteRecord"]);
  });
});
```

- `render(<SiteVisitPage />)` draws the page in jsdom; the tests wait until Save is enabled, which
  is when loading has finished.
- Controls are found by their ids here; `screen.getByRole("checkbox", { name: "Sam Inspector" })`
  finds one by what a user sees.
- `userEvent` types, clicks and picks like a user. Everything that changes the screen is awaited
  with `waitFor` or `findBy...`.

The browser test (`test/e2e/site-visit.spec.ts`) drives the real page in a browser against the
preview, for the main path:

```ts
import { expect, test } from "@playwright/test";
import { FeaturePage } from "./helpers/featurePage";

/** The Site visit page in the local preview, on test/fixtures/mock-data.json. */
test.describe("site-visit", () => {
  let page: FeaturePage;

  test.beforeEach(async ({ page: browserPage }) => {
    page = new FeaturePage(browserPage, "site-visit");
    await page.open({ table: "account", id: "a1000000-0000-0000-0000-000000000001" });
  });

  test("loads the status options from the table metadata", async () => {
    await expect(page.field("visitStatus").locator("option")).toHaveText([
      "Select an option",
      "Planned",
      "Done",
      "Cancelled",
    ]);
  });

  test("asks for the reason only for a cancelled visit", async () => {
    await expect(page.field("cancelReason")).toBeHidden();
    await page.select("visitStatus", "Cancelled");
    await expect(page.field("cancelReason")).toBeVisible();
  });

  test("creates the visit linked to the account", async () => {
    await page.fill("name", "Hangar 3");
    await page.fill("visitDate", "2030-01-01");
    await page.select("visitStatus", "Planned");
    await page.click("save");
    await expect(page.field("status")).toHaveText(/Saved/);
    const [write] = await page.writes();
    expect(write.type).toBe("createRecord");
    expect(write.payload).toMatchObject({
      pidev_name: "Hangar 3",
      "pidev_Account@odata.bind": "/accounts(a1000000-0000-0000-0000-000000000001)",
    });
  });
});
```

More on tests, including what each layer is for: [test and check](../test-and-check.md).

---

**Previous:** [3. The page component and mock data](03-page-and-mock-data.md) · **Next:** [5. Button and build](05-button-and-build.md)
