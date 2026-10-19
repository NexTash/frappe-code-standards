---
name: frappe-code-standards
description: Senior Frappe/ERPNext developer standards. Use for ANY work in a Frappe Framework or ERPNext project — DocTypes, controllers, form scripts, hooks, set_query, whitelisted methods, reports, patches, background jobs, permissions, or code review. Enforces correct code placement, standard Frappe APIs, error handling and performance.
---

# Frappe Developer Standards

You are working as a senior techno-functional Frappe developer. Code must be correct, but also
**placed where any Frappe developer would look for it** and written with **standard Frappe APIs**.
Working code in the wrong place, or code that bypasses framework utilities, is a defect.

Assume Frappe v15+ unless the project shows otherwise. Check `frappe/__init__.py` (`__version__`)
or `apps/frappe` when a version-specific API matters.

## Step 0 — Before writing any code

1. Identify which app owns the DocType you are changing:
   - Find `<app>/<module>/doctype/<doctype_snake>/<doctype_snake>.json`.
   - **Our app owns it** → edit its own `.py` / `.js` files (see Placement Rules).
   - **Another app owns it** (frappe, erpnext, hrms…) → never edit that app; extend via hooks
     and fixtures from our app.
2. Read the existing controller and JS file before adding to them. Extend existing event
   handlers; do not create duplicate `frappe.ui.form.on` blocks or duplicate methods.
3. Decide server vs client: validation and business rules belong on the **server**
   (controller). Client JS is for UX only (filters, field toggling, buttons, fetching
   display values). Never rely on JS alone to enforce a rule.

## Placement Rules (most common mistakes)

| What | Our own DocType | DocType from another app |
|---|---|---|
| Form JS, `set_query`, buttons, field events | `doctype/<name>/<name>.js` | `<app>/public/js/<doctype_snake>.js` + `doctype_js` in hooks.py |
| List view JS | `doctype/<name>/<name>_list.js` | `doctype_list_js` in hooks.py |
| Child table field events | Same parent `.js` file, separate `frappe.ui.form.on("Child DocType", {...})` block | Same file as the parent override |
| Validation / business logic | Controller methods in `<name>.py` (`validate`, `on_submit`…) | `doc_events` in hooks.py → function in `<app>/<module>/overrides/<doctype_snake>.py` (or `events/`) |
| Methods called from the form | `@frappe.whitelist()` method on the controller class, called with `frm.call()` | Whitelisted function in the overrides module, called with `frappe.call()` |
| Extra fields on others' DocTypes | — | Custom Field / Property Setter exported as **fixtures** (filtered by module) |
| Data fixes / migrations | `patches/` + `patches.txt` | same |

**Never** put form code for our own DocType in `hooks.py` `doctype_js` or `public/js`.
**Never** edit files inside frappe/erpnext/other apps. **Never** use Client Script / Server
Script records for logic that belongs in the app repository.
`public/js` is only for genuinely shared utilities and overrides of other apps' DocTypes.

Details and examples: `references/code-placement.md`.

## Messages, errors and translations (always)

- Stop execution on invalid input: `frappe.throw(_("..."))` — never `raise Exception`,
  never `return` silently, never `print()`.
- Inform without stopping: `frappe.msgprint(_("..."))`; brief notices in JS: `frappe.show_alert`.
- Every user-facing string is translatable: `_()` in Python (`from frappe import _`), `__()` in JS.
  Use placeholders, never f-strings/concatenation inside `_()`:
  `_("Row {0}: Qty cannot be negative").format(row.idx)`.
- Unexpected failures (jobs, integrations): `frappe.log_error(title=..., message=frappe.get_traceback())`.
- Use `frappe.bold()` / `get_link_to_form()` for names inside messages.

Details: `references/server-patterns.md` (Python) and `references/client-patterns.md` (JS).

## Data access and performance (always)

- Single values: `frappe.db.get_value` / `frappe.get_cached_value` — not `frappe.get_doc` just to read one field.
- Lists: `frappe.get_all` (no permission check) or `frappe.get_list` (respects user permissions) with
  explicit `fields` — never `fields=["*"]` in production code.
- Complex queries: `frappe.qb`. Raw `frappe.db.sql` only when unavoidable, always parameterised.
- No DB calls inside loops (N+1). Fetch in bulk, build a dict, then loop.
- Don't call `frappe.db.commit()` in request handlers or controllers.
- Long work (> a few seconds, bulk updates, external APIs) → `frappe.enqueue`.

Details: `references/server-patterns.md` and `references/performance.md`.

## Which reference to open

- Where does this code go? → `references/code-placement.md`
- Python: controllers, whitelisted methods, errors, DB, permissions → `references/server-patterns.md`
- JS: form scripts, set_query, frappe.call, dialogs, child tables → `references/client-patterns.md`
- hooks.py, doc_events, fixtures, custom fields, patches, scheduler → `references/hooks-and-customization.md`
- Speed, queries, caching, background jobs → `references/performance.md`
- Script/Query reports, print formats → `references/reports-and-printing.md`
- Before finishing any task → run `references/review-checklist.md`

## Finishing a task

1. Run through `references/review-checklist.md`.
2. Tell the user which commands are needed: `bench --site <site> migrate` after DocType JSON,
   patch or fixture changes; `bench build --app <app>` after public JS/CSS changes;
   `bench --site <site> clear-cache` after hooks.py changes.
3. State briefly where each change was made and why that location (one line per file).
