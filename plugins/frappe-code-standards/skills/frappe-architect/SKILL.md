---
name: frappe-architect
description: Plan and design Frappe/ERPNext solutions before coding - choose custom app vs customization, DocType and data model design, app structure, dependencies, which extension mechanism to use. Use for new features, new apps, or "how should I build X" questions.
---

# Frappe Solution Design

Do this before writing code for any non-trivial feature. Give the user a short plan.

## 1. Customize or build?
| Situation | Choose |
|---|---|
| Fewer than about 10 extra fields on an existing DocType | Custom Fields (fixtures) |
| Small validation, UI-only access | Server Script (see frappe-server-scripts) |
| Real business logic, tests, version control | **Custom app** (default for teams) |
| Change behaviour of an ERPNext DocType | hooks: `doc_events`, or v16 `extend_doctype_class`. Override a class only as a last resort. |
| New business object | New DocType in the custom app |

Never modify ERPNext/Frappe source.

## 2. Data model
- One DocType per real-world thing. Link to existing data, don't copy it (no duplicate customer name fields
  except `fetch_from` read-only display).
- Child tables only for rows that live and die with the parent. Otherwise make a separate DocType + Link.
- Every DocType gets naming (`naming_series` / `autoname`), mandatory fields, permissions per role, and `track_changes` if auditable.
- Submittable (`is_submittable`) when the doc becomes final (orders, approvals).
- Single DocType for settings. Use Password fields for secrets.
- Add indexes (`search_index`) on fields used in filters/reports.

## 3. App structure
- `required_apps` in hooks.py lists dependencies (base apps first). No circular dependencies.
- Group by module (`modules.txt`). Keep shared helpers in `<app>/utils/`, and overrides in `<module>/overrides/`.
- Ship config as fixtures filtered by module: Custom Field, Property Setter, Workflow, Role, Notification.

## 4. Plan output (give this to the user)
1. Approach and why (app vs customization).
2. DocTypes/fields to add (name, key fields, links).
3. Logic locations (file per rule) and hooks used.
4. Permissions/roles.
5. Data migration (patches) if any.
6. Version notes (v14/v15/v16), then implement with frappe-core.
