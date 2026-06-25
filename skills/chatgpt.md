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
  connected**, so ChatGPT can read that repo's code when answering.

### Channel 命名规则与窗口隔离（严格执行）

**Channel 名以你所在的 tmux 窗口名为前缀。不要用串。**

每次调用 `ask-gpt.py` 前，必须确认两件事：

1. **确认自己的窗口名**：`tmux display-message -p -t "$TMUX_PANE" '#W'`
2. **只用自己窗口前缀的 channel**：
   - dm 窗口 → 用 `dm`, `dm1`, `dm2`, `dm3`, `dm4`
   - research 窗口 → 用 `research`, `research1`, `research2`
   - family 窗口 → 用 `family1`, `family2`, `family3`
   - cron 窗口 → 用 `cron1`, `cron2`
   - pbook 窗口 → 用 `pbook1`, `pbook2`
   - chan-work 窗口 → 用 `chan1`, `chan2`

3. **查看当前可用 channel**：`curl -s http://localhost:8801/api/channels | python3 -m json.tool`

**禁止跨窗口用 channel**——dm 窗口不能用 cron1，pbook 窗口不能用 dm2。
LLM 跑久了会忘记自己在哪个窗口、该用哪个 channel，开始串台。
**每次调用前检查，不要靠记忆。**
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

## Answer modes — git-drop is the DEFAULT and ALWAYS first priority (2026-06-24)

**git-drop is not an option you choose — it is the default for EVERY call, and it
is automatic.** As of 2026-06-24 `ask-gpt.py` auto-appends the git-drop instruction
to every question (per `scripts/gitdrop-targets.json`) and detects completion from
the COMMIT. You do **not** decide per-task whether to use git-drop, you do **not**
hand-write the git-drop instruction, and you do **not** fall back to bridge-capture
because it seems simpler. git-drop first, always — because DOM scraping is
fundamentally unreliable (truncation, stuck "working", baseline-gating, SSR junk)
and the git commit is byte-perfect and gh-verifiable.

| Mode | Fidelity | Role |
|------|----------|------|
| **git-drop** | Perfect (git commit) | **DEFAULT — every call, automatic. The answer IS the commit.** |
| **bridge-capture** | Imperfect (DOM scraping) | **Fallback only** — when the connector can't write (read-only/unconfigured channel). Never the primary path. |
| **github-write (legacy)** | Perfect | Deprecated; ignore. |

- Configured channels (dm/chan/family/pbook per `gitdrop-targets.json`) → git-drop
  is automatic; the answer lands as a commit and `ask-gpt.py` prints
  `GIT-DROP OK [VERIFIED] …`.
- The ONLY time you skip git-drop is `ASK_NO_GITDROP=1` for a genuinely
  throwaway discussion question where no artifact is wanted — and even then the
  answer comes via the (fragile) DOM path, so prefer git-drop unless there's a
  reason not to.
- If a channel isn't in `gitdrop-targets.json` yet and you're doing real work on
  it, ask Xiang to add it rather than settling for bridge-capture.

### GPT job mode — the DEFAULT: ChatGPT reads the repo, you do NOT paste source (2026-06-23)

Every ChatGPT tab/channel runs as a **GPT job**: a GitHub repo is connected to it,
and inside the job ChatGPT both **READs** that repo's source and **WRITEs** its answer
back as a git commit (git-drop). Read + write together is the full job loop, fully
unattended. **This is the default working mode** — use it instead of pasting source
into the prompt.

**The connector reads the repo's DEFAULT branch — there is NO branch picker.** ChatGPT's
GitHub connector reads whatever GitHub has configured as the repo's *default* branch
(usually `main`). You cannot say "read branch X" in the prompt and have the connector
honor it. Consequence: **for ChatGPT to read your working source, that source must be on
the connected repo's default branch.** Files that live only on a non-default working
branch are INVISIBLE to the job (ChatGPT 404s on those paths).

**Read-side workflow (do this BEFORE dispatching a code/proof task):**
1. Ensure the files ChatGPT must read are on the connected repo's **default branch**.
   Push them there — a normal `git push`, or per-file
   `gh api repos/<owner>/<repo>/contents/<path> -X PUT -f message=... -f content=<base64> -f branch=<default-branch> [-f sha=<current>]`.
   Keep the pushed copy CURRENT: if you edit a file locally, re-push before asking ChatGPT
   to read the new version, or it reads a stale copy.
