---
description: "Ask ChatGPT (web, incl. Pro/Extended) via the local bridge for collaborative research/audit. Call it with one script — `python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> \"question\"` — which blocks and prints ChatGPT's answer (the channel's tab has a GitHub repo connected). Often run as multiple rounds (R1, R2, ...). PROACTIVE DEFAULT, not a last resort: the reflex on ANY hard research/proof sub-point (a hard lemma, a tactic chain you can't see through, a modeling/design fork, an 'is X true') is to fire a grounded question to ChatGPT FIRST and verify in parallel — don't do long solo analysis while ac/ac2 sit idle (Xiang 06-15: '你为什么不会主动找 chatgpt 呢?'). Also fires whenever Xiang says '问 ChatGPT', '让 ChatGPT 看看', '和 ChatGPT 磋商', 'ChatGPT 协作模式', etc. Sub-command '/chatgpt clear' (also '清/重启/全清 bridge') runs scripts/bridge-clear.sh: a full bridge reset — stop, wipe all data (keep login token), restart clean."
user-invocable: true
---

# /chatgpt — ChatGPT webapp collaboration mode

This skill encodes the working pattern that produced the BAC PSPACE-decider
paper (DNA32 follow-up). You ask ChatGPT (web, incl. Pro/Extended) through a
local bridge and get the answer back programmatically. ChatGPT is not just an
API — it is a research collaborator we iterate with across many rounds.

This page is the CALLER's guide: how to call it and how to ask well. The bridge's
own internals (capture paths, debug/version handshake, markers) live in
`~/repos/chatgpt-bridge-pr3/UNDERSTANDING.md` — a caller doesn't need them.

## Proactive engagement — the default reflex (NOT a last resort)

