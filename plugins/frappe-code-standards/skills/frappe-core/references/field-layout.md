# Field Layout — Section, Column & Tab Breaks

Form layout is part of the task, not an afterthought. Fields dumped in one block
break the form: related fields scatter, child tables land in the wrong place, and
the user has to re-do the layout by hand.

## Step 0 — Read the form first

1. Open the DocType JSON (`<app>/<module>/doctype/<name>/<name>.json`) or the
   Custom Field fixtures and read the **current field list in order** — names,
   fieldtypes, and every existing Section Break / Column Break / Tab Break.
2. Never add fields blind. Know which section they belong in and which field they
   follow (`insert_after`).
3. If the DocType belongs to another app, new fields are Custom Fields in fixtures
   — breaks are Custom Fields too.

## Breaks — what each one does

- **Section Break**: starts a new labelled group (e.g. "Pricing", "Approvals").
  One logical group = one section. Give it a label; mark `collapsible: 1` for
  groups users rarely need.
- **Column Break**: splits a section into columns. Frappe shows 2 columns per
  row by default — put a Column Break after every 2nd field in the row. Never
  leave a section with a single field in the second column and nothing after it.
- **Tab Break**: starts a new tab for a major group (e.g. "Items", "Taxes",
  "Attachments"). Use sparingly — tabs hide fields; sections are the default.
- **Order**: Section Break, then fields, then Column Break, then fields. A Column
  Break without a preceding Section Break in the same group is a mistake.

## Placing new fields

1. Group related fields together under one Section Break; don't scatter one
   feature's fields across sections.
2. Max 2 fields per row. After the 2nd field, add a Column Break before the next
   row — or start a new section.
3. `insert_after` must name a real existing field. Verify it in the JSON; a wrong
   `insert_after` drops the field at the end of the form.
4. Long fields (Text, Text Editor, Table, Attach) go full-width: put them in their
   own row, never squeezed next to another field.
5. Mandatory fields first in their section; read-only/computed fields last.
6. `depends_on` hides a field — but it still takes layout space in the JSON order.
   Keep dependent fields next to the field they depend on.

## Child tables

- A child table is a `Table` field pointing at a child DocType — place it under
  its own Section Break (e.g. "Items"), never inline among normal fields.
- Child DocType columns: mark `in_list_view: 1` on at most 4–5 fields — the grid
  can't show more without horizontal scrolling. Set `columns` (width) on wide ones.
- Child table events stay in the parent's `.js` (`frappe.ui.form.on("Child", ...)`).

## Custom Fields on another app's DocType

- Each new field, Section Break and Column Break is a separate Custom Field
  fixture, each with `insert_after` pointing at the field it follows.
- Add breaks first, then the fields between them — in fixture order.
- Export with `bench --site <site> export-fixtures --app <app>` and check the
  fixture JSON order matches the intended form order.

## Verify

- `bench --site <site> migrate`, then open the form and check: sections in the
  right order, 2 fields per row, no orphan column, child table under its own
  section, grid columns readable without scrolling.
- Run the layout items in `references/review-checklist.md`.
