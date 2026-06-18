---
description: "Codex workflow for asking ChatGPT web/Pro through the local ask-gpt bridge and consuming answers through bridge capture or git-drop. Trigger when Xiang says /chatgpt, '问 ChatGPT', '让 ChatGPT 看看', '和 ChatGPT 磋商', or when a hard research/proof/design sub-point appears. Uses Codex-native shell, files, git, and gh CLI; no Telegram-specific reply tools, Claude run_in_background, MCP, tmux send-keys, or Agent subagents."
user-invocable: true
---

# /chatgpt — Codex ChatGPT Collaboration

ChatGPT is a standing research collaborator, not a last resort. Use it early for
hard proof, modeling, audit, paper, or implementation sub-points. Codex asks,
waits or monitors, verifies the answer, and lands only the parts that survive
local checks.

This is the Codex caller's guide. It removes Claude-Code-only mechanics and
uses local shell, files, `git`, and `gh`.

## Proactive Engagement

Do not spend long solo time on a hard sub-point while ChatGPT channels are idle.

The reflex:

- **Fire first, verify in parallel.** When a research-level sub-point appears,
  send a grounded question before sinking time into solo derivation. Then verify
  independently.
- **Keep channels useful.** If multiple independent hard questions exist and
  multiple channels are known-good, dispatch them separately.
- **Dispatch-before-process.** When an answer returns and the next round is
  obvious, send the next grounded question before doing long processing.
- **No permission needed for hard sub-points.** Xiang's explicit "问 ChatGPT" is
  a trigger, not the only trigger.

## Trigger

Any of:

- `/chatgpt`
- 问 ChatGPT
- 让 ChatGPT 看看
- ChatGPT 协作模式
- 和 ChatGPT 磋商
- 丢给 ChatGPT
- 提问 ChatGPT

If Xiang's intent is unclear, ask once whether to dispatch.

## Codex-Native Calling Surface

Use the bridge script directly from shell:

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py <channel> "question"
```

For long prompts:

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py <channel> <<'EOQ'
question with file paths, commit SHA, and exact requirements
EOQ
```

Rules:

- Do not use `ask-chatgpt.sh`.
- Do not use Claude `run_in_background`, MCP, Telegram tools, tmux send-keys, or
  Agent subagents.
- Do not add an outer timeout. The bridge has its own long deadline.
- Do not resubmit a query that is still thinking.
- Confirm the channel name from known-good history or Xiang; never invent
  `cron1` because `cron` and `cron2` exist.

## Channel and Ledger Discipline

Keep a persistent numbered ledger:

`~/.chatgpt-bridge/ASK_LEDGER.md`

One row per dispatch:

```markdown
Q# | time | channel | topic | mode | status | RUN#/SHA | notes
```

Statuses:

- `dispatched`
- `captured`
- `needs-paste`
- `git-drop fetched`
- `verified`
- `recovered`

Prefix every outgoing question with the ledger number:

```text
Q42 (ac): R3 audit at commit abc1234...
```

Lifecycle:

1. Read and bump the next `Q#` cursor.
2. Append a `dispatched` row before sending.
3. On answer, update status immediately.
4. If capture fails, mark `needs-paste`; do not silently retry.
5. If Xiang pastes the answer, mark `recovered`.
6. After local verification, mark `verified` with the file/build evidence.

## Three Answer Modes

| Mode | Fidelity | Unattended | Use |
| --- | --- | --- | --- |
| bridge-capture | imperfect DOM capture | yes | short text, audits, discussion |
| git-drop | byte-perfect git file | yes | code, Lean, LaTeX, scripts, long answers |
| github-write legacy | perfect but click-heavy | no | avoid unless git-drop unavailable |

Default to **git-drop** for artifacts.

## Bridge-Capture Mode

