---
name: frappe-upgrade
description: Frappe/ERPNext version differences and upgrades v14 to v15 to v16. Use when code must work on a specific version, when migrating or upgrading, or when replacing removed or deprecated APIs.
---

# Version Changes (v14 -> v15 -> v16)

Upgrade one major version at a time. Check `__version__` before using anything below.

## v14 -> v15

| Old / removed | Use |
|---|---|
| `frappe.db.set()` | `doc.db_set()` |
| `frappe.db.update()` | `frappe.db.set_value()` |
| `frappe.db.set_value()` on Singles | `frappe.db.set_single_value()` |
| `frappe.db.set_temp/get_temp` | `frappe.cache` |
| `frappe.compare()` | `from frappe.utils import compare` |
| `enqueue(job_name=...)` | `job_id=` + `is_job_enqueued()` |
| `frappe.new_doc(dt, parent_doc, ...)` positional | keyword args only |
| `convert_utc_to_user_timezone()` | `convert_utc_to_system_timezone()` |
| `get_time_zone()` | `get_system_timezone()` |
| JS `window.get_today` / `show_alert` / `user` / `roles` | `frappe.datetime.get_today`, `frappe.show_alert`, `frappe.session.user`, `frappe.user_roles` |
| `this` in Client Scripts | `frm` / `cur_frm` |

Also: Server Scripts are disabled by default, Vue 3, Node 18+, `pyproject.toml`, and the
last method override wins. Event Streaming moved to its own app.

## v15 -> v16

| Change | Action |
|---|---|
| `extend_doctype_class` hook added | Prefer it over `override_doctype_class`. Prefix methods with the app name. |
| `override_doctype_class` | Class must inherit the original |
| `has_permission` hooks | Must return explicit `True` (not `None`) |
| DB commit inside document hooks | Not allowed |
| Default list sort | `creation` (was `modified`), so pass `order_by` explicitly |
| `frappe.get_doc(dt, name, field=val)` | No longer sets values |
| `frappe.sendmail(now=True)` | No longer commits |
| `db.get_value` on Singles | Returns typed values, not strings |
| `frappe.flags.in_test` | `frappe.in_test` |
| State-changing API methods | POST only |
| Report/Page JS | Runs as IIFE with no globals |
| PDF | Chrome renderer (wkhtmltopdf deprecated) |
| Data masking (field `mask`) | Not applied to raw SQL, so mask manually |
| Translation `get_lang_dict`, `get_translated_dict` hook | Removed |
| Blog, Newsletter, Energy Points, Offsite backups | Separate apps |
| `/app` URL | `/desk` |
| Runtime | Python 3.14+, Node 24+ |
| Studio, Flow, Insights dashboards | New in v16.60 — see **frappe-studio**, **frappe-flow**, **frappe-insights** |

Details, runtime requirements, and new v16 / v16.60 features:
`references/upgrade-v15-v16.md`.

## References

| Task | File |
|---|---|
| v15 -> v16 breaking changes, runtime, new APIs and features | `references/upgrade-v15-v16.md` |
