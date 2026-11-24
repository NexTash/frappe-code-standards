---
name: frappe-core
description: Core Frappe/ERPNext developer rules (v14-v16). Use for ANY Frappe or ERPNext coding - DocTypes, controllers, form/client JS, hooks.py, set_query, whitelisted APIs, fixtures, patches, permissions or code review. Enforces correct code placement and standard APIs.
---

# Frappe Developer Rules

Write code a senior Frappe developer would accept: correct, **in the file where any Frappe
developer would look for it**, using **standard Frappe APIs**, and safe for the project's version.
Working code in the wrong place, or code that bypasses the framework, is a defect.

## Step 0 - Before writing code

1. **Version**: check `apps/frappe/frappe/__init__.py` (`__version__`). Default to v15 if unknown.
   For version-specific APIs, use the **frappe-upgrade** skill.
2. **Owner app**: find `<app>/<module>/doctype/<name>/<name>.json`.
   - Our app owns it -> edit its `.py` / `.js`.
   - Another app owns it (frappe, erpnext, hrms...) -> never edit it; extend from our app via hooks + fixtures.
3. **Read before adding**: extend existing handlers; no duplicate `frappe.ui.form.on` blocks or methods.
4. **Server vs client**: rules and validation go on the server. JS is UX only. Never enforce a rule in JS alone.
5. **App vs Server Script**: if a custom app exists, use it. Server Scripts only when there is UI-only
   access and the logic is short -> use the **frappe-server-scripts** skill (NO imports).
6. **New fields**: read the DocType's current field list first; place with `references/field-layout.md`
   (Section/Column/Tab Break, field order, child tables).

## Placement rules

| What | Our DocType | Another app's DocType |
|---|---|---|
| Form JS, set_query, buttons | `doctype/<name>/<name>.js` | `<app>/public/js/<name>.js` + `doctype_js` in hooks.py |
| List view JS | `<name>_list.js` | `doctype_list_js` in hooks.py |
| Child table events | same parent `.js`, separate `frappe.ui.form.on("Child", {...})` | same override file |
| Validation / business logic | controller methods (`validate`, `on_submit`...) | `doc_events` -> `<app>/<module>/overrides/<name>.py`; on v16 prefer `extend_doctype_class` |
| Methods called from form | `@frappe.whitelist()` on controller, `frm.call()` | whitelisted function in overrides, `frappe.call()` |
| Extra fields | - | Custom Field / Property Setter as **fixtures** filtered by module; layout per `references/field-layout.md` |
| Data fixes | `patches/` + `patches.txt` | same |

Never edit other apps' files. Never put our DocType's form code in `public/js`.

## Always

- **Errors**: `frappe.throw(_("..."))` to stop; `frappe.msgprint` to inform; `frappe.show_alert` in JS.
  Never `raise Exception`, silent `return`, or `print()`.
- **Translate**: `_()` in Python, `__()` in JS, with placeholders: `_("Row {0}: Qty < 0").format(row.idx)`.
  No f-strings inside `_()`.
- **Logging**: `frappe.log_error(title=..., message=frappe.get_traceback())` for jobs and integrations.
- **Reads**: `frappe.db.get_value` / `frappe.get_cached_value` for single fields, not `get_doc`.
- **Lists**: `frappe.get_list` (permission-aware) or `frappe.get_all` with explicit `fields`. No `["*"]`.
- **Queries**: `frappe.qb`; raw `frappe.db.sql` only parameterised.
- **No N+1**: fetch in bulk, build a dict, then loop.
- **No `frappe.db.commit()`** in controllers, hooks or requests (blocked in v16 hooks).
- **Long work** (>few seconds, bulk, external APIs) -> `frappe.enqueue(..., job_id=...)`.
- **Permissions**: whitelisted methods check `frappe.has_permission` / use `get_list`; never
  `ignore_permissions=True` without a reason in a comment. `allow_guest` only for truly public endpoints.

## References in this skill (open only when needed)

| Task | File |
|---|---|
| Where does code go? | `references/code-placement.md` |
| Python controllers, whitelist, DB, permissions | `references/server-patterns.md` |
| JS forms, set_query, dialogs, child tables | `references/client-patterns.md` |
| hooks.py, fixtures, custom fields, patches, scheduler | `references/hooks-and-customization.md` |
| Field layout: Section/Column/Tab Break, field order, child tables | `references/field-layout.md` |
| Before finishing | `references/review-checklist.md` |

## Other branches (separate skills, load only for that task)

| Task | Skill |
|---|---|
| Server Script in UI | frappe-server-scripts |
| Reports, print formats, Jinja | frappe-reports-print |
| Slow code, caching, jobs | frappe-performance |
| v14/v15/v16 APIs, upgrading | frappe-upgrade |
| Error / traceback | frappe-debug |
| Tests, CI | frappe-testing |
| bench, deploy, backup, Frappe Cloud | frappe-ops |
| New feature / app design, "how to build X" | frappe-architect |
| Workflows, approvals | frappe-workflow |
| Email, Notification rules, realtime | frappe-notifications |
| REST API, webhooks, OAuth, external APIs, Data Import | frappe-integrations |
| Website, portal, Web Forms, www pages | frappe-website |
| Workspaces, dashboards, custom Pages, dialogs, Vue SPA | frappe-ui |

## Finishing

1. Run `references/review-checklist.md`.
2. List commands needed: `bench --site <site> migrate` (DocType JSON, patches, fixtures);
   `bench build --app <app>` (public JS/CSS); `bench --site <site> clear-cache` (hooks.py).
3. One line per changed file: where and why there.
