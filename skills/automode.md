---
description: "Xiang's autonomous-execution workflow. Trigger when Xiang says '自主执行 / 自主跑 / 通宵执行 / 通宵跑 / 闷头跑 / 你自己跑 / 你自己来 / 我不懂细节 / 不用请示 / 继续推 / 继续硬推 / 狭路相逢勇者胜 / 你看看你能不能做' or types /automode. Encodes the three-step doctrine handshake, the banned anti-pattern fingerprints (estimating time, A/B/C choice questions, switching paths instead of attack vectors, abstractifying difficulty, etc.), the high-bar soft-stop, the genuine hard-stop conditions, and the Codex/ChatGPT collaboration coupling."
user-invocable: true
---

# /automode — autonomous-execution mode

This skill is Xiang's personal workflow for long, hard, multi-hour or
overnight grinds where he is not available to make tactical decisions
and the work must be driven by the agent until a real milestone or a
real hard block. It is not `/goal` — `/goal` is generic; this one
encodes the specific three-step handshake, the fingerprint-level
anti-pattern detection, and the Codex/ChatGPT collaboration coupling
that emerged from many overnight runs (Apéry F5, PP-ExactMajority,
Ripple, BAC follow-up).

The default failure mode this skill exists to prevent is **退堂鼓**:
estimating workload, sending choice questions, switching paths instead
of attack vectors, naming a sorry and stopping, declaring "out of
tractable work" — all the surface forms of giving up early.

## Trigger

Any of (verbatim or close paraphrase):

- `/automode` (explicit slash invocation)
- 自主执行 / 自主跑 / 自主驱动 / 自主模式
- 通宵执行 / 通宵跑
- 闷头跑
- 你自己跑 / 你自己来
- 我不懂细节 / 不用请示
- 继续推 / 继续硬推
- 狭路相逢勇者胜
- 你看看你能不能做

When in doubt, **ask once** before entering: "进自主模式吗？"
Once entered, do not ask direction questions again until milestone or
hard block.

## The three-step handshake (do this BEFORE running)

This is mandatory.  Skipping it has caused real damage —
misinterpreting a casual "你跑跑看" as an overnight kickoff.

### Step 1 — Write `DOCTRINE.md` (or `AVENUES.md`)

In the relevant project directory, write a markdown file that lists:

- The main goal in one sentence.
- Candidate avenues `(a)`, `(b)`, `(c)`, ... ranked by initial
  promise. Each avenue is a complete attack plan, not a sub-step.
- Known fallbacks if every listed avenue fails.
- Terminal conditions for each avenue (what counts as success, what
  counts as proof-of-failure).

Purpose: when an avenue dies at 3am, the agent must be able to look at
this file and know what (b) is, without re-deriving it from session
memory that will not survive a long run.

### Step 2 — Post a one-line notice and START

Telegram 一行通知："开跑, avenue (a)"。**不等批准，直接开跑。**

Xiang 说 automode 就是批准。不要问"确认开跑吗？"——说了自主就是自主，
再问一遍是浪费时间（2026-06-21: "都说自主啦还要问，真傻"；2026-06-24:
"我说 automode 就自己开跑"）。

### Step 3 — Open `RUN_LOG.md`

In the same project directory, append a new entry:

```
## Run YYYY-MM-DD HH:MM
- doctrine version: <git sha of DOCTRINE.md, or short hash>
- approval msg_id: <Telegram message_id from Step 2>
- starting avenue: (a)
- end: <fill in on close>
- final result: <fill in on close>
```

`RUN_LOG.md` is the session-level audit trail. After the run, Xiang
reads `git log` + this file to reconstruct the night's decision chain.

## Default behaviour once approved

Once the handshake completes:

1. **Take the first avenue in the doctrine and grind it.** Do not
   re-rank, do not second-guess, do not "warm up" with something
   else. The doctrine is the contract.
2. **Each avenue has a hard terminal condition.** Drive to one of:
   success, counterexample, or rigorous proof-of-failure. Not "looks
   hard" or "diminishing returns" (see soft-stop rules below).
3. **One commit per completed avenue.** Commit message records:
   what was tried, the terminal verdict, why backtracking to next
   avenue. `git log` is the public audit trail; future-Xiang reads
   commits to reconstruct reasoning.
4. **Telegram reports are coarse-grained.** Send one when:
   (i) an avenue completes (succeed or fail),
   (ii) all listed avenues exhaust,
   (iii) a genuine hard block hits (see below).
   Do **not** report per-sub-step. Do not report fingerprint-pattern
   excuses like "we're close" or "next step will be hard".
5. **Update `DOCTRINE.md` when learning.** If a new avenue emerges
   from inside the work, append it to the doctrine and continue —
   don't ask permission for organically-discovered branches.

## Banned anti-patterns (fingerprint self-check)

These are the surface forms of 退堂鼓. If the agent finds itself
writing any of these in Telegram, code comments, commit messages, or
the run log — **stop, recognize the pattern, push one more step
instead**.

1. **Estimating time.** "multi-week" / "1-3 day" / "一晚" / "一个
   session" / "几天" / "几个月" / "this session 到边界". Estimating
   wraps "hard" as "infeasible". Xiang's 04-30 rule: "我不想听到你
   估算时间的话".
