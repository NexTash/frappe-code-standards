---
name: frappe-ui
description: Frappe Desk UI - Workspaces, Number Cards, Dashboard Charts, custom Pages, dialogs, list/report view settings, and Vue/frappe-ui frontends. Use when building desk screens, dashboards or custom pages rather than form logic.
---

# Frappe Desk UI

## Workspaces & dashboards
- Create in the UI, set `module`, and export with the app: with developer_mode on, workspaces are saved to the module's `workspace/` folder.
- Public workspaces **must** have `module`. Without it every desk user sees them.
- Never set `for_user` on shipped workspaces (that makes them private).
- Restrict sensitive ones (finance, HR) with Roles on the workspace and on each card/chart.
- When setting `content` JSON in code, also add the matching child rows (shortcuts, links, charts, number_cards).
- Number Card / Dashboard Chart: prefer "Document Type" based ones. For custom ones use
  `method` = a whitelisted function that returns the value/data. Mind permissions.

## Custom Page (`<module>/page/<name>/`)
```js
frappe.pages["my-page"].on_page_load = function (wrapper) {
  const page = frappe.ui.make_app_page({ parent: wrapper, title: __("My Page"), single_column: true });
  page.set_primary_action(__("Refresh"), () => load());
  const field = page.add_field({ fieldname: "company", fieldtype: "Link", options: "Company", change: load });
  function load() { frappe.call("myapp.api.get_data", { company: field.get_value() }).then(r => {/* render */}); }
};
```
On v16 page JS runs as an IIFE, so don't rely on globals.

## Dialogs
```js
const d = new frappe.ui.Dialog({
  title: __("Set Rate"),
  fields: [{ fieldname: "rate", fieldtype: "Currency", label: __("Rate"), reqd: 1 }],
  primary_action_label: __("Apply"),
  primary_action(values) { frm.call("apply_rate", values).then(() => d.hide()); },
});
d.show();
```
Use `frappe.prompt` for one field and `frappe.confirm` for yes/no. Use Frappe field types, not raw HTML inputs.

## Separate SPA (Vue + frappe-ui)
Use for portal-like apps (like Helpdesk/CRM): `<app>/frontend` with Vite and the `frappe-ui` library,
calling `/api/method/...` through `createResource`. Build output goes to `<app>/public/frontend`, with a
www route for the SPA. Use it only when Desk can't do the job.

## Form field layout
- Field order, Section/Column/Tab Breaks and child-table grids are form layout, not just
  code: follow the `frappe-core` skill's `references/field-layout.md` before adding fields.
