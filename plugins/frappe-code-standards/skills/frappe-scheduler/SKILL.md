---
name: frappe-scheduler
description: Frappe/ERPNext scheduler events, background jobs and queues. Use when running code on a schedule, moving slow work out of requests, debugging jobs that never run, or choosing the right queue and timeout.
---

# Scheduler and Background Jobs

## 1. Scheduled work — `scheduler_events` in hooks.py

```python
# hooks.py
scheduler_events = {
    "all": ["my_app.tasks.ping"],
    "hourly": ["my_app.tasks.hourly_sync"],
    "daily": ["my_app.tasks.send_daily_summary"],
    "weekly": ["my_app.tasks.weekly_cleanup"],
    "monthly": ["my_app.tasks.monthly_archive"],
    "cron": {"0 2 * * *": ["my_app.tasks.nightly_job"]},
}
```

- Keys: `all`, `hourly`, `hourly_long`, `daily`, `daily_long`, `weekly`,
  `weekly_long`, `monthly`, `monthly_long`, `cron`.
- `cron` values are standard cron strings (minute hour day month weekday).
- `*_long` variants run on the long queue. Keep each task short and
  idempotent — the scheduler will run it again on the next tick.

## 2. Background jobs — `frappe.enqueue`

```python
frappe.enqueue(
    "my_app.jobs.recalculate_balances",
    queue="long",
    timeout=1500,
    job_id=f"recalc-{company}",
    enqueue_after_commit=True,
    company=company,
)
```

- Queues: `short` (quick jobs), `default`, `long` (heavy jobs, 1500s timeout).
- `enqueue_after_commit=True` so the job only runs if the transaction commits.
- `job_id` deduplicates: the same id will not run twice at once. Check it with
  `frappe.utils.background_jobs.is_job_enqueued(job_id)`.
- Pass simple arguments (strings, numbers). Never pass a document object.

## 3. Never block a request

- Never call an external API in `validate` / `before_save`. Enqueue from
  `on_update` / `on_submit` instead.
- Slow work (bulk processing, bulk email, report generation, imports) always
  goes through `frappe.enqueue`.
- Tell the user when it finishes: `frappe.publish_realtime` or a notification.

## 4. Scheduler on / off

- `bench --site <site> enable-scheduler` / `disable-scheduler`.
- Pause without disabling: `"pause_scheduler": 1` in `site_config.json`.
- Test one event by hand: `bench --site <site> trigger-scheduler-event daily`.
- On Frappe Cloud shared benches there is no shell — manage the scheduler
  from the dashboard.

## 5. Health checks

- `bench doctor` — are the workers and the scheduler alive?
- **Scheduler Log** in Desk — when did each event last run, and did it fail?
- **Error Log** — tracebacks from failed jobs.
- Job never runs: scheduler paused, workers down, or the site disabled.
  `bench doctor` shows all three.

## 6. Writing safe jobs

- Jobs must be **idempotent**: running twice must not double-post, double-send
  or double-charge. Store the external id on the document and check it first.
- Catch errors and log them: `frappe.log_error(title=..., message=frappe.get_traceback())`.
- Commit in batches in long jobs (`frappe.db.commit()` every few hundred
  records) so one failure does not lose hours of work — and so the job can
  resume where it stopped.
- A job that sends email must not use `sendmail(now=True)`; queue it.
