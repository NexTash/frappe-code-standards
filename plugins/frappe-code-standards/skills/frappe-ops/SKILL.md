---
name: frappe-ops
description: Frappe/ERPNext bench, sites, apps, deployment, backup and restore. Use for bench commands, new-site, get-app/install-app, migrate, production setup (nginx, supervisor, SSL, Docker), backups, restores or server issues.
---

# Frappe Ops

## Safety rules
- Never run bench as root. Use a dedicated `frappe` user.
- **Back up before** update, migrate, uninstall-app or restore.
- Pin versions: `--frappe-branch version-15` / `--branch version-15`.
- Upgrade one major version at a time (see the frappe-upgrade skill).
- Never delete app folders by hand. Use `bench remove-app`.
- Test restores on a staging site. A backup you never restored is not a backup.

## Everyday commands

```bash
bench init frappe-bench --frappe-branch version-15
bench new-site site.local
bench get-app erpnext --branch version-15        # download
bench --site site.local install-app erpnext       # activate
bench --site site.local migrate                   # after DocType/patch/fixture/code changes
bench build --app myapp                           # after public JS/CSS changes
bench --site site.local clear-cache               # after hooks.py changes
bench --site site.local console                   # Python shell
bench --site site.local execute myapp.tasks.run --kwargs '{"name": "X"}'
bench --site site.local enable-scheduler
bench --site site.local set-config key value
bench doctor                                      # workers / scheduler health
bench update --pull --patch --build               # update with care, back up first
```

## Backup / restore

```bash
bench --site site.local backup --with-files --compress
bench --site site.local restore db.sql.gz --with-public-files pub.tar --with-private-files priv.tar
bench --site site.local migrate && bench --site site.local clear-cache
bench setup backups    # cron
```
Backups go to `sites/<site>/private/backups/`. Copy them off the server.

## Production (bench)

```bash
sudo bench setup production frappe          # nginx + supervisor
sudo -H bench setup lets-encrypt site.example.com
bench config dns_multitenant on && bench setup nginx   # multiple sites, site name = domain
```
Disable the default nginx site. Use SSH keys only, no root login, and set `allow_cors` only to trusted domains.

## Production (Docker)
For new installs, Frappe recommends `frappe_docker` with Compose. Build a custom image with your apps
listed in `apps.json`, and keep sites and the DB in volumes.

## Frappe Cloud
There are no Server Scripts on shared benches, and no shell. Deploy custom apps from a Git repo through the dashboard.

### Frappe Cloud rules
- Make a manual backup before major changes, and keep your own off-platform copies.
- Test app updates on a staging site first.
- Custom domain: add a CNAME (never transfer DNS). SSL can take time after DNS propagates.
- Change site config through the dashboard, not by editing files.
- Use `pause_scheduler` instead of disabling the scheduler. Turn on maintenance mode for big imports.
