# Server-side Patterns (Python)

## Controller skeleton

```python
import frappe
from frappe import _
from frappe.model.document import Document
from frappe.utils import flt, cint, getdate, nowdate, get_link_to_form


class LoanApplication(Document):
    def validate(self):
        self.validate_dates()
        self.validate_items()
        self.set_totals()

    def before_submit(self):
        self.check_credit_limit()

    def on_submit(self):
        self.create_ledger_entries()

    def on_cancel(self):
        self.reverse_ledger_entries()

    # --- validations -------------------------------------------------
    def validate_dates(self):
        if self.to_date and getdate(self.to_date) < getdate(self.from_date):
            frappe.throw(_("To Date cannot be before From Date"))

    def validate_items(self):
        for row in self.items:
            if flt(row.qty) <= 0:
                frappe.throw(_("Row {0}: Qty must be greater than 0").format(row.idx))

    def set_totals(self):
        self.total = sum(flt(row.amount) for row in self.items)
```

Rules:
- One small, named method per concern; `validate()` reads like a table of contents.
- Use the right hook: `before_insert`, `validate`, `before_save`, `on_update`, `before_submit`,
  `on_submit`, `before_cancel`, `on_cancel`, `on_trash`. Don't do submit-only work in `validate`.
- Inside `validate`/`before_save`, set fields on `self` directly; never call `self.save()`.
- In `on_update`/`on_submit`, if you must change own fields use `self.db_set("field", value)`.
- Use `frappe.utils` helpers (`flt`, `cint`, `getdate`, `nowdate`, `add_days`, `date_diff`) —
  never `float()`/`int()` on doc fields (they may be `None`).

## Messages and errors

```python
frappe.throw(_("Customer {0} is disabled").format(frappe.bold(self.customer)))
frappe.throw(_("Not permitted"), frappe.PermissionError)
frappe.throw(msg, title=_("Missing Data"))

frappe.msgprint(_("{0} items updated").format(count), alert=True)   # toast
frappe.msgprint(_("Check these rows: {0}").format(", ".join(rows)), title=_("Warning"), indicator="orange")
```

- `frappe.throw` → validation failure, stops and rolls back. Use specific exception classes
  (`frappe.ValidationError`, `frappe.PermissionError`, `frappe.DoesNotExistError`,
  `frappe.DuplicateEntryError`) when callers may catch them.
- Never `raise Exception("...")`, never `print()`, never bare `except:`.
- Catch only what you expect, then either re-throw with a clear message or log:

```python
try:
    response = make_request(...)
except requests.exceptions.RequestException:
    frappe.log_error(title=_("Payment Gateway Error"), message=frappe.get_traceback())
    frappe.throw(_("Could not reach the payment gateway. Please try again."))
```

- Logging for developers: `frappe.logger("my_app").info(...)`; errors visible in Error Log:
  `frappe.log_error(...)`.

## Whitelisted methods

```python
# On our own DocType controller — called from JS with frm.call("get_outstanding")
class LoanApplication(Document):
    @frappe.whitelist()
    def get_outstanding(self):
        return frappe.db.get_value("Loan", self.loan, "outstanding_amount")
```

```python
# Module-level API — called with frappe.call({ method: "my_app.api.get_rates", args })
@frappe.whitelist()
def get_rates(item_code: str, price_list: str):
    frappe.has_permission("Item Price", "read", throw=True)
    return frappe.get_all(
        "Item Price",
        filters={"item_code": item_code, "price_list": price_list},
        fields=["price_list_rate", "currency"],
    )
```

- Every whitelisted function is a public endpoint: check permissions
  (`frappe.has_permission`, `doc.check_permission("write")`) or use `frappe.get_list`.
- Use type hints on arguments (Frappe validates them in recent versions).
- `allow_guest=True` only with explicit reason and input validation.
- Use `methods=["POST"]` for anything that writes.

## Documents and DB

```python
doc = frappe.get_doc("Customer", name)                      # full doc, needed to modify/save
doc = frappe.new_doc("ToDo"); doc.update({...}); doc.insert()
value = frappe.db.get_value("Customer", name, "customer_group")
row = frappe.db.get_value("Customer", name, ["customer_group", "territory"], as_dict=True)
setting = frappe.db.get_single_value("Selling Settings", "cust_master_name")
cached = frappe.get_cached_value("Company", company, "default_currency")
exists = frappe.db.exists("Customer", {"tax_id": tax_id})
frappe.db.set_value("Task", name, "status", "Completed")   # skips validate — use deliberately
```

- `doc.insert(ignore_permissions=True)` only for system-created records, with a comment why.
- `frappe.db.set_value` bypasses controller logic; use `doc.save()` when validations must run.
- Query Builder for joins/aggregates:

```python
from frappe.query_builder.functions import Sum

SI = frappe.qb.DocType("Sales Invoice")
rows = (
    frappe.qb.from_(SI)
    .select(SI.customer, Sum(SI.grand_total).as_("total"))
    .where((SI.docstatus == 1) & (SI.posting_date[from_date:to_date]))
    .groupby(SI.customer)
).run(as_dict=True)
```

- Raw SQL only if QB can't express it, always with `%(name)s` parameters — never f-strings.

## Naming

- Prefer `autoname` in DocType settings (naming series, `field:`, `format:`). Use `autoname(self)`
  in the controller only for logic the settings can't express.
