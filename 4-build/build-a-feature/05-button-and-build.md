---
sidebarTitle: "Button and build"
---

# Build a feature: 5. Button and build

> Part 5 of 5 of the worked example in [Build a feature](../build-a-feature.md).

## 9. The button: `webresources.config.mjs`

The dialog opens from a button on the account form. Describe it next to the feature:

```js
export default {
  prefix: "pidev",
  features: ["quick-task", "site-visit"],
  buttons: {
    "quick-task": { label: "Quick task", table: "account" },
    "site-visit": { label: "Site visit", table: "account" },
  },
};
```

`pnpm new:feature` already added `"site-visit"` to `features`; the button is yours to add. No
`function` is given, so the button runs `open`, the function the scaffold's `command/index.ts`
exports.

## 10. Check and build

```powershell
pnpm check
pnpm test:e2e
pnpm test:dataverse        # needs pac signed in: the table, columns and relationship exist in dev
pnpm build
```

The build lists the web resources and the command's entry point:

```text
  pidev_site_visit_command.js
  pidev_site_visit_page.js
  pidev_site_visit_page.html
  pidev_site_visit_page.css

Command entry points (command designer: Library and Function):
  Library  pidev_site_visit_command.js
  Function SiteVisit.open
```

`webresources.links.xml` now links the four files to their web resources, and `dist/deploy.json`
holds the button:

```json
{
  "feature": "site-visit",
  "uniqueName": "pidev_sitevisit_open_account",
  "label": "Site visit",
  "table": "account",
  "location": "form",
  "library": "pidev_site_visit_command.js",
  "function": "SiteVisit.open",
  "parameters": ["PrimaryControl"]
}
```

Nothing is typed by hand from here: [Deploy](../../6-ship/deploy.md) pushes the files with the
links file and creates the button from `deploy.json`. Something wrong on the way? See
[mistakes people make](../build-a-feature.md#mistakes-people-make).

**Next:** build one yourself: [Your first feature](../../3-first-feature/first-feature.md).

---

**Previous:** [4. Tests](04-tests.md)
