---
name: frappe-testing
description: Frappe/ERPNext tests and CI. Use when writing unit or integration tests for a DocType or app, running bench run-tests, or setting up CI (GitHub Actions) for a Frappe app.
---

# Frappe Testing

## Where
`<app>/<module>/doctype/<name>/test_<name>.py`. The file must start with `test_`, or the runner ignores it.

## Base class

| Version | Needs DB? | Class |
|---|---|---|
| v15+ | no (pure logic) | `from frappe.tests.classes import UnitTestCase` |
| v15+ | yes | `from frappe.tests.classes import IntegrationTestCase` |
| v14 | any | `from frappe.tests.utils import FrappeTestCase` |

Each test runs in a transaction that rolls back afterwards. Never call `frappe.db.commit()` in tests.
If you override `setUpClass`, call `super().setUpClass()`.

```python
import frappe
from frappe.tests.classes import IntegrationTestCase

class TestLoanApplication(IntegrationTestCase):
    def test_amount_must_be_positive(self):
        doc = frappe.get_doc({"doctype": "Loan Application", "amount": -1})
        self.assertRaises(frappe.ValidationError, doc.insert)
```

Rules:
- Test behaviour: validations throw, totals compute, submit/cancel work, permissions deny.
- Create the test data inside the test, or with `test_records.json` next to the DocType.
  Don't depend on production data.
- Check permissions with `frappe.set_user("test@example.com")`, and restore with `frappe.set_user("Administrator")`.
- Mock external HTTP calls (`unittest.mock.patch`). Never call real APIs in tests.

## Run

```bash
bench --site test_site set-config allow_tests true
bench --site test_site run-tests --app myapp
bench --site test_site run-tests --doctype "Loan Application"
bench --site test_site run-tests --doctype "Loan Application" --test test_amount_must_be_positive
bench --site test_site run-tests --app myapp --failfast --junit-xml-output report.xml
```

Use a separate test site, never production.

## CI (GitHub Actions)
Use the workflow that `bench new-app` generates (`.github/workflows/ci.yml`). It runs MariaDB and Redis
services, `bench init --frappe-branch version-15`, `get-app` the repo, `new-site`, `install-app`, then
`run-tests --app myapp`. Pin the same Frappe branch the app targets.
