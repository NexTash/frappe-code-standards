---
name: frappe-integrations
description: Frappe/ERPNext REST API, whitelisted endpoints for external use, webhooks, OAuth, calling external APIs, and Data Import. Use when integrating Frappe with another system in either direction.
---

# Frappe Integrations

## Inbound (others call Frappe)
- REST: `/api/resource/<DocType>[/<name>]` (CRUD), `/api/method/<dotted.path>` (whitelisted).
  v15+: `/api/v2/document/...` and `/api/v2/method/...` are also available.
- Auth: `Authorization: token <api_key>:<api_secret>`, or OAuth `Bearer <token>`. Send `Accept: application/json`.
- Create a **dedicated API user** with minimal roles. Never use Administrator keys.
- Whitelisted endpoints:
  ```python
  @frappe.whitelist(methods=["POST"])
  def create_order(customer: str, items: list | str):
      frappe.has_permission("Sales Order", "create", throw=True)
      items = frappe.parse_json(items)
      ...
  ```
  Validate input, check permissions, paginate lists, use POST for state changes.
  Never use `allow_guest=True` on anything that writes. Rate-limit guest endpoints (`@frappe.rate_limit`).

## Outbound (Frappe calls others)
- Never call external APIs in `validate` / `before_save`. Use `on_update` / `on_submit` + `frappe.enqueue`.
- Always set a timeout (`requests.post(url, json=..., timeout=30)`), wrap in try/except, and `frappe.log_error`.
- Credentials go in a Settings Single DocType with a **Password** field (`doc.get_password("api_key")`)
  or in `frappe.conf`. Never hardcode, and never send secrets to JS.
- Retry with backoff. Make calls idempotent (store the external id on the doc).

## Webhooks
- **Webhook** DocType: doc event -> URL, JSON body with Jinja (`{{ doc.name }}`, `{{ doc.items | tojson }}`).
- Set a **Webhook Secret** in production. Receivers verify the `X-Frappe-Webhook-Signature` HMAC-SHA256.
- Incoming webhooks: whitelisted method, verify the signature/token first, then enqueue the processing.

## OAuth
- Frappe as provider: OAuth Client DocType. Use HTTPS only.
- Connected App + Token Cache for third-party OAuth. Handle token expiry and refresh.
  For Google use `access_type=offline`.

## Data Import
- Download the template from Data Import (exact column names). Test 5-10 rows first, then batches of 5,000 or fewer.
- Check the Import Log for row errors. For huge or scripted loads use a patch / `bench execute` with bulk inserts.