Run `ask-gpt.py` and capture stdout:

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py ac <<'EOQ' >/tmp/gpt_Q42.txt 2>&1
Q42 (ac): ...
EOQ
```

For a single question, foreground is simplest: let the command block until it
prints the answer. For parallel independent questions, use separate shell
processes and separate files; poll or wait explicitly. Do not rely on
Claude-style background task notifications.

After completion:

1. Read `/tmp/gpt_Q42.txt`.
2. Read `~/.chatgpt-bridge/runs.log` for the latest matching run.
3. Print a visible banner in Codex's progress/final response:

```text
═══ BRIDGE RUN #N ═══ <channel> | task <id8> | <bytes> B | COMPLETE
```

If bytes are low or the output contains `[BRIDGE:` markers, mark
`needs-paste` and ask Xiang to paste from the tab. Do not act on truncated text.

## Git-Drop Mode

Git-drop is the default for code/proof artifacts. ChatGPT writes a complete
markdown answer into a preconfigured drop file on a scratch branch of the
connected GitHub repo.

The answer file is markdown:

- prose reasoning;
- suggestions;
- code in fenced blocks such as ```lean or ```python;
- commit SHA reported back in chat when possible.

### Prompt Template

Put the task in the question text, not in the drop file:

```text
Q<N> (<channel>): <full question with context, file paths, commit SHA, requirements>

IMPORTANT: Write your COMPLETE response into <drop-file> on <scratch-branch>
via the GitHub connector. UPDATE the existing markdown file.
Use markdown:
- Explanation and reasoning as prose
- Code in fenced blocks with language tags
- Include all imports in code blocks
After committing, report the commit SHA.
```

### Fetching With `git`

If the connected repo is cloned locally:

```bash
git fetch <remote> <scratch-branch>
git show <remote>/<scratch-branch>:<drop-file> > /tmp/chatgpt_Q<N>_drop.md
```

### Fetching With `gh` CLI

Use `gh` when the local checkout is not already on the connected repo or when
you want a direct GitHub read:

```bash
gh api \
  repos/<owner>/<repo>/contents/<drop-file> \
  -f ref=<scratch-branch> \
  --jq '.content' | base64 -d > /tmp/chatgpt_Q<N>_drop.md
```

If ChatGPT reports a commit SHA, verify the file at that SHA:

```bash
gh api \
  repos/<owner>/<repo>/contents/<drop-file> \
  -f ref=<commit-sha> \
  --jq '.content' | base64 -d > /tmp/chatgpt_Q<N>_drop.md
```

If the drop file needs initialization, use `gh api` only in a repo and branch
Xiang has designated for scratch work:

```bash
printf '# ChatGPT drop\n' | base64 | tr -d '\n' >/tmp/drop.b64
gh api repos/<owner>/<repo>/contents/<drop-file> \
  -X PUT \
  -f branch=<scratch-branch> \
  -f message='init chatgpt drop file' \
  -f content="$(cat /tmp/drop.b64)"
```

Do not initialize scratch branches in sensitive or shared repos without explicit
permission.

### Extracting Code

```bash
sed -n '/^```lean/,/^```/p' /tmp/chatgpt_Q<N>_drop.md | sed '/^```/d' > /tmp/answer.lean
sed -n '/^```python/,/^```/p' /tmp/chatgpt_Q<N>_drop.md | sed '/^```/d' > /tmp/answer.py
```

Then compile, test, or diff locally. ChatGPT output is not accepted until Codex
has verified it.

## Monitoring

Codex does not have Claude `run_in_background` notifications. Use one of these
patterns:

### Foreground for one hard question

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py ac < /tmp/Q42.txt > /tmp/gpt_Q42.txt 2>&1
```

Let it run. Do other local reasoning only if Codex can use a separate command
session without losing the foreground process.

### Separate shell processes for independent questions

```bash
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py ac  < /tmp/Q42.txt > /tmp/gpt_Q42.txt 2>&1 &
PID42=$!
python3 -u ~/.openclaw/workspace/scripts/ask-gpt.py ac2 < /tmp/Q43.txt > /tmp/gpt_Q43.txt 2>&1 &
PID43=$!
```

Record PIDs and output files in the ledger. Poll explicitly:

```bash
ps -p "$PID42" >/dev/null || echo Q42 done
wc -c /tmp/gpt_Q42.txt /tmp/gpt_Q43.txt
```

Harvest as soon as an output is complete. Do not batch-harvest only at the end
if one channel finished earlier.

### Wrong channel triage

If output stays 0 bytes:

```bash
grep -q "ch=<channel> .*verdict=OK" ~/.chatgpt-bridge/runs.log || \
  echo "channel likely wrong or never succeeded"
```

Wrong channel is more likely than a long blank answer. Re-fire only after
confirming the channel name; do not ask Xiang to paste from a phantom tab.

## Asking Style

1. **Tactic-level questions beat completion requests.** Ask whether claim X
   follows from setup Y, not "finish this proof".
2. **Do not over-organize Xiang's critiques.** Send his concern close to
   verbatim so ChatGPT can disagree with the framing.
3. **Use round numbering.** R1, R2, ... with what changed since last round.
4. **Cite actual file paths and commit SHA.** Never say "recent version".
5. **One main question per round.** Split independent topics.
6. **No internal codenames.** Replace private project codenames with descriptive
   names on first mention.
7. **Patience.** Pro/Extended thinking can take a long time. Do not resubmit or
   reset mid-flight.
8. **Verify, don't transcribe.** Re-derive, compile, test, or grep before acting.
9. **Parallelize independent sub-questions.** Do not split one coherent question.

## After the Answer

1. Surface the bridge run banner or git-drop commit SHA in the visible Codex
   response.
2. Read the answer critically.
3. Verify repository claims against actual files.
4. Compile/test extracted code.
5. If iterating, commit verified changes first, then ask the next round against
   that commit SHA.
6. Update the ask ledger.

## If the Answer Fails to Come Back

- Do not resubmit or reset a pending Pro/Extended answer.
- If output is empty for a long time, first suspect a wrong channel.
- If output contains `[BRIDGE:` or is obviously truncated, mark
  `needs-paste` and ask Xiang to paste.
- If a file has content but process remains alive, inspect the file and check
  size stability before killing the process.
- Do not act on partial output.

## Do Not

- Do not paraphrase Xiang's critique into your own framing before dispatch.
- Do not silently retry bridge failures.
- Do not commit ChatGPT claims without verification.
- Do not use `ask-chatgpt.sh`.
- Do not resubmit a query that is still thinking.
- Do not use Claude-specific `run_in_background`, MCP, Telegram reply tools,
  tmux send-keys, or Agent subagents.
- Do not mass-reload tabs.
