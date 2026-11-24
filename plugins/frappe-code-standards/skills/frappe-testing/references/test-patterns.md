# Test Patterns

## Where tests live

- DocType tests: `<app>/<module>/doctype/<name>/test_<name>.py`
- App tests: `<app>/<module>/tests/test_<thing>.py`
- Shared test data: `test_records.json` next to the DocType, loaded with
  `frappe.get_test_records("DocType")`.

## Running tests

```bash
bench --site <site> run-tests --app <app>                    # whole app
bench --site <site> run-tests --app <app> --doctype <Name>   # one DocType
bench --site <site> run-tests --app <app> --test test_method_name
bench --site <site> run-tests --app <app> --failfast
bench --site <site> run-tests --app <app> --coverage
```

## Rules

- Tests are `unittest.TestCase` subclasses. Each test runs in a transaction
  that is rolled back after it. Never call `frappe.db.commit()` in a test.
- Create documents with `frappe.get_doc({...}).insert()`; use
  `frappe.get_test_records` for shared fixtures, not hand-built dicts in
  every test.
- Test the server, not the form: call controller methods and whitelisted
  methods directly. JS behaviour belongs in UI tests.
- One test, one assertion theme. Name tests after the behaviour:
  `test_blocks_negative_qty`, not `test_1`.
- A test that needs a submitted document must submit it (`doc.submit()`);
  rollback cleans it up — do not delete documents by hand in tests.

## Mocking

Mock external calls with `unittest.mock.patch`, never with a live API:

```python
from unittest.mock import patch

@patch("myapp.api.send_to_gateway")
def test_payment_posts_on_success(self, mock_send):
    mock_send.return_value = {"status": "ok"}
```

- Mock `frappe.sendmail` / `frappe.enqueue` the same way when testing code
  that sends mail or queues jobs. Assert the mock was called, not that mail
  arrived.
- Never mock `frappe.db` — if the test needs the database, let it use the
  test site's database (it rolls back).

## UI tests (Cypress, built in)

- Frappe's built-in UI tests run in Cypress. Helpers live in
  `frappe/tests/ui_test_helpers.py` (login, form, grid).
- Run them with `bench --site <site> run-ui-tests <app>` (add `--headless`
  for CI).
- UI tests cover what Python can't: form rendering, field layout
  (Section/Column breaks), buttons, dialogs, list view. Keep them short —
  one flow per test.
- Playwright is an alternative, not built into bench: put specs in
  `tests/ui/` and run `npx playwright test`. Use it only if the team already
  runs Playwright; don't mix Cypress and Playwright in one app.

## What to test first

1. Validation that must block a save (`validate` raising `frappe.throw`).
2. Money/stock math (totals, taxes, ledger effects) — assert on the created
   GL/SLE rows.
3. Permissions on whitelisted methods (a forbidden user gets an error).
4. Field layout for new fields (UI test that the form renders with the right
   sections).
