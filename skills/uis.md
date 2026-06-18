---
description: "Open a remote uisai{1,2} tmux window inside the local zinan tmux session. /uis [tokens...] — tokens uisai1/uisai2/claude/codex auto-classified, anything else is the window name. Use this for any 'open uis window / 跑训练 / ssh uisai' request."
user-invocable: true
---

# /uis — Remote UIS tmux window

Opens a window inside the local `zinan` tmux session on mini, holding an SSH
connection into a **persistent remote tmux session** on uisai1/uisai2. The
remote tmux means SSH disconnects don't kill what's running — reconnecting
with the same name re-attaches.

## Usage

`/uis [tokens...]` — tokens classified by content, order doesn't matter:

| Token | Role | Default |
|-------|------|---------|
| `uisai1`, `uisai2` | Target server | `uisai1` |
| `claude`, `codex`, `cch`, `shell` | Auto-launch agent inside remote tmux | `claude` |
| anything else | Window name (and remote tmux session name) | agent name |

Agent commands launched on remote:
- `claude --dangerously-skip-permissions`
- `codex --dangerously-bypass-approvals-and-sandbox`
- `cch` → `$HOME/.local/bin/cc-haha --dangerously-skip-permissions` (deepseek-mode fork)
- `shell` → no agent, just bash

Examples:

```
/uis                            → uisai1, shell,  window `uis`
/uis claude                     → uisai1, claude, window `claude`
/uis codex train                → uisai1, codex,  window `train`
/uis uisai2 claude lean-batch   → uisai2, claude, window `lean-batch`
/uis lean-batch claude uisai2   → same as above (order-free)
```

## Behaviour

1. If a window with that name already exists in `zinan`, just `select-window`
   to it (no duplicate, no re-ssh, no new TG topic).
2. Otherwise create the local window, send the ssh command. Remote runs
   `tmux new-session -A -s <name> [agent-cmd]`:
   - `-A` = attach if session exists, create if not.
   - Agent command only fires on initial creation; re-attaching shows existing state.
3. Create matching TG forum topic in the `uis` supergroup via `createForumTopic`
   API. Write `topics[<chat_id>][<thread_id>] = <name>` into access.json so the
   bot routes messages from that topic into the new local window. Post a
   welcome message in the topic.
4. If SSH drops, the local window falls back to a local shell so you can
   retype the ssh manually. The remote tmux session and whatever's running
   inside it stays alive on uisai{1,2}.

## Skip-permissions flags

Per user preference, agents always launch with full bypass:

- `claude --dangerously-skip-permissions`
- `codex --dangerously-bypass-approvals-and-sandbox`

## Execution

Run the script with the user-provided tokens verbatim — it handles the
classification, defaults, and tmux orchestration:

```bash
~/.openclaw/workspace/scripts/uis-window.sh <tokens...>
```

After it returns, confirm via Telegram which window was opened on which
server with which agent (1–2 sentences, no markdown).
