# AI Company Handbook

A self-bootstrapping AI software company powered by OpenCode. 11 specialized agents — architect, frontend, backend, reviewer, devops, and more — that work together to build software. Initialize in any project directory with a single command.

## Quick Start

```bash
mkdir my-project && cd my-project
opencode
```

Then tell the build agent (URL, local path, or local file):

```
Initialize company from https://github.com/you/handbook
Initialize company from ~/handbook
Initialize company from ./handbook.zip
```

The agent will:
1. Create all agent configuration files
2. Set up the skills directory for company knowledge

Restart OpenCode, and say:

```
@lead what should we build?
```

## What gets created

```
my-project/
├── opencode.json              # Minimal config (handbook as instruction file)
├── ai-company-handbook.md     # Company constitution (copied from this repo)
├── .opencode/
│   ├── agents/
│   │   ├── lead.md           # Tech Lead — task analysis & routing recommendations
│   │   ├── po.md              # Product Owner — breaks large requests into epics
│   │   ├── architect.md       # Architect — technical planning & task decomposition
│   │   ├── architect-context.md # Architect (Context) — 1M-token codebase analysis
│   │   ├── knowledge.md       # Knowledge Manager — research, skills, documentation
│   │   ├── frontend.md        # Frontend Dev — React, TypeScript, Tailwind, Leaflet
│   │   ├── frontend-pro.md    # Frontend Dev (Pro) — complex state, perf, bug hunts
│   │   ├── backend.md         # Backend Dev — APIs, CRUD, database, services
│   │   ├── backend-ops.md     # Backend Ops — Docker, SRE, production debugging
│   │   ├── reviewer.md        # Code Reviewer — read-only review + retro facilitation
│   │   └── devops.md          # DevOps — build config, CI, deployment
│   └── skills/
│       └── README.md          # Skills created by @knowledge on first research request
└── docs/
    └── model-catalog.md       # AI model reference (snapshot)
```

## The Team

| Layer | Agents |
|---|---|
| **Build** (primary) | `build` — Default agent, handles delegation |
| **Planning** (subagents) | `@lead`, `@po`, `@architect`, `@architect-context`, `@knowledge` |
| **Specialists** | `@frontend`, `@frontend-pro`, `@backend`, `@backend-ops`, `@reviewer`, `@devops` |

## How it works

Ask `@lead` to analyze any task. @lead determines the scale and recommends which specialist to use. The build agent delegates via the Task tool.

| Scale | Example | Routing |
|---|---|---|
| **Small** | "Fix this button color" | Lead → Specialist directly |
| **Medium** | "Build MapContainer from plan" | Lead → Architect → Specialist |
| **Large** | "Build a ride hailing system" | Lead → PO → Architect → Specialists (parallel) |

The company improves itself over time through hiring pipelines, performance evaluations, retrospectives, and a living knowledge base of skills.

## Requirements

- [OpenCode](https://opencode.ai) installed
- OpenCode Go subscription ($5/month)

## Repo structure

```
handbook/
├── README.md                   ← This file
├── BOOTSTRAP.md               ← Bootstrap instructions read during init
├── ai-company-handbook.md      ← Full company constitution
├── opencode.template.json      ← Generated as opencode.json during init
├── agent-templates/            ← Agent files with full system prompts
└── docs/
    └── model-catalog.md        ← AI model reference
```
