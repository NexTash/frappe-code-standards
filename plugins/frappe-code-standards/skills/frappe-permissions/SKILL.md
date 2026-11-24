---
name: frappe-permissions
description: Frappe/ERPNext permissions - roles, User Permissions, has_permission hooks, permission queries and whitelisted-method checks. Use when a user cannot see, open, edit or submit a document, when an API call returns 403, or when designing who may do what.
---

# Frappe Permissions

Access is granted only when **every** layer allows it: role permission,
User Permissions, document sharing, and any `has_permission` hook.

## 1. Role permissions (DocType level)

- Set them in **Role Permissions Manager**, or ship them as fixtures from the app.
- Per role, per DocType: read, write, create, delete, submit, cancel, amend,
  print, email, export, report, import, `set_user_permissions`.
- A role with no read permission cannot open the DocType at all — sharing one
  document does not fix that.
- Check in code with `frappe.has_permission("Sales Order", "read", doc, throw=True)`.

## 2. User Permissions (which documents)

- A User Permission restricts a user to specific values, e.g. one Company or
  Branch: `frappe.permissions.add_user_permission("Company", "My Co", user)`.
- It filters lists, links and reports. A user who sees an empty list usually
  has a User Permission hiding the documents.
- `frappe.get_list` and `frappe.get_all` apply User Permissions automatically.
  `frappe.db.sql` bypasses **all** permission checks — never use it for
  user-facing reads.

## 3. `has_permission` and `permission_query_conditions`

- `has_permission` in hooks.py maps a DocType to a function returning
  True/False for one document:

```python
# hooks.py
has_permission = {"Sales Order": "my_app.overrides.sales_order.has_permission"}
```

- On v16 the function **must return an explicit `True`**; returning `None`
  denies access. On v14/v15, `None` meant "no opinion".
- `permission_query_conditions` adds a SQL condition to list queries:

```python
def get_permission_query_conditions(user):
    if "Sales Manager" not in frappe.get_roles(user):
        return "`tabSales Order`.owner = {0}".format(frappe.db.escape(user))
```

- Build conditions with `frappe.db.escape`, never with f-strings.

## 4. Whitelisted methods

```python
@frappe.whitelist()
def get_order_total(name):
    doc = frappe.get_doc("Sales Order", name)
    frappe.has_permission("Sales Order", "read", doc, throw=True)
    return doc.grand_total
```

- Check permission **before** returning data, not after.
- A method without `@frappe.whitelist()` cannot be called from the form or
  the API at all (403). Internal Python calls do not need the decorator.
- Never use `allow_guest=True` on anything that writes. Guest endpoints are
  read-only, rate-limited, and must validate every input.

## 5. Sharing

- Share one document: `frappe.share.add("Sales Order", name, user, read=1)`;
  remove it with `frappe.share.remove("Sales Order", name, user)`.
- Sharing grants access to that one document only. It does **not** replace
  the role permission — the user still needs read on the DocType.
- Sharing is for exceptions. Routine access belongs in roles.

## 6. Desk vs API

- Desk and `/api/resource/...` run the same checks. If it works in Desk but
  fails from the API, the API user (token owner) lacks the role.
- `frappe.get_list` is permission-aware; `frappe.get_all` is too, unless
  `ignore_permissions=True` is passed in trusted server code.

## 403? Check in this order

1. Does the user's role have read/write on the DocType?
2. Is a User Permission hiding the document (Company, Branch, Territory)?
3. Does a `has_permission` hook return `None` (v16) or False?
4. Is the whitelisted method missing the decorator or the permission check?
5. For the API: does the token's user have the role?
