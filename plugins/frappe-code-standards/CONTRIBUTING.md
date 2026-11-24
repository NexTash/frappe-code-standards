# Contributing

Thanks for helping make these skills better. Rules should come from real Frappe
work, not theory — if Claude kept making a mistake on a real project, that is
the best kind of contribution.

## Adding a new skill

1. Create `skills/<kebab-case-name>/SKILL.md` with YAML frontmatter:
   - `name:` in kebab-case, matching the folder name.
   - `description:` under 1024 characters, starting with what the skill is
     and a `Use when...` trigger (for example: "Frappe/ERPNext X. Use when
     doing Y.").
2. Keep the `SKILL.md` body short: tables over prose, concrete right/wrong
   examples over general advice.
3. Put depth in `references/` files (worked examples, error tables, checklists)
   and link to them from the `SKILL.md` — do not bloat the skill body itself.
4. Add the skill to the table in `README.md`, the routing table in
   `templates/CLAUDE.md`, and the list in `AGENTS.md`.

## Fixing a rule

- Show a concrete right and wrong example where it helps.
- Cite the official [Frappe documentation](https://docs.frappe.io/framework)
  when a rule depends on framework behaviour or a version difference.
- Never advise editing another app's files (Frappe, ERPNext, HRMS) — extend
  through hooks and fixtures instead.

## Before submitting

- Run the CI checks locally if you can (see `.github/workflows/quality.yml`):
  every `SKILL.md` needs frontmatter with `name:` and `description:`
  (description under 1024 chars), every `*.json` file must be valid JSON,
  and every local `.md` link in `README.md` must resolve.
- Run `claude plugin validate` on your machine if you changed anything under
  `.claude-plugin/`.
- Bump the version in `.claude-plugin/plugin.json` **and**
  `.claude-plugin/marketplace.json` **and** add an entry to `CHANGELOG.md`.

## Pull requests

- Open PRs against the `develop` branch. The default branch is `develop`,
  not `main`.
- One change per PR, with a short description of the bad output Claude
  produced and what the fix should be.
