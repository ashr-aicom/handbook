# Bootstrap Your AI Company

This handbook lets you set up your own AI software company powered by OpenCode agents. Every team member is an AI agent with a defined role, model assignment, and trigger phrases.

## Prerequisites

- [OpenCode](https://opencode.ai) installed
- A GitHub account (for company infrastructure)
- Git installed

## Quick Start

### 1. Clone the handbook

```bash
git clone https://github.com/ashr-aicom/handbook.git ai-company/handbook
```

### 2. Clone the knowledge repo

```bash
git clone https://github.com/ashr-aicom/knowledge.git ai-company/knowledge
```

Or create your own:

```bash
mkdir -p ai-company/knowledge/.opencode/skills
```

### 3. Set up operations

```bash
mkdir -p ai-company/operations/docs/hiring ai-company/operations/docs/retrospectives
```

### 4. Create your project

```bash
mkdir -p ai-company/projects/my-project
cd ai-company/projects/my-project

# Add knowledge as a submodule
git submodule add https://github.com/ashr-aicom/knowledge.git knowledge

# Create opencode.json
cat > opencode.json << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["../handbook/ai-company-handbook.md"]
}
EOF
```

### 5. Configure agents

Copy agent templates into your project:

```bash
mkdir -p .opencode/agents
cp ../handbook/agent-templates/*.md .opencode/agents/
```

Customize model assignments and prompts as needed.

### 6. Start building

Launch OpenCode from your project directory and delegate work to agents:

```
@lead build a login page with email/password form
```

## Customization

### Replace model assignments

Edit each agent file in `.opencode/agents/` to use models available in [your OpenCode model catalog](https://opencode.ai/zen/go/v1/models).

### Create your own skills

Skills live in `knowledge/.opencode/skills/`. Each skill is a directory with a `SKILL.md` file:

```markdown
---
name: my-domain
description: Description of the domain. Use when working with X, Y, or Z in any project.
---

# My Domain Skill

## Patterns
...

## Gotchas
...
```

### Add new roles

Follow the hiring process in the handbook's Agent Hiring & Evaluation section.

## Directory Structure

```
ai-company/
├── handbook/          # This repo — company constitution
├── knowledge/         # Shared skills (Git submodule in each project)
├── operations/        # Internal records (private)
└── projects/
    └── my-project/
        ├── handbook/   # Git submodule → handbook repo
        ├── knowledge/  # Git submodule → knowledge repo
        └── .opencode/
            └── agents/
```

## References

- [AI Company Handbook](./ai-company-handbook.md) — full constitution
- [OpenCode Documentation](https://opencode.ai)
