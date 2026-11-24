---
description: Scan a Frappe project for APIs removed or changed before upgrading to a target version.
---

Check this project for upgrade blockers before moving to `$ARGUMENTS`
(default: v16 if no version is given).

## How to check

1. Read `__version__` in `apps/frappe/frappe/__init__.py` — that is the
   current version. Upgrade one major version at a time (v14 -> v15, then
   v15 -> v16).
2. Load `skills/frappe-upgrade/SKILL.md` and use its v14 -> v15 and
   v15 -> v16 tables.
3. Search the project (`*.py`, `*.js`, Server Scripts, `hooks.py`) for every
   old or removed API in those tables, for example `frappe.db.set`,
   `frappe.db.set_temp`, `frappe.compare`, `enqueue(job_name=...)`,
   `get_time_zone`, `frappe.flags.in_test`, `frappe.sendmail(now=True)`,
   and v16 changes such as `has_permission` hooks not returning explicit
   `True`, DB commits inside document hooks, and `/app` URLs.

## Output

1. A table of what was found:

   | Old API | New API | Files found in |
   |---|---|---|
   | ... | ... | ... |

   Only list APIs actually found in the code. Do not list table entries
   with no matches.
2. A checklist of manual steps: patches to write (`patches/` +
   `patches.txt`), fixtures to export, tests to run
   (`bench --site <test_site> run-tests --app <app>`), and a backup
   (`bench --site <site> backup --with-files --compress`) before any
   migrate.
3. End with the exact commands to run, in order. Do not run `bench migrate`
   or `bench update` yourself — wait for the user to confirm.