2. In the prompt, reference files by **path + line range** (e.g. "read `scratch/Foo.lean`
   L100-200 on the connected repo"), NOT by pasting their contents. Pasting large source
   into the prompt is the anti-pattern to avoid (Xiang 2026-06-23: "不要这样来发请求") —
   GPT-job mode exists precisely so you don't have to.
3. ChatGPT reads the source itself, reasons, and writes its answer via git-drop (below).

**404-on-a-path-that-exists ⇒ wrong branch, not missing file.** If ChatGPT reports it
can't find a file you know is in the repo, the file is almost certainly on a NON-default
branch (the job only sees the default). FIX: push that file to the default branch. Do NOT
fall back to pasting source — that just hides the setup gap.

**Stay generic — never hardcode a channel.** Each `<channel>` maps to its own connected
repo + default branch + drop file; different sessions own different channels (research
uses its set, cron/family/ac/dm each use theirs). Confirm a channel's
repo / default-branch / drop-path ONCE (ask Xiang, or read it off a prior successful
round), then drive that channel by path-reference. Write prompts and tooling so they take
the channel/repo/branch as parameters — do not bake in any one session's channel names.

### git-drop — the write half of GPT job mode (2026-06-18)

ChatGPT's GitHub connector asks for write permission ONCE per
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

### git-drop completion = the COMMIT, fully decoupled from DOM (2026-06-24)

**The completion signal in git-drop mode is the git commit landing — NOT DOM
capture, NOT `/api/wait` status, NOT the tab's purple/working color.** Those are
all DOM-scraping-derived and unreliable; a tab can sit "working" purple for 40+
min after the answer was already committed, and the old `/api/wait` (driven by
DOM finalize) made the caller 空等 long after the commit landed (Xiang 06-24:
"答案明明出来了，但是 llm 还是在空等").

As of 2026-06-24 this is mechanized in `ask-gpt.py`: git-drop is the DEFAULT
(auto-appended per `scripts/gitdrop-targets.json`), and the script **polls the
target drop-file commit** and returns the instant a NEW commit appears,
independent of DOM. On success it prints ONE line:

```
GIT-DROP OK [VERIFIED] <sha> <owner>/<repo>@<branch>:<file> — answer is the commit, fetch from repo.
```

**Caller rules:**
- When you see `GIT-DROP OK [VERIFIED] …`, the task is **DONE**. Fetch the answer
  from that commit (`gh api repos/<repo>/contents/<file>?ref=<branch>` or
  `git show <remote>/<branch>:<file>`). Do **NOT** keep waiting, re-poll, re-fire,
  or look at the tab color — the commit IS the answer.
- Never infer "still running" from a purple/working tab or a 0-byte DOM capture in
  git-drop mode. Those are decoupled now; ignore them.
- `[reported]` (not `[VERIFIED]`) means the SHA came from the chat reply but gh
  couldn't confirm it — treat as suspect (possible hallucinated commit), verify
  the commit yourself before trusting.
- `ASK_NO_GITDROP=1` for a pure-discussion question where no commit is wanted
  (then completion falls back to the normal DOM/`/api/wait` path).

### git-drop monitoring — one tracked waiter per channel (2026-06-18)

Even with commit-polling, dispatch each channel as its own `run_in_background`
Bash so each notifies on its OWN return — don't batch into one `wait` (blocks on
the slowest) or fire-and-forget with bare `&` (orphans it; observed 06-18:
dispatch two, check once at 0 bytes, miss that they finished — "怎么感觉没提交").

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

