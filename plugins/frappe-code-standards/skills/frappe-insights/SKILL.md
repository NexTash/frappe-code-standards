---
name: frappe-insights
description: Frappe Insights open-source BI app - queries, workbooks, charts and dashboards. Use when building analytics or dashboards across DocTypes, choosing between Insights and Frappe reports, or sharing a dashboard.
---

# Frappe Insights

Insights is a completely open-source BI tool (the `insights` app). Connect to
your databases, analyse data, and build reports and dashboards without writing
complex SQL. From v16.60 the Framework supports Insights dashboards, and native
ERPNext and HR dashboards run on the Insights engine.

## Install

Install the Insights app on the bench (`bench get-app insights`), then install
it on the site. In v16.60, **System Settings** has a **Dashboards** option to
choose Insights or classic dashboards: new sites use Insights by default, while
upgraded sites keep classic dashboards until the setting is changed.

## Concepts

| Piece | What it is |
|---|---|
| Workbook | Top-level container — holds queries, charts, and dashboards |
| Query | Fetches and transforms data from a data source |
| Chart | Visualises one query's results |
| Dashboard | Several charts in a grid, with filters and text blocks |

## Queries — three ways to build

1. **Visual Query Builder** (recommended): no-code, step-by-step.
2. **SQL Editor**: write SQL directly.
3. **Script Editor**: Python scripts for complex transformations.

Query operations, each building on the previous step: **Source**, **Join Table**,
**Append Table**, **Filter Rows**, **Choose Columns**, **Add New Column**,
**Group & Summarise**, **Custom Operation** (Ibis expressions). Aggregations:
count, count distinct, sum, avg, min, max. The expression language uses Python
syntax and is built on the Ibis library.

Data sources: Site DB, demo data, uploads, other queries in the workbook, and
external databases — MySQL, PostgreSQL, DuckDB, and BigQuery.

## Charts and dashboards

- Charts: pick a chart type for a query's results; Insights suggests the best
  chart for a given result set.
- Dashboards: drag charts onto a grid and arrange them. One filter can control
  multiple charts; use dependent filters (e.g. Country, then City) for linked
  filtering. Add text blocks for headers and explanations.
- Share a dashboard via a public link, let it be viewed without login, or embed
  it in another website.

## Insights vs Frappe reports

| Need | Use |
|---|---|
| Cross-DocType analysis, ad-hoc exploration, filtered dashboards | Insights |
| Operational report inside Desk, with print format and export | Query / Script Report — see **frappe-reports-print** |
| Numbers on a Desk workspace (single value, small chart) | Number Card / Dashboard Chart — see **frappe-ui** |
| Scheduled email of a report | Report + **frappe-notifications** |

## Permissions — read this first

- A dashboard shared by public link shows its query results to **anyone with
  the link** — it does not re-check each viewer's DocType permissions. Never
  put salary, margin, or customer data on a publicly shared dashboard.
- Keep internal dashboards to logged-in users and share workbooks only with
  the roles that should see them.
- Insights reads the database directly. If a viewer must not see a row, do not
  put that row in the query — filtering it in the chart is not enough.