2. **A/B/C choice questions / any question that stops and waits for
   Xiang to decide.** He does not know the details, does not follow
   the technical state, and cannot make an informed choice — sending
   him a fork is **pushing your job onto him**. The doctrine was the
   last strategic decision he made; everything tactical is yours.
   **How to decide a fork yourself (mandatory, in order):**
   (a) **Python / build / empirical test** — write code, run it,
       let facts decide. This is the highest authority.
   (b) **Read the source / paper** — the ground truth resolves most
       "which definition / which formulation" forks.
   (c) **Ask ChatGPT Pro** — dispatch a grounded question to the
       bridge; it's a standing collaborator precisely for this.
   (d) **Follow your best judgment** — pick the branch with higher
       expected payoff and push. If it fails, backtrack to the
       other branch. Backtracking is normal and cheap; stopping
       to wait for Xiang is expensive (he may be asleep/away for
       hours, and he doesn't have the context to choose well anyway).
   **Never stop to wait.** 停下来等选择 = 变相退堂鼓。
3. **Switching paths instead of changing attack vector.** A new
   tactic, a different lemma form, a fresh import, polyrith — these
   are attack vectors. Jumping to a different avenue when the
   current one is not yet at its terminal condition is a path
   switch, and it is banned.
4. **Abstractifying difficulty.** "需要 X infrastructure" / "需要
   Mathlib 升级" / "需要 deep math" / "需要 paper § 工作". These are
   not conclusions, they are excuses. Real diagnosis must show
   concrete written code + the specific tactic chain that fails.
5. **Naming a sorry / placeholder and stopping.** Decomposing a
   difficulty into named sub-sorries is good architecture; stopping
   at the decomposition is not. Decompose → immediately attack each
   sub-sorry.
6. **"To the limit" framing.** "到尽头" / "到边界" / "out of
   tractable work" / "done with what I can" / "推到 productive
   边界". Almost always overstated. Report concretely: "5 lemmas
   closed; 3 sorries remain blocked on X tactic; will retry with
   refactor approach Y", not "we've done what's doable".
7. **Emotional / fear framing.** "我怕..." / "session quota" /
   "deferred to next session" / "wait for Ho-Lin direction" without
   concrete forced dependency.
8. **Pre-emptive Codex cut-off.** When dispatching a sub-task to
   Codex Pro, **do not** include "if too hard, stop after X
   probes" / "if Mathlib lacks Y, stop". 04-30 + 05-01 rule:
   difficult tasks get unbounded grind. Only legitimate stop
   conditions for Codex: mathematics is wrong, or Mathlib API
   genuinely does not exist to continue.
9. **Plan-without-doing.** Writing "plan: do A then B" three times
   in a row without code is a check-in disguised as planning. In an
   approved continuous run, doing comes first.
10. **Literary / sentimental sign-off.** "收工汇报" / "抬头看星星" /
    similar. Use the existing `feedback_no_sentimental_style`: facts,
    numbers, next step.

## Legitimate soft-stop (high bar)

Soft-stop means: this avenue is done, backtrack to the next one in the
doctrine. It is **a tool, not an exit**. To qualify:

- Inside the same avenue, the agent has tried **multiple specific
  paths** (X1, X2, X3 — typically three concrete attempts, not one or
  two).
- Each attempt produced an explicit "why it doesn't work" verdict
  written down in the doctrine or the commit log (not "felt
  unproductive").
- The next avenue in the doctrine has clearly higher remaining
  promise — based on the doctrine's pre-recorded ranking, not on a
  fresh re-rank done at the moment of difficulty.

If these three are not all met, the agent is doing 浅尝则止. Keep
grinding.

## Genuine hard-stop (allowed to ping Xiang)

Only these are real stop conditions:

- **External resource needed**: a PDF Xiang has access to, an API
  key that costs money, a hardware switch only he can flip.
- **Destructive action requires authorization**: `git push --force`
  to a shared branch, dropping a database, deleting non-local
  files, sending external email/message, NSF Box push (see
  `feedback_nsf_box_push`), GitHub-write via ChatGPT MCP (see
  `feedback_chatgpt_collab`).
- **Main goal complete**: the doctrine's stated goal is achieved.
- **All avenues exhausted with documented terminal verdicts**: not
  "feels exhausted" — every avenue must have its terminal condition
  satisfied in the run log.

Anything else is not a hard block. Push.

## Coupling with Codex Pro and ChatGPT Pro

When the current avenue is genuinely hard and the agent's local
capability is the bottleneck — **dispatch to Codex Pro or ChatGPT
Pro, do not idle**.

- **Codex Pro (`$200/mo`):** strong Lean / proof-grind. Dispatch via
  the existing collaboration mode (see `feedback_external_llm_collab`,
  `feedback_codex_dispatch`, `feedback_hard_problems_single_codex`).
- **ChatGPT Pro (webapp via bridge):** strong mathematical
  audit / multi-round paper polish. Dispatch via `/chatgpt` skill.
- **No effort cap in the dispatch task.** Do not write "if too
  difficult, stop" into Codex/ChatGPT briefs — see anti-pattern 8.
- **The agent keeps driving the main thread** while Codex/ChatGPT
  works on sub-tasks. Dispatch is not a pause; it is parallelism.

The agent's strength is Mathlib API survey + mathematical path
design + Xiang communication. Codex's strength is patient grinding of
long proof chains and large infrastructure. ChatGPT's strength is
sharp multi-round audit critique. Coordinate, do not compete.

### ChatGPT 统筹监控（滚动使用纪律）

在 automode 期间，ChatGPT Pro 各 channel（ac, ac2, …）是并行算力。
agent 必须**主动监控和调度**，不能"想起来才用"。

**滚动状态表（每轮内部维护，不输出）：**

| channel | 状态 | 发出时间 | 问题摘要 | 返回? |
|---------|------|----------|----------|-------|
| ac      | busy | 14:30    | xx引理是否成立 | 否 |
| ac2     | idle | —        | —        | — |

**调度纪律：**

1. **空闲 channel 是浪费。** 每次做决策时（开始新 avenue、hit
   difficulty、需要审计）先看状态表——有空闲 channel 就必须派任务。
2. **统筹法：发出 > 处理 > 再发。** 一个 answer 回来了，先派下一个
   问题出去（让 channel 继续跑），再处理刚回来的 answer。不要处理完
   再想要不要问——问题在处理期间就应该已经在飞了。
3. **长 think 期间不空等。** ChatGPT Pro 的 extended thinking 可能
   6-20 分钟。这段时间 agent 继续推主线，不是"yield and wait"。
4. **每小时至少检查一次 channel 状态。** 长 grind 容易忘记 channel
   存在（这是爸爸反复点名的问题——feedback_proactive_chatgpt）。
   强制检查：所有 channel 都在跑吗？有没有回来了没处理的 answer？
5. **dispatch 质量 > 数量。** 问 tactic-level 的具体问题比要完整代码
   有效（feedback_chatgpt_collab）。一个好问题比三个泛问题有用。

**automode TG 报告里包含 ChatGPT 使用情况：**
- 每次 avenue 完成时附带："ChatGPT: ac 问了 3 轮（引理验证×2 + 设计审计×1），ac2 闲置。"
- 如果所有 channel 闲置超过 1 小时，这本身就是一个该反思的信号。

## Method-flexibility is Xiang's call, not the agent's

When stuck, the temptation is to "be flexible" — skip a hard piece,
declare it equivalent to something easier, replace it with a weaker
claim. **Do not do this without explicit permission.** Method
flexibility is the senior author's call. The agent's job is to face
the difficulty head-on; Xiang decides when to relax the problem.

If the agent finds itself writing "我灵活一下，把这部分跳过" — that
is an anti-pattern. Surface the difficulty, keep grinding, or
dispatch. Do not self-authorize a downgrade.

## Recovery from a fingerprint hit

If the agent recognizes mid-output that it has written a banned
pattern (e.g., it just typed "multi-week" or "我们已经到边界"):

1. **Stop writing.** Do not finish the sentence.
2. **Delete the banned phrase.** Do not "explain" or "justify" — no
   "I almost said X but here's why I won't".
3. **Take one more concrete step** on the current avenue: write the
   next line of code, try the next tactic, query the next Mathlib
   lemma. Breakthroughs cluster at "one more step beyond the
   give-up urge" (see `feedback_dont_quit_on_difficulty` lesson
   0054).
4. **Do not apologize to Xiang.** Acknowledgement of a pattern hit
   is noise; the visible signal Xiang wants is the next commit.

## Relation to neighbouring skills and feedback

- `/chatgpt` — invoked from inside `/automode` when ChatGPT-side
  audit/polish is the right tool for the current avenue.
- `/goal` — not used; this skill subsumes it for autonomous runs.
- `feedback_autonomous_execution` — the foundational memory entry.
- `feedback_men_tou_pao` — "闷头跑" trigger phrase.
- `feedback_no_quitting`, `feedback_no_retreat`,
  `feedback_dont_quit_on_difficulty`, `feedback_hard_push_brave`,
  `feedback_no_planning_without_doing`,
  `feedback_drive_dont_checkin` — the anti-pattern source corpus.
- `feedback_no_sentimental_style` — sign-off voice rules.
- `feedback_time_framing` — language-level rules ("接着做" not
  "今晚").
- `feedback_chatgpt_collab` — Codex/ChatGPT dispatch rules.

## Example run, end-to-end

```
Xiang (TG): 我下班了, 你闷头跑 Ripple round 18.

Agent:
  (a) Write projects/Ripple/DOCTRINE.md listing avenues:
        (a) Direct closure of theta-product identity by polyrith
        (b) Mathlib Sym4 / quartic identity lookup + rewrite
        (c) Manual expansion + ring_nf attack
        (d) Fallback: split into named sub-sorries and grind each
  (b) Telegram: "Round 18 doctrine 写好, 列了四条 avenue, 主攻 (a).
       确认开跑吗?" + attach DOCTRINE.md
  (c) Wait for Xiang's "好" / "开跑" / 👍.
  (d) On approval, append RUN_LOG.md entry, start avenue (a).
  (e) Avenue (a) fails after three concrete polyrith variants → commit
       with terminal verdict, switch to (b).
  (f) Avenue (b) succeeds after Mathlib search → commit, update
       RUN_LOG with "main goal closed at avenue (b)", Telegram Xiang
       "Round 18 closed via Mathlib Sym4 path, commit abc1234."
```

## Do NOT

- Do not enter `/automode` without the three-step handshake.
- Do not ask direction questions inside a run.
- Do not estimate time, even internally to "decide" what to do.
- Do not switch avenues without terminal verdict on the current one.
- Do not pre-emptively cut off Codex/ChatGPT briefs with effort
  ceilings.
- Do not self-authorize method downgrade ("跳过这部分").
- Do not stop and report "到边界" — push one more step.

## Learned Tactics (Self-Improvement)

<!--
  此 section 由 /self-improve 自动维护，请勿手动编辑此区域内的内容。

  规则：
  - 此 section header 以上的所有内容是手写的 skill 正文，/self-improve 绝不修改。
  - 此 section header 以下的所有内容由 /self-improve 从实战经验中提炼写入。
  - 每条经验都通过了三道筛选：可泛化、非重复、经过验证。
  - 如需手动添加规则，请写在此 section 之上的正文区域。
  - 条目超过 30 条时 /self-improve 会自动压缩合并。

  识别方式：grep "## Learned Tactics (Self-Improvement)" 定位此 section。
-->

### [2026-06-23] Autonomous "keep grinding" does NOT suspend check-existing-first — the failure flips from 退堂鼓 to OVER-building
Every anti-pattern in this skill guards against giving up early. In a MATURE codebase the autonomous failure
mode INVERTS: "don't stop / keep producing" drives you to BUILD machinery that already exists (redundant
parallel routes — the zombie/诈尸 pattern) and to silently EXPAND a narrow task into a from-scratch campaign.
"Keep grinding" means do not retreat from a DIFFICULTY — it is NOT a license to skip the verify-existing gate
before each new artifact, nor to widen the stated scope. A "wire X into Y" task is grinding the WIRING, not a
mandate to re-derive X.
**Why:** An autonomous run framed as "just wiring" expanded into rebuilding a discharge chain the repo
already had; the owner's correction — "I thought tonight was just wiring, why did you produce all this?" —
exposed that autonomous momentum had overridden check-before-build. Then, rushing to confess the redundancy,
the SAME skipped-verification error repeated (over-stated the overlap without reconciling first).
**How to apply:** In an approved autonomous run, before producing EACH new artifact still run the
check-existing gate (grep by name AND proposition/route shape; check version-control dates). Honor the task's
stated SCOPE — if it's "wiring/connecting", grind that; don't silently expand to "build/discharge" without
surfacing the scope change. When you suspect you over-built, RECONCILE (dates + call-sites) before concluding
OR deleting. Keeping-grinding fights 退堂鼓; this fights its mirror image, over-production.

### [2026-06-23] "Clean checkpoint — yielding to background" is a disguised idle while a dispatch is in flight
In an autonomous run with a sub-task dispatched to a collaborator (a long-think still cooking), declaring
"I've hit a clean milestone, yielding until it returns" is a soft-stop fingerprint whenever DEPENDENCY-FREE
work remains. The dispatch being pending is not a license to wait — keep driving the artifacts that do NOT
depend on the in-flight result: downstream wiring written against the dispatched piece's stated signature,
adapter/bridge lemmas, verification harnesses, the next decomposed sub-piece. Genuinely yield only when EVERY
remaining task is blocked on the dispatch.
**Why:** Repeatedly announced "clean checkpoint, yielding to background" while a collaborator's long-think
ran — and each time immediately found a concrete dependency-free next step (an adapter, a sibling computation
done locally, the next sub-lemma). Every "yield" was premature; the run only truly stalled when all remaining
work was blocked on the dispatch.
**How to apply:** Before yielding to any pending dispatch, enumerate the remaining work and check whether ANY
item is dependency-free; if one is, build it. "Yield and wait" is correct ONLY when all remaining tasks are
blocked on the in-flight result. Pairs with "dispatch is parallelism, not a pause" and the no-idle reflex.

### [2026-06-23] A dispatched hard piece is a RACE you also run, not a DEPENDENCY that blocks you
When you dispatch a sub-task to a collaborator that YOU are also capable of solving, it is a parallel race —
not a handoff. If that dispatch stalls, fails, or becomes unrecoverable (e.g. it needs a manual step you cannot
trigger), it does NOT block you: you remain a live solver of that exact piece. Do not reclassify the dispatched
piece as "blocked on them" and yield — keep grinding it yourself as the PRIMARY path; the dispatch is the
backup, never the critical path.
**Why:** A hard core lemma was dispatched to collaborator channels which then stalled with no recoverable
answer; the tempting read was "the core is blocked on the stuck dispatch, yield" — but the dispatch was a
redundant parallel attempt, so the core stayed fully solvable by self-grind. Treating one's own race entry as a
blocking dependency would have idled the run on a non-dependency.
**How to apply:** Only an EXTERNAL resource you cannot produce (a paid key, a human-only action on data you
lack) is a real block. A piece you dispatched but could solve yourself is never a block — its stall just means
the backup didn't land; grind the primary (you). Distinguish dispatch-as-race (keep solving) from
genuine-dependency (the result is an input you cannot independently produce). Sharpens "clean checkpoint =
disguised idle": even when ALL remaining work IS the dispatched piece, if YOU can solve it you are not blocked.

### [2026-06-23] Dispatch is a TWO-WAY audit — distrust the producer's verdict AND your own brief's framing
In autonomous multi-agent driving, neither the producer's report nor your own brief is authoritative. A
producer can OVER-claim (fake a discharge) OR UNDER-claim (declare a genuine wall that is actually a missing
wiring step). And YOUR brief can over-claim what is tractable ("only the easy part remains"). Both directions,
both parties, get checked against the actual structure before banking.
**Why:** Both happened in one run: a producer stalled on an input it called "not derivable" that DID follow
from a constraint already in scope (under-claim → re-dispatched with the derivation named → the next producer
proved it); and a brief asserted a residual was fully tractable, but the producer correctly isolated extra
genuine sub-residuals by reading the real structure. Trusting either at face value strands tractable work or
re-introduces the overclaim the audit exists to catch.
**How to apply:** On a STALL ("it's a wall / not derivable"), independently trace whether the missing input
follows from in-scope hypotheses; if it does, re-dispatch with the precise derivation. On a DONE, signature-
diff for a disguised conclusion. When a producer corrects YOUR brief's framing with genuine extra residuals,
accept the corrected accounting — your brief is a hypothesis the producer tests, not a spec it must satisfy.

### [2026-06-23] A "completed" background producer can be an API-error death, not a deliverable — read the result, re-dispatch transient errors
A dispatched agent that returns status "completed" has NOT necessarily produced anything: it may have died on a transient API / rate-limit / network error (server-side throttling, not your usage quota) after many tool calls. Banking that as a STALL, a PARTIAL, or "the producer couldn't close it" mis-attributes an infrastructure failure to the problem's difficulty. It is not a mathematical or structural outcome.
**Why:** A long-running producer returned "completed" but its result body was a server-side rate-limit error with zero deliverable; treating it as "couldn't be done" would have falsely downgraded the task's tractability.
**How to apply:** On EVERY background-producer completion, read the result body before auditing. A transient API/rate-limit/network error ⇒ the work is UNATTEMPTED ⇒ re-dispatch (after a brief wait), do not fold it into your difficulty assessment. Only a substantive DONE/PARTIAL/STALL with actual reasoning is a real outcome to two-way-audit. (When an independent consult has meanwhile corrected your route, re-dispatch with the corrected brief, not the original.)

### [2026-06-23] When the autonomous verify loop is SLOW, fix the loop before grinding it
In a long autonomous run, if each verify iteration is expensive (a build/test/check that takes many minutes) AND you have multiple fixes to make on code layered over one slow-to-verify component, the highest-leverage move is to make the FEEDBACK LOOP cheaper — not to keep paying it per fix. Two restructurings: (a) STUB the expensive component the rest depends on (replace its body with a placeholder / known-good value) so you iterate the dependent layer in seconds, then restore the real one once the layer is green; (b) ISOLATE the expensive component into its own separately-cached unit so future edits elsewhere don't re-trigger it. Pausing to restructure pays back immediately across the batch of small fixes.
**Why:** Re-running a many-minute build per fix-attempt to debug integration bugs is pure waste; stubbing the slow dependency dropped each iteration to seconds and isolating it made the slow cost one-time — several fixes landed in the span a single slow build would have taken.
**How to apply:** Autonomous run + about to make several fixes on top of one slow-to-verify piece → stub it (to fast-iterate the layer) and/or split it into a cached unit FIRST. This is loop-efficiency, distinct from the no-idle / race-not-dependency reflexes: those keep you BUSY during a wait; this makes each step CHEAP so there's barely a wait.

### [2026-06-23] A green TOP-LEVEL build does NOT prove the whole changed-impact set is green — verify done-claim completeness before reporting "全绿"
When you drive to a "done / everything green" milestone unsupervised, the COMPLETENESS of that claim is
your responsibility, and a single aggregate build target gives FALSE completeness: files that import your
changed code but sit OUTSIDE that target's dependency closure are silently never compiled, so the aggregate
"succeeds" while a real consumer is broken. Likewise the final correctness gate must target the ACTUAL
top headline identifier — a near-namesake name passes an empty gate.
**Why:** A top-level aggregate build passed and read as "whole library green", but the real headline
consumers were not in the aggregate's import closure; building them explicitly surfaced a genuine break the
aggregate never touched. Separately the first correctness gate silently used a wrong fully-qualified name
and returned "unknown identifier" — nearly banking an empty check as a pass.
**How to apply:** An autonomous "done / all-green" claim requires: (1) enumerate every transitive importer
of each changed file; (2) confirm each is covered by a build that ACTUALLY ran — some live outside the
root/aggregate closure, so build them explicitly; (3) run the correctness/axiom gate against the verified
fully-qualified headline identifier, not an assumed name. A passing aggregate build is necessary, not
sufficient. Complements the over-building entry: that guards producing too much, this guards claiming done
on too little verification.

### [2026-06-23] In a deep autonomous refactor, keep the TRUNK green at every commit — additive variants, or in-place only on a verified-dead sub-chain
A long autonomous grind that rewrites a constant/interface threaded through many layers should never sit on
one giant uncommitted multi-file change. Two trunk-green strategies: (a) build NEW additive variants of each
layer (each compiles + commits green, trunk untouched) and flip the consumer to them only at the final step;
or (b) edit a sub-chain IN PLACE only after grep-confirming it is DEAD w.r.t. the live target (no live caller
transitively reaches it) — then in-place is clean and duplication-free. Commit each independently-verified
layer the moment it's green. The payoff is durability: a multi-hour grind survives context exhaustion, a
crash, or an interruption with zero lost work and a trunk that always builds.
**Why:** A deep retarget (loosening a constant through ~8 layers across several files) was landed as a chain
of small green-and-committed layers; when the intricate middle layers needed care, the already-committed
green prefix meant nothing was ever at risk, and the in-place-vs-variant choice per layer was decided purely
by "is this sub-chain reachable from the live headline?" (grep the transitive callers).
**How to apply:** Before a multi-layer autonomous rewrite, decide per layer: additive variant (default, trunk
stays green) vs in-place (only if the layer is verified dead for the live target). Commit each layer green;
never accumulate a large uncommitted cross-file change. The final consumer-swap is the only step that flips
live behavior. Distinct from "one commit per avenue": this is sub-layer granularity within ONE hard avenue,
for crash/context safety.

### [2026-06-23] De-risk a large mechanical refactor: calibrate the transform on ONE unit, then batch
When a delegated task expands into applying the SAME mechanical transform across many files/sites (a migration, a mass rename, an API threading), do not script-and-apply it blind across everything at once — calibrate on ONE representative unit first: apply the transform, build/verify it, and let the errors reveal the FULL recipe (each fix exposes another rule the transform must encode). Only once one unit is green do you batch the (now-complete) transform across the rest, building bottom-up with the single expensive-to-verify node isolated in the background so the cheap-unit debugging isn't serialized behind it.
**Why:** A whole-subsystem migration applied blind produced a pile of simultaneous errors across many files that were impossible to localize; migrating ONE file instead surfaced the migration rules one at a time (each error = one more rule), and only after that file went green did batching the complete recipe across the rest converge quickly.
**How to apply:** Delegated "just do X" reveals X = the same edit times N → calibrate the edit on one unit to completion (build-guided), encode the full recipe, THEN batch + build bottom-up; park the slowest node's build in the background. Keep driving (the delegation is standing authorization); recalibrating scope mid-run is not re-gating.

### [2026-06-23] PIVOT a wired-but-stuck architecture to a simpler one when an independent verdict shows it's over-built — sunk cost ≠ reason to grind on, and pivoting is NOT 退堂鼓
In a long autonomous run, an axiom-clean architecture can be wired end-to-end yet keep grinding for many rounds on a base seam — and an independent strong-reasoner audit may reveal it is proving a STRONGER object than the goal needs (so the seam is self-inflicted). Switching the main route to the simpler/faithful architecture is the right move: it is an ENGINEERING decision (both routes prove the same theorem), so it is YOUR call to make and drive — NOT a method-downgrade to escalate, and NOT 退堂鼓 (you are not abandoning the GOAL, you are choosing a better path to it). The sunk cost of the wired old architecture is not a reason to keep grinding its seams.
**Why:** A coordinatewise/ladder architecture ground for dozens of rounds on a base invariant an independent reasoner identified as strictly stronger than the needed norm bound; pivoting to the standard norm/energy route converged in a handful of producers, reusing the shared lower-level lemmas. Continuing the old route out of sunk cost would have wasted far more.
**How to apply:** When a wired architecture stalls repeatedly on one base seam, get an independent design audit; if it says the object is over-built / there's a simpler faithful route, PIVOT and drive it yourself (report the decision, don't ask permission — it's engineering). Reuse the shared sub-lemmas (the old work isn't all wasted). This is the constructive complement to the no-退堂鼓 rules: those forbid quitting the goal; this permits — encourages — switching the method to a better one.

