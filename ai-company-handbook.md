# AI Company Handbook

> Mental model: a software company. Agents are employees with defined roles, strengths, and weaknesses. Skills are the company's living knowledge base — they start as research, evolve through experience, and represent current best knowledge.

---

## Architecture

### How the company is structured

| Layer | Agents | Role |
|---|---|---|
| **You** | — | The CEO. All work flows through the build agent. |
| **Build** (primary) | `build` | Default OpenCode agent. Handles all tool access. Invokes planning agents and specialists via Task tool. You can also @mention any agent directly. |
| **Planning** (gated) | `@lead`, `@po`, `@architect`, `@architect-context`, `@knowledge` | Lead analyzes tasks and recommends routing. Product Owner breaks large requests into epics/stories. Architect creates technical plans. Knowledge Manager researches, maintains skills, verifies plans. Invoked based on task scale. |
| **Specialists** | `@frontend`, `@frontend-pro`, `@backend`, `@backend-ops`, `@reviewer`, `@devops` | Execute work. Frontend and backend each have standard and pro variants. Reviewer is read-only. DevOps handles config/deploy. |
| **Knowledge** | `.opencode/skills/` | Living documents — company brain. Skills start as research, evolve through experience. Auto-loaded when specialists encounter matching keywords. Shared across projects via Git submodule. |
| **GitHub** | `handbook`, `knowledge`, `operations`, `project-*` repos | Company infrastructure on GitHub. Handbook is public for others to bootstrap. Knowledge is shared via submodule. Operations is private for internal records. Projects have dedicated repos. |

### Task-scale gating

| Request scale | Example | Agents invoked |
|---|---|---|
| **Small** | "Fix this button color" | Lead → Specialist directly |
| **Medium** | "Build MapContainer from plan" | Lead → Architect → Specialist |
| **Large** | "Build a ride hailing system" | Lead → PO → Architect → Specialists |

PO and Architect are **optional layers** — the Lead decides whether to engage them based on request complexity. For small tasks, the Lead routes directly to a specialist. No agent is invoked unnecessarily.

---

## Design Principles

1. **Split only where task profile is distinct** — not just model quality, but what kind of work it is
2. **Lead never writes code** — its only job is analysis, routing, monitoring, knowledge capture, and synthesis
3. **Shared budget pool** — all agents draw from the same $12/5hr cap; model cost affects everyone
4. **Static models on subagents** — each specialist has a fixed model chosen for its strength profile
5. **Lead inherits model** — you choose the model for the primary session; the Lead uses it for reasoning/routing
6. **Task-scale gating** — PO and Architect are only invoked for requests that need them
7. **Lead as monitor** — no separate monitoring agent; the Lead collects results, checks quality, and re-delegates if needed
8. **Skills are living knowledge** — a skill is created from research, updated through implementation discoveries, and matures over time. The single source of truth for any domain.
9. **Evidence-based model selection** — model assignments are based on our own screening results, not provider benchmark claims. Every agent earns their role through a hiring process.
10. **Evolutionary governance** — the company reflects on itself, learns, and changes. Processes, structure, and even this document evolve through measured and regular retrospectives.

---

## Company Knowledge (Skills)

### Philosophy

Skills are the company's brain. They are **not static references** — they are living documents that evolve:

```
Research phase → Skill created with initial knowledge
      │
      ▼
Implementation → Discoveries surface → Skill is updated
      │
      ▼
Future projects → More experience → Skill matures
```

One file per domain. One source of truth. When a specialist works on routing code, the `osrm-routing` skill auto-loads — including all accumulated learnings from prior work.

### Knowledge lifecycle

```
Before project:   @knowledge researches OSRM → creates .opencode/skills/osrm-routing/SKILL.md
During project:   @frontend discovers rate limit is 50/min, not 100
                  → Lead: "@knowledge update osrm-routing with rate limit finding"
                  → @knowledge updates the skill
After project:    @architect references osrm-routing for next routing project
                  → Skill already contains correct rate limit
```

### @knowledge verifies

```
Before project: @architect tech plan says "use OSRM, no key needed"

@knowledge checks osrm-routing skill:
  → Found: "OSRM public API throttles at 50 req/min. For production,
    self-host via Docker or implement request queue."
  → Flags: "This plan doesn't account for rate limiting."
  → Lead catches the gap before implementation starts
```

### Skill structure

```
.opencode/skills/
├── osrm-routing/
│   └── SKILL.md          # "OSRM routing API: endpoints, rate limits, self-hosting, quirks"
├── leaflet-patterns/
│   └── SKILL.md          # "React Leaflet patterns: performance, markers, custom icons"
├── tailwind-conventions/
│   └── SKILL.md          # "Our Tailwind conventions: breakpoints, spacing, dark mode"
└── learnings/
    └── SKILL.md          # "General engineering learnings: anti-patterns, lessons learned"
```

Skills are **not project-specific** — they are company-wide knowledge that applies to any project that touches that domain.

---

## Team Roster

### Lead

| Agent | Model | Tools | Role |
|---|---|---|---|
| `@lead` | `opencode-go/deepseek-v4-flash` | read, glob, grep, webfetch (no edit/write/bash) | Analyzes tasks, determines scale, and recommends which specialist to route to. Called by build agent via Task tool when routing guidance is needed. Does not execute work directly. |

