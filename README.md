<p align="center">
  <img src="plugins/frappe-code-standards/.claude-plugin/icon.png" alt="Frappe Code Standards" width="140">
</p>

<h1 align="center">Frappe Code Standards <sup>beta</sup></h1>

<p align="center">
  Make Claude write Frappe and ERPNext code the way an experienced Frappe developer would —<br>
  in the right file, with the framework's own APIs, and easy for the next developer to read.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-beta-orange" alt="Beta">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-black" alt="MIT License"></a>
  <img src="https://img.shields.io/badge/Frappe-v14%20%7C%20v15%20%7C%20v16-red" alt="Frappe v14 | v15 | v16">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-white" alt="Claude Code plugin">
</p>

---

> [!NOTE]
> **This plugin is in beta.** It will keep maturing as more people use it on real projects, and
> Claude can still make mistakes with it switched on, so review the output the way you would review
> a teammate's code. If something slips through, please [open an issue](../../issues/new/choose).
> Thank you for the support.

## Why this exists

AI assistants can write Frappe code that *works* but doesn't *belong*. The usual problems:

- Form logic for your own DocType dumped into `hooks.py` + `public/js` instead of the DocType's own `.js` file
- `raise Exception(...)` or silent returns instead of `frappe.throw(_("..."))`
- Untranslated user-facing strings
- `set_query` written in `refresh`, or replaced with hand-rolled filtering
- Validation done only in JavaScript
- Database calls inside loops, `fields=["*"]`, raw SQL built with f-strings

The code runs, but it's not where anyone on the team would look for it, and it fights the framework
instead of using it. This plugin gives Claude a clear set of standards so the output looks like it
was written by someone who has shipped Frappe apps before.

## Before and after

You ask: *"On Loan Application, only show active loan types for the selected company, and stop the
user if To Date is before From Date."*

**Without the plugin**

```python
# hooks.py
doctype_js = {"Loan Application": "public/js/loan_application.js"}   # our own DocType!
```

```python
# loan_application.py
def validate(self):
    if self.to_date < self.from_date:
        raise Exception("Invalid dates")
```

**With the plugin**

```js
// my_app/lending/doctype/loan_application/loan_application.js
frappe.ui.form.on("Loan Application", {
    setup(frm) {
        frm.set_query("loan_type", () => ({
            filters: { company: frm.doc.company, disabled: 0 },
        }));
    },
});
```

```python
# my_app/lending/doctype/loan_application/loan_application.py
def validate(self):
    self.validate_dates()

def validate_dates(self):
    if self.to_date and getdate(self.to_date) < getdate(self.from_date):
        frappe.throw(_("To Date cannot be before From Date"))
```

## What it enforces

- **Code placement** — your DocTypes keep logic in their own files; other apps are extended through hooks, never edited
- **Errors and messages** — `frappe.throw`, `frappe.msgprint` and the Error Log, used the way the framework intends
- **Translations** — every user-facing string wrapped for translation
- **Form scripts** — `set_query` in `setup`, form API for buttons and server calls
- **Data access** — explicit fields, no queries in loops, Query Builder over raw SQL
- **Security** — permission checks in whitelisted methods, parameterised queries
- **Review** — a checklist before finishing, plus the `bench` commands to run
- **Version safety** — v14, v15 and v16 differences, and what to change when upgrading

The core rules live in [`frappe-core/SKILL.md`](plugins/frappe-code-standards/skills/frappe-core/SKILL.md).
Focused skills cover the rest — reports, permissions, workflows, performance, debugging, testing,
ops and more — and Claude loads only the one the task needs, so context stays small.

## Installation

### Claude Code

```bash
claude plugin marketplace add NexTash/Frappe-Code-Standards
claude plugin install frappe-code-standards@nextash
```

To keep it up to date automatically, open `/plugin` in Claude Code → **Marketplaces** → `nextash`
→ **Enable auto-update**.

Turn it off for non-Frappe work, and back on later:

```bash
claude plugin disable frappe-code-standards@nextash
claude plugin enable frappe-code-standards@nextash
```

### claude.ai

Each skill is its own folder. Zip the ones you want and upload them under **Skills** in your
claude.ai settings. Start with `frappe-core`:

```bash
cd plugins/frappe-code-standards/skills
zip -r frappe-core.zip frappe-core
```

## Using it in a project

Copy [`templates/CLAUDE.md`](templates/CLAUDE.md) into the root of your Frappe project and fill in
the project facts: Frappe version, which apps are yours, your site name and the prefix you use for
custom fields. Claude reads this file at the start of every session, which tells it which DocTypes
it can edit directly and which ones must be extended through hooks.

Spot a mistake Claude keeps making in that one project? Add a line under **Project-specific rules**.

## What's inside

```
plugins/frappe-code-standards/
├── skills/
│   ├── frappe-core/              Core rules: placement, standard APIs, field layout, review checklist
│   ├── frappe-architect/         Plan a feature or app before coding
│   ├── frappe-data/              Data Import, bulk updates, migration and cleanup
│   ├── frappe-debug/             Tracebacks, 403/404/417, errors that don't show up
│   ├── frappe-erpnext-modules/   Selling, buying, stock, manufacturing, accounts, HRMS, CRM
│   ├── frappe-flow/              Frappe Flow AI assistant and agent framework
│   ├── frappe-insights/          Frappe Insights dashboards and queries
│   ├── frappe-integrations/      REST API, webhooks, OAuth, external APIs, Data Import
│   ├── frappe-notifications/     Email, Notification rules, alerts, realtime
│   ├── frappe-ops/               bench, sites, deployment, backup and restore
│   ├── frappe-performance/       Slow queries, N+1, caching, indexes, background jobs
│   ├── frappe-permissions/       Roles, User Permissions, permission hooks
│   ├── frappe-reports-print/     Script/Query Reports, Print Formats, Jinja, PDF
│   ├── frappe-scheduler/         Scheduler events, queues and jobs
│   ├── frappe-security/          Whitelisted methods, SQL injection, XSS, guest access
│   ├── frappe-server-scripts/    Server Scripts and the RestrictedPython sandbox
│   ├── frappe-studio/            Visual low-code app builder
│   ├── frappe-testing/           Unit and integration tests, CI
│   ├── frappe-ui/                Workspaces, dashboards, custom pages, frappe-ui
│   ├── frappe-upgrade/           v14 → v15 → v16 differences and removed APIs
│   ├── frappe-website/           Web pages, web forms, portal
│   └── frappe-workflow/          Workflows and approval chains
├── agents/
│   ├── frappe-builder.md         Builds a full feature: DocType, controller, JS and tests
│   ├── frappe-reviewer.md        Reviews code against the core rules
│   ├── frappe-doctor.md          Diagnoses errors
│   └── frappe-migrator.md        Plans a version upgrade
└── commands/
    ├── frappe-review.md          /frappe-review — findings table for your changes
    ├── frappe-new-field.md       /frappe-new-field — add a field the right way
    └── frappe-upgrade-check.md   /frappe-upgrade-check — scan for removed APIs
```

The plugin is plain Markdown. It runs no scripts, makes no network calls and collects no data.

## Contributing

Found a pattern Claude still gets wrong, or a rule that doesn't match how the Frappe community
works? [Open an issue](../../issues/new/choose) using the bug report or feature request template,
with a short example of the bad output and what it should have been. Pull requests against the
files in `skills/` are welcome too. Rules should be concrete, show a right and wrong
example where it helps, and follow the official
[Frappe documentation](https://docs.frappe.io/framework).

When changing the plugin, bump `version` in
[`plugin.json`](plugins/frappe-code-standards/.claude-plugin/plugin.json) so installed copies pick up
the update.

## Support

No paywall, no tip jar, no fine print. Just cleaner Frappe code. If it earned its place in your
workflow, a [star](https://github.com/NexTash/Frappe-Code-Standards) is the most eloquent thanks
you can offer. The coffee's on us. The star's on you.

## License

[MIT](LICENSE) © NexTash

<p align="center">
  <i>"Any fool can write code that a computer can understand. Good programmers write code that humans can understand."</i><br>
  — Martin Fowler<br><br>
  <sub>Crafted with curiosity and passion by <a href="https://github.com/faisal-zahoor">Faisal Zahoor</a> · <a href="https://github.com/NexTash">NexTash</a></sub>
</p>