### [2026-06-23] Recover a deadline-exceeded long answer from the bridge task store — don't treat it as lost
When a Pro/Extended long-think exceeds the bridge client's hard deadline (~45 min), the blocking call returns
PENDING / an empty-or-truncated result — but the FULL answer keeps streaming into the bridge's own persistent
task store. Extract it from there (the task DB, keyed by the task id shown in the run banner) instead of
declaring the answer lost or asking for a manual paste.
**Why:** The client's stdout is only a view; the authoritative answer accumulates in the bridge store even
after the client process exits on its timeout. A 150KB+ answer is fully recoverable from the store when stdout
shows nothing.
**How to apply:** On a PENDING/truncated return, query the bridge task store for that task id. Large/growing
data + status processing|completed = the answer is present or still streaming (poll size to stability, then
extract). Tiny data (question-only) + status pending = a genuinely stuck tab (surface to the user). This
diagnostic separates "long answer in flight" from "dead tab" WITHOUT resubmitting — resubmitting would truncate
a live one.

### [2026-06-23] Smoke-test the delivery channel before fanning out a batch
Before committing a multi-question campaign to an external collaborator's delivery
mechanism (a file drop, a scraped capture, a callback), verify the FIRST answer
actually lands end-to-end — a non-empty, retrievable result. A dispatched task that
reports "completed" but yields an empty / zero-byte result is a BROKEN delivery
channel, not a slow answer: stop dispatching into it, surface the breakage once, and
switch to driving the work yourself. Firing more questions into a dead channel only
multiplies wasted dispatches (the already-generated answers may still be
recoverable from the tool's own task store — but stop fanning out first).
**Why:** A whole batch of dispatched questions silently failed to deliver (every task
"completed" but with empty captures and unwritten drop files); continuing to fire wasted
the channels, while the actual breakthrough came from own parallel investigation.
**How to apply:** On the first dispatch of a batch, confirm the answer is retrievable
before sending the rest. On a completed-but-empty result, treat the channel as down and
go solo — do not re-dispatch into it.

### [2026-06-23] After a bridge reset, runs-log history lies — confirm channel liveness from the live registry, and re-fire stale dispatches
A full bridge clear/restart WIPES the run history and recreates an empty task DB. Two consequences bite if you
forget it happened: (1) the "this channel never succeeded in the run log ⇒ wrong channel name" heuristic gives
FALSE positives — the history was erased, not the channel missing; confirm a channel is real/live from the
bridge's LIVE registry (`/api/diag` registered-tabs), not from run-log history. (2) Any question dispatched
just before/around the reset is DEAD — its task id no longer exists in the fresh DB, so the blocking call sits
PENDING at 0 B forever. Kill those in-flight dispatches and re-fire fresh; since they never produced an answer,
re-firing truncates nothing.
**Why:** Post-reset, several dispatches sat 0 B / PENDING and the run log showed every channel "never OK",
which read as wrong channel names — but the live registry showed all the tabs present and connected; the real
cause was the dispatches had hit the pre-reset bridge. Kill + re-fire to the same names landed them.
**How to apply:** If you (or the user) just cleared/restarted the bridge, distrust run-log-based channel
diagnostics; check the live `/api/diag` registry for the tab, and kill+re-fire any dispatch that predates the
reset. Pairs with the phantom-channel ("0 B never fills") and smoke-test entries.

### [2026-06-23] A returned self-verifying script only proves consistency with the collaborator's OWN transcription
When a collaborator (ChatGPT, a subagent) returns a self-contained script that asserts its own correctness
(`assert … == 0; print OK`), that OK proves the result is consistent with THEIR transcription of your problem —
NOT that it matches YOUR actual definitions. A silent mis-transcription of the prompt (a flipped sign, a
dropped term, a wrong index) passes its own assert cleanly. Before trusting/integrating the artifact, re-run
the check against your OWN independent re-encoding of the inputs (your definitions, not theirs).
**Why:** A returned cofactor/identity script printed OK, but the OK only certified it against the script's
own restatement of the problem; re-checking the same output against an independently-coded version of the
definitions is what actually confirmed (or would have caught a mismatch in) the result.
**How to apply:** Any machine-checkable artifact from a collaborator — extract just the result, drop their
scaffolding, and verify it against inputs YOU defined. This sharpens "verify, don't transcribe": a passing
self-test is the collaborator grading its own paper.

### [2026-06-23] Pre-verify any factual premise / mechanism HINT you embed in a prompt — a false premise wastes the long-think
When your question embeds a strategy hint, a claimed mechanism, or a factual premise ("use the X-pairing",
"since A = B, reindex by ..."), numerically/independently verify that premise BEFORE dispatching. A false
embedded premise is not harmlessly ignored — the collaborator builds a confident answer ON it (or burns a
6–20 min long-think chasing it), and you then must detect the error and re-dispatch a correction, wasting both
rounds. The bug is in YOUR prompt, not their reasoning.
**Why:** A dispatched proof-strategy hint turned out numerically false (a pairing identity failed at a concrete
test point); the channel ground on the false premise and the fix cost a full extra round — the error originated
in the prompt I sent, not the answer I got back.
**How to apply:** Before sending any prompt that asserts a mechanism/premise the collaborator should use, run a
quick numerical or independent check on that assertion. "Verify, don't transcribe" applies to your OUTGOING
hints too — don't poison the prompt with an unchecked claim. Pairs with the self-verifying-script entry: verify
BOTH directions of the exchange.

### [2026-06-23] For one hard problem you doubt, split TACTICAL detail vs STRATEGIC route across two channels
With two collaborator channels and ONE hard problem where you doubt your own approach, don't send two pieces of the same derivation (that splits a coherent question). Dedicate one channel to the TACTICAL question (does this specific lemma/bound/step close) and the other to the STRATEGIC question (is this the right approach at all — is the object I'm building even necessary, what is the canonical technique, is there a slicker route). The strategic answer can MOOT the tactical one (revealing the whole sub-structure is an artifact of your design), which a tactical-only consult never surfaces.
**Why:** Driving a hard formalization crux, a tactical consult ("does the fixed-supersolution margin close for arbitrary data") was paired with a strategic one ("is the coordinatewise envelope even needed, or does the weaker norm bound suffice"); the strategic angle could reveal the entire tactical struggle is a self-inflicted design artifact, saving the tactical work entirely.
**How to apply:** Two channels + one doubted approach ⇒ one tactical (HOW), one strategic (WHETHER / what's-canonical). Especially fire the strategic channel when your own framing has already been wrong — it audits the framing itself, not just the step. Distinct from parallelizing genuinely-independent sub-questions.

### [2026-06-23] A wedged refinement round is not a dependency — recover from the PRIOR round + your own work
In multi-round collaboration, if a LATER round (a refinement / re-ask) wedges past its deadline — the task store shows the PROMPT only (no answer text, regardless of byte size) plus a requeue / "junk" event — do not resubmit (re-wedges / truncates) and do not block on it. Check whether an EARLIER round of the same thread already delivered the substantive answer/design, and whether your own parallel verification covers the corrections the re-ask was targeting; if so, proceed on those and abandon the wedged round. The re-ask was an upgrade, not a prerequisite.
**Why:** A re-dispatch wedged with no recoverable answer, but the prior round had already given the complete design and own analysis covered the very correction the re-ask targeted — so blocking on or resubmitting the wedge would have cost time for a result already in hand.
**How to apply:** A re-ask round hangs/wedges (question-only store + requeue event) → mine the prior round's answer + your own work; only re-dispatch fresh-and-short, or surface to the user, if nothing already covers the need. This is the no-answer-to-recover branch that complements the "recover a still-streaming deadline-exceeded answer" lesson.

### [2026-06-23] Verify a DESIGN/architecture answer against an independent STRUCTURAL signal in the code — not the collaborator's prose
"Verify, don't transcribe" is easy for a math identity (re-run a numerical check), but a DESIGN
recommendation (gate this predicate to a sub-band, restructure that interface, split here) has no number
to check. The verification is structural: find an INDEPENDENT place in the codebase that the proposed
design implies a constraint on, and confirm the code already behaves that way. If a separate, pre-existing
component already makes the exact distinction the design relies on, that independent corroboration is far
stronger evidence the design is faithful than the collaborator's argument for it.
**Why:** A collaborator proposed gating a uniform obligation to a sub-band of cases; the decisive
confirmation was not its explanation but discovering that a downstream producer — written earlier and
independently — ALREADY case-split on exactly that sub-band and ignored the obligation outside it. That
independent structural witness confirmed the gate matches the real dependency, which no amount of prose could.
**How to apply:** When a collaborator returns an architecture/design change you cannot numerically check,
before adopting it locate an independent component that the design constrains and confirm the code already
satisfies it. Absence of such corroboration — or a component that contradicts the design — is a red flag to
push back, not adopt. Extends "verify, don't transcribe" to the design/architecture case.

### [2026-06-23] A GLOBAL transport repair does not restore ALL channels — delivery is per-channel; verify each, route to a confirmed-working sibling
After a global fix to a multi-channel collaborator transport (bridge restart, connector re-auth), do NOT assume
every channel recovered. Delivery health is PER-CHANNEL (per-tab config/state): one channel can commit answers
cleanly while a sibling still silently drops every answer for the same kind of task. When a channel stays dead
post-fix, stop firing it and RE-ROUTE the same self-contained question to a channel you have SEEN deliver this
session — don't wait on the dead one or declare the work blocked (a self-contained task doesn't care which tab
answers).
**Why:** After the transport was globally repaired, one channel resumed committing answers reliably while a
sibling produced nothing (empty drops, no commits) for the same task class; re-routing to the working channel
landed it immediately, whereas re-firing the dead channel kept producing nothing.
**How to apply:** Treat each channel's delivery as independently verifiable — confirm liveness per-channel (did
THIS channel just produce a real committed artifact?), not "the transport is up." Surface the dead channel once;
route dispatches to a proven-live sibling. Pairs with the smoke-test and post-reset-diagnostics entries.

