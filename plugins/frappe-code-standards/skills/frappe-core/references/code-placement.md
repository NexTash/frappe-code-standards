# Code Placement

The rule: put code where a Frappe developer opening the project would look for it first.

## App layout (reference)

```
apps/<app>/<app>/
├── hooks.py
├── patches.txt
├── <module>/                      # one folder per Module Def
│   ├── doctype/<doctype_snake>/
│   │   ├── <doctype_snake>.json   # schema (edit via Desk in developer mode, or carefully by hand)
│   │   ├── <doctype_snake>.py     # controller class
│   │   ├── <doctype_snake>.js     # form script
│   │   ├── <doctype_snake>_list.js
│   │   └── test_<doctype_snake>.py
│   ├── report/<report_snake>/     # script / query reports
│   ├── page/                      # custom desk pages
│   └── overrides/ (or events/)    # hooks for OTHER apps' DocTypes, one file per DocType
├── patches/<version>/             # one patch per file
├── public/js/                     # shared utils + overrides of other apps' forms only
├── templates/ and www/            # portal / web pages
└── fixtures/                      # exported Custom Field, Property Setter, etc.
```

## Decision tree

1. **Is the DocType defined in our app?**
   - Yes → form logic in `<name>.js`, server logic in `<name>.py`. Done. No hooks entry.
   - No → continue.
2. **Server logic on another app's DocType** → `doc_events` in hooks.py pointing to
   `<app>.<module>.overrides.<doctype_snake>.<function>`. Signature: `def validate(doc, method=None):`.
3. **Client logic on another app's DocType** → `public/js/<doctype_snake>.js` and
   ```python
   doctype_js = {"Sales Invoice": "public/js/sales_invoice.js"}
   ```
4. **Need to change a controller method of another app** → first try `doc_events`. Only if a
   method must be replaced, use `override_doctype_class` (or `extend_doctype_class` where the installed version supports it),
   subclass the original and call `super()`.
5. **Need a field on another app's DocType** → Custom Field, exported as a fixture filtered by our
   module. Never edit the other app's JSON.
6. **Logic used by several DocTypes** → plain Python module in our app (e.g. `<app>/<module>/utils.py`),
   called from each controller. JS shared helpers → `public/js/` + bundle, only if truly shared.

## Child tables

Child DocType events go in the **parent's** JS file, in their own block:

```js
frappe.ui.form.on("Sales Order Custom", {
    setup(frm) { /* set_query here */ },
    refresh(frm) {},
});

frappe.ui.form.on("Sales Order Custom Item", {
    qty(frm, cdt, cdn) { calculate_row_amount(frm, cdt, cdn); },
    rate(frm, cdt, cdn) { calculate_row_amount(frm, cdt, cdn); },
    items_remove(frm) { calculate_totals(frm); },
});
```

Server-side logic on child rows goes in the **parent controller** (loop over `self.items`), not the
child DocType's controller, unless it is truly row-independent.

## Wrong vs right

- ❌ `hooks.py: doctype_js = {"My Own DocType": "public/js/my_own_doctype.js"}`
  ✅ `my_app/my_module/doctype/my_own_doctype/my_own_doctype.js`
- ❌ Validation in `frappe.ui.form.on(... validate(frm) {...})` only
  ✅ Validation in controller `validate()`; JS may add a UX hint.
- ❌ `doc_events = {"My Own DocType": {...}}` for a DocType we own
  ✅ Controller method `def validate(self):`
- ❌ Logic in a Server Script record created on the site
  ✅ Code in the app repository (versioned, reviewable, deployable).
- ❌ Editing `apps/erpnext/...` files
  ✅ hooks + overrides + fixtures in our app.
