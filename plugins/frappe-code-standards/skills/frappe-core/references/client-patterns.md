# Client-side Patterns (JS)

## Form script skeleton (our own DocType: `<name>.js`)

```js
frappe.ui.form.on("Loan Application", {
    setup(frm) {
        // set_query belongs here (runs once per form load)
        frm.set_query("loan_type", () => ({
            filters: { company: frm.doc.company, disabled: 0 },
        }));
        frm.set_query("item_code", "items", (doc, cdt, cdn) => ({
            filters: { is_stock_item: 1 },
        }));
    },

    refresh(frm) {
        if (frm.doc.docstatus === 1 && frm.doc.status !== "Closed") {
            frm.add_custom_button(__("Close"), () => close_application(frm), __("Actions"));
        }
        frm.toggle_display("approval_section", frm.doc.status === "Pending Approval");
    },

    customer(frm) {
        if (!frm.doc.customer) return;
        frm.call("get_outstanding").then((r) => frm.set_value("outstanding", r.message));
    },
});

frappe.ui.form.on("Loan Application Item", {
    qty: calculate_row_amount,
    rate: calculate_row_amount,
    items_remove(frm) { calculate_total(frm); },
});

function calculate_row_amount(frm, cdt, cdn) {
    const row = locals[cdt][cdn];
    frappe.model.set_value(cdt, cdn, "amount", flt(row.qty) * flt(row.rate));
    calculate_total(frm);
}

function calculate_total(frm) {
    frm.set_value("total", (frm.doc.items || []).reduce((s, r) => s + flt(r.amount), 0));
}
```

## set_query rules

- Put `frm.set_query` in `setup` (or `onload`), not in `refresh` or field events.
- Child table: `frm.set_query("fieldname", "child_table_fieldname", fn)`.
- Filters as object `{ field: value }` or list `[["field", "in", [...]]]`.
- Custom server query (only when filters are not enough):

```js
frm.set_query("supplier", () => ({
    query: "my_app.my_module.queries.supplier_query",
    filters: { company: frm.doc.company },
}));
```

```python
@frappe.whitelist()
@frappe.validate_and_sanitize_search_inputs
def supplier_query(doctype, txt, searchfield, start, page_len, filters):
    ...  # use frappe.qb / get_list, respect permissions
```

- Simple fetches (copy a field from a linked doc) → set **Fetch From** on the field in the DocType,
  not JS.
- Static field visibility/mandatory → **Depends On / Mandatory Depends On / Read Only Depends On**
  in DocType, not JS.

## Server calls

- Method on the current doc's controller → `frm.call("method_name", { args })`.
- Module-level whitelisted function → `frappe.call({ method: "app.module.file.fn", args, freeze: true })`.
- Use promises (`.then`) or `async/await`; never synchronous `async: false`.
- Don't save via `frappe.call` + `frappe.client.set_value` for the open form — use `frm.set_value` then `frm.save()`.

## Messages (JS)

```js
frappe.msgprint(__("Please select a customer first"));
frappe.throw(__("Qty must be positive"));            // stops the current JS flow
frappe.show_alert({ message: __("Saved"), indicator: "green" }, 5);
frappe.confirm(__("Close this application?"), () => close_application(frm));
```

- All strings wrapped in `__()`, placeholders: `__("Row {0}: missing rate", [row.idx])`.
- No `alert()`, no `console.log` left in committed code.

## Dialogs

```js
const d = new frappe.ui.Dialog({
    title: __("Set Due Date"),
    fields: [{ fieldname: "due_date", fieldtype: "Date", label: __("Due Date"), reqd: 1 }],
    primary_action_label: __("Update"),
    primary_action(values) {
        frm.call("set_due_date", values).then(() => { d.hide(); frm.reload_doc(); });
    },
});
d.show();
```

## Don'ts

- No direct DOM manipulation (`$(...)` on form fields) when a `frm.*` API exists.
- Don't duplicate server validation as the only check in JS.
- Don't use `cur_frm`; use the `frm` argument.
- Don't create a second `frappe.ui.form.on` for the same DocType in the same file.
