---
name: frappe-website
description: Frappe website and portal - Web Pages, Web Forms, www/ pages, portal pages for customers/suppliers, website routes, guest access and website security. Use when building anything a website visitor or portal user sees.
---

# Frappe Website / Portal

## Choose
| Need | Use |
|---|---|
| Static/marketing page editable by users | **Web Page** DocType |
| Public or portal form that creates/edits a doc | **Web Form** (ship as part of the app module) |
| Custom route with Python context | `<app>/www/<route>.html` + `<route>.py` with `get_context(context)` |
| Page per document (e.g. /projects/abc) | DocType with "Has Web View" + `route` + `templates/generators` |
| Portal menu item | `portal_menu_items` in hooks.py, or Portal Settings |

## www page pattern
```python
# myapp/www/orders.py
import frappe
no_cache = 1
def get_context(context):
    if frappe.session.user == "Guest":
        frappe.throw(_("Please login"), frappe.PermissionError)
    context.orders = frappe.get_list("Sales Order", fields=["name", "status"], limit=20)
```
`get_list` applies the portal user's permissions. Never use `get_all` for user-scoped data.

## Security
- Keep CSRF on. Never `ignore_csrf` in production.
- Guest endpoints: `allow_guest=True` only when needed, read-only where possible, rate-limited.
- Never trust `frappe.form_dict`. Validate it and escape output (Jinja autoescapes, so don't use `| safe` on user data).
- Use `has_website_permission` hooks for doc-level portal access.
- Don't expose internal DocType names or private files in guest URLs.

## Assets
Put CSS/JS in `<app>/public/`, include them via `web_include_css/js` in hooks.py, and run `bench build --app <app>`.
After route changes: `bench --site <site> clear-website-cache`.
