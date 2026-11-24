---
name: frappe-erpnext-modules
description: Core ERPNext module flows - selling, buying, stock, manufacturing, accounts, plus HRMS, CRM, Helpdesk, Projects and Assets. Use when building or debugging an ERP flow, when stock or ledger balances look wrong, or when deciding which document owns a change.
---

# ERPNext Module Flows

## Document status

Every transactional document has `docstatus`: `0` = Draft (editable),
`1` = Submitted (final, locked), `2` = Cancelled. State changes happen by
submitting or cancelling the owning document — never by setting fields or
writing to the database directly. To correct a submitted document, cancel it
and amend it; amending creates a new draft linked by `amended_from`.

## Selling

Quotation → Sales Order → Delivery Note → Sales Invoice → Payment Entry.

- A Delivery Note moves stock out; a Sales Invoice books the sale.
- An invoice can be created from the order without a delivery (service sales).
- Returns use a Sales Invoice with `is_return = 1` against the original.

## Buying

Material Request → Purchase Order → Purchase Receipt → Purchase Invoice → Payment Entry.

- A Purchase Receipt moves stock in and books the stock value.
- A Purchase Invoice books the supplier liability; it can arrive before or
  after the receipt.
- Returns use a Purchase Invoice with `is_return = 1`.

## Stock

- A **Stock Ledger Entry (SLE)** is created automatically when a stock
  transaction is submitted (Delivery Note, Purchase Receipt, Stock Entry).
  **Never insert SLE rows directly.**
- Stock Entry types: Material Issue, Material Receipt, Material Transfer,
  Manufacture, Repack, Material Transfer for Manufacture.
- Stock balances live on the **Bin** (per Item + Warehouse): `actual_qty`,
  `projected_qty`, `reserved_qty`. Read quantities from Bin or from a stock
  report — never by summing SLE rows in Python.
- Wrong balance? Use **Stock Reconciliation** to correct it, with a posting
  date and a reason. Never edit Bin or SLE by hand.

## Manufacturing

BOM → Work Order → Stock Entry (Manufacture) → finished goods in stock.

- The BOM lists raw materials and operations for one finished item.
- A Work Order tracks the required raw materials and the produced quantity.
- The Manufacture Stock Entry consumes raw materials and adds finished goods.
- Track time with Job Cards / Timesheets where the process needs it.

## Accounts

- A **GL Entry** is created automatically when an invoice, payment or journal
  is submitted. **Never insert GL rows directly.**
- Payment Entry records money in/out against invoices; Journal Entry is for
  manual adjustments between accounts.
- Balances come from the General Ledger report, not from summing GL Entry in
  code — and never from a document's own totals after submit.
- Every transaction posts to a Company and a Cost Center; a document without
  the right company posts to the wrong books.

## HRMS

App: `frappe/hrms`. Employee itself lives in ERPNext
(`erpnext/setup/doctype/employee`).

Key DocTypes: Employee, Leave Application, Leave Allocation,
Leave Ledger Entry, Attendance, Employee Checkin, Shift Type,
Shift Assignment, Salary Structure, Salary Structure Assignment,
Salary Slip, Payroll Entry, Expense Claim, Employee Advance,
Job Opening, Job Applicant, Job Offer.

- Employee → Leave Allocation → Leave Application → Attendance →
  Salary Structure → Salary Structure Assignment → Payroll Entry →
  Salary Slip. Expense Claim → approval → Payment Entry.
- Leave balances come from **Leave Ledger Entry** — never compute them
  from Leave Applications in code.
- Attendance comes from Employee Checkin + Shift Assignment / Shift Type
  — never insert Attendance rows by hand.
- Payroll order is fixed: Salary Structure → Salary Structure Assignment
  → Payroll Entry → Salary Slip. Never create a Salary Slip by hand.
- Never edit a submitted Salary Slip — cancel and amend it like any
  submitted document.
- HRMS is a separate app — extend it with hooks/fixtures, never edit its
  files.

## CRM

Two CRMs exist — do not mix them. ERPNext built-in: Lead, Opportunity,
Customer. Separate app `frappe/crm` (Vue frontend): CRM Lead, CRM Deal,
CRM Organization, CRM Contacts, CRM Task, CRM Call Log, CRM Lead Status,
CRM Deal Status.

- ERPNext: Lead → Opportunity → Quotation → Sales Order.
  frappe/crm: CRM Lead → CRM Deal → won / lost.
- Convert, don't copy: a Lead becomes a Customer / Deal through
  conversion, never by creating the target by hand.
- Deal stages come from CRM Deal Status / Opportunity stages — don't
  hardcode stage names in code.
- SLA and response times come from CRM Service Level Agreement — don't
  compute them from timestamps.
- The frappe/crm frontend is Vue — customize with CRM Form Script /
  CRM Fields Layout, never by editing its frontend files.

## Helpdesk

App: `frappe/helpdesk` (Vue frontend).

Key DocTypes: HD Ticket, HD Agent, HD Team, HD Ticket Type,
HD Ticket Priority, HD Ticket Status, HD Service Level Agreement,
HD Article, HD Customer.

- Ticket arrives (email / portal / API) → triage (type, priority) →
  assign (team, agent) → reply → resolve → feedback.
- Tickets arrive through email / portal / API — never insert HD Ticket
  rows directly in code without the proper defaults.
- The SLA clock comes from HD Service Level Agreement + HD Ticket
  Priority — don't compute response / resolution deadlines in code.
- Assignment goes through HD Team / HD Agent — don't set the agent by
  hand on the ticket.
- Customer replies live in HD Ticket Comment; the knowledge base in
  HD Article.
- Helpdesk is a separate Vue app — customize with HD Form Script /
  HD Field Layout, never by editing its frontend.

## Projects

ERPNext module.

Key DocTypes: Project, Task, Timesheet, Timesheet Detail, Project Type,
Project Template, Activity Type, Activity Cost.

- Project → Task (with dependencies) → Timesheet → Sales Invoice (billing).
- Project % complete comes from its Tasks — never set it by hand.
- Task dependencies use Task Depends On — don't order tasks by creation
  date.
- Time is tracked in Timesheet (linked to Project / Task); billing a
  Timesheet creates a Sales Invoice.
- Costs come from Activity Type / Activity Cost — don't hardcode rates.
- A Task belongs to one Project; moving work means moving the Task, not
  copying fields.

## Assets

ERPNext module.

Key DocTypes: Asset, Asset Category, Asset Maintenance, Asset Movement,
Asset Repair, Asset Value Adjustment, Asset Capitalization, Location,
Depreciation Schedule.

- Purchase Receipt / Invoice → Asset → depreciation (Depreciation
  Schedule) → maintenance / movement / repair → disposal.
- An Asset is created from a Purchase Receipt / Invoice or Asset
  Capitalization — never insert Asset rows by hand.
- Depreciation comes from the Asset Category finance books + Depreciation
  Schedule — never post depreciation with a Journal Entry by hand.
- Move assets with Asset Movement, fix them with Asset Repair, adjust
  value with Asset Value Adjustment.
- Maintenance is scheduled in Asset Maintenance and logged in
  Asset Maintenance Log.
- Disposal is a value adjustment / sale against the Asset — never delete
  an Asset with history.

## Golden rules

1. Business state changes only through submit/cancel of the owning document.
2. Never insert Stock Ledger Entry or GL Entry directly.
3. Never edit a submitted document's fields — cancel and amend instead.
4. Quantities come from Bin / reports; money comes from the ledger reports.
5. One document, one job: a Delivery Note moves stock, an Invoice books money.
