---
name: frappe-workflow
description: Frappe/ERPNext Workflows and approvals. Use when designing or fixing a Workflow - states, transitions, approval chains, roles, conditions, docstatus mapping, or applying workflow actions in code.
---

# Frappe Workflow

## Design first
1. List every stage (Draft, Pending Manager, Approved, Rejected, Cancelled).
2. Give each state a `doc_status`: 0 = draft/editable, 1 = submitted, 2 = cancelled.
3. Give each transition: Action, Next State, Allowed role, optional Condition.

## Rules
- There must be a state with docstatus 0. New docs start there.
- docstatus only moves forward: 0->0, 0->1, 1->1, 1->2. Never go from submitted back to draft, and never transition out of cancelled.
- **Intermediate approval states stay at docstatus 0.** Only the final "Approved" state is 1.
- Rejected is usually docstatus 0 (so it can be edited and resubmitted), or a terminal 0 state.
- Conditions use `doc.fieldname`: `doc.grand_total > 100000`, `frappe.session.user == doc.owner`.
- `allow_self_approval = 0` when the creator must not approve their own doc.
- Set the "Workflow State" field (`workflow_state`). Don't drive state with a custom Select field.
- Workflow is for **who may move a doc**. Put business validation in controller `validate` / `before_submit`.
- Ship the workflow from the app as a **fixture** (`Workflow`, `Workflow State`, `Workflow Action Master`).

## In code
```python
from frappe.model.workflow import apply_workflow, get_transitions
apply_workflow(doc, "Approve")           # respects roles/conditions
get_transitions(doc)                      # actions available to current user
```
Never set `doc.workflow_state = ...` directly to skip approvals.

## Debug
- Button missing: check the user's role, the transition condition, `allow_self_approval`, and that the workflow is active (only one active per DocType).
- "Not allowed to change state": the role is missing on the transition.
- Emails for workflow actions: enable "Send Email Alert" on the workflow, and check Email Account / Email Queue.