### [2026-06-23] When the user explicitly assigns a channel but git-drop isn't configured, set it up yourself (repo-owner gh CLI), don't block
When the user explicitly designates a collaborator channel to use for a PROJECT repo, but its git-drop delivery (scratch branch + drop file in the connected repo) doesn't exist yet, create it proactively with the repo-owner-authed gh CLI — branch from main (`gh api .../git/refs -X POST -f ref=refs/heads/<scratch> -f sha=<main-sha>`), drop file via `gh api .../contents/<path> -X PUT -f branch=<scratch> -f content=<base64>` — then dispatch. Don't block or ask the user to configure it.
**Why:** The user named the channels to use for the work repo, but the git-drop scratch branch + drop file didn't exist; setting them up via gh (authed as the repo owner) took two API calls and the dispatch landed byte-perfect. Asking the user to set it up would have stalled an explicitly-greenlit task.
**How to apply:** On a git-drop dispatch to a channel the user has EXPLICITLY assigned, if the drop file/branch is missing AND the repo is the designated project repo with owner auth, create the scratch branch + drop file yourself, then fire. The skill's "confirm with the user first" caution is for ambiguous/sensitive/shared repos — an explicit channel assignment on the work repo IS the confirmation.

### [2026-06-24] For a big problem with several routes, fan out independent route-ASSESSMENTS and act on CONVERGENCE
Before grinding a big problem that has multiple plausible attack routes, dispatch the SAME problem to 2+ channels asking each to ASSESS and rank the routes (with the obstruction for each), NOT to solve it — one route emphasis per channel, or open-ended. When independent channels CONVERGE on the same verdict (route A tractable, route B high-risk for reason X), that agreement is a cheap, strong signal to commit to route A before sinking effort. Pair each assessment with a concrete sub-question (the governing invariant/formula, the specific inductive-step obstruction) so the verdict arrives with evidence, not just opinion; then verify the chosen route yourself on the first concrete instance before the full grind.
**Why:** Two independent channels asked to assess one proof's attack routes converged — both flagged the recurrence-induction route as high-risk (a derivative blowup; induction hypotheses failing in a degenerate sub-case) and both recommended the resultant route; one also supplied the exact invariant formula, which a quick local CAS check then confirmed on the first case. The convergence de-risked the route choice for the price of two parallel long-thinks.
**How to apply:** Big problem + several plausible routes + expensive to grind the wrong one → fan out route-ASSESSMENT (rank + obstruction + evidence) to independent channels; commit on convergence, dig deeper on divergence; self-verify the winner on one concrete case first. Distinct from splitting one question's tactical/strategic halves — here each channel judges the whole route space.