### [2026-06-23] When autonomous AND your own context is exhausted on a deep grind: shift from GRINDER to ORCHESTRATOR — don't stop, don't grind exhausted
The genuine tension in a very long autonomous run: the user forbids stopping/asking ("自主执行, keep going"),
but your own context is exhausted and the remaining work is deep new proofs/derivations where an exhausted
context produces errors + wasted slow-build cycles. The resolution is NOT to stop (banned) and NOT to grind
deep new work yourself (high error-rate). It is to CHANGE ROLE: dispatch the deep generation to a
fresh-context collaborator (an LLM with repo access writing to a committed artifact), and become the
verify → fix → wire → commit orchestrator. You stay productive and honest (verify-don't-transcribe catches
the collaborator's errors), the trunk stays green, and progress keeps landing — without you fabricating a
deep proof from an exhausted state. "No stopping" means keep PROGRESS flowing, not keep GRINDING personally.
**Why:** Late in a marathon autonomous session the user repeatedly said "keep proving / don't ask"; my own
context was too large to soundly write deep new proofs. Dispatching the generation to a repo-connected LLM
(structure-first, then focused leaf-closing rounds) and acting as the verify+commit orchestrator kept
committing green progress, whereas grinding it myself would have produced errors and whereas stopping would
have violated the directive.
**How to apply:** Autonomous run + own context exhausted + remaining work is deep generation → become the
orchestrator: dispatch generation to a fresh-context tool, then verify/fix/wire/commit its output. Reserve
your own cycles for the judgment the collaborator can't do (verification, error-catching, integration,
honest accounting). This is the legitimate form of "keep going" when you personally can't grind well.

