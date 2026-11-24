---
name: frappe-flow
description: Frappe Flow AI assistant and agent framework (Framework v16.60+). Use when setting up Flow, building an agent with triggers and knowledge, or deciding between Flow and server-side workflow code.
---

# Frappe Flow

Flow puts an AI assistant in your Frappe site, plus a framework for building
your own agents. It ships with Frappe from v16.60. The blog calls it alpha and
the docs call it beta — either way it is pre-release: test it on a copy of your
site before trusting it with live data.

## What Flow can do

- **Assistant**: ask in plain language in the Desk chat. It answers questions
  about your data, runs your reports, and — with your approval — creates,
  updates, and submits documents and takes them through a workflow.
- **Customise**: it can change the site — DocTypes, custom fields, workflows,
  print formats.
- **Agents**: build an agent for one kind of work, with its own instructions,
  tools, and knowledge.
- **Triggers**: run an agent on a schedule, or whenever a document changes.
- **Knowledge**: give an agent content it can search; agents also have memory.

Flow works with the permissions of the person asking, so it can never see or
change more than they can. Flow is not the same as Automation Flow, which runs
fixed steps without AI.

## Set it up

1. As System Manager, open `/flow` on the site.
2. Choose a model provider (Anthropic or OpenAI) and paste its API key, then
   pick the model new chats start with.
3. To run a model on your own server, choose **Custom provider** and enter the
   server's URL — any server that speaks the OpenAI API works, including
   Ollama and LM Studio.
4. Give users access with the **Flow User** role.
5. Optional, under **Settings > General**: **Helper Model** (small jobs such
   as naming chats — a cheaper model keeps costs down) and **Search Provider**
   (the service agents use to search the web).

You bring your own AI provider keys. Frappe says AI credits will eventually be
bundled directly on Frappe Cloud. ERPNext and Frappe HR will ship with skills
Flow can use — those are not out yet.

## Flow vs server-side workflow code

| Need | Use |
|---|---|
| "Show me undelivered orders from last month", draft a reply, fill a form | Flow assistant |
| Same recurring job, same steps, same result every time | **frappe-scheduler** / **frappe-workflow** |
| Approval chains, document states, audit trail | **frappe-workflow** (Workflow DocType) |
| Rules that must never be skipped (validation, money, stock) | Server code — see **frappe-core** |
| Recurring agent work (e.g. "every morning, list overdue quotations") | Flow agent + trigger |

## Rules

- **Never give Flow an irreversible action without approval.** It can make
  mistakes — the docs say so. Keep it off anything that moves money or stock
  until it leaves beta.
- **Permissions are the guardrail.** If a user cannot see a document, Flow
  cannot see it for them. Do not work around this with a privileged user.
- **One agent, one job.** A narrow agent with clear instructions and few tools
  beats a general agent that does everything badly.
- **Keep secrets out of knowledge bases.** Anything in a knowledge base can
  surface in an answer.
- **Extending Flow**: ship tools and agents from your app, and run Flow on your
  own bench with Flow Server — see the Flow docs under "Extending Flow" and
  "Flow Server".

## Do not confuse

- The `frappe-flow` package on PyPI is a separate CLI agent harness — it is
  not the Flow in your site.
