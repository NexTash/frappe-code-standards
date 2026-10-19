# Frappe Code Standards

Makes Claude write Frappe Framework and ERPNext code the way an experienced Frappe developer
would: in the right file, with the framework's own APIs, and easy for the next developer to read.

**What it enforces**

- **Correct code placement.** Logic for your own DocTypes goes in the DocType's `.py` and `.js`.
  DocTypes from other apps (ERPNext, HRMS) are extended through `doc_events`, `doctype_js`, an
  `overrides/` module and fixtures, never by editing the other app.
- **Standard framework APIs.** `frappe.throw`, `frappe.msgprint`, `_()` / `__()` translations,
  `frm.set_query` in `setup`, `frappe.qb`, `frappe.enqueue` and `frappe.log_error`.
- **Performance and security.** No queries in loops, explicit fields, background jobs for slow
  work, permission checks in whitelisted methods and parameterised queries.
- **A review checklist** before every task is finished, plus the `bench` commands to run.

The plugin is plain Markdown guidance. It runs no scripts, makes no network calls and collects no
data.

**Beta:** the plugin will keep maturing with real-world use, and Claude can still make mistakes with
it switched on, so review its output as you would a teammate's. Issues and suggestions are welcome on
GitHub. Thank you for the support.

## Install (Claude Code)

```bash
claude plugin marketplace add NexTash/Frappe-Code-Standards
claude plugin install frappe-code-standards@nextash
```

## Use in a project

Copy `templates/CLAUDE.md` from the repository into your project root and fill in your Frappe
version, your apps, your site and your custom field prefix.

Full documentation: https://github.com/NexTash/Frappe-Code-Standards

> *"Any fool can write code that a computer can understand. Good programmers write code that humans can understand."*
> — Martin Fowler

Crafted with curiosity and passion by Faisal Zahoor at NexTash. MIT licensed.