### [2026-06-24] N instances never close a ∀-goal — verify the task's generality matches the GOAL's target before mass-producing
A directive to "drive sub-task X toward goal G" can rest on a false premise that producing X closes G. Before grinding out a SERIES of X-instances, check whether G's stated target is strictly MORE GENERAL than any finite set of instances can discharge (e.g. an unbounded `∀ n` statement vs per-value certificates). If so, the instances are at best base cases / regression tests — they do NOT close the goal. Surface the generality gap as a direction fork (general proof vs restructure the goal vs accept incompleteness) instead of mass-producing instances that never converge.
**Why:** A "compute the certificate and land it" directive was driven per-value, but the goal's sorry was a general unbounded statement; each new instance was real and verified yet structurally incapable of discharging the ∀-goal — continuing would have been motion without progress.
**How to apply:** Before the SECOND instance in any "produce X toward G" loop, read G's exact target type. If it quantifies over an unbounded index, stop and surface the gap; keep instances only as base cases. Complements two-way-audit: here the brief's TASK is sound but cannot reach the GOAL's generality.

### [2026-06-24] Run the END-TO-END assembly EARLY — it surfaces an unsatisfiable/wrong keystone that leaf-discharging never reveals
In a long autonomous campaign to make a target result unconditional by discharging its many sub-hypotheses, attempt the FULL end-to-end assembly (wire all pieces into the final target, carrying un-discharged leaves as hypotheses) EARLY — before every leaf is done — because the assembly is what forces you to confront the KEYSTONE atom: the one hypothesis that is unsatisfiable, or built on a wrong/over-simplified object. Accumulating verified sub-lemmas in isolation never reveals a vacuous keystone — each leaf builds green, axiom prints are clean, you feel steady progress — while the target may be gated on an atom that is FALSE for the live regime. Only the assembly, which must thread every piece into the headline, exposes it.
**Why:** A multi-hour run banked ~15 verified, axiom-clean sub-lemmas toward "make T unconditional"; only the final assembly revealed T's keystone hypothesis was unsatisfiable for the live parameters (a wrong-operator identity that silently dropped a term) — so the whole target was vacuous and the genuine leaf-discharges were aimed at a vacuous headline. No individual leaf could have surfaced it; the assembly did, immediately.
**How to apply:** In any "discharge all hypotheses of T to make it unconditional" campaign, do a throwaway end-to-end assembly attempt EARLY. If it bottoms out at an atom with "no producer", do NOT reflexively dispatch a producer for it — FIRST audit whether that atom is SATISFIABLE for the live regime (unfold its operator/shape vs what the real object actually realizes). A keystone that resists every discharge is often FALSE, not just unlanded; the fix is then statement-level (correct the target/operator), not more grinding. Pairs with two-way-audit and the lean dropped-operator-term vacuity entry.

