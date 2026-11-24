---
name: frappe-data
description: Data Import, bulk updates, migration and cleanup for Frappe/ERPNext — templates, preview validation, patches, backup before deletes. Use when moving, fixing or importing large amounts of data.
---

# Frappe Data

Data work is irreversible by default. Every import, bulk update and delete gets
a backup first and a preview before it runs.

## Data Import (Desk)

Flow: download the template → fill it → upload → **read the preview** → import.

- Download the template from the Data Import doc — it has the exact columns,
  mandatory fields and fieldtypes for that DocType. Never hand-write the header row.
- Link fields need the exact document name (e.g. Customer `CUST-00001`), not the
  label. Select fields need an exact option value. Dates in `YYYY-MM-DD`.
- Upload, then read the **preview**: it lists every row that will fail and why.
  Fix the file and re-upload — don't import with errors and "fix later".
- `start_import` runs in the background; watch the import log. Errored rows can
  be exported (`export_errored_rows`) and re-imported after fixing.
- Child tables: add the child fields as columns and repeat the parent row once
  per child row.
- Import runs through the normal controller — `validate`, `before_insert`,
  autoname and naming series all run. The importing user needs write permission
  on the DocType.
- Never import into production without testing the same file on a test site first.

## Bulk updates

| Method | When | Watch out |
|---|---|---|
| Data Import, Import Type "Update Existing Records" | Fixing field values in bulk | Match on the name column — a wrong name creates a duplicate instead of updating |
| Data Import, "Insert or Update Records" | Mixed new + existing rows | Same name-column rule; check the preview for inserts vs updates |
| `patch` in `patches.txt` | One-off data fix shipped with the app | Must be idempotent — a failed patch blocks migrate; test on a copy first |
| `bench console` / `bench execute` | One-off fix on one site | No undo; take a backup first |
| Server Script / whitelisted method | User-triggered bulk action | Enqueue it (`frappe.enqueue`) for large sets; never loop `frappe.db.commit()` per row |

- Bulk updates go through `doc.save()` or `frappe.db.set_value` — `set_value`
  skips `validate` and controller hooks. Use it only when you mean to skip them,
  and say so in a comment.
- Never `frappe.db.sql("update ...")` on user data unless there is no API for
  it — it bypasses versioning, permissions and hooks.
- For >1000 rows, enqueue a background job and show progress; a web request
  that runs for minutes will time out.

## Migration between sites

- Full site move: `bench --site <site> backup --with-files --compress` on the
  source, `bench --site <site> restore <db-file> --with-public-files <pub> --with-private-files <priv>`
  on the target, then `bench --site <site> migrate`. Backup includes files and database.
- Data-only move: export with Data Import templates, then import on the target —
  slower, but lets you clean the data on the way.
- Check versions first: restoring a backup from a newer Frappe/ERPNext version
  onto an older one is not supported — upgrade the target first.
- site_config values (API keys, tokens) don't travel with the database backup —
  copy them separately with `bench set-config`.
- Test the restore on a staging site before touching production.

## Cleanup and deletion

- Before any bulk delete: `bench --site <site> backup`. Deleted documents are
  hard to recover.
- Prefer cancelling submitted docs over deleting — cancelled docs keep the audit
  trail; deleted docs don't.
- Delete with `frappe.delete_doc(doctype, name)` — never raw SQL deletes — so
  child tables, links and files are handled.
- Don't delete master data (Customer, Item, Account) that transactions reference
  — Frappe blocks it with a link error. That's protection, not a bug. Disable
  (deactivate) instead.

## Exports

- List view → Menu → Export: CSV/Excel with the current filters and visible columns.
- Programmatic export: Data Import template download, or `frappe.get_list` + CSV
  in a whitelisted method (permission-checked, enqueued for large sets).
- Never export sensitive data (salaries, passwords, API keys) to a shared file
  without a reason and a permission check.

## Data checklist

- [ ] Backup taken; tested on staging before production.
- [ ] Template downloaded, not hand-written; preview read, zero error rows.
- [ ] Update matched on the name column — no accidental duplicates.
- [ ] Bulk work enqueued, not in a web request; no raw SQL writes.
- [ ] Deletes via `frappe.delete_doc`; submitted docs cancelled, not deleted, where possible.
