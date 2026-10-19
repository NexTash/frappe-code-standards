# hooks.py, Customization, Fixtures, Patches

## hooks.py — keep it a registry, not a code file

```python
doctype_js = {"Sales Invoice": "public/js/sales_invoice.js"}          # other apps' DocTypes only
doctype_list_js = {"Sales Invoice": "public/js/sales_invoice_list.js"}

doc_events = {
    "Sales Invoice": {
        "validate": "my_app.selling.overrides.sales_invoice.validate",
        "on_submit": "my_app.selling.overrides.sales_invoice.on_submit",
    },
}

scheduler_events = {
    "daily": ["my_app.tasks.daily.send_overdue_reminders"],
    "cron": {"0 2 * * *": ["my_app.tasks.nightly.sync_rates"]},
}

fixtures = [
    {"dt": "Custom Field", "filters": [["module", "=", "My Module"]]},
    {"dt": "Property Setter", "filters": [["module", "=", "My Module"]]},
]
```

- No logic in hooks.py — only dotted paths.
- No `doc_events` for DocTypes our app owns (use the controller).
- Never use `"*"` doc_events unless the logic truly applies to every DocType; it runs on every save.

## Override function style

```python
# my_app/selling/overrides/sales_invoice.py
import frappe
from frappe import _


def validate(doc, method=None):
    validate_discount_limit(doc)


def validate_discount_limit(doc):
    limit = frappe.db.get_single_value("My App Settings", "max_discount")
    if doc.additional_discount_percentage > limit:
        frappe.throw(_("Discount cannot exceed {0}%").format(limit))
```

## Custom Fields

- Created on site via Customize Form with **Module = our module**, then exported:
  `bench --site <site> export-fixtures --app <app>`.
- Or created in code during install / patch with
  `from frappe.custom.doctype.custom_field.custom_field import create_custom_fields`.
- Field names on others' DocTypes: prefix with app short code (e.g. `nx_approval_status`) to avoid collisions.

## Patches (data changes)

```
# patches.txt
[pre_model_sync]
my_app.patches.v1_0.rename_old_field
[post_model_sync]
my_app.patches.v1_0.set_default_status
```

```python
# my_app/patches/v1_0/set_default_status.py
import frappe


def execute():
    frappe.db.set_value("Loan Application", {"status": ("is", "not set")}, "status", "Draft")
```

- One patch per file; idempotent (safe to run twice); no `print`.
- Patches that need new fields → `[post_model_sync]`.

## Settings

- App configuration → a Single DocType (`My App Settings`), read with
  `frappe.db.get_single_value` / `frappe.get_cached_doc`. Not hardcoded constants, not site_config,
  unless it is infra config (keys, URLs per environment).

## Permissions

- Roles and DocPerm in DocType JSON; extra rules via `has_permission` / `permission_query_conditions`
  hooks. Don't hide fields in JS as a security measure — use permlevel.
