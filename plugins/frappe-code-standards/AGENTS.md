# Frappe / ERPNext rules for any AI coding assistant

This works with Claude, Codex, Cursor, Gemini, Copilot and others.

1. For any Frappe or ERPNext work, read `skills/frappe-core/SKILL.md` and follow it.
2. Then read **only** the branch that matches the task:

| Task | Read |
|---|---|
| Server Script (UI) | `skills/frappe-server-scripts/SKILL.md` |
| Reports, print formats, Jinja | `skills/frappe-reports-print/SKILL.md` |
| Slow code, caching, background jobs | `skills/frappe-performance/SKILL.md` |
| v14/v15/v16 differences, upgrades | `skills/frappe-upgrade/SKILL.md` |
| Errors, tracebacks | `skills/frappe-debug/SKILL.md` |
| Tests, CI | `skills/frappe-testing/SKILL.md` |
| bench, deploy, backup | `skills/frappe-ops/SKILL.md` |
| Design a feature / app | `skills/frappe-architect/SKILL.md` |
| Workflows, approvals | `skills/frappe-workflow/SKILL.md` |
| Email, notifications, realtime | `skills/frappe-notifications/SKILL.md` |
| REST API, webhooks, OAuth, Data Import | `skills/frappe-integrations/SKILL.md` |
| Website, portal, Web Forms | `skills/frappe-website/SKILL.md` |
| Workspaces, dashboards, Pages, dialogs | `skills/frappe-ui/SKILL.md` |
| Permissions, roles, User Permissions, 403 | `skills/frappe-permissions/SKILL.md` |
| Scheduler events, background jobs, queues | `skills/frappe-scheduler/SKILL.md` |
| ERPNext module flows (selling, stock, manufacturing, HRMS, CRM, Helpdesk, Projects, Assets) | `skills/frappe-erpnext-modules/SKILL.md` |
| Studio visual app builder (v16.60) | `skills/frappe-studio/SKILL.md` |
| Frappe Flow AI workflows | `skills/frappe-flow/SKILL.md` |
| Insights BI, dashboards, queries | `skills/frappe-insights/SKILL.md` |
| Security audit | `skills/frappe-security/SKILL.md` |
| Data Import, bulk updates, migration | `skills/frappe-data/SKILL.md` |

Don't read branches the task doesn't need. That keeps token use low.

For project setup, copy `templates/CLAUDE.md` into the Frappe project root and fill
in the project facts there.