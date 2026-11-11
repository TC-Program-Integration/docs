---
sidebarTitle: "UI components: data and dialogs"
---

# UI components: lists, tables, charts, dialogs and tabs

> **Reference.** More of the React kit in `src/ui`; import from `"@ui"`. The form fields, layout,
> save bar and feedback components (`Alert`, `Spinner`, `Badge`) are on [UI components](ui-components.md).
> Tests in `test/unit/ui/`.

In the starter, every component of the kit is shown live in the example feature: run `pnpm dev`,
open Quick task and look at the "Component showcase" tabs (`src/features/quick-task/page/showcase/`).
A project that deleted `quick-task` ([start a new project](../../2-set-up/02-start-a-new-project.md))
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
| `<Modal open title closeLabel wide id onClose>` | A modal for any content (a list, a calendar, a form), where `ConfirmDialog` only asks a question. The children are the body. The focus moves to the first control inside (not the close button), Tab and Shift+Tab stay among the dialog's controls, Escape and a click on the backdrop call `onClose`, and the focus goes back to where it was when it closes. `closeLabel` is the close button's accessible name in the page's language; `wide` gives it more room; `id` (default `modal`) names the title (`<id>-title`). Draws nothing while `open` is false; the page owns `open`. |
| `<SegmentedControl label value options onChange compact showLabel id>` | One choice of a few, as joined buttons with icons (how to show something, how to group it). `options` are `{ value, label, icon }`; `compact` shows only the icons (each keeps its label as its name). An ARIA radio group: Left/Right and Up/Down move the choice, Home and End go to the ends, and only the chosen button is a tab stop. Use it for a choice that is worth seeing all at once, not for actions. |
| `<SettingsButton />` / `<SettingsDialog open onClose />` | A gear button for the title row (`<Page titleActions={<SettingsButton />}>`) and the dialog it opens: the language (English or French), the theme (match system, light or dark) and accessibility: how much the page moves (match system, reduced, off), the contrast (match system, standard, high) and a strong keyboard focus ring. Each choice applies at once and is kept in the browser (`pp-preferences`; the language in `pp-language`). `mountPage` calls `startPreferences()`, which puts the choices on the root element (`data-theme`, `data-motion`, `data-contrast`, `data-focus`) before the page is drawn; the CSS (`theme/theme.css`, `motion/motion-modes.css`) and the animation code (`motion/motion.ts`) read them. Dark and high contrast are not applied when printing. |
| `useFormFactor()` | `{ narrow, compact, coarse }` for the few things CSS cannot decide (which panel starts open). `narrow` is a pane under `SHEET_BELOW` (800 px), `compact` under `COMPACT_BELOW` (480 px), `coarse` a finger or a pen. A web resource runs in an iframe, so the width is the pane's. The CSS (`ui/responsive/responsive.css`) uses the same widths: touch targets (`--pp-target`, 32 px, 44 px for a finger), hover-free grips, safe areas, dialogs as sheets under 600 px, icon-only choices under 480 px, and `DataTable` as cards under 720 px (each cell carries `data-label`). Text, gaps and padding are fluid (`tokens.css`); a feature's own `page.css` is built last, so a responsive rule in another file needs one more class (such as `.pp-page`) to override it. |
| `<MenuButton label icon items id>` | An icon button that opens a small menu of actions used now and then. `items` are `{ id, label, icon, onSelect }`. An ARIA menu button: the focus goes to the first item when it opens, the arrow keys, Home and End move between items, Escape closes it and gives the focus back, Tab and a press outside close it, and a choice closes it and runs. Also in `SchedulerEvent`: `alert` (the red marker is only for entries that need action; absent means any flag) and `initials` (drawn on the bar). |
| `<Tabs label tabs initial onChange>` | Tabs; `tabs` are `{ id, label, content }`, `initial` is the id of the tab shown first (default: the first) and `onChange(id)` is called when the user switches. Left/Right (wrapping), Home and End move. Every panel stays mounted (the others are hidden), so what the user typed survives a switch. A tab is `#<tab id>-tab` and its panel `#<tab id>-panel`. |

Ids: keep the ids you give `Pagination`, `ConfirmDialog` and `Tabs` (and the ids they derive) unique
on the page, like field ids.

## Scheduler (`src/ui/scheduler`)

A planning calendar for whole-day entries. It knows nothing about Dataverse: the page owns the
data, the date and the view, and answers the callbacks. Days are `"YYYY-MM-DD"` strings (the
Eastern day the business means); the library only ever sees local-midnight dates, converted in
one tested function (`logic/dates.ts`), so daylight saving and the user's time zone cannot move
an entry to another day.