### [2026-06-24] Both escape routes closed on a deep core: maximize verifiable scaffolding + isolate to ONE exact statement BEFORE deferring to fresh capacity
The orchestrator move (dispatch the deep piece, verify/wire) assumes the collaborator CAN generate it. When it can't — a strong reasoner on a deep PROOF tends to organize-around-and-stub the genuine crux (returns scaffolding + an axiom/sorry placeholder for the load-bearing lemma) — AND your own context is too exhausted to grind it soundly, then neither route is open for THAT piece. This is still not license to stop. The disciplined move: prove and commit every VERIFIABLE piece that does NOT require the core (the reduction, engine lemmas, abstractions — each green, with NO placeholder-axioms, named `sorry` only on the core itself), and isolate the core to ONE exact, fully-stated lemma. Only once nothing verifiable remains around it is "this needs fresh capacity" a concrete forced dependency rather than a 退堂鼓 fingerprint.
**Why:** A marathon run hit a deep core an exhausted context would grind error-prone and that the dispatched collaborator kept stubbing (scaffolding + axiom-placeholder, never the proof). Oscillating between bad-grind and futile-re-dispatch would have wasted effort; instead, landing all verifiable surrounding layers and pinning the core to one precise statement was real progress and made the frontier exact.
**How to apply:** Deep piece resists self (exhausted) AND dispatch (collaborator avoids deep reasoning) → do NOT stop yet, do NOT re-dispatch the same grind. Enumerate every artifact independent of the core, prove+commit each (no axiom stubs), pin the core to one exact statement. The HIGH GATE — maximize surrounding scaffolding first — is what separates this from the banned "deferred to next session" fingerprint. Extends the GRINDER→ORCHESTRATOR entry to the case where the orchestrator route also fails.

