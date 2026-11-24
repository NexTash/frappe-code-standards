---
name: frappe-debug
description: Debug Frappe/ERPNext errors. Use when there is a traceback, error message, HTTP 403/404/417, PermissionError, ValidationError, ImportError, or a change that does not show up.
---

# Error -> Cause -> Fix

Check **Error Log** (desk) or `bench --site <site> console` / `logs/` first.
Read the traceback bottom-up: the last frame is where it failed. For the
workflow (reproduce, isolate, confirm) and worked fixes with code, read
`references/common-errors.md`.

| Error | Cause | Fix |
|---|---|---|
| `ImportError: __import__ not found` | `import` in a Server Script | Use `frappe.utils.*`, see the frappe-server-scripts skill |
| `SyntaxError: raise/try not allowed` | sandbox | `frappe.throw()` / `if` checks |
| 403 `PermissionError` | missing `@frappe.whitelist()`, role, or `allow_guest` | add decorator / role permission / User Permission |
| `CSRFTokenError` | raw fetch / stale session | use `frappe.call()` |
| 401 `AuthenticationError` | bad token | header `Authorization: token key:secret` |
| 404 `DoesNotExistError` | `get_doc` on missing record, wrong path | `frappe.db.exists()` first, verify `/api/method/dotted.path` |
| 409 `DuplicateEntryError` | unique / naming clash | check exists, fix `autoname` / naming series |
| 417 `ValidationError` | `frappe.throw` ran | fix the input or the rule |
| `MandatoryError` | required field empty | set before insert/save |
| `LinkValidationError` | Link points to missing record | create or validate the target |
| `LinkExistsError` | delete blocked by linked docs | show links, cancel/delete dependents |
| `CharacterLengthExceededError` | value too long | truncate / increase length |
| `TimestampMismatchError` | doc changed since load | reload (`doc.reload()` / `frm.reload_doc()`) |
| `AttributeError: module has no attribute` | wrong dotted path in hooks.py | match function name and path exactly |
| Override does nothing / breaks | missing `super()` call | call `super().method()` |
| `QueryDeadlockError` | competing transactions | smaller transactions, retry |
| `QueryTimeoutError` | slow query | index, paginate, narrow fields |
| `InternalError` (gone away / too many) | DB connection | `frappe.db.connect()`, check `max_connections` |
| Change not visible | cache / build | `clear-cache`, `bench build --app <app>`, `migrate` |
| Fixture not loading | missing `fixtures` filter or export | `bench --site <site> export-fixtures`, check hooks.py filter |
| Job never runs | worker / scheduler off | `bench doctor`, `bench --site <site> enable-scheduler` |