| Component | Gives |
|---|---|
| `<SchedulerCalendar events resources splitByPerson view date today selectedId onNavigate onViewChange ...>` | The toolbar (Previous, Next, Today, Month/Week/Day) and the calendar (React Big Calendar, MIT, restyled from the `--pp-*` tokens). Controlled: it changes nothing itself. `events` are `SchedulerEvent`; `splitByPerson` gives each person a band in the week and day views. A bar is a real button with an accessible name (title, kind, status, days, Pending, flags). |
| `onMove(change)`, `onResize(change)` | **Keyboard:** with an editable bar focused, Left and Right move it by a day, Shift with Left or Right changes its last day, Up and Down change the person (when `splitByPerson` shows people); the same callbacks answer, and the focus goes back to the bar when it is drawn again. A line of help under the toolbar says so. **Mouse:** an entry was dragged to other days (and, in a person band, another person), or its first or last day was dragged. `change` is `{ id, start, end, resourceId? }`. Only entries with `editable: true` can be dragged. |
| `onSelect(id)`, `onCreateRange(range)` | An entry was clicked or activated with Enter or Space; empty days were clicked or dragged over (`{ start, end, resourceId? }`). |
| `outsideItem`, `onDropFromOutside(drop)` | Set `outsideItem` (`{ title, days }`) while the page drags an item in from outside; the drop is `{ start, end, resourceId? }`. |
| `<TeamGrid events resources from days today selectedId onNavigate onWeeks ...>` | The person-by-day view: a row for each person, one for the entries nobody has ("Unassigned") and one for entries marked `team: true` such as holidays; a column for each day (`days` is 7, 14 or 28, from the Sunday `from`). Entries are bars in lanes with their travel as bands. Same callbacks as the calendar: `onSelect`, `onMove`, `onResize`, `onCreateRange` (a click on an empty day of a row), `outsideItem` and `onDropFromOutside`. A bar can be dragged to another day or person (a drop on "Unassigned" gives `resourceId: null`; while something is dragged the bars let the drop through to the day under them), its first or last day can be dragged by the edge of the bar (`onResize`, never past the other edge; no edge on a cut side), or it is moved and resized with the arrow keys as in the calendar. Where a bar sits comes from CSS classes, not inline styles. |
| Week and day views | The all-day rows hold the entries, and under them is the **hour grid**, 0:00 to 24:00. `workHours` (`{ start, end }`, default 7 and 17) is the working day: its slots have the class `pp-sc-work` (the page colour, a line at its start and end, bold hours) and the rest `pp-sc-off` (shaded). The grid opens scrolled one hour before work starts, and a line under the toolbar says what the shading means. In the day view with people each person has a column with the hours. |
| Motion | `scheduler-motion.css`: bars lift on hover, a pending bar settles in with a glow, a selected bar pulses once, the cell or day a drop lands on lights up, a view fades in when it is switched. Meaning is never carried by movement alone, and with `prefers-reduced-motion` everything is switched off. |
| `<Toolbar label view onPrevious onNext onToday onViewChange>` | The calendar's own toolbar (Previous, Next, Today, Month/Week/Day) for a page that draws the same entries another way, such as a list. `step(day, view, direction)` and `rangeLabel(day, view, locale)` give the next day and the title. |
| `<BacklogList label items onDragStart onDragEnd>` | A list of items (`{ id, title, days }`) that can be dragged onto the calendar. Dragging has no keyboard alternative yet: give the page a button that does the same. |
| `<Legend items>` | What each colour stands for (`{ colour, label }`), plus the pending, travel and flag cues. |

The day helpers the page needs are exported with it: `addDays`, `daysBetween`, `weekday`, `weekStart`, `monthStart`, `monthEnd`, `rangeOf` and `daysLabel`; they work on the day strings and never on a time zone.

A `SchedulerEvent` has `id`, `title`, `start` and `end` (both inclusive), `resourceIds` (empty
when unassigned), `kind`, `status`, `colour` (`c1` to `c6`, an Outlook category colour such as `blue` or `dark-red` from scheduler-outlook.css, or `neutral`), `travelBefore`,
`travelAfter`, `editable`, `pending` and `flags`. A pending entry is drawn with a dashed edge, a
pattern and the word "Pending"; travel (`travelBefore`, `travelAfter`) is drawn as lighter, dashed-edged ends of the same bar as the visit, saying "Travel", on one track per day so the edges fall on the day lines; `colourTo` (an Outlook colour) makes the bar a gradient from `colour` to `colourTo`; a flag is a "!"
with the flag names as text. Meaning is never carried by colour alone.

- **Limits.** The week view with people shown separately stacks a band per person (no hour grid: 21 narrow columns would not read); the person-by-day view is `TeamGrid`.
  An entry with no person is not drawn in a view split by person.
- **A crowded month day.** Where the entries of a day do not fit, the cell shows two buttons instead of "+N more": "Details (N)" opens a short list of only the entries that are not drawn (a click selects one; Escape or Close shuts it), and "Expand" makes the whole month as tall as its entries need, easing the height and showing a "Collapse" button above the grid. The frame is the body, so the border stays around the month while it grows. A printed month is always expanded. Week, day and `TeamGrid` have no such limit: they grow with their entries.
- **Do not mount it in a hidden tab panel.** The library measures zero height there and shows no
  "+N more" until the window is resized; `Tabs` keeps its panels mounted but hidden.
- **Size.** It adds about 256 KB minified (79 KB gzipped) to a page that draws it; other pages
  are unchanged. `react-big-calendar` is pinned in `package.json`; a new version can change the
  class names the stylesheet replaces.
- **Tests.** jsdom has no layout, so the library's mouse selection cannot run: tests stub it, and
  drag, resize and drop are checked in the browser.
