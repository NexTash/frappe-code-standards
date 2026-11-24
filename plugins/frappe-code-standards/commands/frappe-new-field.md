---
description: Add fields to a Frappe/ERPNext DocType following the field-layout rules — Section/Column/Tab Breaks, insert_after, child tables — and output the changed files and commands to run.
---

Add fields to the DocType named in `$ARGUMENTS`.

- `$ARGUMENTS` must name the DocType and the fields, for example:
  `/frappe-new-field Sales Order: delivery_date (Date), priority (Select: Low/Medium/High) after customer`
- If `$ARGUMENTS` is empty, or names no DocType or no fields, ask the user
  for them instead of guessing.

## How to add fields

1. Load `skills/frappe-core/SKILL.md` and
   `skills/frappe-core/references/field-layout.md` and follow them.
2. Read the DocType's current field list first — from
   `<app>/<module>/doctype/<name>/<name>.json`, or from the Custom Field
   fixtures if the DocType belongs to another app. Never add fields blind.
3. Place the fields: group related fields under one labelled Section Break;
   max 2 fields per row with a Column Break after every 2nd field; long
   fields (Text, Text Editor, Table, Attach) full-width in their own row;
   a child table under its own Section Break with at most 4–5 `in_list_view`
   columns. Every field needs a real `insert_after` naming an existing field.
4. If the DocType belongs to another app (frappe, erpnext, hrms), add the
   fields as Custom Field fixtures — breaks are fixtures too, each with
   `insert_after` — and export with
   `bench --site <site> export-fixtures --app <app>`.
5. If a field needs behaviour (fetch from, depends on, mandatory), set it
   in the DocType, not in JS. If it needs a `set_query`, add it to the
   form script's `setup` in the correct file.
6. Run the layout items in
   `skills/frappe-core/references/review-checklist.md` before finishing.

## Output

- The field layout as a short list: section → fields, so the user can see
  the form shape without opening it.
- One line per file created or changed: where and why.
- Commands to run: `bench --site <site> migrate` (DocType JSON, fixtures);
  `bench build --app <app>` (public JS/CSS, only if the form script changed);
  `bench --site <site> clear-cache` (hooks.py, only if fixtures changed).
- Do not add fields the user did not ask for, and do not re-layout existing
  fields unless the user asks for it.
