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

### Step 2 — Send to Xiang, wait for explicit approval

Telegram the doctrine summary (or attach the .md file) and ask
literally: **"确认开跑吗？"**  Do not start until Xiang sends an
approving message. Approval looks like "开跑" / "好" / "go" / 👍 — any
clear positive.

This step exists because casual voice messages have been mis-heard as
launch commands. The explicit handshake prevents runaway sessions.

**SKIP the approval-wait when the invocation is a CONTINUATION of an
already-explicit autonomous run.** If Xiang has, in the SAME live
session, already been driving the agent autonomously — repeated
"继续"/"继续推", an explicit "自主模式"/"闷头跑", or an ongoing approved
grind that is plainly still in flight — then the `/automode` invocation
is a *continuation*, not a fresh kickoff, and the invocation itself IS
the approval. Still write `DOCTRINE.md` (Step 1) and `RUN_LOG.md`
(Step 3) and post a ONE-LINE "开跑, avenue (a)" notice, but do NOT stop
to ask "确认开跑吗？" — that re-ask is the redundant-handshake
anti-pattern Xiang flagged (2026-06-21: "都说自主啦还要问，真傻"). The
approval-wait is ONLY for a COLD start from a casual/ambiguous message
where runaway is the risk. When in doubt about cold-vs-continuation,
default to continuation if there is ANY explicit autonomous directive
already live in the session.

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
2. **A/B/C choice questions** sent to Xiang mid-run. He approved the
   doctrine; tactical branches are the agent's call.
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
