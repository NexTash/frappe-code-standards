---
name: frappe-security
description: Security audit for Frappe/ERPNext apps — SQL injection, XSS, whitelisted-method permissions, allow_guest, secrets and file access. Use when reviewing code, exposing an API, or handling user-supplied data.
---

# Frappe Security

Assume every input is hostile: form fields, URL params, file uploads, webhook
payloads, imported rows. The framework gives you safe APIs — use them, don't
work around them.

## SQL injection — never string-format SQL

| Wrong | Right |
|---|---|
| `frappe.db.sql(f"select * from `tabUser` where name='{user}'")` | `frappe.db.sql("select * from `tabUser` where name=%s", user)` |
| `frappe.db.sql("... where name='" + name + "'")` | `frappe.qb.from_("User").select("*").where(Field("name") == name)` |

- `frappe.db.sql` only with `%s` placeholders and values passed separately.
  Never f-string, `%`, or `+` user input into the query string.
- Prefer `frappe.qb` or `frappe.get_list` / `frappe.get_all` with `filters` —
  they parameterise for you.
- Table/column names can't be placeholders. If a DocType name comes from input,
  validate it against a fixed allowlist first.

## XSS — Frappe Jinja does NOT autoescape

Frappe renders Jinja in a sandboxed environment, but it does **not** autoescape
HTML. `{{ user_text }}` in a print format, email or web template renders raw HTML.

| Wrong | Right |
|---|---|
| `{{ doc.remarks }}` in a print format / web template | `{{ doc.remarks \| e }}` |
| `{{ html \| safe }}` on user-supplied content | `\|safe` only on HTML your own code generated |
| `$(el).html(frm.doc.notes)` in JS | `$(el).text(frm.doc.notes)` or `frappe.utils.escape_html(frm.doc.notes)` |

- Never `|safe` on anything a user typed, uploaded, or imported.
- In JS, build DOM with `.text()` or `frappe.utils.escape_html()` — never
  `.html(user_input)`.
- `frappe.render_template(..., safe_render=True)` is the default — keep it.
  `safe_render=False` allows server-side scripting in templates.

## Whitelisted methods — permissions are your job

- Every `@frappe.whitelist()` method must check permissions itself:
  `frappe.has_permission("DocType", "read", doc)` or `doc.has_permission("write")`
  before reading/writing. Whitelisting does NOT check permissions for you.
- `frappe.get_list` is permission-aware; `frappe.get_all` is not — use `get_list`
  for user-facing reads.
- `ignore_permissions=True` only with a comment explaining why (e.g. a
  system-generated doc in a method that already checked a business rule).
- `allow_guest=True` only for truly public endpoints (signup, public page data).
  A guest method must never return private data or accept writes without its
  own validation.
- Don't trust `frappe.form_dict` / request params for identity — use
  `frappe.session.user`.

## Secrets — never hardcode

| Wrong | Right |
|---|---|
| API key in code, fixtures, or a committed JSON | `bench set-config <key> <value>` (site_config.json, not in git) |
| Password in a Data field | `Password` fieldtype — encrypted at rest; read with `frappe.utils.password.get_decrypted_password` |
| Token in a Client Script / JS | Keep it server-side; expose only via a whitelisted method that checks permission |

- site_config.json is per-site and not committed — that's where keys, tokens and
  webhook secrets live. Read with `frappe.get_site_config()` / `frappe.conf`.
- Never log secrets, and never put them in error messages, fixtures, or test data.
- A leaked secret must be rotated immediately — deleting the file doesn't remove
  it from git history.

## Files — private stays private

- Uploads attached to docs are `File` docs with `is_private`. Private files need
  a logged-in user with permission on the attached doc to download.
- Don't serve user files from `public/` or link the `file_url` of a private file
  on a guest page.
- Validate uploads server-side (file type/extension) before saving; never execute
  uploaded content.
- Don't bypass file-download permission checks with `ignore_permissions` to
  "make it work".

## Desk vs portal exposure

- Desk assumes a logged-in user with roles. Website/portal pages (`www/`, Web
  Forms, `allow_guest`) are public — every value shown there must be safe for
  anonymous eyes.
- A DocType with a Web Form is a public write endpoint: validate every field
  server-side (`validate` / `before_insert`), and never expose internal fields
  (cost, margin, internal notes) in the form.
- Don't rely on "hidden" fields or JS to hide data — the API returns what the
  DocType allows. Use field-level permissions and keep sensitive fields off
  guest-visible doctypes.

## Security review checklist

- [ ] No f-string / `+` / `%` user input in `frappe.db.sql`; `frappe.qb` or placeholders everywhere.
- [ ] No `{{ user_value }}` without `|e` in Jinja; no `|safe` on user content; no `.html(user_input)` in JS.
- [ ] Every `@frappe.whitelist()` checks `has_permission` (or documents why not); `allow_guest` only where truly public.
- [ ] No `ignore_permissions=True` without a comment.
- [ ] No secrets in code, fixtures, logs, or JS; keys in site_config; passwords in `Password` fields.
- [ ] Private files stay private; uploads validated server-side.
- [ ] Guest/portal surfaces show no internal data; writes validated server-side.
