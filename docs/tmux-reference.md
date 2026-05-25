# tmux Reference

Quick reference for the maintainer. Not loaded by agents.

## Session management

```bash
tmux new-session -d -s <name> -c <directory>    # Create detached session
tmux send-keys -t <name> "command" Enter          # Send command
tmux capture-pane -t <name> -p                    # Capture entire pane
tmux capture-pane -t <name> -p | tail -40         # Last 40 lines
tmux kill-session -t <name>                       # Kill session
tmux ls                                           # List sessions
```

## Sending keys to OpenCode TUI

| Purpose | Command |
|---|---|
| **Chat input (literal)** | `tmux send-keys -l -t <name> "text" Enter` |
| **Chat input (interpreted)** | `tmux send-keys -t <name> "text" Enter` — RISK: shell mode |
| **Navigate permissions** | `tmux send-keys -t <name> Right Enter Enter` (Allow always) |
| **Allow once** | `tmux send-keys -t <name> Enter` |
| **Enter subagent view** | `tmux send-keys -t <name> C-x Down` |
| **Exit subagent view** | `tmux send-keys -t <name> Up` |
| **Navigate subagents** | `tmux send-keys -t <name> Left` (prev) / `Right` (next) |
| **Escape/dismiss** | `tmux send-keys -t <name> Escape` |

## Testing recipe

```bash
# 1. Start
tmux new-session -d -s test -c /path/to/project
tmux send-keys -t test "opencode" Enter

# 2. Wait for TUI (10-15s)
sleep 12

# 3. Check agent
tmux capture-pane -t test -p | grep "Build\|Plan\|Lead"

# 4. Send chat (LITERAL)
tmux send-keys -l -t test "Your task" Enter

# 5. Wait
sleep 15

# 6. Check status
tmux capture-pane -t test -p | tail -40

# 7. Handle permission prompt
# tmux send-keys -t test Right Enter Enter  (Allow always)

# 8. Enter subagent view if needed
tmux send-keys -t test C-x Down
tmux capture-pane -t test -p | tail -10

# 9. Clean up
tmux kill-session -t test
```

## Critical rules

- ALWAYS use `-l` (literal) for chat input. Without it, text becomes shell commands.
- NEVER kill a session while subagents are working. Use subagent view (`C-x Down`) to check.
- The `Right Enter Enter` pattern for Allow always: Right selects "Allow always", first Enter confirms selection, second Enter confirms the dialog.
