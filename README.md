# AgenTeX

**Agentic QA for Claude Code — an agent plans, runs, and reports your tests so you don't click through them by hand.**

[![Version](https://img.shields.io/badge/version-0.8.1-blue.svg)](./CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-8A2BE2.svg)](https://docs.anthropic.com/en/docs/claude-code)
[![Playwright](https://img.shields.io/badge/Playwright-CLI-2EAD33.svg?logo=playwright&logoColor=white)](https://www.npmjs.com/package/@playwright/cli)
[![Azure DevOps](https://img.shields.io/badge/Azure%20DevOps-integration-0078D7.svg?logo=azuredevops&logoColor=white)](https://azure.microsoft.com/en-us/products/devops)

AgenTeX (Agentic Test eXecution) takes manual test execution off your plate. Instead of clicking the
same scenarios by hand, an agent plans them, drives a **real browser** via
[`@playwright/cli`](https://www.npmjs.com/package/@playwright/cli), captures screenshot/log evidence,
and produces a consolidated defect report — either **sequentially** (human-in-the-loop) or in
**parallel** (autonomous, one session per spec file). It **never modifies your application code**.

## [Getting Started](./docs/getting-started.md)

New here? **[Getting Started](./docs/getting-started.md)** walks you through install → browser driver
→ `/init-test` → permissions → first run. The short version:

```
/plugin marketplace add MhmdElGazzar/elgazzar-plugins
/plugin install agentex@elgazzar-plugins
/init-test
/execute-test https://example.com
```

## Features — how each one works

| Feature | How it works | Docs |
|---------|--------------|------|
| **Browser testing** | An agent plans scenarios, drives a real `playwright-cli` browser, screenshots each one, and reports defects — sequential (approve each step) or parallel (one `qa-executor` subagent per spec file). | [browser-testing](./docs/browser-testing.md) |
| **API & DB steps** | `api:` / `db:` scenario steps run **only** the named, parameterized requests/queries in your `integration/` catalog — the agent never composes its own SQL or HTTP; DDL is refused. | [api-db-steps](./docs/api-db-steps.md) |
| **Ask the KB** | `kb:` steps (or `/ask-kb`) query your project's KB Ask API for advisory context — informs testing, **never** used as PASS/FAIL evidence. | [ask-kb](./docs/ask-kb.md) |
| **Optimize login** | Pay a web app's login once per session: drive it live, verify by landmark (never by URL), save the browser session, and reload it into a fresh browser to continue. | [optimize-login](./docs/optimize-login.md) |
| **Azure DevOps planning** | `/estimate-story` estimates QA effort and creates 5 `[Testing]` tasks per story; `/design-test` turns story ACs into linked test cases — both via the `az` CLI, with confirmation. | [azure-devops](./docs/azure-devops.md) |
| **Azure DevOps bug filing** | After a run, `bug-report-azure` files found defects as ADO **Bugs** via the `az` CLI — recommends severity/priority, links each to its parent User Story, validates & attaches screenshots, optionally fails the related test case; all behind one confirmation. | [azure-devops](./docs/azure-devops.md) |
| **HTML report** | At the end of a run, generates a standalone, self-contained `extent-report.html` dashboard (donut chart, status cards, expandable per-test-case steps). | [extent-report](./docs/extent-report.md) |
| **Configuration** | A keys-only `.env` drives targets and integrations; catalog files hold only env-var *names*, so secrets stay in the environment. | [configuration](./docs/configuration.md) |

See [docs/](./docs/) for the full reference on any feature.

## Usage at a glance

```
# Sequential (human-in-the-loop) — natural language:
Test https://example.com — the signup form: happy path plus empty and bad-email cases.

# Parallel (autonomous) — one subagent per spec file:
Run a parallel regression against https://example.com from the specs in test/suite1/.

# Slash commands:
/execute-test https://example.com
/estimate-story 12345 12346
/design-test 12345
/ask-kb acme-store: how does the checkout flow work?
```

Every run writes to a timestamped `executions/execu_<timestamp>/` folder — `report.md`,
`extent-report.html`, per-session logs/screenshots, and a merged bug list.

One-time setup:

1. Azure CLI + DevOps extension: install `az` (see `skills/azure-integration/references/azure-cli.md`),
   then `az extension add --name azure-devops`.
2. Fill the `AZURE_*` keys in `.env` (`/init-test` scaffolds it keys-only): organization URL,
   project, team, default assignee.
3. Auth: `az login`, or for non-interactive use export a PAT in your shell:
   `export AZURE_DEVOPS_EXT_PAT=<your-pat>` (never committed, never printed by the agent).

Then run `/estimate-story` for the current sprint, or `/estimate-story 12345 12346` for
specific stories. The agent processes **one story at a time** and never creates tasks without
your confirmation.

## What's inside

| Component | File | Purpose |
|-----------|------|---------|
| Skill | `skills/browser-testing/SKILL.md` | The orchestrator workflow — modes, output layout, defect format, rules |
| Skill | `skills/azure-integration/SKILL.md` | Reach Azure resources during a run via the `az` CLI |
| Skill | `skills/task-estimation/SKILL.md` | Estimate QA effort and create `[Testing]` tasks on Azure DevOps stories |
| Skill | `skills/test-design/SKILL.md` | Analyze story ACs into test conditions; create & link test cases in ADO |
| Skill | `skills/api-integration/SKILL.md` | Execute cataloged API calls in test steps (`api:`) via a runner script |
| Skill | `skills/db-integration/SKILL.md` | Execute cataloged DB queries in test steps (`db:`) via a runner script |
| Skill | `skills/ask-kb/SKILL.md` | Ask the project's KB Ask API in test steps (`kb:`) for advisory answers (never evidence) |
| Skill | `skills/extent-report/SKILL.md` | Interactive HTML dashboard (`extent-report.html`) for a finished run |
| Skill | `skills/figma-integration/SKILL.md` | Read a Figma design → user stories & test conditions that feed `test-design`/`browser-testing`; compare a design against a live build; optionally sync design↔code via the `@figma/code-connect` CLI |
| Skill | `skills/jira-integration/SKILL.md` | Reach Jira & Confluence via the `acli` CLI — file defects, build issue hierarchies, plan sprints, publish reports |
| Agent | `agents/qa-executor.md` | Subagent that runs one test spec in its own isolated browser session |
| Reference | `skills/browser-testing/references/playwright-cli.md` | The browser driver — setup & gotchas |
| Reference | `skills/azure-integration/references/azure-cli.md` | `az` CLI — install/auth/common commands |
| Reference | `skills/azure-integration/references/azure-devops-cli.md` | `az boards` / `az devops` basics — shared by the ADO skills |
| Reference | `skills/test-design/references/test-case-mechanics.md` | Test Case creation — Steps XML, linking direction, gotchas |
| Template | `skills/test-design/templates/test-template.md` | Project conventions template — scaffolded to `.agentex/` in your project |
| Reference | `skills/api-integration/references/api-requests.md` | Runner usage + curl fallback for cataloged API requests |
| Reference | `skills/db-integration/references/sqlcmd.md` | Runner usage + sqlcmd (SQL Server) for cataloged queries |
| Reference | `skills/ask-kb/references/kb-ask-api.md` | KB Ask API contract, result handling & curl fallback |
| Reference | `skills/figma-integration/references/figma-cli.md` | Figma REST reads (design → stories/specs) + `@figma/code-connect` CLI (install/auth/authoring) |
| Reference | `skills/jira-integration/references/atlassian-cli.md` | `acli` for Jira — install/auth, issues, hierarchy, boards, sprints, attachments |
| Reference | `skills/jira-integration/references/confluence-cli.md` | `acli` for Confluence — spaces, pages (REST), labels, comments, restrictions |
| Reference | `skills/jira-integration/references/admin-cli.md` | ⚠️ opt-in org-admin user management — separate auth, confirm-per-write |
| Scripts | `skills/*/scripts/*.js` | Deterministic runners & helpers: `run_api`, `run_db`, `ask_kb`, `extract_visible_text`, `preflight`, `init_run`, `merge_run` |
| Templates | `skills/{api,db}-integration/templates/sample_{api,db}.json` | Catalog samples — scaffolded to `integration/` in your project |
| Script | `skills/extent-report/scripts/make_html_report.js` | Standalone HTML dashboard generator (run via `node`) |
| Command | `commands/init-test.md` | `/init-test` — scaffold sample specs + `executions/` in your project |
| Command | `commands/execute-test.md` | `/execute-test <url or scope>` — run the tests |
| Command | `commands/ask-kb.md` | `/ask-kb <question>` — ask the project's Knowledge Base a question (advisory only) |
| Command | `commands/estimate-story.md` | `/estimate-story [ids]` — estimate & create QA tasks on ADO stories |
| Command | `commands/design-test.md` | `/design-test <ids>` — design & create linked test cases on ADO stories |
| Permissions | `settings.example.json` | Recommended permission rules to copy into your project |
| Example specs | `test/suite1/` | Ready-to-adapt sample test specs — one file per browser session |
| Output | `executions/` | Where each run's report, screenshots & defect list land (auto-created) |
| Config | `.env.example` | Optional operational values (target URL, Azure DevOps) |

## Install

AgenTeX installs through the **`elgazzar-plugins`** marketplace — that's the one path to use.
From Claude Code:

```
/plugin marketplace add MhmdElGazzar/elgazzar-plugins
/plugin install agentex@elgazzar-plugins
```

> **Developing or testing a local clone?** Point the marketplace at your local copy of the
> `elgazzar-plugins` repo instead of GitHub, then install the same way:
> ```
> /plugin marketplace add /path/to/elgazzar-plugins
> /plugin install agentex@elgazzar-plugins
> ```
> (`/plugin marketplace add` needs a repo that contains `.claude-plugin/marketplace.json`,
> which is the `elgazzar-plugins` repo — not this plugin repo.)

## Contributing

New to the codebase? **[docs/contributing/](./docs/contributing/README.md)** teaches Claude
Code concepts from zero, AgenTeX's architecture, and walks through adding a skill end to end.
Open issues and PRs on the [GitHub repository](https://github.com/MhmdElGazzar/agentex).

## Contributors

- **Mohamed Elgazzar** — creator & maintainer
- **Marwah Zain**
- [**@mabdel130**](https://github.com/mabdel130) — `extent-report` skill (PR #1)
- **YoussefKhalilTester**
- **Hager-Helmy**

## License

MIT — see [LICENSE](./LICENSE).
