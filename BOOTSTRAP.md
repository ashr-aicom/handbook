# Bootstrap the AI Company

Use these instructions ONLY when initializing the AI company in a new directory. Do not apply during normal operation.

## How to detect initialization

The user will say something like:
- "Initialize company from <url-or-path>"
- "Setup the AI company"
- "Bootstrap the company"

If `.opencode/agents/` already exists with more than 2 agent files, the company is already initialized. Report status. Do not overwrite unless the user explicitly asks.

## Step 1: Fetch the handbook

The user provides a source — GitHub URL, local directory path, or nothing.

| Source | Action |
|---|---|
| `https://github.com/...` | Clone the repo into a temp location. Read files from there. |
| `/absolute/path` or `~/path` | Read directly from that directory. |
| Not provided | Ask the user for the source URL or path. |

## Step 2: Read the constitution

Read `ai-company-handbook.md`. This contains:
- **Architecture** (§): company structure, task-scale gating
- **Design Principles** (§): 10 principles to embed in every agent
- **Team Roster** (§): every agent's role, model, tools, and trigger phrases
- **Model Assignment Rationale** (§): which model goes to which agent

## Step 3: Create opencode.json

Generate a minimal `opencode.json`. The build agent remains the primary agent — no default_agent override needed. The handbook is loaded as an instruction file:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["ai-company-handbook.md"]
}
```

## Step 4: Create subagent files

For each subagent in the Team Roster (§), create `.opencode/agents/<name>.md`.

See the Appendix (§) in the handbook for the file format.

If the handbook repo contains `agent-templates/<name>.md`, copy those files directly (they have full system prompts).

If no templates exist, generate the prompt from the role description, strength, and trigger phrases in the roster table. The prompt must include:
1. Role name and purpose
2. Strengths (from roster)
3. When to accept tasks (trigger phrases)
4. The design principles from the handbook (§ Design Principles)
5. Relevant conventions: "Never add comments unless asked. Follow the existing codebase conventions."

## Step 5: Create skills directory

Create `.opencode/skills/` directory with a single `README.md`:

```
# Skills

Skills are created by @knowledge on first research request.
See @knowledge's role for details.

Empty directory is intentional — no research has been done yet.
```

## Step 6: Copy handbook and report completion

Copy `ai-company-handbook.md` into the project root (needed for the `instructions` field in opencode.json). Then report:

```
Company initialized.

11 agents ready:
  Planning: @lead, @po, @architect, @architect-context, @knowledge
  Specialists: @frontend, @frontend-pro, @backend, @backend-ops, @reviewer, @devops

Next steps:
  1. Quit opencode and restart (config is loaded at startup)
  2. Ask @lead for routing guidance: "Use @lead to analyze: <task>"
  3. Set up GitHub: "@lead setup github" (separate meta-task — creates repos,
     sets up knowledge submodule, pushes handbook)
```

## Note on Git and GitHub

Git setup is a **separate meta-task**, not part of initialization. The company is fully operational locally without Git. When the user is ready, they run `@lead setup github` which creates the GitHub org, repos, and submodule links as described in the handbook's Company Infrastructure section.