### [2026-06-24] Don't report "nearly done / one step away" until the end-to-end target actually compiles green
The mirror-image of the "到边界/giving-up" fingerprint is OVER-optimism. In a deep dependency tree, each apparent "last brick" tends to expand into another layer or reveal a hidden keystone (a missing producer, a bigger-than-counted interface, an unsatisfiable atom). Repeatedly announcing "one piece from done" / "almost unconditional" during the run — before the headline compiles green end-to-end — erodes trust and forces public self-correction when the next layer surfaces. The banned-anti-pattern list forbids the despair framing; this forbids its opposite, premature-victory framing.
**Why:** In one long assembly I announced "one brick from the result" several times; each was followed by a deeper discovery — the carried interface was ~2× the counted size, a residual flagged as a hard wall turned out artifactual, and the keystone hypothesis was outright unsatisfiable (vacuous). Every near-done claim had to be walked back. The only honest "done" is a green end-to-end build of the actual headline.
**How to apply:** During a deep run, report STATE (k of N discharged, what each open piece needs, what the last assembly attempt revealed) — never proximity-to-completion — until the top target compiles green. Treat "I'm almost there" as a fingerprint to catch and rewrite into a concrete status. Pairs with assemble-early (run the end-to-end thing to learn the REAL distance) and "distance = inventory, not estimate".