### [2026-06-24] Harvest git-drops where the connector/gh CLI lives — empty output on a remote worker is a missing tool, not a missing answer
A git-drop answer is fetched with the gh CLI (or git) against the connected repo, authenticated only on the bridge host. Run the harvest on a different machine (a remote build/compute host) and it can silently return EMPTY because the CLI is absent or unauthenticated — indistinguishable from "the collaborator wrote nothing." Before concluding a drop is empty/failed, confirm the harvest tool exists and is authenticated in THAT environment; otherwise fetch on the bridge host and ship the file to the worker.
**Why:** Every harvest attempt on a remote build host returned 0 bytes; the answer had committed fine — the host simply had no gh CLI, so an environment gap was mistaken for a missing answer (and nearly for a wedged round).
**How to apply:** When a git-drop harvest comes back empty, first check `which gh` / auth on the current host before re-dispatching or declaring the round failed. Pin harvests to the bridge host.

### [2026-06-24] The git-drop answer can land (and be harvestable) BEFORE the dispatch process notifies — poll the drop branch, don't wait only on the process signal
In git-drop mode the answer is a git COMMIT the collaborator pushes; that commit lands independently of — and often BEFORE — the dispatch process (`ask-gpt.py`) exits and fires its completion task-notification. Treating the process notification as the SOLE harvest trigger leaves a ready answer sitting unread while the process still shows "running." Once a question is in flight, proactively fetch the drop branch and check whether YOUR round's answer is already committed, rather than passively blocking on the process signal.
**Why:** A dispatched question's full answer was already committed to the drop branch and readable via `git show` while its background dispatch task still showed "running / empty output"; waiting on the process notification alone would have delayed a harvestable answer, and the user flagged the passivity ("你自己不好好监控的吗").
**How to apply:** After dispatching a git-drop question, poll the drop branch (`git fetch <remote> <scratch-branch>; git show …:<drop-file>`) and harvest the moment your round's answer is committed (verify the Q#/header per the freshness rule) — the process-exit notification is a BACKSTOP, not the only trigger. Generalizes: when a collaborator's deliverable is an artifact decoupled from the dispatch process, watch the ARTIFACT, not just the process. Pairs with the freshness-check and per-channel-monitoring entries.

