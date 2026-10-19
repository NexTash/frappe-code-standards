# Reports and Printing

## Choosing the report type

- **Report Builder** (no code) → simple filtered lists of one DocType. Prefer it when it is enough.
- **Query Report** → a single SQL query, no Python logic.
- **Script Report** → anything with logic, multiple sources, or computed columns. Default choice for custom reports.
- Reports belong in our app as standard reports (`Is Standard = Yes`, module set) so they are versioned.

## Script Report structure

```
<module>/report/<report_snake>/
├── <report_snake>.json
├── <report_snake>.py
└── <report_snake>.js
```

```python
import frappe
from frappe import _


def execute(filters=None):
    filters = frappe._dict(filters or {})
    validate_filters(filters)
    return get_columns(), get_data(filters)


def validate_filters(filters):
    if filters.from_date > filters.to_date:
        frappe.throw(_("From Date cannot be after To Date"))


def get_columns():
    return [
        {"label": _("Customer"), "fieldname": "customer", "fieldtype": "Link", "options": "Customer", "width": 180},
        {"label": _("Total"), "fieldname": "total", "fieldtype": "Currency", "width": 120},
    ]


def get_data(filters):
    SI = frappe.qb.DocType("Sales Invoice")
    ...  # query builder, filters applied in the query, not in Python
```

- Columns as dicts with proper `fieldtype` (Currency, Link, Date) — not `"Customer:Link/Customer:180"` strings in new code.
- Filters defined in the report's `.js` (`frappe.query_reports["Report Name"] = { filters: [...] }`),
  with defaults and `reqd` where needed.
- Respect permissions (`frappe.get_list` or filter by allowed companies).

## Print formats

- Prefer the Print Format Builder / Jinja print formats stored in our app (standard, module set).
- Jinja: use `doc.get_formatted("field")` for currency/date, `_()` for labels,
  `frappe.db.get_value` sparingly (no queries in loops).
- Shared Jinja helpers → `jinja = {"methods": ["my_app.utils.jinja.helper"]}` in hooks.py.
