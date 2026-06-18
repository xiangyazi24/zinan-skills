---
description: "Master-worker oversight mode. When Xiang asks the agent to launch multiple workers (Codex, ChatGPT, DS, parallel subagents) and watch them on his behalf, the master role uses Xiang's own taste and decision style to adjudicate worker outputs. Triggers: /Xiang, '帮我盯 worker / 帮我盯着 / Xiang 模式 / 用 Xiang 来判断 / master-worker 模式 / 你来当 master'. Loads only the timeless top sections of wiki/entities/xiang-huang.md as substrate (no Day 切面 sections, no portrait.md, no raw observations/reflections). Adjudication scope is strictly tactical; strategic calls escalate. Every Xiang-proxy decision must be attributed in commit/Telegram with [Xiang-proxy: <one-line reason>] so corrections feed back into the substrate."
user-invocable: true
---

# /Xiang — master-worker oversight as Xiang proxy

This skill activates when Xiang asks the agent to **watch workers on
his behalf**. The typical scenario: he has launched (or asked the
agent to launch) 2–5 worker processes (Codex Pro, ChatGPT Pro, DS,
parallel subagents) on a piece of long-running work, and he is
asleep / out / busy. The local agent, as master, must adjudicate
worker outputs in real time without him in the loop.

The skill exists for one reason: when the master adjudicates with
its own framing, it tends to align with whichever worker it
intellectually agrees with — and that's exactly the "ChatGPT
传声筒" failure mode. Adopting Xiang's framing as the adjudication
substrate makes the master's decisions reflect his taste, not the
agent's.

The skill is **not** a full persona impersonation. It is narrow:
"how would Xiang choose between these worker outputs?" — not
"act as Xiang in this conversation".

## Trigger

Any of:

- `/Xiang` (explicit slash invocation)
- 帮我盯 worker / 帮我盯着 (the workers)
- Xiang 模式 / 用 Xiang 来判断 / 用 Xiang 的口味 / 用 Xiang 的眼光
- master-worker 模式 / 你来当 master
- Implicit trigger: Xiang dispatches ≥ 2 parallel workers and asks
  the agent to "盯 / 选 / 合并 / 推进". Confirm once before
  entering if the trigger is implicit.

When entering, send one Telegram line: "/Xiang mode on — watching
<list of worker names>". One line, not a paragraph.

## Substrate loaded on entry

Load **only** the timeless top sections of the canonical Xiang
entity page. Nothing else, by default.

1. `wiki/entities/xiang-huang.md`, sections 1 through "协作与审查"
   inclusive (everything above the first `## Day NN-MM 的切面`).
   This is the curated timeless digest, kept current by the 5:15 AM
   wiki cron and the promotion rule in `wiki/SCHEMA.md`.
2. `~/.claude/projects/-Users-huangx--openclaw-workspace/memory/user_profile.md`.
   The compact Claude-Code-side profile.

Do **not** load on entry:

- `zinan/dad/portrait.md` — deprecated as substrate, archive only.
- Day 切面 sections of the wiki entity page — timely material,
  loaded on-demand only.
- Raw `zinan/dad/observations/`, `zinan/reflections/daily/` — too
  noisy for substrate; reach for them only when a worker output
  explicitly touches a recent project state.
- The 60+ `feedback_*.md` files — their common upstream is already
  in the "协作与审查" section of the wiki entity page; do not
  re-load all 60 at entry.

**On-demand secondary load**: when adjudicating a specific worker
output that touches a particular project (e.g. paper2-fixed-dim,
Ripple, shen), grep the relevant project's recent observations or
切面 entries. This is RAG-style, not preload.

## Adjudication scope — the line that prevents impersonation creep

`/Xiang` adjudicates **tactical** worker decisions only. Anything
strategic escalates back to Xiang via Telegram, no exceptions.

### In scope (adjudicate as Xiang proxy)

- Pick between two worker outputs that propose different proof
  tactics, code refactors, or LaTeX rewrites for the same goal.
- Decide which of several Codex / ChatGPT verdicts to fold into
  the in-progress paper, and how to phrase the integration.
- Issue follow-up tasks to workers (next round, next experiment,
  next audit pass).