### [2026-06-23] Audit-LLM that keeps "confirmed true + isolated, but I can't produce it" = TOOL MISMATCH — stop re-dispatching the grind
A strong-reasoning chat collaborator (audit-class) excels at confirming a claim, refuting a bad route, and
ISOLATING the irreducible residual — but it does NOT produce large mechanical proofs / derivations. If two-plus
rounds on the same target all come back "verified true + here's the exact remaining lemma, but I can't fill it,"
that is not a cue to re-brief it sharper — it is a tool-mismatch signal. The channel's value (the isolation) is
already extracted; further proof-dispatches are wasted long-thinks. Route the mechanical grind to a proof-grinder
(or grind it yourself); keep the audit channel for the next isolate/verify step.
**Why:** The same hard mechanical lemma was re-dispatched many rounds; every round re-confirmed truth and
re-isolated the residual but produced no proof. It was finally closed by self-grind, not another audit round —
the repeated dispatches after round ~2 added nothing.
**How to apply:** After ~2 rounds that confirm-but-can't-produce on a MECHANICAL target, stop re-briefing it as
a proof task. Harvest the reduction it DID give (often a clean sub-lemma + the surrounding assembly), then grind
the core yourself or hand it to a grinder. Pairs with "verify, don't transcribe" and matching the dispatch to
the collaborator's strength (audit vs grind).

### [2026-06-24] A faithful PORT of existing code is usually faster SOLO than the long-think latency — reserve dispatch for no-template hard sub-problems
The collaborator's strong-reasoning long-think runs 10-20 min. That latency is worth paying for a GENUINELY hard sub-problem (no existing template, an "is this even true" doubt, a route you can't see). It is NOT worth paying for a task that is a faithful PORT/mirror of code that already exists in the repo (adapt lemma X for object Y; specialize the proven A to case B) — there your solo time is typically SHORTER than the round-trip, so dispatching wastes both the channel and the wait. Calibrate WHAT to dispatch by solo-difficulty: low-difficulty-because-a-template-exists ⇒ lead solo; high-difficulty-no-template ⇒ dispatch (and still race solo as primary).
**Why:** Dispatched a self-contained task that was a near-mechanical port of an already-proven lemma, then completed it AND the surrounding supply chain solo while the long-think was still processing — the dispatch added nothing but a redundant in-flight job. The signal was visible up front: a complete template existed, so the task was "transcribe-and-adapt," not "discover."
**How to apply:** Before dispatching, ask "is there already a proven template I'm just porting?" If yes, solo it (the long-think latency exceeds your port time) and keep the channel free for a harder sub-problem. Dispatch when the sub-problem is genuinely open (no template, real doubt, wide solution space). Complements race-not-dependency (how to treat an in-flight dispatch) with a selection rule (whether to dispatch at all).

