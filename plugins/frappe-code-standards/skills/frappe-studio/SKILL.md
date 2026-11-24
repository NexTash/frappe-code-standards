---
name: frappe-studio
description: Frappe Studio visual low-code app builder (Framework v16.60+). Use when building a custom internal app or bespoke UI without writing code, exporting a Studio app into a Frappe app, or deciding between Studio and writing code.
---

# Frappe Studio

Studio is a visual, low-code app builder. It is part of Frappe Framework from
v16.60: build an entire app — pages, layout, data — without writing code, and it
goes live immediately. It also works with AI agents (including Flow), so an agent
can build the app for you.

## What it is

- **Studio App**: the top-level record — title, home page, icon, public route.
- **Studio Page**: one screen — route, title, a block tree (live + draft), data
  resources, variables, and a page script.
- **Blocks**: drag-and-drop layout built from frappe-ui components. Edit any
  component's props and slots in the editor.
- **Data**: wire Frappe data sources into a page with minimum configuration;
  form and CRUD utilities speed up standard screens.
- **Behaviour**: reactive apps with dynamic variables, scripts, and watcher support.
- **Components**: reuse Studio components, or drop in custom Vue components.

## Open it

- Go to `/studio` on the site, or click the framework icon → Studio on the dock.
- Developer Mode is **not** required — unless you want to export the app.

## Studio vs writing code

| Need | Use |
|---|---|
| Small internal app, custom screen on top of ERPNext, portal | Studio |
| DocType business logic, validation, server rules | Code — see **frappe-core** |
| Custom Desk screens, dashboards, dialogs | Code — see **frappe-ui** |
| New feature / app design from scratch | **frappe-architect** first, then Studio or code |

Studio pages do not replace server logic. Rules, validation, and permissions
still belong on the server (see **frappe-core**).

## Export to code

By default Studio writes **no files** — the app lives in the database as Studio
App and Studio Page records. To ship it with your own Frappe app:

1. Turn on Developer Mode.
2. Open the app in Studio and click **Export App** on the toolbar.
3. Turn on **Enable App Export**, select the Frappe App, click **Update**.
4. Studio writes the app to `apps/<your_app>/studio/<app_name>/`.

Commit that folder to git like any other app code. Saving an exported app writes
its definition back to the repository.

## AI agents and Studio

- MCP is on by default. Point an agent (Claude Code, Codex, Cursor, VS Code)
  at `<site>/api/v2/mcp`.
- The guide agents use to build Studio apps is in the Frappe repo at
  `skills/studio-app-building`.
- Ask the agent for one screen at a time, then check it in Studio before
  publishing — a published page copies the draft blocks to live.

## Limits

- Studio is new in v16.60 and its docs are still in progress. Expect the editor
  and the export format to change in later v16 releases.
- Keep Studio apps small and export to git early; do not let a business-critical
  app live only in the database.
- Test on a staging site first. Studio is not a substitute for tested server
  code on anything that moves money or stock.
- Do not confuse Studio with Frappe Builder — Builder builds website pages,
  Studio builds apps.