### Product Owner

| Agent | Model | Tools | Role | Trigger |
|---|---|---|---|---|
| `@po` | Inherited from Lead | read, write | Breaks large requests into epics and user stories with acceptance criteria | Only invoked for project-scale requests (multi-feature, multi-system, or "build a ... system" requests) |

### Architect

| Agent | Model | Strength | Trigger phrases |
|---|---|---|---|
| `@architect` | `opencode-go/minimax-m2.5` | SOTA planning & task decomposition, design-first workflow. 80.2% SWE-bench, 200K context, 6,300 req/5hr | "design the system", "architecture review", "should we use X or Y?", greenfield planning, medium-task technical planning |
| `@architect-context` | `opencode-go/deepseek-v4-flash` | 1M context for full-codebase analysis, near-Pro quality. 7,450 req/5hr | "how does this impact the whole codebase?", large monorepo evaluation, cross-cutting concerns |

### Knowledge Manager

| Agent | Model | Tools | Role | Trigger |
|---|---|---|---|---|
| `@knowledge` | `opencode-go/qwen3.6-plus` | read, write, glob, grep (no edit/bash — markdown only) | Creates and maintains skill files as living knowledge. Conducts pre-project research. Updates skills with implementation discoveries. Verifies plans against existing knowledge. Curates and deduplicates. | Lead says `@knowledge research/osrm`, `@knowledge update osrm-routing with X`, or `@knowledge verify this plan` |

### Frontend

| Agent | Model | Strength | Trigger phrases |
|---|---|---|---|
| `@frontend` | `opencode-go/deepseek-v4-flash` | Near-Pro coding, 1M context, 7,450 req/5hr. Daily workhorse. | Standard components, hooks, forms, styling, state management, Leaflet integration |
| `@frontend-pro` | `opencode-go/deepseek-v4-pro` | Best raw coding (80.6% SWE-bench), 1M context, 1,300 req/5hr | Complex state machines, race conditions, performance bottlenecks, intricate React patterns, bug hunts |

### Backend

| Agent | Model | Strength | Trigger phrases |
|---|---|---|---|
| `@backend` | `opencode-go/deepseek-v4-flash` | 1M context, strong general coding, 7,450 req/5hr | Standard API routes, CRUD, database queries, simple services, middleware |
| `@backend-ops` | `opencode-go/minimax-m2.7` | SRE-level debugging, log analysis, real-world engineering. 3,400 req/5hr | Docker/container issues, production debugging, rate limiter design, infrastructure, incident response |

### Reviewer

| Agent | Model | Strength | Trigger phrases |
|---|---|---|---|
| `@reviewer` | `opencode-go/deepseek-v4-flash` | High quality + high volume (7,450 req/5hr). Read-only (edit/write: deny) | Code review, architecture review, security audit, best-practices check. Also acts as neutral facilitator for retrospectives (see Company Evolution). |

### DevOps

| Agent | Model | Strength | Trigger phrases |
|---|---|---|---|
| `@devops` | `opencode-go/qwen3.6-plus` | Sufficient for config/setup, 3,300 req/5hr | Build config, CI pipelines, Vercel/Netlify deploy, Docker Compose, environment setup |

---

## Model Assignment Rationale

| Model | Assigned to | Why |
|---|---|---|
| DeepSeek V4 Pro | `@frontend-pro` | Best raw coding. Only used for genuinely complex frontend work. |
| DeepSeek V4 Flash | `@frontend`, `@backend`, `@architect-context`, `@reviewer` | Near-Pro quality at 5.7x volume. The team's workhorse. 1M context is critical for full-codebase agents. |
| MiniMax M2.5 | `@architect` | Uniquely designed for "architect-level planning, decomposing tasks before coding." |
| MiniMax M2.7 | `@backend-ops` | Built for SRE, log analysis, real-world engineering. When "it works on my machine" isn't enough. |
| Qwen3.6 Plus | `@devops`, `@knowledge` | Config files and documentation don't need a premium model. Good enough, preserves budget. |
| Inherited | `@lead`, `@po` | Lead and PO inherit the primary session model. You control the routing intelligence quality. PO's model quality determines how well it breaks down large systems. |

---

## Model Lifecycle Management

Models come and go. New models launch with better benchmarks. Old models get deprecated. Without a process for this, the team silently degrades — agents fail on removed models, or stick to obsolete models while better options exist.

### The three risks

| Risk | Impact |
|---|---|
| **Deprecation** | Agent silently fails when invoked. Task goes unfulfilled, no clear error surfaced. |
| **New models** | Better quality or cheaper models exist but go unused. Team leaves performance on the table. |
| **Reference doc rot** | `docs/model-catalog.md` becomes stale and misleading. |

### Audit process

The Lead runs a model audit — either on session start (every ~10 sessions, not every time) or when the user explicitly asks. The audit does not block work; it runs in the background.