### [2026-06-24] On a deep analytic frontier you intend to FORMALIZE, ask for the ROADMAP, not the proof
A strong reasoner asked to PROVE a deep analytic frontier (a PDE regularity bootstrap, a hard estimate) tends to stub the load-bearing step. The far more productive ask — feeding it the EXISTING banked lemmas + the precise definitions/framework — is the FORMALIZATION DECOMPOSITION: the explicit constant, the minimal machine-checkable hypotheses, and the lemma sequence in dependency order. This converts a "deep open frontier" into a NAMED BRICK ROADMAP, each brick small enough to dispatch + machine-verify. The collaborator becomes route-architect; you + the proof-grinder build and check each brick.
**Why:** A multi-round dialogue on a deep regularity bootstrap, structured this way (PDE + the banked lemmas + "what's the exact bound / the minimal hypotheses / the lemma decomposition"), produced a complete N-lemma roadmap with an explicit closed-form constant — and the foundational bricks then built + verified first try. Asking it to "prove the bootstrap" would have returned a sketch or a stub.
**How to apply:** When ChatGPT's role is a deep math frontier you intend to formalize: feed the exact definitions + the lemmas you've ALREADY banked + the framework, and ask for (a) the explicit constants, (b) the minimal machine-checkable hypotheses, (c) the lemma decomposition in dependency order — NOT the proof. Then build the named bricks and verify each. Pairs with the body rule "tactic-level questions beat completion requests" and the split-tactical-vs-strategic entry.

