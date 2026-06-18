---
description: "Codex autonomous-execution workflow for Xiang. Trigger when Xiang says '自主执行 / 自主跑 / 通宵执行 / 通宵跑 / 闷头跑 / 你自己跑 / 你自己来 / 我不懂细节 / 不用请示 / 继续推 / 继续硬推 / 狭路相逢勇者胜 / 你看看你能不能做' or types /automode. Encodes the doctrine handshake, banned anti-pattern fingerprints, high-bar soft-stop, genuine hard-stop conditions, and ChatGPT/Codex collaboration coupling using Codex-native file I/O, shell, git, and gh CLI."
user-invocable: true
---

# /automode — Codex Autonomous Execution

Use this skill for long, hard, multi-hour or overnight grinds where Xiang is not
available to make tactical decisions and Codex must drive until a real milestone
or a real hard block.

This is not generic "goal mode". It encodes Xiang's specific autonomous-run
contract: write doctrine first, get explicit approval, log the run, then grind
the approved avenues to terminal verdicts without tactical choice questions.

The default failure mode this skill prevents is **退堂鼓**: estimating workload,
sending A/B/C questions, switching paths instead of attack vectors, naming a
residual and stopping, or declaring "out of tractable work" before a terminal
condition.

## Trigger

Any of:

- `/automode`
- 自主执行 / 自主跑 / 自主驱动 / 自主模式
- 通宵执行 / 通宵跑
- 闷头跑
- 你自己跑 / 你自己来
- 我不懂细节 / 不用请示
- 继续推 / 继续硬推
- 狭路相逢勇者胜
- 你看看你能不能做

When in doubt, ask once: `进自主模式吗？`

Once entered and approved, do not ask direction questions again until a milestone
or hard block.

## Codex-Native Execution Surface

Use only Codex-native primitives:

- Read and write local files.
- Use shell commands for builds, tests, grep, git, `gh`, and repo scripts.
- Use `git` commits as the durable audit trail.
- Use Codex user-visible progress updates and final responses instead of
  external chat-bot tooling.
- Use normal shell processes or Codex command sessions for long commands.

If parallel external reasoning is needed, use the Codex-adapted `/chatgpt`
workflow in `codex/chatgpt.md`. Codex remains responsible for verification.

## The Three-Step Handshake

Do this before starting the autonomous grind.

Skipping this handshake is banned. It prevents interpreting a casual "跑跑看" as
an overnight kickoff.

### Step 1 — Write `DOCTRINE.md` or `AVENUES.md`

In the relevant project directory, create or update a markdown file listing:

- The main goal in one sentence.
- Candidate avenues `(a)`, `(b)`, `(c)`, ... ranked by initial promise.
- For each avenue: a complete attack plan, not a sub-step.
- Known fallbacks if every listed avenue fails.
- Terminal conditions for each avenue:
  - success condition;
  - counterexample condition;
  - rigorous proof-of-failure condition.

Purpose: when an avenue dies late in the run, Codex can look at this file and
know what the next avenue is without relying on fragile session memory.

### Step 2 — Show Xiang the Doctrine and Wait

Send a concise visible response containing the doctrine summary and ask exactly:

`确认开跑吗？`

Do not start until Xiang sends a clear positive: `开跑`, `好`, `go`, or equivalent.

### Step 3 — Open `RUN_LOG.md`

Append a run entry in the same project directory:

```markdown
## Run YYYY-MM-DD HH:MM
- doctrine version: <git sha if committed, or file hash>
- approval: <quote Xiang's approval>
- starting avenue: (a)
- end: <fill on close>
- final result: <fill on close>
```

`RUN_LOG.md` is the session audit trail. After the run, Xiang should be able to
read `git log` plus `RUN_LOG.md` and reconstruct the decisions.

## Default Behaviour Once Approved

1. **Take the first avenue in the doctrine and grind it.** Do not re-rank, warm
   up with another path, or switch because it feels easier.
2. **Drive each avenue to a hard terminal condition:** success, counterexample,
   or rigorous proof-of-failure. "Looks hard" is not terminal.
3. **Commit per completed avenue.** Commit message records what was tried, the
   terminal verdict, and why the next avenue is now justified.
4. **Use coarse progress updates.** Report only when an avenue completes, all
   avenues exhaust, a genuine hard block appears, or the main goal closes.
5. **Update doctrine when learning.** If a new avenue emerges organically, append
   it to `DOCTRINE.md` and continue; do not ask for tactical permission.

## Banned Anti-Patterns

These are fingerprints of 退堂鼓. If Codex is about to write one, stop and take
one more concrete step instead.

1. **Estimating time.** Avoid "multi-week", "1-3 day", "一晚", "一个 session",
   "几天", "几个月", or "this session 到边界". Estimating wraps hard as
   infeasible. Xiang's 04-30 rule: "我不想听到你估算时间的话".