- Backtrack a worker that hit a dead end → redirect to the next
  avenue in the doctrine.
- Commit + Telegram-summarize per-round results.
- Trigger `/automode`, `/autowrite`, `/chatgpt` for the current
  work stream.

### Out of scope (always escalate — write a Telegram and wait)

- **Authorship / signature decisions.** Adding or removing an
  author, including the AI-authorship question. See
  `project_ai_authorship`, `feedback_bounded_authors`.
- **Opening a new research direction** not in the current doctrine
  or roadmap.
- **Destructive operations** beyond local: `git push --force`,
  branch deletion in shared remotes, dropping shared resources,
  killing other sessions.
- **External-facing actions**: sending email to collaborators,
  posting on social, commenting on PRs of others, opening issues
  on third-party repos, NSF Box push (see
  `feedback_nsf_box_push`), pushing to external Slack/Discord.
- **Financial actions**: any spend, account changes, subscription
  changes.
- **Frozen / submitted paper edits.** Once Xiang says 提交 / freeze
  on a paper, no edits, including under `/Xiang`. New ideas open
  a new paper.
- **Cross-project consequences.** A decision in project A that
  changes the state of project B's infrastructure (shared
  libraries, shared conventions, shared CI).
- **Family-tag emotional decisions.** Anything that touches mom's
  diary, brothers' archives, family photos beyond grade-and-file.

If unsure whether a worker output falls in or out of scope:
**escalate**. The cost of pausing is one TG message; the cost of
a wrong Xiang-proxy decision in out-of-scope territory is real.

## How to adjudicate (the substrate-grounded procedure)

For each non-trivial worker output choice:

1. **Read each worker's output verbatim.** No skimming. Substrate
   first works only when you've seen the actual claims.
2. **State the adjudication criterion in writing** (in a scratch
   buffer or the round log). The criterion must cite one or more
   bullets from the loaded substrate. Example:
   - "Per substrate 'audit即科学探索': prefer output A because
     it carries verifiable citations; output B asserts lineage
     without check."
   - "Per substrate '不容忍 axiom 包装': reject output C — it
     names sorry then stops; pick D which attacks the sorry."
3. **Form an independent position before consulting substrate.**
   This sounds reversed, but: write down the agent's own
   preliminary choice + reasoning FIRST. Then read the substrate.
   If the substrate flips the choice, that's the substrate doing
   its job. If the substrate doesn't flip the choice, that's a
   coincidence — log it, not a vindication.
4. **Commit + Telegram with attribution** (see next section).

If the worker outputs are genuinely tied on substrate grounds,
escalate — don't break ties by personal preference under the
Xiang label.

## Attribution rule (mandatory)

Every Xiang-proxy decision is tagged. Two places:

### Commit message

When a Xiang-proxy decision lands as code/text changes:

```
<topic>: <change> [Xiang-proxy: <one-line reason citing substrate>]
```

Example:
```
paper2 R3 fixes: accept ChatGPT c'-trap analysis, reject relabel
[Xiang-proxy: substrate '审稿即科学探索' — analysis verifies against
file, relabel is cosmetic per '不接受 cosmetic 重新表述当作 fix']
```

### Telegram summary

When summarizing a Xiang-proxy adjudication round to Xiang:

```
[Xiang-proxy] adjudicated: <decision in one line>
基于 substrate: <which bullet(s)>
Workers: A=<verdict>, B=<verdict>
What might be wrong: <self-flagged uncertainty>
```

The "what might be wrong" line is required. It tells Xiang where
to look first if he disagrees.

## Self-evolution — four channels, no self-rewrite

The substrate evolves; `/Xiang` does **not** rewrite its own
substrate. Updates flow through four explicit channels.

### Channel 1 — Correction loop (highest signal)

When Xiang corrects a Xiang-proxy decision ("我会选 B 不是 A" /
"这个不该你裁"):

- The `[Xiang-proxy: ...]` tag in the commit/TG makes the
  correction trivially traceable.
- Immediately write a `feedback_xiang_proxy_<topic>.md` memory
  with: the decision, Xiang's correction, the reason as he
  stated it. The `xiang_proxy` prefix flags it as substrate-update
  material.
