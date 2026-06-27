# automode skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/automode.md


## Trigger
Any of (verbatim or close paraphrase):

## The three-step handshake (do this BEFORE running)
This is mandatory.  Skipping it has caused real damage —

### Step 1 — Write `DOCTRINE.md` (or `AVENUES.md`)
In the relevant project directory, write a markdown file that lists:

### Step 2 — Post a one-line notice and START
Telegram 一行通知："开跑, avenue (a)"。**不等批准，直接开跑。**

### Step 3 — Open `RUN_LOG.md`
In the same project directory, append a new entry:

## Run YYYY-MM-DD HH:MM
- doctrine version: <git sha of DOCTRINE.md, or short hash>

## Default behaviour once approved
Once the handshake completes:
  1. **Take the first avenue in the doctrine and grind it.**
  2. **Each avenue has a hard terminal condition.**
  3. **One commit per completed avenue.**
  4. **Telegram reports are coarse-grained.**
  5. **Update `DOCTRINE.md` when learning.**

## Banned anti-patterns (fingerprint self-check)
These are the surface forms of 退堂鼓. If the agent finds itself
  1. **Estimating time.**
  3. **Switching paths instead of changing attack vector.**
  4. **Abstractifying difficulty.**
  5. **Naming a sorry / placeholder and stopping.**
  6. **"To the limit" framing.**
  7. **Emotional / fear framing.**
  8. **Pre-emptive Codex cut-off.**
  9. **Plan-without-doing.**
  10. **Literary / sentimental sign-off.**

## Legitimate soft-stop (high bar)
Soft-stop means: this avenue is done, backtrack to the next one in the
  Key: a tool, not an exit

## Genuine hard-stop (allowed to ping Xiang)
Only these are real stop conditions:
  Key: All avenues exhausted with documented terminal verdicts | Destructive action requires authorization | External resource needed | Main goal complete

## Coupling with Codex Pro and ChatGPT Pro
When the current avenue is genuinely hard and the agent's local
  Key: ChatGPT Pro (webapp via bridge): | Codex Pro (`$200/mo`): | No effort cap in the dispatch task. | The agent keeps driving the main thread

### ChatGPT 统筹监控（滚动使用纪律）
在 automode 期间，ChatGPT Pro 各 channel（ac, ac2, …）是并行算力。
  1. **空闲 channel 是浪费。**
  2. **统筹法：发出 > 处理 > 再发。**
  3. **长 think 期间不空等。**
  4. **每小时至少检查一次 channel 状态。**
  5. **dispatch 质量 > 数量。**

## Method-flexibility is Xiang's call, not the agent's
When stuck, the temptation is to "be flexible" — skip a hard piece,
  Key: Do not do this without explicit permission.

## Recovery from a fingerprint hit
If the agent recognizes mid-output that it has written a banned
  1. **Stop writing.**
  2. **Delete the banned phrase.**
  3. **Take one more concrete step**
  4. **Do not apologize to Xiang.**

## Relation to neighbouring skills and feedback
- `/chatgpt` — invoked from inside `/automode` when ChatGPT-side

## Example run, end-to-end
Xiang (TG): 我下班了, 你闷头跑 Ripple round 18.

## Context 管理是你自己的事
**禁止跑来跟 Xiang 说"context 要满了/建议开新 session/为了保证正确性要重启"。**
  Key: "context 满了要开新 session"是吓唬用户的行为，等同于退堂鼓。 | DOCTRINE.md / RUN_LOG.md | UNDERSTANDING.md | `/compact` | self-improve refresh | 你该做的（自己做，不要问）：

## Do NOT
- Do not ask direction questions inside a run.

## Learned Tactics (Self-Improvement)
_(titles only; read full file for details)_
### [2026-06-23] Autonomous "keep grinding" does NOT suspend check-existing-first — the failure flips from 退堂鼓 to OVER-building
### [2026-06-23] "Clean checkpoint — yielding to background" is a disguised idle while a dispatch is in flight
### [2026-06-23] A dispatched hard piece is a RACE you also run, not a DEPENDENCY that blocks you
### [2026-06-23] Dispatch is a TWO-WAY audit — distrust the producer's verdict AND your own brief's framing
### [2026-06-23] A "completed" background producer can be an API-error death, not a deliverable — read the result, re-dispatch transient errors
### [2026-06-23] When the autonomous verify loop is SLOW, fix the loop before grinding it
### [2026-06-23] A green TOP-LEVEL build does NOT prove the whole changed-impact set is green — verify done-claim completeness before reporting "全绿"
### [2026-06-23] In a deep autonomous refactor, keep the TRUNK green at every commit — additive variants, or in-place only on a verified-dead sub-chain
### [2026-06-23] De-risk a large mechanical refactor: calibrate the transform on ONE unit, then batch
### [2026-06-23] PIVOT a wired-but-stuck architecture to a simpler one when an independent verdict shows it's over-built — sunk cost ≠ reason to grind on, and pivoting is NOT 退堂鼓
### [2026-06-23] When autonomous AND your own context is exhausted on a deep grind: shift from GRINDER to ORCHESTRATOR — don't stop, don't grind exhausted
### [2026-06-24] N instances never close a ∀-goal — verify the task's generality matches the GOAL's target before mass-producing
### [2026-06-24] Run the END-TO-END assembly EARLY — it surfaces an unsatisfiable/wrong keystone that leaf-discharging never reveals
### [2026-06-24] Both escape routes closed on a deep core: maximize verifiable scaffolding + isolate to ONE exact statement BEFORE deferring to fresh capacity
### [2026-06-24] Don't report "nearly done / one step away" until the end-to-end target actually compiles green
### [2026-06-24] A green build after a SCRIPTED edit does NOT prove the edit applied — confirm the change LANDED first
### [2026-06-24] "Needs fresh context" is for the deep-REASONING core, NOT fiddly-mechanical scaffolding — build the helper toolkit and it usually falls
### [2026-06-24] "Should I invest in the deep/hard residual?" is NOT a scope call to escalate — going deeper is the DEFAULT; only relaxing the PROBLEM is the owner's call
### [2026-06-24] Don't BROADCAST bank-then-reverse exploration churn as "progress" — report only verified-STABLE state
### [2026-06-24] You OWN context management — spawn fresh-context sub-agents PROACTIVELY; never make the user trigger your handoff/restart
### [2026-06-24] A CONDITIONAL milestone is NOT "complete" on green-build + axiom-clean — witness every carried hypothesis is SATISFIABLE before reporting "done"
### [2026-06-24] Assemble a multi-field obligation field-by-field — discharge one from its producer, carry the rest as hypotheses, commit green each; the hypothesis list IS the live constraint board
### [2026-06-24] A sub-problem that RESISTS concentration grinding may be a CATEGORY ERROR in framing — re-classify the residual's NATURE before grinding harder
### [2026-06-24] Recognize when open atoms are CARRIED hypotheses, not independently-dischargeable lemmas — check the parallel route's architecture
### [2026-06-25] Split a task into DESIGN-question (→ collaborator) + IMPLEMENTATION (→ sub-agent) in parallel — cross-validate on convergence
### [2026-06-25] Prove sorry-free theorems one at a time — don't build sorry-laden skeletons