```
You: "@lead audit models"   (or: Lead decides it's time, based on elapsed
                               sessions since last audit)
Lead:
  ├── @knowledge: "Fetch the live model catalog from
  │     https://opencode.ai/zen/go/v1/models and compare against
  │     all agent model assignments in .opencode/agents/"
  │
  ├── @knowledge reports:
  │     ├─ DEPRECATED: miniMax M2.5 → needs replacement for @architect
  │     ├─ NEW: deepseek-v5-flash at same cost as v4-flash, 5% better SWE-bench
  │     │   → Consider upgrading @frontend, @backend, @architect-context
  │     └─ All other models active and current
  │
  ├── Lead presents findings to user
  │
  └── User: "Upgrade flash agents to v5. Find replacement for architect."
        Lead: @knowledge: "@knowledge update agent models per the audit"
          → Updates .opencode/agents/*.md with new model IDs
          → Updates docs/model-catalog.md
          → Updates this concept doc if needed
```

### Audit components

| Component | What gets checked |
|---|---|
| **Agent model assignments** | Each `.opencode/agents/*.md` — is the `model:` field still in the live catalog? |
| **Model recommendations** | For each role, are there new models that are a clear upgrade (better quality at same cost, or same quality at lower cost)? |
| **Reference doc freshness** | When was `docs/model-catalog.md` last updated? Is it more than 30 days stale? Does it still match the live catalog? |

### Guardrails

- **Never auto-switch models** — the audit reports findings; the user decides. A model upgrade can change behavior in subtle ways.
- **Deprecation is blocking** — if a model is removed, the agent stops working. Must be fixed immediately.
- **Upgrades are optional** — "better benchmark" doesn't always mean better for your specific workflow. User judgment required.
- **Record audit results** — each audit writes a timestamped note to `docs/model-audit-log.md` so there's a history of what changed and why.

---

## Agent Hiring & Evaluation

Benchmarks from model providers are marketing claims. Real performance depends on the agent's prompt, the task domain, and the interaction with other agents. The team needs its own evidence.

### The hiring pipeline

```
Role defined → Screen candidates (models) → Interview (validate prompt+model)
                                              │
                                              ▼
                                           Evaluate
                                             │
                           ┌─────────────────┴─────────────────┐
                           ▼                                   ▼
                        HIRE                                REJECT
                           │                              (adjust prompt
                           ▼                               or try new model)
                    Probation period
                    (monitor early 5-10 tasks)
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
          CONFIRM                    EXIT PROBATION
       (pass evaluation)            (investigate why)
              │
              ▼
       Regular performance
       evaluations (ongoing)
```

### Phase 1: Model screening

When creating a new role or reassigning a model, run the same task against multiple candidate models and compare results.

```
User: "@lead hire frontend"

Lead:
  ├── @knowledge: "Create a screening task for the @frontend role.
  │     The task should test: React component creation, TypeScript
  │     typing, Tailwind styling, and Leaflet map integration."
  │
  ├── @knowledge creates: docs/hiring/frontend-screening.md
  │     # Frontend Screening Task
  │     Build a MapMarker component that:
  │     1. Renders a clickable marker on a Leaflet map
  │     2. Shows a popup with formatted coordinates on click
  │     3. Uses TypeScript with proper types
  │     4. Styled with Tailwind
  │     5. Handles loading and error states
  │     Evaluation criteria: correctness, types, readability, edge cases
  │
  ├── Lead delegates the SAME task to each candidate:
  │     task(agent="frontend-candidate-v4-pro")
  │     task(agent="frontend-candidate-v4-flash")
  │     task(agent="frontend-candidate-mimo-v2.5-pro")
  │     task(agent="frontend-candidate-qwen3.6-plus")
  │     (temporary candidate agents, same prompt as @frontend,
  │      different models)
  │
  ├── @reviewer: "Compare the 4 implementations. Score each on:
  │     1. Correctness (does it work?)
  │     2. TypeScript quality (proper types, no `any`)
  │     3. Edge case handling (loading, error, empty state)
  │     4. Code readability and structure
  │     Output: ranked results with explanation"
  │
  └── Lead presents: "DeepSeek V4 Flash scored highest overall.
        V4 Pro wins on edge cases but costs 5.7x more.
        Qwen3.6 Plus failed loading state.
        MiMo tied on quality but 1.7x fewer requests than Flash.
        Recommendation: V4 Flash for @frontend."
```

### Phase 2: Agent interview (prompt validation)

Even with the right model, a bad prompt produces bad behavior. The interview tests the complete agent (model + prompt + constraints) on role-specific scenarios.

```
Lead: "@knowledge create interview scenarios for @frontend"

@knowledge creates docs/hiring/frontend-interview.md:
  Scenario 1: "Add a speed slider to the ControlPanel that updates
              animation speed in real time"
  Scenario 2: "The map renders but no tiles appear. Debug and fix."
  Scenario 3: "Review this VehicleMarker component for performance issues"
  Scenario 4: "Handle the case where OSRM returns zero coordinates"

Lead runs scenarios → evaluates:
  Scenario 1 → Good. Clean implementation, proper React state.
  Scenario 2 → Missed the tile URL format issue. Prompt needs
               more Leaflet-specific guidance.
  Scenario 3 → Caught the missing useMemo, but missed unnecessary
               re-renders. Prompt needs performance awareness.
  Scenario 4 → Handled correctly with user-friendly message.

Lead adjusts prompt with learnings → re-run failed scenarios → PASS
```

### Phase 3: Hire

Decision based on combined screening + interview results:

