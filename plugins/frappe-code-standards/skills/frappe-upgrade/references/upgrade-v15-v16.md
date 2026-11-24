# Upgrade v15 -> v16 — Breaking Changes and New Features

Sources: Frappe Framework v16.0.0 release notes (GitHub, 12 Jan 2026), Frappe
Framework / ERPNext / HR v16.60 announcement (Frappe Blog, 17 Nov 2026),
Frappe product updates (wkhtmltopdf, `never_skip_patches`), Frappe Framework
v16 webinar page. Items marked "community-reported" come from migration guides,
not from Frappe's own docs — verify them before relying on them.

## Rules

- Upgrade **one major version at a time**: v14 -> v15, verify, then v15 -> v16.
  Never skip v15 — v16 patches assume v15 schema changes are already in place.
- Do it on a **staging copy first**, with a tested backup (`bench backup
  --with-files`). Never upgrade a broken bench.
- Every custom app needs a version-16 branch (or be removed) before migrating.

## Runtime

| Component | v15 | v16 |
|---|---|---|
| Python | 3.10–3.11 | **3.14+** (older versions fail with SyntaxError) |
| Node.js | 18 / 20 | **24+** |
| MariaDB | 10.6+ | 10.6+ (stay current) |
| OS | Ubuntu 22.04 | Ubuntu 24.04 recommended |

## Breaking changes — code

| Old / removed | Use / action |
|---|---|
| `extend_doctype_class` hook added | Prefer it over `override_doctype_class`. Prefix methods with the app name. |
| `override_doctype_class` | Class must inherit the original |
| `has_permission` hooks | Must return explicit `True` (not `None`) |
| DB commit inside document hooks | Not allowed |
| Default list sort | `creation` (was `modified`), so pass `order_by` explicitly |
| `frappe.get_doc(dt, name, field=val)` | No longer sets values |
| `frappe.sendmail(now=True)` | No longer commits |
| `db.get_value` on Singles | Returns typed values, not strings |
| `frappe.flags.in_test` | `frappe.in_test` |
| State-changing API methods | POST only (`logout`, `upload_file`, `send_login_link`) |
| Report/Page JS | Runs as IIFE with no globals |
| Data masking (field `mask`) | Not applied to raw SQL, so mask manually |
| Translation `get_lang_dict`, `get_translated_dict` hook | Removed |
| SQL in templates, Server Scripts, Query Reports | Blocked from reading `__Auth` (password hashes, secrets) |
| Blog, Newsletter, Energy Points, Offsite backups | Separate apps (offsite backups: `frappe/offsite_backups`) |
| `/app` URL | `/desk` |
| Standard workspace customisations | Reset on upgrade — keep customisations in your own app |

## Breaking changes — PDF and print

- PDF renders with the **Chrome** renderer. **wkhtmltopdf is deprecated** in
  v16 and removed in v17. Print Settings has a PDF Generator dropdown (Chrome
  or wkhtmltopdf); switch formats to Chrome or **PlutoPrint** now and check
  how they print.
- **Classic print formats are deprecated** and stop working in v17.

## New in v16.0

- Redesigned Workspace and Desktop (navigation, module cells, sidebar);
  navigation bar shows the document title, not the system ID.
- Enhanced List View: scrollable, resizable columns; scrollable child tables
  with sticky columns.
- Role-based field masking.
- ~2× faster performance (per Frappe).
- **SQLite support**: Frappe can run on SQLite without MariaDB.
- Email Template "Use HTML" option; PDF Generator dropdown in Print Settings.
- Patches that must never be skipped: list them in hooks.py as
  `never_skip_patches = ["app.patches.v16_0.critical_data_fix"]` — if one
  fails, migration stops. `bench migrate --skip-failing` skips other failures.

## New in v16.60 (Framework + ERPNext + HR)

- **Studio** merged into Framework — see the **frappe-studio** skill.
- **Frappe Flow** AI assistant/agent framework (alpha/beta) — see the
  **frappe-flow** skill.
- **Insights dashboards** — Framework supports Insights dashboards, charts,
  and number cards; System Settings has a Dashboards option (Insights or
  classic). New sites use Insights by default; upgraded sites keep classic
  until changed. See the **frappe-insights** skill.
- Vue Islands, improved mobile UX, child-table importer, improved web form
  editor (visual builder, field picker, page stepper), better report printing,
  upload-first flow for documents.
- ERPNext: Day Book report, ten new standard print formats, overview
  dashboards inside Customer and Supplier forms, first phase of a Projects
  revamp.

## Community-reported, not confirmed in Frappe's own docs

Verify these on staging before relying on them:

- Custom apps migrate from `webpack.config.js` to `vite.config.js`;
  CommonJS `require()` is not supported in Vite ESM mode.
- The old RQ-based background worker model is removed in v16; re-run
  `bench setup supervisor` and check custom Procfile entries.
- Desk frontend Vue 3 migration is completed in v16; custom Vue 2 components
  using the Options API need a rewrite.

## Upgrade checklist

1. Staging copy, tested backup, runtime on Python 3.14 / Node 24.
2. Every app on a version-16 branch; remove unsupported apps first.
3. `bench switch-to-branch version-16` (frappe, erpnext, hrms, custom apps),
   then `bench migrate` — watch for `never_skip_patches` failures.
4. Check: list sort order, print formats on Chrome, `has_permission` hooks,
   POST-only endpoints, `/desk` links, offsite backups app installed.
5. Switch dashboards to Insights only after checking each dashboard (System
   Settings > Dashboards).
