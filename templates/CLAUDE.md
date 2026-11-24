# Project instructions

This is a Frappe Framework / ERPNext project.

- For ALL code work in this repository, load and follow the `frappe-core` skill
  before writing code. Its placement rules, error-handling rules and review
  checklist are mandatory.
- Then load **only** the branch that matches the task:

| Task | Load |
|---|---|
| Server Script in the UI (sandbox, no imports) | `frappe-server-scripts` |
| Script/Query reports, print formats, Jinja | `frappe-reports-print` |
| Slow code, N+1, caching, background jobs | `frappe-performance` |
| v14/v15/v16 differences, upgrades, removed APIs | `frappe-upgrade` |
| Errors, tracebacks | `frappe-debug` |
| Tests, CI | `frappe-testing` |
| bench, deploy, backup/restore, Frappe Cloud | `frappe-ops` |
| Design a feature or a new app | `frappe-architect` |
| Workflows, approvals | `frappe-workflow` |
| Email, Notification rules, realtime | `frappe-notifications` |
| REST API, webhooks, OAuth, Data Import | `frappe-integrations` |
| Website, portal, Web Forms | `frappe-website` |
| Workspaces, dashboards, Pages, dialogs, Vue SPA | `frappe-ui` |
| Permissions, roles, User Permissions, 403 | `frappe-permissions` |
| Scheduler events, background jobs, queues | `frappe-scheduler` |
| ERPNext module flows (selling, buying, stock, manufacturing, accounts, HRMS, CRM, Helpdesk, Projects, Assets) | `frappe-erpnext-modules` |
| Studio visual app builder (v16.60) | `frappe-studio` |
| Frappe Flow AI workflows | `frappe-flow` |
| Insights BI, dashboards, queries | `frappe-insights` |
| Security audit | `frappe-security` |
| Data Import, bulk updates, migration | `frappe-data` |

- Do not load branches the task does not need. That keeps token use low.
- Do not use other skills for code in this repository unless I ask.
- Before finishing a task, run the review checklist in `frappe-core`
  (`skills/frappe-core/references/review-checklist.md`).

## Project facts

- Frappe version: <v14 / v15 / v16>
- ERPNext version: <v14 / v15 / v16, or not installed>
- Our app(s): <app_name> — DocTypes here are "ours", edit their `.py` / `.js` directly
- Other installed apps: <erpnext, hrms, ...> — extend only via hooks and fixtures, never edit
- Site for commands: <site_name>
- Field prefix for custom fields on other apps' DocTypes: <nx_>

## Project-specific rules

<!-- Add one line here every time Claude makes a mistake you don't want repeated. -->