| Input | Weight | Source |
|---|---|---|
| Screening scores (model quality) | 40% | `@reviewer` comparison of candidate outputs |
| Interview results (prompt quality) | 40% | Scenario pass/fail rate |
| Cost efficiency (req/5hr budget) | 20% | Model catalog pricing |

The hiring decision is documented in `docs/hiring/<role>-<date>.md` with:
- Candidate models tested
- Screening task used
- Interview scenarios and results
- Final recommendation with rationale
- Baseline metrics (for future comparison)

### Phase 4: Probation period

First 5-10 real tasks from the agent are tracked. The Lead monitors closely.

```
Lead monitors early tasks:
  Task 1: "Build MapContainer" → ✅ Good. Matched screening quality.
  Task 2: "Add responsive breakpoints" → ⚠️ Used fixed widths, not Tailwind responsive.
                                     → Lead: "@knowledge document: @frontend
                                        needs explicit Tailwind responsive
                                        instruction"
  Task 3: "Implement geolocation" → ✅ Good. Edge cases handled.
  ...
  Task 10: "Optimize marker rendering" → ✅ Good. Used useMemo correctly.

Probation review:
  ✅ 8/10 tasks met expectations
  ⚠️ 2 tasks needed rework (both Tailwind-related)
  → Result: CONFIRM. Prompt adjusted for Tailwind. Probation passed.
```

| Outcome | Trigger | Action |
|---|---|---|
| **CONFIRM** | >80% tasks pass first submission | Agent moves to permanent roster. Prompt is frozen (minor adjustments only). |
| **EXTEND** | 60-80% pass rate | Extend probation 5 more tasks. May adjust prompt. |
| **REASSIGN** | <60% pass rate or critical failures | Reject. Re-enter screening with different model or major prompt rewrite. |

### Phase 5: Regular performance evaluations

Ongoing cycle — not punishment, just reality checks. Models that were great at hiring may underperform in real work. Prompts that worked for small tasks may fail at scale.

```
Evaluation schedule:
  ├─ Monthly: Light review (last 10 tasks, pass/fail rate, rework rate)
  ├─ Quarterly: Full review (screening task re-run, model audit integration)
  └─ Ad-hoc: Triggered by Lead when failure rate spikes or user raises concern
```

| Metric | How measured | Good | Warning | Action |
|---|---|---|---|---|
| **First-attempt pass rate** | Tasks that pass without rework | >80% | 60-80% | <60% |
| **Rework rate** | Tasks needing 2+ re-delegations | <10% | 10-25% | >25% |
| **Knowledge consistency** | Does agent use available skills? | Always | Sometimes | Rarely |
| **Budget efficiency** | Request count per task vs. expected | Within range | 20% over | 50%+ over |
| **Screening re-run** | Same task, same model — has quality changed? | Stable or improved | Minor regression | Significant regression |

### Evaluation outcomes

| Outcome | Trigger | Action |
|---|---|---|
| **Maintain** | All metrics green. Agent performing as hired. | No action. Continue monitoring. |
| **Adjust prompt** | One or two metrics degraded. Model still sound, prompt needs tuning. | `@knowledge` updates agent prompt based on failure patterns. |
| **Re-screen models** | Model quality degraded (screening re-run shows regression) or better models now exist. | Return to Phase 1. Run screening with current + new candidate models. |
| **Reassign role** | Agent consistently fails across multiple metrics. | Remove from roster. Re-open hiring for the role. |

### Evaluation calendar integration

```
Week 1:   Hiring runs (new role or replacement)
Week 2-3: Probation (monitoring)
Week 4:   Monthly light review (all active agents)
Quarterly: Full review + screening re-run for all agents
On-demand: Triggered by Lead on failure spike
Model audit: Runs quarterly alongside full review, or on user request
```

### Storage

```
docs/hiring/
├── frontend-screening.md       # Standard screening task for frontend role
├── backend-screening.md        # Standard screening task for backend role
├── architect-screening.md      # ... (one per role)
├── frontend-interview.md       # Interview scenarios
├── ...
├── frontend-2026-05-23.md      # Hiring decision + baseline metrics
├── backend-2026-06-15.md       # ...
├── evaluations/                # Ongoing evaluations
│   ├── frontend-2026-06.md     # Monthly review
│   ├── frontend-2026-Q3.md     # Quarterly review
│   └── ...
└── model-audit-log.md          # Model audit history
```

---

## How the Lead Routes Tasks

The Lead is a subagent invoked via `@lead`. It analyzes tasks and recommends routing. The build agent (primary) handles the actual delegation via the Task tool.

### Step 1: Scale assessment

On every request, the Lead first determines the scale:

| Scale | Indicators |
|---|---|
| **Small** | Single component, one-file change, bug fix with clear location, simple question |
| **Medium** | New feature spanning multiple files, architectural decision needed, new component from a spec |
| **Large** | New system/module, multiple features, "build a ... system/app/platform", spans frontend + backend |

### Step 2: Route by scale

#### Small request — direct delegation

```
You → build → Task(@lead) → Lead: "SMALL, @frontend"
              → build → Task(@frontend)
```

```
You: "@lead fix the reset button color"
Lead: "Scale: SMALL. Single component change. Route: @frontend."
Build: delegates to @frontend via Task tool
```

#### Medium request — architect first

