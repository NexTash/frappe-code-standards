# Changelog

All notable changes to this project are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [2.0.0] - 2026-11-24

### Changed

- The single `frappe-code-standards` skill is replaced by `frappe-core` plus focused skills. Claude loads only the skill a task needs.
- Supports Frappe v14, v15 and v16.

### Added

- Skills: `frappe-architect`, `frappe-data`, `frappe-debug`, `frappe-erpnext-modules`, `frappe-flow`, `frappe-insights`, `frappe-integrations`, `frappe-notifications`, `frappe-ops`, `frappe-performance`, `frappe-permissions`, `frappe-reports-print`, `frappe-scheduler`, `frappe-security`, `frappe-server-scripts`, `frappe-studio`, `frappe-testing`, `frappe-ui`, `frappe-upgrade`, `frappe-website`, `frappe-workflow`.
- `frappe-core/references/field-layout.md` for DocType field layout.
- Agents: `frappe-builder`, `frappe-reviewer`, `frappe-doctor`, `frappe-migrator`.
- Commands: `/frappe-review`, `/frappe-new-field`, `/frappe-upgrade-check`.

### Removed

- `frappe-code-standards` skill. Use `frappe-core`; performance and reports rules moved to `frappe-performance` and `frappe-reports-print`.

## [1.0.5]

- Single `frappe-code-standards` skill with placement, server, client, hooks, performance, reports and review references.
