# Permission Debugging

Five cases that come up again and again. For the rules behind them, read
`../SKILL.md`.

## Case 1: document shared, user still gets PermissionError

- **Symptom:** you shared a Sales Order with a user; they still get 403.
- **Likely cause:** their role has no read permission on Sales Order.
- **Fix:** sharing grants access to one document only — it never replaces the
  role permission. Give the role read on the DocType in Role Permissions
  Manager, then the share works.

## Case 2: list shows nothing

- **Symptom:** the list view is empty for one user, fine for everyone else.
- **Likely cause:** a User Permission (e.g. Company) or a
  `permission_query_conditions` hook is filtering everything out.
- **Fix:** open the user's **User Permissions** and check for a Company /
  Branch restriction. Then check the hook's condition — test it with the
  user's own roles, never as Administrator.

## Case 3: report is empty for a user

- **Symptom:** a Script or Query Report returns rows for you, none for a user.
- **Likely cause:** the report runs `frappe.get_list` (permission-aware) and
  the user lacks the role, or it filters by a company the user cannot see.
- **Fix:** check the user's roles and User Permissions first. Only then check
  whether the report itself restricts companies or uses `ignore_permissions`.

## Case 4: API call returns 403

- **Symptom:** `GET /api/resource/Sales Order/ABC` works in Desk, fails from the API.
- **Likely cause:** the token belongs to a different user who lacks the role;
  or for `/api/method/...`, the method has no `@frappe.whitelist()` decorator.
- **Fix:** check which user owns the API key and secret, and give that user
  the role. For `/api/method/...`, confirm the decorator is present and that
  the method's own `has_permission` check passes for that user.

## Case 5: workflow action button missing

- **Symptom:** the Approve button does not appear for a user.
- **Likely cause:** the transition's **Allowed** role does not include the
  user's role, or the transition condition fails for this document.
- **Fix:** open the Workflow, check the transition's role and condition
  (`doc.fieldname`), and confirm only one workflow is active for the DocType.

## Checklist — run in order

1. Which user, exactly? Test as that user, never as Administrator.
2. Role Permissions Manager: does one of their roles have read on the DocType?
3. User Permissions: is a Company, Branch or Territory hiding the document?
4. `has_permission` hook: does it return an explicit `True` (v16)?
5. Whitelisted method: decorator present, permission checked first?
6. API: does the token belong to the user you think it does?
