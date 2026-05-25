# AI Company — Maintainer Instructions

These instructions are for you, the human maintainer. They are NOT copied to projects during init — they live in the handbook repo only. OpenCode loads this file automatically when you work in the sandbox.

## Philosophy

### Let the company manage itself

- If the AI Company should do something, ASK it. Don't manually edit agent files, commit to repos, or sync the handbook yourself. Say: "Sync the handbook with the latest changes from the sandbox" or "Create a skill for X."
- If you find yourself about to `cp`, `git commit`, or manually edit `.opencode/`, stop. Ask the build agent to do it.
- The only exception: creating the AI Company in a fresh directory (initialization from an empty directory — this requires manual `opencode` launch since there's no config yet).

### You are the CEO, not an engineer

- Ask @lead for routing guidance
- Ask @po for epic breakdowns
- Ask @architect for technical plans
- Let specialists execute
- Let @knowledge document learnings
- Let @reviewer review
- Your job: approve, direct, and make decisions agents can't make (org names, model choices, architectural direction)

## Testing with tmux

### The problem

When you send text to opencode via `tmux send-keys`, opencode may interpret it as a shell command instead of chat input. You'll see:

```
# Shell
$ Some command here
zsh: command not found: Some
```

### The fix

Use `tmux send-keys -l` (literal mode) to send chat input:

```bash
# WRONG — interpreted as shell command
tmux send-keys -t session "Some text" Enter

# RIGHT — literal, goes to TUI input
tmux send-keys -l -t session "Some text" Enter
```

### Full testing recipe

```bash
# 1. Start session
tmux new-session -d -s ai-test -c /path/to/project
tmux send-keys -t ai-test "opencode" Enter

# 2. Wait for TUI to load (10-15s)
sleep 12

# 3. Check what agent is active
tmux capture-pane -t ai-test -p | grep "Build\|Plan\|Lead"

# 4. Send command (literal mode!)
tmux send-keys -l -t ai-test "Your task here" Enter

# 5. Wait for processing
sleep 15

# 6. Capture output
tmux capture-pane -t ai-test -p | tail -40

# 7. Handle permission prompts
#   tmux send-keys -t ai-test Right Enter Enter  (Allow always)

# 8. Enter subagent view if subagents are running
tmux send-keys -t ai-test C-x Down
tmux capture-pane -t ai-test -p | tail -10

# 9. Clean up
tmux kill-session -t ai-test
```

### Subagent navigation

Subagents run in child sessions. Use these keys to navigate:

| Action | Keys |
|---|---|
| Enter child session (first subagent) | `C-x` → `Down` |
| Cycle to next child | `Right` |
| Cycle to previous child | `Left` |
| Return to parent session | `Up` |
| Session switcher (all subagents) | `ctrl+p` → "Switch Session" |

### Task output is collapsed by default

When a subagent completes, the output shows a summary line (e.g., "Frontend Task · 3 toolcalls · 12.5s"). Use "Click to expand" to see the full subagent response including file contents and tool outputs. The session switcher shows all active/queued subagents at once.

### NEVER kill sessions while subagents are working

Subagents take 30-90 seconds to think before writing files. The parent TUI shows idle status while subagents are actively processing. Always enter subagent view (`C-x Down`) to verify before killing a session. Killing prematurely abandons work in progress.

### Permission prompts

## Git operations

### Don't do these manually

- `cp` files between sandbox and company directories
- `git add` / `git commit` / `git push` manually
- `git submodule add` / `git submodule update --remote`

### Do this instead

```
"Sync the handbook with latest changes from the sandbox"
"Commit and push the new skill to the knowledge repo"
"Create a project with knowledge as a submodule"
"Update the knowledge submodule in all projects"
```

## File locations

| What | Where | Copied to projects? |
|---|---|---|
| Handbook constitution | `ai-company-handbook.md` | Yes (submodule) |
| Bootstrap instructions | `BOOTSTRAP.md` | No (build agent reads from source) |
| Agent templates | `agent-templates/` | Yes (copied to `.opencode/agents/`) |
| Model catalog | `docs/model-catalog.md` | Yes (copied) |
| Testing doc | `docs/testing.md` | No (handbook maintenance only) |
| Maintainer instructions | `AGENTS.md` (this file) | No (maintainer only) |

## Known limitations

1. **Lead can't self-delegate**: The Lead is a subagent that provides analysis. The build agent handles Task tool delegation. Don't expect @lead to "do" work — it analyzes and recommends.

2. **Knowledge agent in submodule context**: @knowledge works best when invoked in the knowledge repo directly (not from within a project that has knowledge as a submodule). For skill creation, open opencode in `~/ai-company/knowledge/`.

3. **Agent prompt templates**: If you change an agent template in `agent-templates/`, it won't affect already-initialized projects. Use the handbook sync flow to update.

4. **Model drift**: Models evolve. Run a model audit periodically: `@knowledge audit models`.