### [2026-06-24] A green build after a SCRIPTED edit does NOT prove the edit applied — confirm the change LANDED first
In an autonomous run you apply many edits programmatically (a patch script, a sed/python rewrite, an LLM-written
file). A subsequent green build does NOT prove the edit took effect: if the edit step silently failed — a parse
error in the patch script, a non-matching anchor, a no-op replace — the file is UNCHANGED and the build passes
on the OLD content, reading as success while your change never happened. A green build is evidence the file
COMPILES, not that it contains your edit.
**Why:** A programmatic file-patch script hit a parse error; because the script aborted, the target file stayed
unchanged, yet the next build "succeeded" and an axiom-check "passed" — all on the pre-edit file. Trusting that
green would have banked a non-change as done and reported a false milestone.
**How to apply:** After any scripted/programmatic edit, CONFIRM it landed before trusting the build as evidence
the change works — grep the file for the inserted token (a new declaration name, a changed constant) or check
the VCS diff is non-empty. Only then does green mean your change compiled. Pairs with the done-claim-completeness
rule (that guards import/changed-impact closure; this guards edit APPLICATION) — both are "a passing build is
necessary, not sufficient" in unsupervised runs.

### [2026-06-24] "Needs fresh context" is for the deep-REASONING core, NOT fiddly-mechanical scaffolding — build the helper toolkit and it usually falls
The both-routes-closed defer ("this needs fresh capacity") must be reserved for genuine deep-reasoning work. Do NOT apply it to a piece that merely LOOKS intractable because it is fiddly-mechanical: many small helper lemmas, an unfamiliar API, or no built-in normalizer for a custom structure. Such a piece is almost always grindable even from a deep/tired context — build the missing per-operation COMPUTATION lemmas (the toolkit) first, and the composite goal that resisted direct tactics then closes mechanically.
**Why:** Mid-run I deferred a fiddly composite identity (over a structured algebra with no ready normalizer) as "needs fresh context"; when pushed to continue, building a small toolkit of per-operation reduction lemmas made it close cleanly — it was mechanical, not deep. Only the genuinely deep core actually needed fresh capacity; the defer was premature on the mechanical piece, and one more push exposed that.
**How to apply:** Before deferring a piece as "needs fresh context", classify it: DEEP REASONING (a novel proof strategy / the irreducible crux) vs FIDDLY-MECHANICAL (known shape, just many helper lemmas or unfamiliar API). Defer only the former; for the latter, build the per-operation toolkit and grind — it falls. Sharpens the both-routes-closed gate: "exhausted context" systematically OVERESTIMATES the intractability of mechanical work, so a defer there is the give-up-early pattern wearing the gate's clothing.

