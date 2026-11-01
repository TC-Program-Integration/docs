# flowkit layout

> **Reference.** Where each helper of `flows/lib/flowkit/` lives, for when you add or change one. Using them:
> [flowkit](flowkit.md) and [flowkit options](flowkit-options.md). A flow file imports everything from
> `lib.flowkit`; the packages below are only how the code is organized.

| Package | Holds |
|---|---|
| `types.py` | `Json`, `Action`, `Actions`, `RunAfter` |
| `actions/` | built-in actions, no connector knowledge: `ordering` (`ok`, `failed`, `when`, `chain`), `variables`, `control` (`foreach`, `condition`, `switch`, `scope`, `try_catch`, `parallel`, `delay`...), `data` (`compose`, `query`, `select`, `parse_json`, `table`), `web` (`http`, `response`, `respond_to_powerapp`), `errors` (`catch`, `terminate_*`), `options` (`with_options`), `conditions` (`and_`, `equals`..., `odata_quote`) |
| `connectors/` | one module per connector, with its actions and its triggers: `base` (`connector`, `CONNECTORS`), `dataverse`, `sharepoint`, `outlook`, `teams`, `users`; and `choices` (the choice-value lookup, which reads stringmaps) |
| `triggers/` | built-in triggers (`recurrence`, `manual_trigger`, `http_request_trigger`) and `trigger_options` |
| `check/` | `check()` (`checker`), its rules (`rules`) and the known expression functions (`functions`) |
| `build/` | `Flow`, `Solution` (the zip), `EnvironmentVariable` (`environment`), `load_flow`, and the XML templates' helpers |

The tests are in `flows/tests/`, in folders with the same names (`actions/`, `connectors/`, `triggers/`, `check/`, `build/`) and shared helpers in `tests/helpers.py`; `pnpm test:flows` runs them.

A new helper goes in the module of its kind (a new connector gets its own module in `connectors/`), is listed
in that package's `__init__.py` and in `flowkit/__init__.py`, and gets a test and a row in the
[flowkit reference](flowkit.md). Packages only import from the ones above them in this table, which keeps the
imports free of cycles.