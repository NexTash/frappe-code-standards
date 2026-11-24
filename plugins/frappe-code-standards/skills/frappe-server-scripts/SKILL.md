---
name: frappe-server-scripts
description: Frappe/ERPNext Server Scripts (Setup > Server Script, RestrictedPython sandbox). Use when writing or fixing a Server Script: Document Event, API, Scheduler or Permission Query. Covers no-import rule, sandbox limits, event name mapping.
---

# Server Scripts (Setup > Server Script)

Run in a RestrictedPython sandbox. Prefer a custom app controller whenever one exists.

## Rule 1: NO import statements

Every `import` fails, even `import frappe`. Use the pre-loaded names:

| Blocked | Use |
|---|---|
| `import json` | `frappe.parse_json()` / `frappe.as_json()` (`json` is also pre-loaded) |
| `from datetime import ...` | `frappe.utils.today()`, `now_datetime()`, `add_days()`, `date_diff()` |
| `from frappe.utils import flt` | `frappe.utils.flt()` |
| `import requests` | `frappe.make_get_request()` / `make_post_request()` |
| `import re`, `os`, `sys` | not available, so move the logic to a custom app |

## Other sandbox limits

- `raise` blocked, so use `frappe.throw()`.
- `try/except` blocked on v14-v15, so use `if` checks such as `frappe.db.exists()`.
- Some builtins are missing, so use `{}` / `frappe._dict()`.
- `frappe.db.commit()` only in Scheduler scripts.
- v15+: disabled by default, enable with `bench set-config -g server_script_enabled 1`.
- Not available on Frappe Cloud shared benches.

## Types

| Type | Trigger | Key variable |
|---|---|---|
| Document Event | doc lifecycle | `doc` |
| API | `/api/method/<name>` | `frappe.form_dict`, set `frappe.response["message"]` |
| Scheduler Event | cron | none |
| Permission Query | list filtering | `user`, set `conditions` |

## Event name trap

| UI label | Hook | | UI label | Hook |
|---|---|---|---|---|
| Before Save | **`validate`** | | After Save | `on_update` |
| Before Insert | `before_insert` | | After Insert | `after_insert` |
| Before Submit | `before_submit` | | After Submit | `on_submit` |
| Before Cancel | `before_cancel` | | After Cancel | `on_cancel` |
| Before Delete | `on_trash` | | After Delete | `after_delete` |

"Before Save" is `validate`, not `before_save`.

## Example

```python
# Document Event: Sales Order, Before Save
if doc.delivery_date and frappe.utils.date_diff(doc.delivery_date, frappe.utils.today()) < 0:
    frappe.throw(_("Delivery Date cannot be in the past"))
```

Over 50 lines, external libraries, or tests needed? Move it to a custom app.