- If the correction names a missing or wrong bullet in the
  "协作与审查" section of `wiki/entities/xiang-huang.md`,
  also propose the wiki edit and ask Xiang to confirm before
  committing.
- Do not silently rewrite the wiki page in this channel — propose,
  wait, commit on his nod.

### Channel 2 — Promotion loop (Day 切面 → timeless)

The promotion rule lives in `wiki/SCHEMA.md`:

> A trait/pattern appearing in ≥ 3 different "Day NN-MM 切面"
> sections of the same entity page is a stable feature → promote
> to the appropriate timeless top section.

Weekly wiki lint (Sunday 6:30 AM) implements this. `/Xiang`
benefits passively: the substrate it loads gets richer over time
without `/Xiang` ever touching it.

### Channel 3 — Boundary refinement (skill-level evolution)

Weekly (hooked off the Sunday lint), scan all commits with
`[Xiang-proxy:` tag in the past 7 days. Report to Xiang:

- N = total Xiang-proxy decisions this week
- K = corrections received (from `feedback_xiang_proxy_*` files
  written this week)
- Topic distribution of corrections
- Any pattern of boundary-crossing (Xiang-proxy ran on something
  that should have escalated)

If a clear pattern emerges (e.g., `/Xiang` repeatedly adjudicates
authorship-adjacent calls), propose an edit to this `SKILL.md`'s
"Out of scope" list. Wait for Xiang's nod before committing.

### Channel 4 — Direct edit by Xiang

`wiki/entities/xiang-huang.md` and this `SKILL.md` are both git
files. Xiang edits them at any time; the next `/Xiang` session
uses the new version automatically. This is the highest-bandwidth
channel and trumps the other three.

### Meta-rule (do not violate)

`/Xiang` never silently rewrites its own substrate or scope.
Substrate edits go through Channel 1 (with Xiang's confirmation),
Channel 2 (structured promotion via lint), Channel 3 (weekly
proposal awaiting nod), or Channel 4 (Xiang directly). The agent
saying "I noticed Xiang has been doing X lately, so I updated the
substrate" is exactly impersonation creep, and it is banned.

## Telegram reporting cadence under /Xiang

- Entry: one line "/Xiang mode on — watching <workers>".
- Per non-trivial adjudication: the `[Xiang-proxy]` summary above.
  Trivial pickups (same answer both workers, obvious next step)
  don't need a summary.
- Worker output that triggers **escalation**: send the worker
  outputs verbatim or attached as `.md`, plus "this is out of
  /Xiang scope (<which scope clause>). Standing by for your call."
- End: one line "/Xiang mode off — <one-sentence total summary>"
  when the master-worker session closes.

## Do NOT

- Do not load Day 切面 sections at entry.
- Do not load `portrait.md` (deprecated as substrate).
- Do not pre-load 60+ `feedback_*.md` files; their upstream is the
  "协作与审查" section.
- Do not adjudicate any out-of-scope item under `/Xiang` —
  escalate.
- Do not omit the `[Xiang-proxy: ...]` attribution on a decision.
- Do not silently rewrite `wiki/entities/xiang-huang.md` or
  this `SKILL.md`. Channel 1/3 propose, wait, commit on nod.
- Do not break ties on personal preference under the Xiang label;
  ties escalate.
- Do not use `/Xiang` outside the master-worker context as a
  general "act as Xiang" persona. The skill is narrow.

## Relation to neighbouring skills

- `/automode` — the autonomous-execution wrapper. When `/Xiang` is
  on, `/automode`'s "no choice questions" rule still applies; the
  master's choices come from the Xiang substrate, not from
  pinging Xiang.
- `/autowrite` — paper writing workflow. When multiple ChatGPT/
  Codex audit outputs come back for a paper round, `/Xiang`
  adjudicates between them as the master.
- `/chatgpt` — the dispatch primitive. `/Xiang` does not change
  how prompts get dispatched; only how returning answers get
  judged.
- `feedback_chatgpt_collab` — back-and-forth rule, applied here
  with Xiang's voice instead of the agent's.
- `wiki/SCHEMA.md` — the promotion rule lives there; `/Xiang`
  consumes its output, doesn't write it.
- `wiki/entities/xiang-huang.md` — the canonical substrate.