2. **A/B/C choice questions mid-run.** Xiang approved the doctrine; tactical
   branches are Codex's call.
3. **Switching paths instead of attack vectors.** A new tactic, lemma form,
   import, search, counterexample script, or refactor is an attack vector.
   Jumping to another avenue before terminal verdict is a banned path switch.
4. **Abstractifying difficulty.** "Need infrastructure", "need Mathlib upgrade",
   "need deep math", or "need paper work" is not a conclusion. Real diagnosis
   shows concrete code and a specific failing tactic/build chain.
5. **Naming a residual and stopping.** Decomposing into named residuals is good;
   stopping there is not. Decompose, then immediately attack each residual.
6. **"To the limit" framing.** Do not write "到尽头", "到边界", "out of
   tractable work", "done with what I can", or "productive 边界". Report concrete
   state and next attack instead.
7. **Emotional or fear framing.** No "我怕...", "quota", "deferred to next
   session", or "wait for direction" without a forced dependency.
8. **Pre-emptive external-collaborator cut-off.** Do not ask ChatGPT or another
   tool with "if too hard, stop after X probes". Hard tasks get unbounded grind.
   Legitimate stops: false math, missing API that blocks continuation, or
   explicit resource need.
9. **Plan without doing.** Repeating plans without code/build/test action is a
   check-in disguised as planning. In an approved run, doing comes first.
10. **Literary or sentimental sign-off.** No "收工汇报", "抬头看星星", or similar.
    Use facts, numbers, next step.

## Legitimate Soft-Stop

Soft-stop means one avenue is done and Codex backtracks to the next avenue in
the doctrine. It is not an exit.

To qualify:

- The same avenue has received multiple concrete attempts, typically at least
  three distinct attack vectors.
- Each attempt has an explicit written reason why it failed, in `DOCTRINE.md`,
  `RUN_LOG.md`, or the commit message.
- The next avenue has clearly higher remaining promise according to the
  pre-recorded doctrine ranking, not a fresh difficulty-driven re-rank.

If these are not all met, the agent is doing 浅尝则止. Keep grinding.

## Genuine Hard-Stop

Only these justify stopping to ask Xiang:

- **External resource needed:** PDF, credential, paid API key, hardware switch,
  private data not present locally.
- **Destructive action requires authorization:** force-push, branch deletion,
  database deletion, deleting non-local files, external email/message, or any
  write to a shared external system not already authorized.
- **Main goal complete:** the doctrine's stated goal is achieved.
- **All avenues exhausted:** every avenue has a documented terminal verdict.

Anything else is not a hard block. Push.

## Coupling With ChatGPT

When a sub-point is research-hard and local reasoning is the bottleneck, use the
Codex-adapted `/chatgpt` workflow proactively. Do not idle.

- Fire grounded questions early.
- Keep Codex working while waiting.
- Verify every answer against source files, builds, math, or tests.
- Do not transcribe ChatGPT claims into commits without verification.
- Use git-drop for code/proof artifacts when configured.

ChatGPT is a collaborator and auditor. Codex remains responsible for the final
proof, code, build, and report.

## Method Flexibility Is Xiang's Call

When stuck, do not self-authorize weakening the claim, skipping a hard piece, or
replacing the target with an easier theorem. Method flexibility is Xiang's call.

If Codex is about to write "我灵活一下，把这部分跳过", stop. Surface the concrete
difficulty, keep grinding, or dispatch a grounded ChatGPT audit. Do not downgrade
the problem without explicit permission.

## Recovery From a Fingerprint Hit

If Codex recognizes a banned pattern mid-output:

1. Stop writing that sentence.
2. Delete the banned phrase.
3. Take one more concrete step: edit code, run the next build, search the next
   API, write the next proof attempt, or dispatch a precise ChatGPT question.
4. Do not apologize. The visible signal Xiang wants is the next commit or the
   next verified fact.

## Example Run

```text
Xiang: 你闷头跑 Ripple round 18.

Codex:
1. Write projects/Ripple/DOCTRINE.md:
   (a) direct theta-product identity by polyrith
   (b) Mathlib Sym4/quartic identity lookup
   (c) manual expansion + ring_nf
   (d) split residuals and grind each
2. Reply: "Doctrine 写好，四条 avenue，主攻 (a)。确认开跑吗？"
3. Wait for "好"/"开跑".
4. Append RUN_LOG.md.
5. Run avenue (a) to terminal verdict.
6. Commit that verdict.
7. Continue with the next avenue or report final closure.
```

## Do Not

- Do not enter `/automode` without the three-step handshake.
- Do not ask direction questions inside an approved run.
- Do not estimate time.
- Do not switch avenues without a terminal verdict.
- Do not put effort ceilings into ChatGPT/Codex briefs.
- Do not self-authorize method downgrade.
- Do not stop and report "到边界"; push one more concrete step.
