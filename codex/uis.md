---
description: "Codex workflow for running work on uisai1/uisai2 with persistent remote sessions. Trigger on 'open uis window', '跑训练', 'ssh uisai', '/uis', or requests to build/test/run on uisai. Uses direct ssh, remote tmux, shell, and git; strips local Zinan tmux, Telegram topic creation, and Claude-specific launcher assumptions."
user-invocable: true
---

# /uis — Codex Remote UIS Workflow

Use this when Xiang asks Codex to run work on `uisai1` or `uisai2`, especially
for long builds, Lean checks, training, or GPU/server jobs.

## Defaults

- Server: `uisai1` unless Xiang says `uisai2`.
- Session name: derive from the task/repo name.
- Agent: shell by default. Launch Codex remotely only if Xiang asks for a remote
  agent.

## Codex-Native Surface

Use direct shell commands:

```bash
ssh uisai1
ssh uisai2
```

For persistence:

```bash
ssh uisai2 'tmux new-session -A -s <session-name>'
```

For one-shot remote builds:

```bash
ssh uisai2 'cd /path/to/repo && <command>'
```

Do not create local Zinan tmux windows, Telegram forum topics, or bot routing
entries. Do not use tmux send-keys from Codex unless there is no safer direct
command path.

## Remote Session Pattern

1. Choose server and session name.
2. Ensure source code is present on the remote host.
3. Use `rsync` or `git fetch/pull` as appropriate.
4. Run the command inside a remote tmux session for long jobs:

```bash
ssh uisai2 'cd /path/to/repo && tmux new-session -A -s <session-name>'
```

For non-interactive long jobs:

```bash
ssh uisai2 'cd /path/to/repo && tmux new-session -d -s <session-name> "<command>; echo DONE"'
```

Then inspect:

```bash
ssh uisai2 'tmux capture-pane -pt <session-name> -S -200'
```

Prefer exact session names. Do not kill sessions you did not create.

## Running Codex Remotely

Only when explicitly requested:

```bash
ssh uisai2 'tmux new-session -A -s <session-name> "codex --dangerously-bypass-approvals-and-sandbox"'
```

Use the full bypass flag only when Xiang has requested that operating mode for
the remote agent.

## Reporting

Report:

- server;
- remote path;
- session name;
- command started;
- how to reattach or inspect logs.

Example:

```text
uisai2 上已启动 session `lean-batch`，路径 `/var/tmp/shen_p2`，命令是 `lake build ...`。
查看：ssh uisai2 'tmux capture-pane -pt lean-batch -S -200'
```

## Do Not

- Do not modify Telegram bot routing or forum topics.
- Do not assume a local `zinan` tmux session exists.
- Do not use Claude launcher commands.
- Do not force-kill ambiguous tmux sessions.
- Do not run destructive remote cleanup without explicit permission.
