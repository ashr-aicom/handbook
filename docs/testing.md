# AI Company — Testing & Verification

> What's been tested, what hasn't, and exactly how to reproduce each test. Run after any architectural change or before a new release.

---

## Test Environment

Clean setup for every test run:

```bash
rm -rf ~/Temp/ai-com && mkdir -p ~/Temp/ai-com && cd ~/Temp/ai-com
opencode
```

Then:

```
Initialize company from /path/to/handbook
```

For tmux-based automated testing:

```bash
tmux new-session -d -s ai-com -c ~/Temp/ai-com
tmux send-keys -t ai-com "opencode" Enter
sleep 10
tmux send-keys -l -t ai-com "Initialize company from /path/to/handbook" Enter
# Handle permissions: tmux send-keys -t ai-com Right Enter Enter
sleep 30
tmux capture-pane -t ai-com -p
```

---

## Test Catalog

Status: ✅ Passed · ❌ Failed · ⬚ Not tested · ⚠️ Fragile — re-verify after related changes

### Bootstrap & Config

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-BOOT-01 | Clean init from local path | ✅ | `Initialize company from <path>` in empty directory. Verify: 14 files created (opencode.json, 11 agents, skills/README.md, handbook). No errors. Completion report matches template. |
| TC-BOOT-02 | Init from GitHub URL | ⬚ | `Initialize company from https://github.com/<org>/handbook` |
| TC-GH-01 | Setup meta-task | ✅ | `Set up GitHub for the AI Company. The org is ashr-aicom.` — Verified: 3 repos created (handbook, knowledge, operations), pushed, correct remote URLs, directory structure matches handbook spec. Agent asked 2 questions (company root, repo creation) via structured UI. |
| TC-CONF-01 | No config errors on restart | ✅ | Restart opencode 3+ times in the initialized directory. Verify: no errors in status bar, agent list loads cleanly. |
| TC-CONF-02 | Model audit runs without error | ⬚ | `@lead audit models` — verify: no crashes, report contains deprecation/upgrade info |
| TC-CONF-03 | Handbook loads as instruction | ⚠️ | Verify opencode.json has `"instructions": ["ai-company-handbook.md"]`. Agent should reference handbook content when asked about team structure. |

### Agent Discovery

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-DISC-01 | All 11 agents registered | ✅ | Start opencode. Ask: "List all agents and roles." Verify: @lead, @po, @architect, @architect-context, @knowledge, @frontend, @frontend-pro, @backend, @backend-ops, @reviewer, @devops all present. |
| TC-DISC-02 | Agents appear in @-mention | ⬚ | Type `@` in the TUI input. Verify autocomplete shows all 11 agents. |

### Task Routing (Lead)

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-LEAD-01 | Small task analysis | ✅ | `Use the Task tool: @lead analyze this task: Create src/HelloWorld.tsx — a simple React component.` Verify: Lead responds with "Scale: SMALL" and recommends @frontend. Structured analysis format. |
| TC-LEAD-02 | Medium task delegates to architect | ⬚ | `@lead analyze: Build a vehicle tracking demo with maps and routing.` Verify: recommends @architect first. |
| TC-LEAD-03 | Large task delegates to PO | ⬚ | `@lead analyze: Build a ride hailing system.` Verify: recommends @po → @architect → specialists. |

### Product Owner

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-PO-01 | Epic breakdown | ✅ | `Use the Task tool: @po Break this into epics and user stories with acceptance criteria: Build a time tracking system where employees clock in/out, view weekly hours, managers approve timesheets.` Verify: structured output with 3+ epics, each with user stories and acceptance criteria. Priorities assigned. |
| TC-PO-02 | Empty/invalid request | ⬚ | `@po Break this into epics: ???` Verify: handles gracefully, asks for clarification. |

### Architect

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-ARCH-01 | Technical plan | ✅ | `Use the Task tool: @architect Create a technical plan for a vehicle tracking demo with React, Leaflet, OSRM routing.` Verify: structured plan with component breakdown, data flow, implementation phases. |
| TC-ARCH-02 | Plan from PO stories | ⬚ | Feed architect PO output from TC-PO-01. Verify: plan references story IDs, estimates per story. |
| TC-ARCH-CTX-01 | Full-codebase analysis | ⬚ | Requires codebase with 50+ files. `@architect-context analyze the impact of changing <core-module>.` Verify: lists all affected files, risks, migration path. |

### Frontend

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-FE-01 | Component creation | ✅ | `Use the Task tool: @frontend Create src/Navbar.tsx — responsive navbar, app name 'AI Company', two links, Tailwind styling.` Verify: creates file with TypeScript types, responsive design (mobile menu), Tailwind classes. |
| TC-FE-02 | Edge case handling | ⬚ | `@frontend Create a component that handles loading, empty, and error states.` Verify: all 3 states implemented. |
| TC-FE-PRO-01 | Complex state / performance | ⬚ | `@frontend-pro Debug performance issue in <component> — re-renders on every keystroke.` Verify: identifies cause, applies useMemo/useCallback appropriately. |

### Backend

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-BE-01 | API client | ✅ | `Use the Task tool: @backend Create src/api/timesheets.ts with clockIn, clockOut, getWeeklyHours, approveTimesheet functions.` Verify: TypeScript interfaces, ApiError class, proper error handling, typed request wrapper. |
| TC-BE-02 | Validation | ⬚ | `@backend Add input validation to <endpoint>.` Verify: validates required fields, formats, throws meaningful errors. |
| TC-BE-OPS-01 | Docker compose | ✅ | `Use the Task tool: @backend-ops Create docker-compose.yml for self-hosted OSRM server.` Verify: valid YAML with health checks, restart policies, volume persistence, profile-based init. |
| TC-BE-OPS-02 | Production debugging | ⬚ | `@backend-ops Debug: <container> is restarting continuously.` Verify: analyzes logs, identifies root cause. |

