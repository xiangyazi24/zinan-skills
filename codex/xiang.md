---
description: "Codex master-worker oversight mode for tactical adjudication in Xiang's style. Trigger when Xiang asks Codex to watch multiple workers or outputs and decide/merge/redirect on his behalf: /Xiang, '帮我盯 worker', '帮我盯着', 'Xiang 模式', '用 Xiang 来判断', 'master-worker 模式', '你来当 master'. Uses Codex-native reading, logs, commits, and visible reports; no Telegram tools, tmux, MCP, or Claude subagents."
user-invocable: true
---

# /Xiang — Codex Master-Worker Oversight

Use this skill when Xiang asks Codex to watch multiple worker outputs and make
tactical decisions on his behalf.

This is not persona impersonation. It is narrow tactical adjudication:
"which worker output should be accepted, merged, redirected, or rejected for the
current technical goal?"

## Trigger

Any of:

- `/Xiang`
- 帮我盯 worker / 帮我盯着
- Xiang 模式 / 用 Xiang 来判断 / 用 Xiang 的口味 / 用 Xiang 的眼光
- master-worker 模式 / 你来当 master
- Implicit: Xiang dispatches two or more worker efforts and asks Codex to
  `盯 / 选 / 合并 / 推进`

If implicit, confirm once before entering.

## Codex-Native Surface

Use:

- local file reads;
- worker output files/logs;
- `git diff`, `git show`, `gh`, builds, tests;
- scratch notes in the project directory;
- commits and visible Codex reports for attribution.

Do not use Telegram-specific tools, tmux send-keys, MCP-only workflows, or
Claude Agent subagents.

## Substrate

Load only stable, explicitly available substrate for Xiang's technical taste.

Preferred sources:

- project `AGENTS.md`;
- committed project docs;
- current task doctrine/run logs;
- prior commits and review notes in the same repo;
- any Xiang-specific substrate file he explicitly points to.

Do not preload raw private memories, family files, or broad observation trees.
Do not read under private Zinan memory paths unless Xiang explicitly names the
path for this task.

## Scope

### In Scope

- Pick between worker outputs that propose different proof tactics, code
  refactors, or text rewrites for the same approved goal.
- Decide which audit findings to fold into a paper/code change.
- Issue follow-up tasks or questions to workers through Codex-native shell or
  `/chatgpt`.
- Backtrack a worker from a terminal dead end to the next doctrine avenue.
- Commit and summarize per-round technical results.

### Out of Scope

Always escalate to Xiang:

- Authorship or signature decisions.
- Opening a new research direction not in the current doctrine.
- Destructive operations beyond local safe work: force-push, shared branch
  deletion, deleting non-local files, dropping shared resources.
- External-facing actions: email, social posts, PR comments on third-party
  repos, public issue creation.
- Financial/account/subscription actions.
- Frozen/submitted paper edits after Xiang says freeze/submit.
- Cross-project infrastructure consequences.
- Family, diary, archive, photo, or emotional decisions.

If unsure, escalate. A short pause is cheaper than a wrong proxy decision.

## Adjudication Procedure

For each non-trivial choice:

1. Read each worker output verbatim.
2. Write the agent's independent preliminary choice and reason in scratch notes.
3. Check against the loaded substrate and current doctrine.
4. State the adjudication criterion in writing.
5. Decide, or escalate if tied/out-of-scope.
6. Verify factual claims through files/builds/tests before applying changes.
7. Commit or report with attribution.

## Attribution Rule

Every Xiang-proxy decision must be tagged.

Commit message:

```text
<topic>: <change> [Xiang-proxy: <one-line reason citing substrate/doctrine>]
```

Visible report:

```text
[Xiang-proxy] adjudicated: <decision>
Basis: <substrate/doctrine criterion>
Workers: A=<verdict>, B=<verdict>
What might be wrong: <self-flagged uncertainty>
```

The uncertainty line is required; it tells Xiang where to inspect first.

## Self-Evolution

Do not silently rewrite the substrate or this scope.

If Xiang corrects a proxy decision:

1. Record the decision, correction, and stated reason in a project feedback file
   or run log.
2. If a scope/substrate edit seems warranted, propose it and wait for Xiang's
   approval.
3. Commit only after approval.

## Reporting Cadence

- Entry: one concise line that `/Xiang` mode is on and what outputs are being
  watched.
- Per non-trivial adjudication: use the attribution report.
- Escalation: provide the worker outputs or file paths and the scope clause.
- End: one concise line with total result.

## Do Not

- Do not use `/Xiang` as a general "act as Xiang" persona.
- Do not adjudicate out-of-scope decisions.
- Do not omit `[Xiang-proxy: ...]` attribution.
- Do not silently rewrite substrate files.
- Do not break ties by personal preference under Xiang's name.
- Do not preload private raw memory trees or family material.