```
You → build → Task(@lead) → Lead: "MEDIUM, @architect first"
              → build → Task(@knowledge): check skills
              → build → Task(@architect): create plan
              → build → Task(@frontend): implement per plan
```

```
You: "build the MapContainer from the vehicle tracking plan"
Lead: "Scale: MEDIUM. Multi-file component. Route: @architect → @frontend."
Build: @knowledge checks skills → @architect creates plan → @frontend builds
```

#### Large request — full pipeline

```
You → build → Task(@lead) → Lead: "LARGE, @po → @architect → parallel specialists"
              → build → Task(@po): break into epics
              → build → Task(@architect): technical plan per epic
              → build → parallel Task(@frontend), Task(@backend)
              → build → Task(@reviewer): review
              → build → Task(@knowledge): document learnings
```

```
You: "build a ride hailing system"
Lead: "Scale: LARGE. Multi-system. Route: @po → @architect → specialists."
Build: @po breaks into 3 epics → @architect plans Rider epic →
       parallel @frontend + @backend → @reviewer → @knowledge
```

### Full routing decision tree

```
Scale assessment (by @lead)
│
├─ SMALL → Which domain?
│   ├─ Frontend → Standard?
│   │   ├─ Yes → @frontend
│   │   └─ No (complex/perf/bug) → @frontend-pro
│   ├─ Backend → Ops/debugging?
│   │   ├─ Yes → @backend-ops
│   │   └─ No → @backend
│   ├─ Review → @reviewer
│   ├─ Knowledge → @knowledge (research, update skill, verify plan)
│   └─ Config/deploy → @devops
│
├─ MEDIUM → @po needed? (no)
│   ├─ @knowledge: "check for relevant skills, document learnings after"
│   ├─ @architect-context (if full-codebase scope)
│   └─ @architect (standard planning)
│       └─ Then route to specialists (as in SMALL)
│
└─ LARGE → @po needed? (yes)
    ├─ @knowledge: "research domain, check existing skills"
    ├─ @po → breaks into epics/stories (incorporating knowledge)
    ├─ @architect → technical plan per epic
    ├─ Specialists → parallel execution per epic
    └─ @knowledge: "document all new learnings surfaced"

Note: @lead provides the analysis. Build executes the delegation via Task tool.
```

### Monitoring and intervention

The build agent and @lead share monitoring responsibility:

1. Build delegates tasks to specialists
2. Collect results as they complete
3. If a result is incomplete, wrong, or low quality → re-delegate with corrective instructions
4. If a specialist repeatedly fails → escalate to a higher-tier variant (e.g., @frontend → @frontend-pro)
5. Review all completed work (optionally via @reviewer) before presenting
6. After each significant body of work → capture learnings via @knowledge

### Meta-tasks

Not all company work fits the Small/Medium/Large routing model. Meta-tasks are operational concerns that the user triggers explicitly — they are never auto-routed by the Lead during normal task processing.

