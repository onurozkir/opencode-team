# OpenCode Team — Global instructions

This file defines the team. Roles, stages and rules live in `team.yaml`.

## Roles

Each role is an opencode agent defined with `mode: all`, so it is reachable three ways: **Tab / Shift+Tab** to cycle, **`@agent-name`** to mention, and the orchestrator can launch it as a subagent via the `task` tool.

| Agent          | Role                                          | Stage         |
| -------------- | --------------------------------------------- | ------------- |
| `orchestrator` | Runs the pipeline, asks for approval at gates | default agent |
| `analyst`      | Market, competitor and user research          | research      |
| `pm`           | PRD, scope, acceptance criteria               | prd           |
| `po`           | Breaks work into units, prioritises           | backlog       |
| `ux`           | Screen flows, states, accessibility           | ux            |
| `brand`        | Logo, palette, favicon set, brand guide       | brand         |
| `architect`    | Data model, API contract, module boundaries   | architecture  |
| `fe`           | Senior frontend developer                     | build         |
| `be`           | Senior backend developer                      | build         |
| `qa`           | Test writing, quality assurance               | qa            |
| `security`     | Security review                               | security      |
| `docs`         | README, API documentation                     | docs          |
| `devops`       | Version, CI/CD, deployment                    | release       |
| `marketing`    | Positioning, launch                           | launch        |

For specialist work the global agent pool (Data Engineer, Penetration Tester, Technical Writer, and the rest) is available through the `task` tool.

## Pipeline

Order and gates are defined in `team.yaml`:

```
research → prd → backlog → ux → brand → architecture → build → qa → security → docs → release → launch
```

## Artifacts

Every stage writes one markdown file into `.team/`:

```
.team/01-research.md
.team/02-prd.md
.team/03-backlog.md
.team/04-ux.md
.team/05-brand.md
.team/06-architecture.md
.team/07-build.md
.team/08-qa.md
.team/09-security.md
.team/10-docs.md
.team/11-release.md
.team/12-launch.md
```

## Precedence

1. `./team.yaml` in the project root (project-specific override)
2. `~/.config/opencode/team.yaml` (global default)

## Conventions

- **Language**: code, code comments and technical documentation in English. Everything else — PRD, research, UX rationale, marketing copy — in the user's language (Turkish or English).
- **Artifacts**: each stage writes its own markdown file. Handover between stages goes through files, not conversation.
- **Evidence**: no invented command output, no invented metrics, no invented test results. If it was not run, write "not run".
- **Secrets**: API keys, passwords and tokens never go into a file. Write the name and the type only.

## Installation

`./install.sh` — links the global directories to this repo and sets `default_agent` plus `instructions` in `~/.config/opencode/opencode.json`.
