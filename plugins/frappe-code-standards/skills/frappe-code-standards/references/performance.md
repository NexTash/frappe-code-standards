# Performance

## Queries

- Fetch only needed fields: `frappe.get_all("Item", filters=..., fields=["name", "item_group"])`.
- Never `frappe.get_doc` inside a loop to read values. Bulk fetch and map:

```python
# ❌ N+1
for row in self.items:
    group = frappe.db.get_value("Item", row.item_code, "item_group")

# ✅ one query
item_codes = [row.item_code for row in self.items]
groups = dict(frappe.get_all("Item", filters={"name": ("in", item_codes)},
                             fields=["name", "item_group"], as_list=True))
for row in self.items:
    group = groups.get(row.item_code)
```

- Aggregates in SQL/QB (`Sum`, `Count`), not by loading rows into Python.
- Add `search_index` on fields used in frequent filters (in the DocType field settings).
- Use `pluck="name"` for simple lists of names.
- `frappe.db.get_value` with `cache=True` / `frappe.get_cached_value` / `frappe.get_cached_doc`
  for rarely-changing masters (Company, settings).

## Writes

- Bulk updates: `frappe.db.set_value` with filters, or QB `update`, not doc-by-doc save — but only
  when skipping controller validation is acceptable.
- Don't `frappe.db.commit()` inside requests; the framework commits. Allowed in long background
  jobs / patches processing batches.

## Background jobs

```python
frappe.enqueue(
    "my_app.my_module.jobs.recalculate_balances",
    queue="long",
    timeout=1500,
    company=company,
    enqueue_after_commit=True,
)
```

- Anything slow (bulk processing, emails in bulk, external API calls, report generation) → enqueue.
- Notify the user on completion with `frappe.publish_realtime` or a notification.
- Jobs must handle their own errors with `frappe.log_error`.

## Client

- Avoid `frappe.call` in `refresh` for data that rarely changes; load once in `onload` or use
  `frm.doc.__onload` set by the controller's `onload()` method.
- Debounce calls triggered by typing.
