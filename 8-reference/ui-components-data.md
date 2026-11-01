---
sidebarTitle: "UI components: data and dialogs"
---

# UI components: lists, tables, charts, dialogs and tabs

> **Reference.** More of the React kit in `src/ui`; import from `"@ui"`. The form fields, layout,
> save bar and feedback components (`Alert`, `Spinner`, `Badge`) are on [UI components](ui-components.md).
> Tests in `test/unit/ui/`.

In the starter, every component of the kit is shown live in the example feature: run `pnpm dev`,
open Quick task and look at the "Component showcase" tabs (`src/features/quick-task/page/showcase/`).
A project that deleted `quick-task` ([start a new project](../2-set-up/02-start-a-new-project.md))
no longer has them; read the same folder in the starter.

Like the rest of the kit, these take their data as props and hold no table names: the page loads
the records (see the [core API](core-api.md)) and the component draws them.

```tsx
import { Chart, type Column, DataTable, Pagination, countBy, paginate } from "@ui";

const columns: Column<Task>[] = [
  { key: "name", header: "Task", sortValue: (t) => t.name },
  { key: "team", header: "Team" },
];
const shown = paginate(tasks, 10, page); // page starts at 1
const byTeam = useMemo(() => countBy(tasks, "team"), [tasks]);
const datasets = useMemo(() => [{ label: "Tasks", data: byTeam.data }], [byTeam]);

<DataTable caption="Tasks" columns={columns} rows={shown} rowKey={(t) => t.id} />
<Pagination total={tasks.length} pageSize={10} page={page} onPage={setPage} />
<Chart type="bar" ariaLabel="Tasks per team" labels={byTeam.labels} datasets={datasets} />
```

## Lists and tables

| Component | Gives |
|---|---|
| `<DescriptionList items columns>` | Label/value pairs (`<dl>`) for a record summary. `items` are `{ label, value }`; an empty value shows "-". `columns` is 1 (default) or 2. |
| `<ItemList label items selection selected onSelect onOpen loading emptyText>` | Records as a list. `items` are `{ id, primary, secondary?, badge? }`; `badge` is any element, e.g. a `<Badge>`. `selection` is `"none"` (default), `"single"` or `"multiple"`; `selected` holds the item ids. With a selection or an `onOpen` the list is interactive: Up/Down/Home/End move, Space or a click selects, Enter or a double click calls `onOpen(id)`. With neither it is a plain list. Shows a `Spinner` while `loading`. |
| `<DataTable caption columns rows rowKey selectable selected onSelect emptyText>` | A table. A column is `{ key, header, render?, sortValue?, align? }`. Without `render` the cell shows `row[key]`. With `sortValue` the header is a button that sorts, and sorts the other way on a second click (`aria-sort` says which). `align` is `"right"` or left. `selectable` adds a checkbox per row and "Select all"; `selected` holds the `rowKey` values. |
| `<Pagination total pageSize page onPage id>` | Previous / Next and "11-20 of 25". `page` starts at 1. The buttons are `#<id>-prev` and `#<id>-next` (`id` default `pager`). It does not slice the list: use `paginate`. |
| `paginate(items, size, page)`, `pageWindow(total, size, page)` | The items of one page; the page (kept between 1 and the last), the page count and the index range. |
| `sortRows(rows, valueOf, direction)` | A sorted copy (`direction` is `"asc"` or `"desc"`): numbers by size, text by locale with numbers in order (`item 2` before `item 10`), empty values last in both directions. `DataTable` uses it. |

## Charts

`<Chart type labels datasets ariaLabel title stacked />` draws a chart with
[Chart.js](https://www.chartjs.org) on a canvas.

- `type` is `"bar"`, `"line"`, `"pie"` or `"doughnut"`. `datasets` are `{ label, data }` with one
  number per label. Bar and line charts colour each series; pie and doughnut charts colour each
  slice. `stacked` stacks the series of a bar or line chart and does nothing for pie and doughnut.
- `ariaLabel` is required: the canvas is `role="img"`, and a hidden table holds the same numbers
  for screen readers.
- Give `labels` and `datasets` a stable identity (`useMemo`, as above): a new array on every render
  redraws the numbers every time. A change of `type`, `title` or `stacked` rebuilds the chart.
- Animation is off when the user's system asks for reduced motion.
- `countBy(rows, key)` and `sumBy(rows, key, amount)` turn records into `{ labels, data }`, one
  entry per value of `key` in order of first appearance; a blank value is grouped as "(none)".
  `amount(row)` gives the number to add up for a row.
- **Colours and size.** `--pp-chart-1` ... `--pp-chart-6` (they repeat when there are more series or
  slices) and `--pp-chart-height` (default 280px) are CSS variables; set them in the feature's
  `page.css` to restyle one page. Other kit variables the components use: `--pp-success`,
  `--pp-warning` and `--pp-surface-alt`, next to those in `src/ui/tokens.css`.
- **Size of the page.** Only a page that draws a `Chart` bundles Chart.js (about 0.4 MB
  unminified; `pnpm build --minify` shrinks it). The other pages are unchanged.
- **Tests.** jsdom has no canvas, so a test that renders a `Chart` replaces `chart.js` with a fake:
  `test/helpers/fakeChartJs.ts`, used as in `test/unit/quick-task.test.tsx`.

## Dialogs and tabs

| Component | Gives |
|---|---|
| `<ConfirmDialog open title confirmLabel cancelLabel danger onConfirm onCancel id>` | A modal question; the question is the children. The focus starts on Cancel, Tab stays between the two buttons, Escape cancels, and the focus goes back to where it was when the dialog closes. The buttons are `#<id>-confirm` and `#<id>-cancel` (`id` default `confirm`, labels "OK" and "Cancel"). `danger` makes Confirm red. Draws nothing while `open` is false; the page owns `open` and closes the dialog in both callbacks. |
| `<Tabs label tabs initial onChange>` | Tabs; `tabs` are `{ id, label, content }`, `initial` is the id of the tab shown first (default: the first) and `onChange(id)` is called when the user switches. Left/Right (wrapping), Home and End move. Every panel stays mounted (the others are hidden), so what the user typed survives a switch. A tab is `#<tab id>-tab` and its panel `#<tab id>-panel`. |

Ids: keep the ids you give `Pagination`, `ConfirmDialog` and `Tabs` (and the ids they derive) unique
on the page, like field ids.