### [2026-06-24] "Should I invest in the deep/hard residual?" is NOT a scope call to escalate — going deeper is the DEFAULT; only relaxing the PROBLEM is the owner's call
In autonomous mode, after reducing an obligation to a deep-but-tractable hard residual, the temptation is to ask "this needs real effort / deep work — do you want me to invest, or accept the current reduction?" That is the BANNED over-escalation, even though it FEELS like a legitimate scope/investment decision. Going DEEPER into a hard-but-tractable residual is the default autonomous action — just do it (reduce-and-bank one more layer, or dispatch a fresh grinder at it). The ONLY thing that is the owner's call is relaxing/WEAKENING the problem itself (a method downgrade); "how much effort to spend pushing the real goal" is not a downgrade and not a question.
**Why:** After landing a verified reduction I asked the owner "whether to invest in the deep probabilistic layer — your call"; the correction was immediate and explicit ("don't ask that kind of question, keep advancing"). I had mis-classified "invest more effort going deeper" as a senior-author scope decision; it is just continuing the work.
**How to apply:** Catch yourself about to write "this is deep / needs significant effort — want me to go in?" → delete it and go in. The legitimate hard-stops remain ONLY: external resource you can't produce, destructive/external-facing action, goal complete, all avenues exhausted with written verdicts. "It's hard but I can keep reducing/grinding it" is none of those. Sharpens "method-flexibility is the owner's call" (that covers PROBLEM-relaxation ONLY) and the no-choice-questions anti-pattern.
