# Review Checklist (run before finishing every task)

## Placement
- [ ] Own DocType logic is in its own `<name>.py` / `<name>.js` — nothing in hooks.py or public/js for it.
- [ ] Other apps' DocTypes are extended only via hooks.py + `overrides/` + fixtures; no files edited in frappe/erpnext.
- [ ] `set_query` is in `setup`/`onload` of the correct form script.
- [ ] Business rules enforced on the server, not only in JS.
- [ ] No duplicate `frappe.ui.form.on` blocks or duplicate controller methods.

## Framework APIs
- [ ] Errors: `frappe.throw(_())`; info: `frappe.msgprint(_())` / `frappe.show_alert`.
- [ ] All user-facing strings use `_()` / `__()` with `{0}` placeholders.
- [ ] No `print`, `console.log`, `alert`, bare `except`, `raise Exception`.
- [ ] `flt`/`cint`/`getdate` used for doc values.
- [ ] Field-level behaviour (fetch from, depends on, mandatory) set in DocType, not JS, where possible.

## Security
- [ ] Whitelisted methods check permissions; writes use POST.
- [ ] No f-string SQL; parameters or QB only.
- [ ] `ignore_permissions` only with a comment explaining why.

## Performance
- [ ] No DB calls inside loops.
- [ ] Explicit `fields`, no `*`.
- [ ] Slow work enqueued; no `frappe.db.commit()` in request path.

## Delivery
- [ ] Tell the user the commands to run (migrate / build / clear-cache) and the files changed with one-line reasons.
