---
name: frappe-notifications
description: Frappe/ERPNext email, Notification doctype, alerts, system notifications and realtime. Use when sending emails, building Notification rules, email templates, in-app notifications, or frappe.publish_realtime.
---

# Frappe Notifications

## Choose
| Need | Use |
|---|---|
| Email/alert on a doc event, configurable by users | **Notification** DocType (ship as fixture) |
| Email from code | `frappe.sendmail` |
| Bell-icon notification | `Notification Log` via `frappe.get_doc({...}).insert()` or `enqueue_create_notification` |
| Live UI update | `frappe.publish_realtime` |
| Reusable email body | Email Template |

## frappe.sendmail
```python
frappe.sendmail(
    recipients=[doc.owner],
    subject=_("Order {0} approved").format(doc.name),
    template="order_approved", args={"doc": doc},   # templates/emails/order_approved.html
    reference_doctype=doc.doctype, reference_name=doc.name,  # shows in timeline
)
```
- An Email Account with "Enable Outgoing" is required.
- Keep the default queueing. Never `now=True` in production (it blocks, and on v16 it doesn't commit).
- Send from `on_submit` / `on_update`, not `validate` (validation may still fail).
- Bulk mail goes through `frappe.enqueue`.

## Notification DocType
- Channel: Email / System Notification / SMS / Slack.
- Event: New, Save, Submit, Cancel, Days Before/After, Value Change, Method.
- Condition: `doc.status == "Overdue"`. Message: Jinja with `{{ doc.name }}`.
- Recipients: by field (`owner`, a Link to User/Email), by role, or CC.

## Realtime
```python
frappe.publish_realtime("my_event", {"name": doc.name}, user=frappe.session.user)  # or doctype/docname
```
```js
frappe.realtime.on("my_event", (data) => { ... });
```

## Debug
Check **Email Queue** (status, error), **Error Log**, that the scheduler is enabled (`bench doctor`),
and that `mute_emails` is not set in site_config.
