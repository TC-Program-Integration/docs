# Flow expressions

> **Reference.** The functions you use most inside flow actions, with examples, and the traps around them.
> The complete list is Microsoft's
> [workflow expression functions reference](https://learn.microsoft.com/azure/logic-apps/workflow-definition-language-functions-reference).

## Writing them in Python

An action's value is a text. It is an **expression** when it starts with `@`, and it can hold expressions
inside text with `@{ ... }`:

| You write | Result |
|---|---|
| `"@triggerBody()?['Name']"` | the value itself (a number stays a number) |
| `"Hello @{triggerBody()?['Name']}"` | text with the value pasted in |
| `"@@"` | a literal `@` |

In a Python **f-string** the braces must be doubled: `f"Task @{{items('Loop')?['subject']}} is due"`. If you
build an expression from parts, keep the parts in variables (`task = "items('For_each_task')"`) and
`pnpm flows:build` will still check the names.

## Reading values

| Expression | Gives |
|---|---|
| `triggerOutputs()?['body/<column>']` | a column of the row that started a Dataverse flow |
| `triggerBody()?['<name>']` | an input of a manual or HTTP flow |
| `outputs('<Action>')?['body/<column>']` | a column of the row an action returned |
| `outputs('<List>')?['body/value']` | the array of rows a list returned |
| `body('<Action>')` | the body of an action's answer |
| `items('<Loop>')` | the current element of that loop (works in nested loops) |
| `item()` | the current element inside `query`, `select` and `table` |
| `variables('<Name>')` | a variable |
| `result('<Scope>')` | the results of every action in a scope, for error handling |
| `actions('<Action>')?['status']` | `Succeeded`, `Failed`, `Skipped`... |

`?[...]` reads a property **safely**: if anything on the way is empty the result is empty, not an error.
Always use it on data that can be missing; use `[...]` without the question mark only for what must exist.

## Empty and missing values

| Expression | Use |
|---|---|
| `empty(x)` | true for empty text, an empty array, an empty object, or nothing |
| `coalesce(a, b, c)` | the first of them that is not empty |
| `if(test, yes, no)` | one of two values |
| the literal `null` | an empty value to **write**. There is **no** `null()` function: flowkit's check refuses it. |

```python
"@coalesce(triggerOutputs()?['body/pidev_phone'], 'no phone')"
"@if(empty(triggerOutputs()?['body/pidev_notes']), 'none', triggerOutputs()?['body/pidev_notes'])"
```

## Comparing

`equals(a, b)`, `not(x)`, `and(a, b)`, `or(a, b)`, `greater`, `greaterOrEquals`, `less`, `lessOrEquals`,
`contains(collection, value)`, `startsWith`, `endsWith`. **Types matter:** `equals('2', 2)` is false. A Choice
column is a number; a lookup is text.

In a `condition(...)` the same comparisons are written as a dictionary:
`{"and": [{"greater": ["@length(variables('Problems'))", 0]}]}`.

## Text

| Function | Example | Gives |
|---|---|---|
| `concat(a, b)` | `concat('Hello ', 'Ana')` | `Hello Ana` |
| `substring(text, start, length)` | `substring('Callback', 0, 4)` | `Call` |
| `replace(text, old, new)` | `replace('a-b', '-', ' ')` | `a b` |
| `toLower` / `toUpper` / `trim` | `toUpper('ab')` | `AB` |
| `split(text, sep)` / `join(array, sep)` | `join(variables('Names'), ', ')` | `a, b, c` |
| `indexOf(text, find)` | | position, `-1` if absent |
| `length(text)` | | number of characters |
| `string(x)` | | any value as text |

## Collections

| Function | Gives |
|---|---|
| `length(array)` | number of elements |
| `first(array)` / `last(array)` | the first or last element |
| `take(array, n)` / `skip(array, n)` | the first n / all but the first n |
| `union(a, b)` / `intersection(a, b)` | combined / common elements |
| `contains(array, value)` | true if present |
| `createArray(a, b)` | a new array |
| `range(start, count)` | `[start, start+1, ...]` |

## Numbers

`add`, `sub`, `mul`, `div`, `mod`, `min`, `max`, `int(x)`, `float(x)`, `rand(min, max)`. `div(7, 2)` is `3` when both are
integers: convert with `float` first to keep the fraction.

## Dates and times

| Function | Example | Gives |
|---|---|---|
| `utcNow()` | | the current time, UTC, as ISO text |
| `addDays(ts, n)` | `addDays(utcNow(), -7)` | n days later (negative: earlier); also `addHours`, `addMinutes` |
| `startOfDay(ts)` / `startOfMonth(ts)` | | midnight / the first day |
| `formatDateTime(ts, 'yyyy-MM-dd')` | | `2026-10-12` |
| `convertTimeZone(ts, 'UTC', 'Romance Standard Time')` | | the same moment in another zone |
| `dayOfWeek(ts)` | | 0 (Sunday) to 6 |

- Dataverse stores **times in UTC**. Show them to people in their zone with `convertTimeZone`.
- A **date-only** column has no time or zone: format it with `formatDateTime(x, 'yyyy-MM-dd')` and **never**
  convert it (it would shift by a day).
- In an OData `$filter` a date is the ISO text **without quotes**: `scheduledend lt @{utcNow()}`.
- Time zone names are Windows names (`Pacific Standard Time`), not `America/Los_Angeles`.

## OData filters (`$filter`)

| Need | Write |
|---|---|
| a choice | `prioritycode eq 2` |
| a text | `subject eq 'Welcome call'` (a quote inside it is doubled: `odata_quote(...)`) |
| a lookup | `_ownerid_value eq <guid>` (no quotes) |
| active rows | `statecode eq 0` |
| several | `a eq 1 and (b eq 2 or c eq 3)` |
| starts with | `startswith(subject, 'Call')` |
| contains | `contains(subject, 'urgent')` |
| empty | `pidev_phone eq null` |

## Traps

| Symptom | Cause |
|---|---|
| "The template language function 'xyz' is not valid" | a typo, or a function that does not exist (`null()`) |
| The result is the text `@{...}` itself | the value was not an expression: `@{}` works only inside text, and `@` must be the first character |
| An f-string loses its braces or fails | doubled braces `{{ }}` are needed inside f-strings |
| `equals` is false for equal-looking values | different types: `'2'` and `2` |
| A date is off by one day | a date-only column was converted between time zones |
| The run fails on an empty lookup | the path was read with `[...]` instead of `?[...]` |
| A `Terminate` message fails to evaluate | it used `variables()`: put the text in a `compose` first (`terminate_with_list` does) |