### [2026-06-24] Math/design consult with unpushable code: inline the MINIMAL definitions — the "don't paste source, push instead" rule is for code-READ tasks
The "GPT-job-mode reads the default branch; don't paste source, push it" discipline targets tasks where the collaborator must READ your repo. For a MATH/DESIGN consult (is X true / what's the right invariant / which route) where the answer needs REASONING not code-reading, AND your working code is unpushed and you can't/shouldn't push (push timing is the owner's call), the right move is a SELF-CONTAINED question: inline the MINIMAL definitions (the predicate, the obstruction mechanism, the available facts) + the precise question, framed as mathematics. This is NOT the banned "paste large source for a code-read" — it's a self-contained math problem, and it returns a sharp answer you then verify against your own derivation.
**Why:** A design consult was needed while the relevant code sat on an unpushed local branch (owner controls push timing), so repo-read was unavailable and "push it" was not an option; inlining ~5 key definitions + the counterexample mechanism produced a precise per-case answer that independently corroborated the local analysis. Treating "don't paste source" as absolute would have blocked a useful consult.
**How to apply:** Repo-read unavailable (unpushed, can't push) + the question is math/design (not "read my file") → self-contained question with minimal inlined definitions + the precise obstruction; do NOT paste whole source files, and do NOT block on pushing. Reserve the "push it / don't paste" advice for genuine code-READ tasks.

### [2026-06-24] A SLOW long-context collaborator tab is not a DEAD one — discriminate via the live processing state, don't abandon after N empty polls
A bridged collaborator tab that has accumulated many prior turns gets SLOW to respond; an empty/no-answer-yet poll then looks identical to a dead channel, and declaring it a "zombie" after a few polls ABANDONS a still-running long-think. The live registry's per-channel state (a `processing`/`prompt-sent` flag) distinguishes SLOW-but-alive from genuinely-unregistered/dead, which poll-emptiness cannot.
**Why:** A delegate gave up on a collaborator channel after three empty polls and called it a zombie tab; the registry showed it was still processing (a long-context tab with many prior responses) and the answer landed later — the abandonment was premature, and the channel was fine.
**How to apply:** Before declaring a channel dead, check BOTH registry liveness (is the tab registered?) and its per-channel in-flight flag (is a request processing?). Registered + processing = SLOW, keep waiting or proceed in parallel; not-registered = genuinely dead, route to a sibling. Never infer death from poll-emptiness alone. Long-context tabs are the slow ones — prefer a fresh/short tab when latency matters. Tell delegates this so they don't prematurely abandon a working tab.

### [2026-06-24] A cross-check dispatch is NON-LOAD-BEARING when the build/source settles it — fire parallel, proceed on the verifiable signal, harvest retroactively
When you dispatch a collaborator to CROSS-CHECK a result you can settle yourself (a build compiles it, the source decides faithfulness, your own derivation closes it), the dispatch is corroboration, not a dependency. Blocking on it — especially through a slow or degrading channel — wastes the parallelism. Proceed on the ground-truth signal; when the cross-check lands it either confirms (retroactive reassurance) or flags a discrepancy to investigate.
**Why:** A result was verified and committed on the strength of a build + a source-read while the parallel cross-check channel was slow/truncating; the cross-check confirmed it retroactively, after the commit. Waiting on it would have idled the run on a non-dependency.
**How to apply:** Classify each dispatch — LOAD-BEARING (you genuinely cannot settle it yourself: a design judgment, an unknown reachability/truth) vs NON-LOAD-BEARING (build/source/your own derivation decides it). Block only on the former. Fire the latter in parallel and proceed; treat its later agreement as a bonus and disagreement as a flag to re-open. Pairs with "channel liveness from the registry" and "split tactical vs strategic across channels".

### [2026-06-24] Spawning a same-model sub-agent does NOT discharge the parallel-independent-reasoner reflex
Offloading a hard sub-problem to a fresh-context sub-agent of your OWN architecture is not a substitute for firing the external collaborator in parallel. A sub-agent is the same reasoner with a clean context window — it shares your systematic blind spots; the external collaborator's value is INDEPENDENCE (a different model catches failure modes your own architecture is consistently blind to). "I already dispatched a sub-agent on it" must not suppress the parallel-consult reflex.
**Why:** On a hard frontier I spawned same-architecture sub-agents to grind the sub-problems and, feeling the work was "covered," skipped the parallel external consult — losing the independent cross-check exactly where it mattered most. Fresh context ≠ independent perspective.
**How to apply:** When a hard sub-problem appears, fire the external collaborator in parallel REGARDLESS of whether you also spawned a sub-agent — they serve different roles (sub-agent = parallel labor in your own voice; external = independent audit/roadmap). Keep it self-contained: ask a roadmap/tactic-level question with minimal inline context while the sub-agent grinds. Pairs with the proactive-engagement default and "split tactical vs strategic across channels".

### [2026-06-24] When the collaborator can't PRODUCE the artifact, use it to write COMPUTATIONAL DISCOVERY scripts instead
An audit-class collaborator that can't produce a proof/derivation CAN write computational scripts (Python, SageMath)
that DISCOVER the mathematical structure your proof needs — e.g. enumerate both sides of an identity, test candidate
bijections, find grouping patterns, or refute conjectured decompositions with concrete counterexamples. This is a
DIFFERENT tool-match than asking for a proof (which triggers the "confirm-but-can't-produce" anti-pattern). The
collaborator writes the probe script; YOU run it locally and interpret the output as mathematical evidence.
**Why:** After ~8 rounds of "confirm TRUE + can't produce the proof," dispatching the same collaborator to write a
Python bijection-enumerator instead produced the terminal finding (the identity has no per-term bijection — the
cancellation is irreducibly global). The computational probe answered a question no amount of audit rounds could.
**How to apply:** When a collaborator has confirmed an identity's truth and isolated its structure but can't produce
the proof: switch the task from "prove X" to "write a Python script that enumerates both sides of X, tests
candidate groupings, and reports whether a bijection/decomposition exists." Run the script yourself and use the
output as mathematical evidence (empirical test = highest-authority decision method per automode). Complements
the tool-mismatch entry: that says STOP re-dispatching the proof; this says what to dispatch INSTEAD.
