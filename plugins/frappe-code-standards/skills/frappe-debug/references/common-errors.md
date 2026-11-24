# Debug workflow

Reproduce first, guess later. Most Frappe errors are found in three places:
the **Error Log** in Desk, the traceback's **last frame**, and the site's
`logs/` folder.

## 1. Reproduce and capture

1. Reproduce the error once, then stop clicking — every retry writes the same
   traceback again and hides the first one.
2. Open **Error Log** in Desk (or `bench --site <site> console`) and copy the
   full traceback, not just the last line.
3. Read the traceback **bottom-up**: the last frame is where it failed, the
   frames above it are how it got there. The error type on the last line
   (for example `frappe.exceptions.DoesNotExistError`) decides the fix, not
   the message text.

## 2. Isolate

- Does it fail for one user, one document, or everyone? One user points to
  permissions or User Permissions; one document points to its data; everyone
  points to the code or the last deploy.
- Does it fail on `migrate` / after a `bench update`? Check the
  `frappe-upgrade` skill for a removed or changed API first.
- Change not visible at all (no error)? That is usually cache or build, not
  logic: `bench --site <site> clear-cache`, then `bench build --app <app>`,
  then `migrate`.

## 3. Worked fixes

### `DoesNotExistError` (404) — `get_doc` on a missing record

`get_doc()` raises; `get_value()` returns `None`. Check existence first:

```python
if not frappe.db.exists("Customer", customer):
    frappe.throw(_("Customer {0} does not exist").format(customer))
doc = frappe.get_doc("Customer", customer)
```

Or catch it when the record may legitimately be gone:

```python
try:
    doc = frappe.get_doc("Sales Order", name)
except frappe.DoesNotExistError:
    frappe.throw(_("Sales Order {0} was deleted").format(name))
```

### `PermissionError` (403) — user lacks access

Check in this order: Role Permission on the DocType, then User Permission
(a User Permission on Company or Branch can hide every document), then a
`has_permission` hook. On v16 a `has_permission` hook must return an
explicit `True` — returning `None` denies access. For whitelisted methods
called from the form, the method itself needs `@frappe.whitelist()` and must
check permission before returning data. A document shared with a user can
still raise `PermissionError` if the user's role has no read permission on
that DocType.

### `TimestampMismatchError` — document changed since it was loaded

Two people (or a background job) saved the same document. Reload and retry
instead of overwriting:

```python
doc.reload()
doc.set("status", "Approved")
doc.save()
```

In an API, tell the user to refresh rather than silently saving over the
other change.

### `DuplicateEntryError` (409) — insert clashes with an existing name

Check before inserting, and fix the naming rule rather than catching the
error in a loop:

```python
if frappe.db.exists("Item", item_code):
    frappe.throw(_("Item {0} already exists").format(item_code))
```

Common causes: a naming series that restarted, or `autoname` producing a
name that already exists after a data import.

### `QueryTimeoutError` / `QueryDeadlockError` — slow or clashing queries

- Timeout: add an index on the filtered column, pass explicit `fields`
  instead of `["*"]`, and paginate with `limit_page_length`.
- Deadlock: make transactions smaller (save one document per commit in a
  job, not hundreds), and retry the job once instead of failing it.

### Job never runs / scheduler silent

```bash
bench doctor
bench --site <site> enable-scheduler
bench --site <site> trigger-scheduler-event daily
```

If `bench doctor` shows workers down, the job was never picked up — that is
an ops problem, not a code problem (see the `frappe-ops` skill).

## 4. Confirm the fix

Reproduce the original steps once more, then check the Error Log is empty
for that error. A fix that only works in `bench console` but not from the
form is usually a permission or a client-side call difference — test it the
way the user hit it.
