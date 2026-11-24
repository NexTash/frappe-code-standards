# Backup & Restore Runbook

## Backup

```bash
bench --site <site> backup                 # database + site_config
bench --site <site> backup --with-files    # also public + private files
bench backup-all-sites                     # every site on the bench
```

- Backups land in `sites/<site>/private/backups/`. Copy them off the
  machine — a backup on the same disk is not a backup.
- Offsite: rsync or S3-sync the backups folder nightly (cron or a backup
  service). Frappe Cloud: backups run automatically from the dashboard
  (site → Backups); download one and verify it restores.
- Before a backup: check disk space (`df -h`) — a backup needs at least
  the size of the database + files free.

## Restore

```bash
bench --site <site> restore <path-to-sql.gz> \
  --with-public-files <path> --with-private-files <path>
bench --site <site> migrate        # always migrate after a restore
bench --site <site> clear-cache
```

- Restore to a NEW site first when the goal is checking data (`bench
  new-site <site>` then restore into it). Never test a restore on the
  live site.
- A restore replaces the database and files. Anything created after the
  backup is gone — say so to the user before restoring.

## Checks before a restore (do all of these)

1. Backup the current site first — the restore is destructive.
2. Confirm the backup file is complete (size > 0, `gunzip -t` passes).
3. Confirm versions match: the backup's app versions vs installed apps
   (`bench version`).
4. Confirm disk space: database size x 2 free (restore + temp).
5. Put the site in maintenance mode
   (`bench --site <site> set-maintenance-mode on`) and tell users.
6. After restore: `migrate`, `clear-cache`, `bench restart`, then open the
   site and check login + one key document.

## Frappe Cloud

- Backups: dashboard → site → Backups (automatic daily; manual on
  demand). Restore from the dashboard — no bench access needed.
- Never run bench commands over SSH on Frappe Cloud unless support tells
  you to — the setup differs from self-hosted and you can break the bench.