ChatGPT is a standing collaborator, not an escape hatch for when you're already stuck. The failure
mode this section exists to kill (Xiang 06-15, twice: "ac 和 ac2 不能闲着" / "你为什么不会主动找
chatgpt 呢? 是上下文太撑了你忘记这回事了吗??"): doing long **solo** analysis on a hard sub-point
while the ChatGPT channels sit idle, then engaging ChatGPT only as a last resort. Context length is
never the excuse — engaging ChatGPT on hard sub-points is part of the standing job.

The reflex instead:

- **Fire first, verify in parallel.** The moment a research-level sub-point appears — a hard lemma, a
  failed tactic chain you can't see through, a modeling/design fork, an "is X even true" question —
  draft a grounded question and `ask-gpt.py` it **before** sinking time into solo derivation. Then do
  your own derivation *in parallel* to verify/refute the answer (rule 8: verify, don't transcribe).
- **Keep both channels saturated.** `ac` and `ac2` should not both be idle while there is open hard
  work. Pro long-thinks run 6–20 min; that window is exactly when a question should already be in
  flight — not when you first think about asking. When a channel is busy you cannot add to it
  (resubmitting truncates the in-flight answer), so the time to dispatch is *before* you go heads-down.
- **Dispatch-before-process (统筹).** When an answer returns and a next round is obvious, fire the next
  question (plus any independent second question) **before** sitting down to process the one in hand,
  so the channel works while you work.
- You do **not** need Xiang to say "问 ChatGPT" — proactively engaging on hard sub-points IS the job.
  The explicit triggers below are *additional* entry points, not the only ones.

## Trigger

Any of: `/chatgpt`, "问 ChatGPT", "让 ChatGPT 看看", "ChatGPT 协作模式",
"和 ChatGPT 磋商", "丢给 ChatGPT", "提问 ChatGPT". If Xiang's intent is
unclear (e.g. "ChatGPT 怎么看"), ask once whether to dispatch.

**Sub-command `clear`** — `/chatgpt clear` (also: "清 bridge / 重启 bridge /
bridge 全清 / 把 bridge clear / 全面重启 bridge") is NOT a question to ChatGPT.
It runs a full bridge reset; see "## /chatgpt clear" below. Don't call
`ask-gpt.py` for it.

## /chatgpt clear — full bridge reset

`/chatgpt clear` runs ONE script that stops the bridge, wipes ALL accumulated
data, and restarts it clean:

```bash
bash ~/.openclaw/workspace/scripts/bridge-clear.sh
```

What it does (encodes the manual reset Xiang did 2026-06-19):
- `launchctl bootout` the `com.huangx.chatgpt-bridge` agent (stops it AND
  prevents KeepAlive from respawning mid-wipe).
- Trash everything under `~/.chatgpt-bridge` **except `token`** (the ChatGPT
  login credential — never delete it, or Xiang must re-login). This clears
  `bridge.sqlite` (the big one, ~300 MB when bloated), `runs.log`,
  `ASK_LEDGER.md`, etc.
- Trash `/tmp/chatgpt-bridge` (result `.md` / images / push.log / connstate)
  and truncate `/tmp/chatgpt-bridge.log`.
- `launchctl bootstrap` it back → server recreates an empty `bridge.sqlite`
  (`CREATE TABLE IF NOT EXISTS`).
- Verify: process up, `:8801` listening, fresh DB, `/api/diag` responds.

Rules:
- Data goes to **Trash (reversible)**, not `rm`. The script prints how much was
  cleared; tell Xiang he can empty Trash to reclaim the disk.
- The script is self-verifying — relay its final `✅ CLEAR COMPLETE` / `✗` line.
- `:8801` is the bridge. **`:8800` is the wiki-server — never touch it.**
- After clearing, the ChatGPT browser tabs reconnect on their own (extension
  polls); remind Xiang to refresh/restart the plugin if a tab looks stale.
- `clear` wipes in-flight tasks too (for a full reset that's the point) — note
  it also clears `ASK_LEDGER.md`, so any outstanding `✗ NEEDS-PASTE` rows are
  gone after a clear.

### Trigger sources

This skill fires from **both** sides — Claude Code prompt and Telegram:

- **Claude Code prompt** (typing `/chatgpt …` in the tmux window): the
  skill executes inline; ChatGPT's answer arrives in the same window.
- **Telegram message** (`/chatgpt <question>` sent in any DM/group whose
  chat_id is in the bot allowlist): the bot's slash-command handler does
  not match `/chatgpt`, so the message falls through to the normal text
  injector — Claude Code in the matching tmux window sees
  "/chatgpt <question>" as user input and runs this skill. The bot adds
  no per-command acknowledgement, so the skill must explicitly send a
  short Telegram reply at the start (`telegram_reply` with a one-line
  "dispatching to ChatGPT…" so Xiang knows the request landed), and
  again when the answer arrives (`telegram_reply` with the answer
  quoted back, or with a summary if the answer is long).

Regardless of source, the response must always be relayed via
`telegram_reply` to the originating `chat_id` (taken from the most
recent `<channel source="telegram" chat_id="…">` tag). Do not silently
write the answer only to the Claude Code window — Xiang sees Telegram,
not the tmux pane.

## How to call ChatGPT

Use ONE script — **`ask-gpt.py`**. It blocks and prints ChatGPT's answer:

```bash
# short question:
python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> "your question"

# long / multi-line question (pipe via stdin — no quoting headaches):
python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> <<'EOQ'
your long, multi-line question…
EOQ
```

- `<channel>` names an open ChatGPT tab; each channel/tab has a **GitHub repo
  connected**, so ChatGPT can read that repo's code when answering. Ask Xiang
  which channels are live and what repo each is bound to (e.g. `ac` →
  AnalyticCombinatorics, `dm` → Bounded).
- It **blocks until the answer is ready** and prints it. Answers can take from
  seconds (fast mode) to **10-20 minutes** (Pro/Extended long-think) — that is
  NORMAL. **Never resubmit or reset a pending question** — that interrupts the
  in-progress answer and yields a truncated fragment. Just wait.
- **Parallel**: for several INDEPENDENT sub-questions, fire to different
  channels in the background so the long-thinks overlap:
  ```bash
  python3 ~/.openclaw/workspace/scripts/ask-gpt.py dm2 "subproblem A" >/tmp/a 2>&1 &
  python3 ~/.openclaw/workspace/scripts/ask-gpt.py dm3 "subproblem B" >/tmp/b 2>&1 &
  python3 ~/.openclaw/workspace/scripts/ask-gpt.py dm  "hardest one"   # foreground
  wait
  ```

## Waiting for answers — use pipe, don't wait for process exit

`ask-gpt.py` prints the answer to stdout with `flush=True` and calls
`sys.exit(0)`. In most cases a simple pipe just works:

```bash
# Single question — pipe captures the answer as soon as it's printed:
ANSWER=$(python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py <channel> "question")

# Long / multi-line question:
ANSWER=$(python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py <channel> <<'EOQ'
your long question…
EOQ
)
```

The `-u` flag disables Python's output buffering so the answer hits the pipe
immediately on `print()`. The `$()` subshell captures everything and returns
as soon as the process exits.

For **parallel** background calls, pipe each to a file and read when done:

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py dm2 "sub A" > /tmp/gpt_a.txt 2>&1 &
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py dm3 "sub B" > /tmp/gpt_b.txt 2>&1 &
wait
A=$(cat /tmp/gpt_a.txt)
B=$(cat /tmp/gpt_b.txt)
```

### MOST RELIABLE for an autonomous agent: one `run_in_background` Bash PER channel (2026-06-17)

The `nohup … &` + later-`wait`/checkpoint pattern above has a sharp failure mode for an agent that
also runs other long work: the launcher exits, the grind is **orphaned**, nothing fires when that
channel returns, and you end up **batch-harvesting at your next scheduled checkpoint** — so a 6-min
answer sits unread for 20 (Xiang 06-17: "答案到了… 好像没有收到的样子"). The harvest must happen the
**moment** the channel returns, not on your polling cadence.

The fix: launch **each channel as its own `run_in_background: true` Bash tool call** (NOT bare
`nohup &`). The command runs `ask-gpt.py` to a file and ends with a marker line. When that process
exits, the harness fires a **task-notification** → you are re-invoked and harvest THAT channel
immediately, while the others keep thinking.

```bash
# ONE run_in_background Bash per channel — each notifies you on its own return:
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py cron1 < /tmp/q1.txt > /tmp/gpt_q1.txt 2>&1
echo "GPT_DONE cron1 $(wc -c < /tmp/gpt_q1.txt)B"
```

Rules for the reliable pattern:
- **One `run_in_background` Bash per channel.** Never wrap several channels in one `wait` (blocks on
  the slowest — you harvest nothing until the whole batch finishes) and never bare `nohup … &` with no
  tracked waiter (orphans the grind — no notification, you forget it until the user nudges).
- **Harvest on return, not on a checkpoint.** When a channel's task-notification fires, READ that
  answer file now (print the BRIDGE RUN banner, TG-relay), then re-fire that freed channel. Do not
  defer reading to a `ScheduleWakeup` tick — the notification IS the harvest trigger.
- **Do not set a `timeout` on the Bash** — `ask-gpt.py` has its own 45-min deadline; an outer timeout
  kills the Pro long-think early.
- Codex/uisai nohup runs are different — those are NOT harness-tracked, so they still need a
  `ScheduleWakeup` poll. Only the **local** `ask-gpt.py` calls get the per-channel notification.

If a process hangs after printing (rare — the explicit `sys.exit(0)` should
prevent it, but sockets can stall), the output file already has the answer.
Poll file size stability as a fallback:

```bash
# Fallback: if `wait` hangs, poll instead
while [ ! -s /tmp/gpt_a.txt ]; do sleep 5; done
S1=$(wc -c < /tmp/gpt_a.txt); sleep 10; S2=$(wc -c < /tmp/gpt_a.txt)
[ "$S1" = "$S2" ] && kill $PID1 2>/dev/null  # answer stable, kill hanging process
```

**Forbidden patterns**:
- `Bash(command, timeout=300)` wrapping `ask-gpt.py` — kills the process
  before Pro long-think (10-20 min) finishes
- Any custom `timeout` flag or wrapper — the correct timeout is already
  inside `ask-gpt.py` (45 min deadline); adding another layer causes
  premature kills
- Omitting `-u` when piping — Python block-buffers piped stdout by default,
  so the answer may sit in the buffer until process exit

> There is an older `ask-chatgpt.sh` in the bridge repo. **Don't use it** for
> agent/cross-window calls — it has a tmux-window-name guard that rejects a
> mismatched `--channel` (the source of past confusion). `ask-gpt.py` is the one.

(Bridge internals — capture paths, the debug/version handshake, markers — live
in `~/repos/chatgpt-bridge-pr3/UNDERSTANDING.md`, not here. A caller doesn't need
them.)

## ASK LEDGER — persistent numbered question tracker (Xiang 2026-06-17)

The reliable per-channel harvest above fixes *orphaned* answers; it does NOT fix the bridge **DOM-capture
failure** (`verdict=FAIL` / `[BRIDGE:…]` / <500 B) — the answer is generated in the tab but never scraped, and
telling Xiang "this one failed" in chat is easy to lose (Xiang 06-17: "你这样告诉我容易丢… 做成 task 列表一样
的持久性列表… 问题有编号我很容易找回"). So keep a persistent, glance-able, **numbered** ledger.

**File:** `~/.chatgpt-bridge/ASK_LEDGER.md` (shared across windows, beside `runs.log`). One markdown-table row per
`ask-gpt.py` dispatch: `Q# | time | channel | topic | status | RUN#`. Status: `⏳ dispatched` · `✅ captured`
(>500 B, no `[BRIDGE:` marker) · `✗ NEEDS-PASTE` · `↩ recovered`. Header carries the legend + a `<!-- next Q#: N -->`
cursor.

**This is now MECHANIZED in `ask-gpt.py` — do NOT do it by hand.** As of 2026-06-21 the script itself, on every
call: assigns the global `Q<N>` (atomic locked read+bump of the cursor — fixes the duplicate-Q# collisions
hand-maintenance produced), **auto-prefixes the brief** `Q<N> (channel): …` (so the number shows in the tab), writes
the `⏳ dispatched` row, prints a Q#-led `═══ BRIDGE Q<N> →<ch> … ═══` banner to **stderr** (machine task-id demoted
to a `↳ debug:` tail — never headline a raw `task`/bg-id to Xiang), and on return updates the row to
`✅ captured`/`✗ NEEDS-PASTE` + fills `RUN#`. **Do not hand-number, do not hand-prefix the brief, do not hand-write
ledger rows** — doing so double-prefixes (`Q12 (dm1): Q11 (dm1): …`) and re-introduces collisions. Pass an optional
`ASK_LABEL="short topic"` env var if you want a better `topic` than the auto first-line summary.

**What the caller still owns:** (1) relay the script's stderr banner / the answer to Xiang; (2) on a `✗ NEEDS-PASTE`
row do NOT silently re-fire (resubmitting truncates the in-flight tab answer) — surface that `Q<N>` row to Xiang
once, he pastes from the tab; (3) on his paste mark `↩ recovered` (note where banked), then process
verify-don't-transcribe.

Persists across sessions; highest `Q#` is the next to use. The ledger is the source of truth for "what's
outstanding" — surface the `✗` rows when reporting a rolling-harvest round.

## Three answer modes

ChatGPT answers come back through one of three channels. Pick the
right one per task.

| Mode | Fidelity | Unattended? | When to use |
|------|----------|-------------|-------------|
| **bridge-capture** | Imperfect (DOM scraping) | ✅ yes | Short text answers, audits, math discussion |
| **git-drop** (NEW) | Perfect (git commit) | ✅ yes | Code/proof artifacts — Lean files, LaTeX, scripts |
| **github-write (legacy)** | Perfect | ❌ needs click | Only if git-drop isn't set up on a tab |

### git-drop — the DEFAULT mode for all ChatGPT tasks (2026-06-18)

ChatGPT's GitHub connector now asks for write permission ONCE per
session, then writes freely without per-file dialogs. This makes
git-based answer delivery reliable and unattended.

**The drop file is `.md` format.** ChatGPT writes its COMPLETE
response (explanation, reasoning, suggestions, AND code) as markdown
into the drop file. Code goes in ` ```lean ` / ` ```python ` fenced
blocks. This replaces bridge DOM capture entirely — the git commit
IS the answer, byte-perfect, no truncation.

**How it works:**

1. Each channel has a fixed drop file in `.md` format:
   `scratch/_CHATGPT_DROP_<channel>.md` on the scratch branch of the
   connected repo. The exact repo, branch, and drop-file path are
   **per-tab setup** — ask Xiang which tab has git-drop configured.

2. I send the question via `ask-gpt.py <channel>`, with the
   instruction to write the FULL response into the drop file.

3. ChatGPT writes the file and reports the commit SHA in chat.

4. I fetch and read:
   ```bash
   git fetch <remote> <scratch-branch>
   git show <remote>/<scratch-branch>:<drop-file>
   ```
   The drop file gets overwritten each round, no accumulation.

5. I extract code from fenced blocks, compile/verify locally, act.

**Prompt template for git-drop:**

Put the TASK in the question text (inline), NOT in the drop file.
The drop file is only the answer container — ChatGPT overwrites it
with its response. If you put the task in the drop file, ChatGPT
may read the placeholder instead of the task (race condition).

```
<full question with all context, references, and requirements>

IMPORTANT: Write your COMPLETE response into <drop-file> on
<scratch-branch> via the GitHub connector (UPDATE the existing file).
Use markdown format:
- Explanation, reasoning, and suggestions as prose
- Code in ```lean (or ```python etc.) fenced blocks
- Include all imports in code blocks
After committing, report the commit SHA.
```

**Why `.md` not `.lean`:** Bridge DOM capture is unreliable (truncation,
voice-button race, intro-only capture). By having ChatGPT write its
full response — prose AND code — into the drop file, we get everything
in one git commit. No need for bridge capture at all. The bridge
`ask-gpt.py` call still runs (to get the SHA from chat text), but the
authoritative answer is always in the git file.

**Extracting code from the drop file:**
```bash
# After fetching, extract Lean code blocks:
git show <remote>/<scratch-branch>:<drop-file> | \
  sed -n '/^```lean/,/^```/p' | sed '/^```/d' > /tmp/answer.lean
```

**Setup**: before first use on a channel, the drop file + scratch
branch must already exist in the connected repo. This is per-tab setup
owned by whoever configured that tab — confirm the repo/branch/drop-file
with Xiang rather than creating branches yourself. If it genuinely needs
initialising, the drop file can be created with
`gh api repos/<owner>/<repo>/contents/<drop-file> -X PUT -f branch=<scratch-branch> ...`
(or ask ChatGPT to create it on the first round) — but only in a repo
designated for scratch, **never** a sensitive/shared repo without Xiang's OK.

**Sync shortcut** (after ChatGPT reports SHA):
```bash
git fetch <remote> <scratch-branch> && git show <remote>/<scratch-branch>:<drop-file>
```

### git-drop monitoring — MANDATORY (2026-06-18 lesson)

After dispatching a git-drop task, you MUST actively monitor for
completion. The failure mode (observed 06-18): dispatch two channels
in background, check once at 0 bytes, assume "still running," miss
that they finished minutes ago. Xiang had to ask "怎么感觉没提交".

**Rule: one tracked waiter per channel, immediate harvest.**

For each `ask-gpt.py` call, use `run_in_background: true` as a
SEPARATE Bash call so you get a per-channel `task-notification`:

```bash
# CORRECT: each channel gets its own tracked background task
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py dm1 <<'EOQ' > /tmp/gpt_dm1.txt 2>&1
<question>
EOQ
```
(with `run_in_background: true` on the Bash call)

Do the same for dm2 as a separate Bash call. When `task-notification`
fires for dm1, immediately:
1. Read `/tmp/gpt_dm1.txt` for the bridge response (SHA etc.)
2. `git fetch <remote> <scratch-branch>`
3. `git show <remote>/<scratch-branch>:<drop-file>`
4. Compile locally
5. Re-dispatch dm1 with next task

**Do NOT**: batch both channels into one `wait` (blocks on slowest),
fire-and-forget with bare `&` (no notification), or poll only once
and give up.

### bridge-capture (unchanged)

The original mode. `ask-gpt.py` blocks and prints whatever the
bridge captures from the DOM. Works for text answers. Subject to
known bugs (intro-only capture, voice-button race, truncation on
long answers). See the bridge internals in
`~/repos/chatgpt-bridge-pr3/UNDERSTANDING.md`.

### github-write (legacy, deprecated)

The old mode where ChatGPT creates NEW files via GitHub connector.
Each file creation popped a confirmation dialog requiring Xiang at
the keyboard. Superseded by git-drop which reuses one file and
only needs one-time authorization. Keep for reference only.

## Asking style — the rules that matter

These come from the actual BAC paper round trips (R1–R44):

1. **Tactic-level questions beat completion requests.** "Why does step 5
   require the inequality to be strict?" produces sharper answers than
   "rewrite step 5". When asking ChatGPT to audit code or proofs,
   formulate as "check whether claim X follows from setup Y".

2. **Do not over-organise Xiang's critiques before dispatching.** When
   Xiang raises a concern, send it close to verbatim (echo back through
   ASR if voice). The ChatGPT collaboration is most useful when ChatGPT
   sees the raw critique and can disagree with the framing. Over-organised
   prompts pre-commit you to an interpretation.

3. **Use round numbering (R1, R2, …) for multi-turn audits.** Each round
   names what specifically changed since the last round and asks ChatGPT
   to re-verify against the committed state (cite commit SHA + file
   paths). This is how the bridge captures and our memory retain
   coherent reasoning across rounds.

4. **Cite the actual file paths and commit SHA.** ChatGPT can read the
   repo state through the conversation context if pointed at specific
   files. Never use phrases like "the recent version" without a SHA.

5. **One main question per round.** Bundling 3 different issues into one
   message diffuses the answer. Split into separate rounds if the topics
   are not tightly coupled.

6. **No internal codenames in the outgoing prompt.** Replace BAC / DNA32
   etc. with descriptive phrases on first mention (e.g.,
   "bounded analog complexity construction"), then use the short form
   afterwards. ChatGPT's responses come back clean.

7. **Patience — never resubmit a long-thinking query.** Extended/Pro mode
   shows `processing`/len 0 for 10-20+ minutes with no incremental output —
   that is NORMAL, not stuck. Resubmitting or reset-chat mid-flight is
   DESTRUCTIVE: it interrupts the in-progress answer and yields a truncated
   fragment. Just wait (ask-gpt.py blocks to 45 min). When unsure stuck vs
   thinking, ask Xiang (he can see the tab) — don't reset.

8. **Verify, don't transcribe.** ChatGPT is a smart auditor, not an oracle.
   Read its answer critically: agree where the reasoning holds, push back
   where it overreaches (it sometimes asks for false rigor, or hand-waves a
   key step). Land the math in your own domain (e.g. re-derive / check in
   Lean) before acting; the writing/proof layer is ours to drive (see
   `feedback_chatgpt_collab`).

9. **Parallelize independent sub-questions across channels.** When several
   independent hard steps are blocked at once, dispatch them to different
   channels in the background (each long-think is ~20 min) rather than
   serializing. Don't split a single coherent question.

## If an answer doesn't come back

All three answer modes (fast / extended / Pro) auto-deliver — you just wait on
`ask-gpt.py`. But Pro long-thinks 10-20 min: a pending question is NOT stuck.
- **Do not resubmit or reset** a pending question — it interrupts the in-progress
  answer and produces a truncated fragment.
- If `ask-gpt.py` returns `[BRIDGE: …]` / an empty or obviously-truncated answer,
  the task likely parked — **tell Xiang directly**, don't silently retry. He can
  glance at the tab (or paste the answer). Don't act on a truncated answer.
- If the output file has content but the process is still running, **read the
  file anyway** — the answer is there. The process hang is a known bug (see
  "Waiting for answers" above). Don't wait for exit; poll file size stability.
- **0 B that never fills — FIRST suspect a wrong/non-existent channel name (2026-06-17).**
  If the output file is **stuck at 0 B** while `ask-gpt.py` keeps running, the #1 cause is
  **the channel name is not a real open tab** — `ask-gpt.py <typo>` blocks forever on a tab
  that does not exist, captures nothing, and eventually logs `verdict=FAIL:None`. So before
  anything else: **confirm the channel is in the live list** (ask Xiang which tabs are open,
  or check recent `runs.log` `ch=` values for a channel that has succeeded before). The
  2026-06-17 incident: a faithfulness audit fired to `cron1` sat 0 B for 25 min — there is no
  `cron1`, only `cron` and `cron2`. The fix was not "wait for the DOM" but **re-fire to the
  real channel** (`cron`). Don't ask Xiang to paste a phantom-channel answer — there isn't one.
  - **Only after the channel is confirmed real**: a real channel sitting 0 B + process alive +
    aged > ~25 min (past Pro's max long-think), with no `runs.log` line, *can* be a genuine
    DOM-capture hang — then the answer IS in the tab; tell Xiang to paste (don't resubmit).
  - Detection / triage one-liner:
    ```bash
    # is the channel even real? (has it ever succeeded?)
    grep -q "ch=<channel> .*verdict=OK" ~/.chatgpt-bridge/runs.log || echo "channel <channel> never succeeded — likely WRONG NAME, re-fire to a known-good channel"
    S1=$(wc -c </tmp/gpt_X.txt); sleep 8; S2=$(wc -c </tmp/gpt_X.txt)   # 0,0 ⇒ nothing captured
    ```
  Root prevention: **never invent a channel name.** The set of channels is fixed and small
  (e.g. `cron`, `cron2`, `ac`, `ac2`, …) — if unsure, ask Xiang once; do not pattern-guess a
  `cron1` because `cron`/`cron2` exist.

## After the answer arrives — run banner (now auto-emitted)

As of 2026-06-21 **`ask-gpt.py` prints the banner itself, to stderr**, on every
call — a `SUBMITTED` banner at dispatch and a result banner (`✅ COMPLETE` /
`✗ TRUNCATED` / `✗ FAIL` / `✗ PENDING`) on return, both Q#-led, with the machine
task-id demoted to a `↳ debug:` tail. It still also appends the machine line to
`~/.chatgpt-bridge/runs.log`. The caller's job is no longer to *reconstruct* the
banner from `runs.log` — it is to **relay** the stderr banner (and the answer) to
Xiang. Do not headline a raw `task <id>` or a harness background-task id (e.g.
`bdk47lczp`) to Xiang — those are machine tokens; lead with the `Q<N>` banner.
The legacy hand-built format (kept for reference; the script's output supersedes it):

```text
═══ BRIDGE RUN #N ═══  <channel> | task <id8> | <bytes> B | ✅ COMPLETE
═══ BRIDGE RUN #N ═══  <channel> | task <id8> | <bytes> B | ✗ TRUNCATED —需要手贴
═══ BRIDGE RUN #N ═══  <channel> | task <id8> | — | ✗ FAIL (<status>)
```

Rules: read the verdict from `runs.log` (or compute it: <500 B ⇒ truncated);
print the banner in the Claude Code window AND lead the Telegram report with
the same line; if a truncated run is later completed by a manual paste,
append a `RUN#N-manual ... verdict=OK` line to `runs.log` so the ledger
closes out. Other windows follow the same rule — the log is shared.

## After the answer arrives

1. The answer is whatever `ask-gpt.py` printed to stdout.
2. **Always send a Telegram reply** to the originating `chat_id`,
   quoting the key claims. Do not silently fold ChatGPT's reasoning into
   your own — credit the source and let Xiang decide whether to accept
   the framing. If the answer is long, paste it inline (or as an
   attached `.md` file if it exceeds Telegram's message length limit).
3. If ChatGPT makes a strong claim about the repo (a file, a function,
   a citation), **verify it against the actual file** before acting on
   it. The "Verify Before Claim" rule applies here.
4. If iterating: structure the next round as R<n+1>, name what changed,
   commit the change first, then ask ChatGPT to verify against the
   committed state.

## Telegram round-trip — start-to-finish template

```text
Xiang (TG): /chatgpt R5: check whether step 7 still holds at commit abc1234

Skill:
  (a) Send TG ack: "→ R5 已 dispatch (commit abc1234)..."
  (b) python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> "R5: …"
      (blocks until the answer is ready — may be 10-20 min on Pro; don't poll)
  (c) When it returns the answer text:
        If short: telegram_reply with the answer body.
        If long: telegram_reply with a 3-bullet summary + the full text as a
        .md attachment via the `files` parameter.
  (d) If it returns "[BRIDGE: …]" / a truncated fragment: do NOT retry silently.
        Tell Xiang and ask him to glance at the tab / paste the answer.
```

## Examples

```text
Xiang: 问 ChatGPT 这个 lemma 的证明步骤 5 严密吗?

→ /chatgpt: "R3: please check whether step 5 in
   tm-completeness/paper-draft/s01-bac-decision.tex (commit c0c445d,
   around line 410) still holds when h^f has support on two adjacent
   cells. The previous round (R2) flagged a concern about the
   convex-relaxation argument."
```

```text
Xiang: 让 ChatGPT 给个 review score, 我们准备 submit 了.

→ ask-gpt.py <channel> "R44: final readiness assessment. Paper at commit
   f380762 incorporates all R28-R43 audit findings. Please rate readiness
   (X/10) and flag any remaining theorem-blocking concern."
```

## Do NOT

- Do not paraphrase Xiang's critique before sending — verbatim or near-verbatim only.
- Do not silently keep retrying when the bridge intro-only bug triggers — ask Xiang to paste.
- Do not commit to ChatGPT's claims about the codebase without verifying the actual files.
- Do not give other windows/agents `ask-chatgpt.sh --channel X` when their tmux
  window name ≠ X — the channel guard rejects it (exit 1). Hand them
  `ask-gpt.py <channel>` (no guard) instead.
- Do not resubmit / reset-chat a query that is still thinking (10-20 min at
  `processing` is normal for Pro/extended). Resubmitting truncates the answer.
- Do not mass-reload tabs simultaneously (the real rate-limit trigger); tab
  count itself is fine.

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

### [2026-06-23] git-drop harvest: verify the drop file is FRESH (matches your Q#), not just non-empty
When an answer arrives via a shared "drop" file (the channel writes its response into a fixed file you then
fetch), a background-task "completed" + a NON-EMPTY drop file does NOT mean YOUR question's answer landed.
If the write silently failed or is still pending, the file still holds the PREVIOUS round's answer — which
reads like a successful capture and gets acted on as the new answer. Always verify the drop file's
question-identifier (the Q#/topic header the channel echoes back) matches the question you dispatched THIS
round, before trusting or integrating it.
**Why:** A dispatched question returned "completed" and its drop file had full content — but it was a
different, earlier question's answer (the new write never landed); only the question-id header in the file
exposed the staleness. Acting on it would have integrated an off-topic answer.
**How to apply:** On every shared-file / git-drop harvest, grep the fetched file's header for your dispatched
question-id (and/or expected topic); if it doesn't match, treat as NOT-landed — surface it, don't act, and
fall back to your own work. Pairs with the git-drop-monitoring + ASK-LEDGER discipline and "verify, don't
transcribe."
