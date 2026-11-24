# Query & Index Guide

## Read EXPLAIN first

```sql
bench --site <site> mariadb
MariaDB> EXPLAIN SELECT name FROM `tabSales Invoice`
         WHERE posting_date >= '2026-01-01' AND docstatus = 1;
```

- `type: ALL` = full table scan. `key: NULL` = no index used. `rows` = rows
  MariaDB thinks it must read.
- Fix the query or the index, not the hardware. Re-run EXPLAIN after every
  change.

## Adding an index

- Single-column index: set `search_index: 1` on the field in the DocType
  JSON. For Custom Fields on another app's DocType, set it in the Custom Field.
- Composite index: only via a patch — `frappe.db.add_index("DocType",
  ["field_a", "field_b"])`. DocType JSON has no composite index option.
  Order matters: equality columns first, range column last.
- Index the columns in your WHERE/JOIN, not the columns in your SELECT.
- Don't index everything: every index slows inserts and updates. Index
  foreign keys (Link fields) and the 2–3 filters a report actually uses.

## N+1 fixes

- Symptom: one query per row in a loop — visible as slow list views and
  slow reports.
- Fix: fetch once, build a dict, then loop:

```python
items = frappe.get_all("Item", fields=["name", "item_name"], filters={...})
prices = {
    p.item_code: p.price_list_rate
    for p in frappe.get_all(
        "Item Price",
        fields=["item_code", "price_list_rate"],
        filters={"item_code": ["in", [i.name for i in items]]},
    )
}
```

- `frappe.get_all` with `["in", [...]]` beats one `get_value` per row.
- Child tables: `doc.get("items")` is already loaded — never re-query a
  child table row by row.

## Caching

- `frappe.get_cached_doc("DocType", name)` / `frappe.get_cached_value` for
  single fields — use for masters (Company, Item, Customer) read in loops.
- After a deploy that changes DocTypes, run `frappe.clear_cache()` (or
  `bench --site <site> clear-cache`).
- `frappe.cache()` (Redis) for computed values with a TTL; always include
  the site/user in the key when the value differs per user. Never cache
  user-specific data under a shared key.

## frappe.qb vs get_list

- `frappe.get_list` / `frappe.get_all`: simple filters. `get_list` is
  permission-aware; `get_all` is not. Use for ordinary reads.
- `frappe.qb`: joins, group by, aggregates, case expressions. Use when
  `get_all` can't express the query.
- Raw `frappe.db.sql`: only parameterised, only when qb can't do it.
  Never f-string SQL.
- Paginate: `limit_page_length` / `limit_start` on get_list; `LIMIT ...
  OFFSET ...` in qb. Never fetch all rows and slice in Python.

## Checklist

- [ ] EXPLAIN shows an index used (`key` set, `rows` small).
- [ ] No query inside a loop.
- [ ] `get_list` for permission-checked reads; `get_all` only when
      permissions are checked elsewhere.
- [ ] Paginated, not "fetch all then slice".