### Reviewer

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-REV-01 | Code review | ✅ | `Use the Task tool: @reviewer Review <file>.tsx for quality, accessibility, best practices.` Verify: structured output with correctness, accessibility, issues, suggestions. |
| TC-REV-02 | Security audit | ⬚ | `@reviewer Security audit: <auth-endpoint>.` Verify: flags injection, missing validation, insecure practices. |
| TC-REV-03 | Retrospective facilitation | ⬚ | Requires prior task data. See Company Evolution § in handbook. |

### DevOps

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-DEV-01 | Package.json / config | ✅ | `Use the Task tool: @devops Create package.json for React+Vite+TS with leaflet, tailwindcss, vitest.` Verify: valid JSON, correct dependencies, dev/build/test scripts. |
| TC-DEV-02 | Deployment config | ⬚ | `@devops Configure Vercel deployment for this project.` Verify: vercel.json or equivalent. |

### Knowledge Manager

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-KNOW-01 | Skill file creation | ✅ | `Use the Task tool: @knowledge Create a skill file for tailwind-conventions documenting our Tailwind patterns.` Verify: creates valid SKILL.md in .opencode/skills/ with frontmatter (name, description), 50+ lines of content. |
| TC-KNOW-02 | Skill update | ⬚ | `@knowledge update osrm-routing with: rate limit is actually 30/min.` Verify: updates existing skill, adds update note. |
| TC-KNOW-03 | Plan verification | ⬚ | `@knowledge verify this plan against existing skills.` Verify: cross-references, flags contradictions, if any. |

### Process & Quality

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-PARA-01 | Parallel delegation | ⬚ | Ask build to delegate to @frontend AND @backend simultaneously. Verify: both tasks execute in parallel (ctrl+x down to view). |
| TC-ESC-01 | Escalation to pro | ⬚ | Give @frontend a task it fails. Verify: build retries or escalates to @frontend-pro. |
| TC-HIRE-01 | Hiring pipeline | ⬚ | `@lead hire frontend` — verify screening task creation, candidate comparison, interview scenarios. |
| TC-RETRO-01 | Retrospective | ⬚ | `@lead run retro` — verify data compilation, reviewer analysis, report generation. |

### GitHub & Infrastructure

| ID | What | Status | How to reproduce |
|---|---|---|---|
| TC-GH-01 | Setup meta-task | ⬚ | `@lead setup github` — needs gh CLI. Verify: asks org name, creates repos, pushes. |
| TC-GH-02 | Knowledge submodule | ⬚ | After TC-GH-01. Verify: `git submodule status` shows knowledge/ linked. @knowledge creates a skill → committed to knowledge repo. |
| TC-GH-03 | Handbook submodule sync | ⬚ | Change handbook remotely. `git submodule update --remote` in project. Verify: opencode.json instructions point to updated file. |
| TC-GH-04 | Operations repo privacy | ⬚ | After TC-GH-01. Verify: operations repo is private, hiring docs not in public repos. |

---

## Regression Map

Which changes trigger which re-tests:

| Change | Re-run these tests |
|---|---|
| BOOTSTRAP.md edited | TC-BOOT-01, TC-CONF-01 |
| opencode.template.json edited | TC-BOOT-01, TC-CONF-01, TC-CONF-03 |
| Any agent template changed | TC-DISC-01, TC-{agent}-01 |
| Lead prompt or mode changed | TC-LEAD-01, TC-LEAD-02, TC-LEAD-03 |
| Any agent model changed | TC-{agent}-01 |
| Any agent permission changed | TC-{agent}-01, TC-CONF-01 |
| New agent added | TC-BOOT-01, TC-DISC-01, TC-CONF-01 |
| New agent template file added | TC-BOOT-01, TC-DISC-01 |
| Infrastructure section added/changed | TC-GH-01 through TC-GH-04 |
| Handbook architecture changed | TC-LEAD-01 through TC-LEAD-03 |
| TL;DR — any change to bootstrap files | TC-BOOT-01, TC-DISC-01, TC-CONF-01 (minimum safety net) |

---

## Run History

| Date | Trigger | Tests run | Result |
|---|---|---|---|
| 2026-05-24 (Round 2) | Tier 1 specialists | TC-PO-01, TC-ARCH-01, TC-DEV-01, TC-BE-01, TC-BE-OPS-01, TC-KNOW-01 | All passed |
| 2026-05-24 (Round 1) | Lead architecture fix | TC-BOOT-01, TC-DISC-01, TC-CONF-01, TC-LEAD-01, TC-FE-01, TC-REV-01 | All passed |
| 2026-05-24 (Round 4) | GitHub infrastructure test | TC-BOOT-01, TC-GH-01 | Passed — 3 repos created on ashr-aicom org. Agent used structured question UI. |

---

## TL;DR Verification

Bare minimum to run after any change:

```bash
# 1. Bootstrap from scratch
rm -rf ~/Temp/ai-com && mkdir -p ~/Temp/ai-com && cd ~/Temp/ai-com && opencode
# → "Initialize company from /path/to/handbook"
# → Verify: 14 files created, no config errors

# 2. Restart and verify agents
# → Quit, restart opencode
# → "List all agents and roles" → 11 agents present

# 3. Test one specialist
# → "Use the Task tool: @frontend Create a simple React component"
# → Verify: file created, valid TypeScript

# 4. Test Lead
# → "Use the Task tool: @lead analyze: fix button color"
# → Verify: "Scale: SMALL, Route: @frontend"
```