| Meta-task | Trigger | Agents involved | Where documented |
|---|---|---|---|
| **Hire/replace agent** | User: `@lead hire frontend` | Lead → Knowledge → Reviewer → Candidate agents | Agent Hiring & Evaluation (§ above) |
| **Run model audit** | User: `@lead audit models` (or Lead detects it's due) | Lead → Knowledge | Model Lifecycle Management (§ above) |
| **Run retrospective** | User: `@lead run retro` (or project close / month-end) | Lead → Knowledge → Reviewer | Company Evolution & Retrospectives (§ below) |
| **Onboard new project** | User: `@lead onboard project X` | Lead → Knowledge → PO → Architect | Lead delegates PO/Architect per scale gating |
| **Add new role** | User: `@lead create @mobile role` | Lead → Knowledge (designs agent, runs hiring) | Company Evolution (§ below) |
| **Disable/archive role** | User: `@lead disable @backend-ops` | Lead → Knowledge (marks agent `disable: true`) | Company Evolution (§ below) |

Meta-tasks bypass the normal routing tree. The Lead recognizes them by their explicit trigger keywords and routes directly to the documented flow.

---

## Why Not More Granularity?

Roles were considered but rejected:

| Rejected split | Reason |
|---|---|
| `@reviewer` → Pro/Quick | A review is a review. Model speed doesn't change the task profile. |
| `@devops` → Pro/Standard | Config files and deployment scripts are uniformly simple. Qwen3.6 is enough. |
| `@architect` → Pro/Quick/Light | M2.5 covers planning; V4 Flash covers context-heavy analysis. A third tier adds no new capability. |
| `@backend` → CRUD/API/DB | V4 Flash handles all standard backend work. Only ops (M2.7) is a distinct task profile. |
| PO + Architect → Deep Architect | A single agent doing both product breakdown and technical planning creates too much cognitive load. No separation of concerns — product assumptions bleed into tech decisions. A second agent catches bad calls. |
| `@monitor` (separate agent) | OpenCode's Task tool launches a subagent and returns a single result. No continuous "watch and intervene" mechanism exists. The Lead already acts as monitor by collecting and checking results. |
| `learnings/` as separate system | Skills are living documents — they start as research, evolve through discoveries, and mature over time. Separating learnings from skills creates two places to check instead of one source of truth. |

**The rule**: create a specialist only when the *kind of work* (not the model tier) is different.

---

## Company Evolution & Retrospectives

The team evaluates individual agents. But the company also needs to evaluate *itself* — its structure, processes, and how it works. A company that ships features but never improves how it ships them will plateau.

### What can evolve

| Layer | Example change | Trigger |
|---|---|---|
| **Agent-level** | Update `@frontend` prompt to emphasize Tailwind responsive patterns | Probation findings or evaluation metrics |
| **Process-level** | Tighten scale assessment criteria ("30% of Medium tasks mis-routed as Small") | Retrospective data |
| **Structural** | Add `@mobile` role when projects need React Native; remove `@backend-ops` if unused for 3+ months | Quarterly review |
| **Constitutional** | Update routing decision tree, model assignments, hiring thresholds, this doc itself | Quarterly org review |
| **Budget** | Upgrade plan, rebalance model assignments across roles | Model audit + evaluation data |

### Retrospective cadence

| Trigger | Scope | Duration | What it covers |
|---|---|---|---|
| **Per-project close** | Project-level | Light, ~3 agent invocations | What went well? What patterns repeated? Lessons to document? |
| **Monthly** | All active agents | Medium | Process health — routing accuracy, rework patterns, evaluation trends |
| **Quarterly** | The entire system | Deep | Org structure, team composition, constitutional changes, budget review |
| **Ad-hoc** | On-demand | Variable | Triggered by user concern or Lead detecting a pattern across multiple projects |

### Retrospective process (standard flow)

`@reviewer` acts as the facilitator — an objective, outside party with no stake in the outcomes. `@knowledge` gathers data. Lead presents and executes.

```
Trigger: project close / month end / quarter end / user request
  │
  ▼
Lead: "@knowledge prepare retro data for [scope]"
  │
  ├── @knowledge compiles:
  │     ├── /docs/hiring/evaluations/ from the period
  │     ├── Rework/delegation patterns (Lead provides from session memory)
  │     ├── Hiring decisions in the period (any new agents?)
  │     ├── Model audit log (any changes?)
  │     ├── Skills created/updated in the period
  │     └── Compiles into docs/retrospectives/<date>-data.md
  │
  ├── Lead: "@reviewer analyze the retro data and produce findings"
  │
  ├── @reviewer analyzes as neutral facilitator:
  │     1. What went well? (patterns to keep and reinforce)
  │     2. What didn't? (failures, rework patterns, bottlenecks)
  │     3. What repeated? (root causes, not one-off incidents)
  │     4. What's surprising? (metrics that contradict expectations)
  │     5. Recommendations (what should change)
  │     → Writes docs/retrospectives/<date>-findings.md
  │
  ├── Lead reviews findings, adds execution context:
  │     "Reviewer flagged 30% routing errors. Lead's context:
  │      most were ambiguous requests where scale was unclear.
  │      Recommendation: add clarifying questions to Lead prompt."
  │
  ├── Lead presents to user (with executive summary of top 3 items)
  │
  └── User approves → Lead delegates changes:
        ├── @knowledge: Update affected agent prompts
        ├── @knowledge: Update skills if patterns emerged
        ├── @knowledge: Update ai-company-handbook.md if structural
        └── @knowledge: Write docs/retrospectives/<date>.md (full record)
```

### Why `@reviewer` as facilitator

`@reviewer` has no personal stake in routing decisions, hiring outcomes, or process design. It sees what the Lead might miss or justify. A Lead grading its own routing is like a developer reviewing their own code — they have blind spots that compound over time.

| Without neutral facilitator | With `@reviewer` as facilitator |
|---|---|
| Lead: "Routing accuracy is fine for the complexity of requests." | `@reviewer`: "30% of medium tasks were routes as Small. This pattern is consistent across 3 months." |
| Lead: "Probably just edge cases." | `@reviewer`: "The same failure pattern repeats: ambiguous requests with domain-crossing scope." |

The marginal cost (one V4 Flash invocation per retro on the cheapest model tier) is negligible.

### Approval model

Not all changes are equal. The Lead operates with a graduated authority model that evolves as the user builds trust:

| Change type | Examples | Authority |
|---|---|---|
| **Minor tuning** | Update agent prompt with a specific convention ("add Tailwind responsive guidance to @frontend"), add a trigger phrase, fix a typo in a skill | Lead autonomous. Executes immediately, documented in retro log. |
| **Process adjustment** | Tighten scale assessment criteria, change probation threshold, adjust weighting in hiring decision | Lead proposes, user rubber-stamps. User can override but expects to trust these. |
| **Structural change** | Add/remove a role, change the routing decision tree, change the team's org chart | Lead proposes with evidence (retro data), user must explicitly approve. |
| **Constitutional change** | Change design principles, approval model, retro process itself, budget strategy | User must drive or explicitly approve. Lead may recommend but never executes unilaterally. |

#### Trust evolution

```
Early stage (week 1-4):  User approves everything. Lead learns preferences.
                          User calibrates what "good" looks like.
     │
     ▼
Mid stage (month 2-3):   Lead handles minor tuning autonomously.
                          User approves process changes, signs off
                          on structural. User intervention decreases.
     │
     ▼
Mature (month 4+):       Lead handles minor + process autonomously.
                          User involved only for structural and
                          constitutional. Trust is established.
```

This threshold can shift — if Lead makes a bad autonomous decision, the user can dial it back. The model is a safeguard, not a straitjacket.

### Org evolution examples (real scenarios)

| Scenario | Evolution |
|---|---|
| Team starts doing React Native after 3 web projects | Lead detects pattern in retro → proposes `@mobile` role → user approves → hiring runs → agent added to roster, routing tree updated |
| `@architect` invoked on trivial tasks 80% of the time | `@reviewer` flags in retro → Lead adjusts: architect gated to Medium+ only, Small tasks route directly → documented in retro |
| Lead gets overloaded — monitoring takes 40% of sessions | `@reviewer` detects Lead congestion → Lead proposes `@delivery-manager` role to handle task tracking → user approves or rejects |
| Knowledge grows to 30+ skills, conflicts emerge | `@reviewer` flags skill overlap in retro → `@knowledge` runs deduplication cycle → skills consolidated |
| `@backend-ops` unused for 3 months | `@reviewer` flags zero usage → Lead proposes removing the role → user approves → agent disabled, roster updated |

### Retrospective storage

```
docs/retrospectives/
├── 2026-05-23-project-vehicle-tracking.md   # Per-project retro
├── 2026-06-01-monthly.md                    # Monthly review
├── 2026-06-01-data.md                       # Supporting data compiled by @knowledge
├── 2026-06-01-findings.md                   # Neutral analysis by @reviewer
├── ...
├── 2026-Q3-quarterly.md                     # Quarterly org review
├── 2026-Q3-quarterly-data.md
├── 2026-Q3-quarterly-findings.md
└── ...
```

### Doc versioning

`docs/ai-company-handbook.md` is the company constitution. Every structural or constitutional change updates it. A changelog at the bottom tracks the evolution:

```
## Changelog
- 2026-07-Q3: Added @mobile role after first React Native project
- 2026-06-Q2: Added @knowledge agent (knowledge management)
- 2026-06-Q2: Narrowed @architect trigger to Medium+ only (30% over-invocation)
- 2026-05: Initial company formation
```

---

## Company Infrastructure

The AI company lives on your local machine and is backed up to GitHub. Projects, knowledge, operations, and the handbook itself each have dedicated repos.

### GitHub org structure

| Repo | Visibility | Contents | Purpose |
|---|---|---|---|
| `handbook` | Public | Constitution, BOOTSTRAP.md, agent templates, README | What others clone to init their own AI company. Clean, no internal data. |
| `knowledge` | Public | `.opencode/skills/` — osrm-routing, leaflet-patterns, etc. | Shared across all projects via Git submodule. Grows with experience. |
| `operations` | Private | `docs/hiring/`, `docs/retrospectives/`, `docs/model-audit-log.md` | Internal records. Private because hiring docs and retros are company-internal. |
| `project-*` (N) | Public or Private | Project code + `.opencode/` config | One repo per project. Standard Git workflow. |

### Local directory layout

```
~/ai-company/
├── handbook/              ◄── git clone ── github.com/<org>/handbook
│   ├── BOOTSTRAP.md
│   ├── ai-company-handbook.md
│   └── agent-templates/
├── knowledge/             ◄── git clone ── github.com/<org>/knowledge
│   └── .opencode/
│       └── skills/
│           ├── osrm-routing/SKILL.md
│           ├── leaflet-patterns/SKILL.md
│           └── ...
├── operations/            ◄── git clone ── github.com/<org>/operations
│   ├── hiring/
│   ├── retrospectives/
│   └── model-audit-log.md
└── projects/
    ├── vehicle-tracker/   ◄── git ── github.com/<org>/project-vehicle-tracker
    │   ├── knowledge/     ◄── git submodule ── github.com/<org>/knowledge
    │   └── .opencode/
    └── timesheets/        ◄── git ── github.com/<org>/project-timesheets
        ├── knowledge/     ◄── git submodule ── github.com/<org>/knowledge
        └── .opencode/
```

### Knowledge as Git submodule

Each project links to the shared knowledge repo via Git submodule. Skills are a single source of truth — one update benefits all projects.

**Why submodule over alternatives:**

| Approach | Verdict |
|---|---|
| Git submodule | Recommended. Single source of truth, version-pinned, standard tooling. |
| Symlink | Local-only, breaks across machines, no version history. |
| Copy | Drifts across projects, no single source of truth. |
| Git subtree | Bloats project repos, complex commands. |

**How it works:**

```
@knowledge updates osrm-routing/SKILL.md:
  1. Commits to knowledge/ repo
  2. Pushes to github.com/<org>/knowledge
  3. Each project: git submodule update --remote knowledge/
  4. All projects see the updated skill
```

The submodule pointer tracks which commit of knowledge each project uses. This is version history — you can see "project X used skills as of commit abc123."

### GitHub setup meta-task

Git setup is a separate meta-task, not part of init. Triggered after the company is initialized:

```
User: "@lead setup github"

Lead asks:
  What is your GitHub org name?

  Create repos on GitHub?
  [1] Yes — I have gh CLI installed and authenticated
  [2] No — give me the manual instructions

  Which repos should I create?
  [1] handbook + knowledge + operations (recommended)
  [2] handbook only
  [3] Custom selection

Lead delegates to @devops:
  - git init each local directory
  - gh repo create for handbook, knowledge, operations
  - git submodule add knowledge into each project
  - Push all repos to GitHub
  - Link projects to knowledge submodule

Lead: "@knowledge document the repo structure"
```

### Handbook sync in projects

Projects include the handbook as a Git submodule so the constitution stays current:

```
~/ai-company/projects/vehicle-tracker/
├── handbook/   ◄── git submodule ── github.com/<org>/handbook
```

When the handbook evolves (e.g., new agent added, routing rules changed), the project updates:

```
git submodule update --remote handbook/
```

The handbook loaded via `"instructions": ["ai-company-handbook.md"]` in `opencode.json` always reflects the current constitution.

---

## Future: Packaging for Distribution

| Approach | What it is | Best for |
|---|---|---|
| **Clone handbook repo** | `git clone https://github.com/<org>/handbook` and init from local path | Anyone bootstrapping their own AI company |
| **Copy `.opencode/agents/` + submodule knowledge/** | Clone the team config + link shared skills | Simple reuse across projects within the same company |
| **Skill** (`SKILL.md`) | Inject context/rules into an existing agent | Domain knowledge (e.g., "vehicle tracking domain", "OSRM routing patterns") |
| **Plugin** (`plugin.ts`) | Programmatic hooks to register agents, tools, auth | Automated agent registration, dynamic model selection |

### Priority for distribution:
1. **Done**: Bootstrap flow (BOOTSTRAP.md + agent templates) — proven working end-to-end
2. **Now**: Publish handbook to `github.com/<org>/handbook` — make it cloneable by anyone
3. **Next**: Set up knowledge repo + skills, projects with submodule links
4. **Later**: Explore a plugin for auto-registration + dynamic model routing

---

## Appendix: Agent File Structure

**Subagents** live in `.opencode/agents/<name>.md`:

```markdown
---
description: Short description of role, when to use it, and trigger phrases
mode: subagent
model: opencode-go/<model-id>
permission:
  edit: deny
  write: deny
  bash: deny
---

System prompt with role definition, scope, strengths, input/expectations, and conventions.
```

**Skills** live in `.opencode/skills/<name>/SKILL.md`:

```markdown
---
name: osrm-routing
description: OSRM routing API patterns. Use when working with OSRM, route calculation, Vehicle tracking, or map routing in any project
---

# OSRM Routing

## API Endpoints
...

## Rate Limits (updated: we discovered actual limit is 50/min)
...

## Self-Hosting
...
```

Auto-loads when a specialist encounters keywords from the description.

**The opencode.json** is minimal — build agent handles all tool access:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["ai-company-handbook.md"]
}
```

---

## Appendix: Testing

See `docs/testing.md` for the full test catalog, regression map, reproduction commands, and run history. After any architectural change, run the TL;DR verification at the bottom of that document. Add new tests whenever a new feature, agent, or process is introduced.

---

## Appendix: Model Reference

See `docs/model-catalog.md` for the current model catalog snapshot (benchmarks, context windows, request budgets). This document is a point-in-time reference and may become stale. The live catalog is at `https://opencode.ai/zen/go/v1/models`. Run `@knowledge` model audit to detect drifts between the snapshot and the live catalog.

---

## Changelog

This document is the company constitution. Every structural or constitutional change is recorded here.

| Date | Change | Type |
|---|---|---|
| 2026-05-24 | GitHub repos created on `ashr-aicom` org: handbook (public), knowledge (public), operations (private). Bootstrapped from handbook template. Verified: git init, initial commit, push successful. | Infrastructure |
| 2026-05-24 | Added `docs/testing.md` — test catalog (31 tests), regression map, run history, TL;DR verification. Linked from handbook appendix. | Operational |
| 2026-05-24 | Added Company Infrastructure section: GitHub org structure, local directory layout, knowledge as Git submodule, GitHub setup meta-task, handbook sync. Updated Architecture and Distribution sections. | Constitutional |
| 2026-05-24 | Lead rearchitected as subagent (mode: subagent). Build remains primary. Lead provides analysis and routing recommendations; build delegates via Task tool. Knowledge agent: edit allow for skill creation. Architecture diagram simplified. | Architectural |
| 2026-05-23 | Added Company Evolution & Retrospectives section. `@reviewer` as retro facilitator. Graduated approval model (minor/process/structural/constitutional). Trust evolution over time. | Constitutional |
| 2026-05-23 | Added Agent Hiring & Evaluation section. Model screening, prompt interview, probation, regular evaluations. | Structural |
| 2026-05-23 | Added Model Lifecycle Management section. Model audit process, deprecation handling, reference doc freshness. | Structural |
| 2026-05-23 | Added Company Knowledge (Skills) section. `@knowledge` agent. Skills as living documents. | Structural |
| 2026-05-23 | Added `@knowledge` agent to roster. Added `@po` agent (gated on project scale). Renamed Commander → Lead. | Structural |
| 2026-05-23 | Initial company formation. 11-agent AI team across 6 roles (lead, PO, architect, knowledge, frontend, backend, reviewer, devops). Task-scale gating (Small/Medium/Large). Company knowledge via skills. Evidence-based model selection. Evolutionary governance. | Constitutional |
