# chatgpt skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/chatgpt.md


## Proactive engagement — the default reflex (NOT a last resort)
ChatGPT is a standing collaborator, not an escape hatch for when you're already stuck. The failure
  Key: Dispatch-before-process (统筹). | Fire first, verify in parallel. | Keep both channels saturated.

## Trigger
Any of: `/chatgpt`, "问 ChatGPT", "让 ChatGPT 看看", "ChatGPT 协作模式",
  Key: Sub-command `clear`

## /chatgpt clear — full bridge reset
`/chatgpt clear` runs ONE script that stops the bridge, wipes ALL accumulated
  Key: Trash (reversible) | `:8800` is the wiki-server — never touch it. | except `token`

### Trigger sources
This skill fires from **both** sides — Claude Code prompt and Telegram:
  Key: Telegram message

## How to call ChatGPT
Use ONE script — **`ask-gpt.py`**. It blocks and prints ChatGPT's answer:
  1. **确认自己的窗口名**
  2. **只用自己窗口前缀的 channel**
  3. **查看当前可用 channel**

## Waiting for answers — use pipe, don't wait for process exit
`ask-gpt.py` prints the answer to stdout with `flush=True` and calls

### MOST RELIABLE for an autonomous agent: one `run_in_background` Bash PER channel (2026-06-17)
The `nohup … &` + later-`wait`/checkpoint pattern above has a sharp failure mode for an agent that
  Key: Do not set a `timeout` on the Bash | Don't use it | Forbidden patterns | Harvest on return, not on a checkpoint. | One `run_in_background` Bash per channel. | batch-harvesting at your next scheduled checkpoint | each channel as its own `run_in_background: true` Bash tool call | task-notification

## ASK LEDGER — persistent numbered question tracker (Xiang 2026-06-17)
The reliable per-channel harvest above fixes *orphaned* answers; it does NOT fix the bridge **DOM-capture
  Key: This is now MECHANIZED in `ask-gpt.py` — do NOT do it by hand. | What the caller still owns: | auto-prefixes the brief

## Answer modes — git-drop is the DEFAULT and ALWAYS first priority (2026-06-24)
**git-drop is not an option you choose — it is the default for EVERY call, and it
  Key: DEFAULT — every call, automatic. The answer IS the commit. | Fallback only | bridge-capture | github-write (legacy)

### GPT job mode — the DEFAULT: ChatGPT reads the repo, you do NOT paste source (2026-06-23)
Every ChatGPT tab/channel runs as a **GPT job**: a GitHub repo is connected to it,
  Key: 404-on-a-path-that-exists ⇒ wrong branch, not missing file. | Read-side workflow (do this BEFORE dispatching a code/proof task): | Stay generic — never hardcode a channel. | The connector reads the repo's DEFAULT branch — there is NO branch picker. | This is the default working mode | default branch | path + line range

### git-drop — the write half of GPT job mode (2026-06-18)
ChatGPT's GitHub connector asks for write permission ONCE per
  Key: Default: read the CODE file | How it works: | On demand: read `/tmp/gpt_<ch>.md` | Prompt template for git-drop: | Reading the answer — code by default, prose on demand (2026-06-25). | Sync shortcut | The drop file is `.md` format. | Why `.md` not `.lean`:

### git-drop completion = the COMMIT, fully decoupled from DOM (2026-06-24)
**The completion signal in git-drop mode is the git commit landing — NOT DOM
  Key: Caller action on that banner: RE-DISPATCH the question | Caller rules: | Sandbox-instead-of-commit failure (2026-06-25).

### git-drop monitoring — one tracked waiter per channel (2026-06-18)
Even with commit-polling, dispatch each channel as its own `run_in_background`
  Key: Rule: one tracked waiter per channel, immediate harvest.

### bridge-capture (unchanged)
The original mode. `ask-gpt.py` blocks and prints whatever the

### github-write (legacy, deprecated)
The old mode where ChatGPT creates NEW files via GitHub connector.

## Asking style — the rules that matter
These come from the actual BAC paper round trips (R1–R44):
  1. **Tactic-level questions beat completion requests.**
  2. **Do not over-organise Xiang's critiques before dispatching.**
  3. **Use round numbering (R1, R2, …) for multi-turn audits.**
  4. **Cite the actual file paths and commit SHA.**
  5. **One main question per round.**
  6. **No internal codenames in the outgoing prompt.**
  7. **Patience — never resubmit a long-thinking query.**
  8. **Verify, don't transcribe.**
  9. **Parallelize independent sub-questions across channels.**

## If an answer doesn't come back
All three answer modes (fast / extended / Pro) auto-deliver — you just wait on
  Key: 0 B that never fills — FIRST suspect a wrong/non-existent channel name (2026-06-17). | Do not resubmit or reset | Only after the channel is confirmed real | confirm the channel is in the live list | never invent a channel name. | stuck at 0 B | tell Xiang directly | the channel name is not a real open tab

## After the answer arrives — run banner (now auto-emitted)
As of 2026-06-21 **`ask-gpt.py` prints the banner itself, to stderr**, on every

## After the answer arrives
1. The answer is whatever `ask-gpt.py` printed to stdout.
  2. **Always send a Telegram reply**

## Telegram round-trip — start-to-finish template
Xiang (TG): /chatgpt R5: check whether step 7 still holds at commit abc1234

## Examples
Xiang: 问 ChatGPT 这个 lemma 的证明步骤 5 严密吗?

## Do NOT
- Do not paraphrase Xiang's critique before sending — verbatim or near-verbatim only.

## Learned Tactics (Self-Improvement)
_(titles only; read full file for details)_
### [2026-06-23] git-drop harvest: verify the drop file is FRESH (matches your Q#), not just non-empty
### [2026-06-23] Recover a deadline-exceeded long answer from the bridge task store — don't treat it as lost
### [2026-06-23] Smoke-test the delivery channel before fanning out a batch
### [2026-06-23] After a bridge reset, runs-log history lies — confirm channel liveness from the live registry, and re-fire stale dispatches
### [2026-06-23] A returned self-verifying script only proves consistency with the collaborator's OWN transcription
### [2026-06-23] Pre-verify any factual premise / mechanism HINT you embed in a prompt — a false premise wastes the long-think
### [2026-06-23] For one hard problem you doubt, split TACTICAL detail vs STRATEGIC route across two channels
### [2026-06-23] A wedged refinement round is not a dependency — recover from the PRIOR round + your own work
### [2026-06-23] Verify a DESIGN/architecture answer against an independent STRUCTURAL signal in the code — not the collaborator's prose
### [2026-06-23] A GLOBAL transport repair does not restore ALL channels — delivery is per-channel; verify each, route to a confirmed-working sibling
### [2026-06-23] When the user explicitly assigns a channel but git-drop isn't configured, set it up yourself (repo-owner gh CLI), don't block
### [2026-06-24] For a big problem with several routes, fan out independent route-ASSESSMENTS and act on CONVERGENCE
### [2026-06-24] Harvest git-drops where the connector/gh CLI lives — empty output on a remote worker is a missing tool, not a missing answer
### [2026-06-24] The git-drop answer can land (and be harvestable) BEFORE the dispatch process notifies — poll the drop branch, don't wait only on the process signal
### [2026-06-23] Audit-LLM that keeps "confirmed true + isolated, but I can't produce it" = TOOL MISMATCH — stop re-dispatching the grind
### [2026-06-24] A faithful PORT of existing code is usually faster SOLO than the long-think latency — reserve dispatch for no-template hard sub-problems
### [2026-06-24] On a deep analytic frontier you intend to FORMALIZE, ask for the ROADMAP, not the proof
### [2026-06-24] Math/design consult with unpushable code: inline the MINIMAL definitions — the "don't paste source, push instead" rule is for code-READ tasks
### [2026-06-24] A SLOW long-context collaborator tab is not a DEAD one — discriminate via the live processing state, don't abandon after N empty polls
### [2026-06-24] A cross-check dispatch is NON-LOAD-BEARING when the build/source settles it — fire parallel, proceed on the verifiable signal, harvest retroactively
### [2026-06-24] Spawning a same-model sub-agent does NOT discharge the parallel-independent-reasoner reflex
### [2026-06-24] The highest-leverage collaborator question is often "is my FRAMING of this sub-problem correct?" — not "prove/bound/compute X"
### [2026-06-24] When the collaborator can't PRODUCE the artifact, use it to write COMPUTATIONAL DISCOVERY scripts instead
### [2026-06-24] A mis-framed question yields a CONFIDENTLY-WRONG audit — re-read your framing + verify the artifact before acting on a strategic verdict
### [2026-06-24] Dispatch a FEASIBILITY AUDIT of your own wiring to the collaborator — it catches structural impossibilities you can't see from inside the code
### [2026-06-25] For a self-contained Lean-WRITING task that reads multiple files, a sub-agent beats a ChatGPT channel — sub-agent has direct file + build access
### [2026-06-25] Include the project's TERMINAL VERDICTS on dead routes when asking for strategic route advice — the collaborator doesn't know your history
### [2026-06-25] CAS-verify with SYMBOLIC indeterminates to discover whether a proof needs quotient-ring machinery
### [2026-06-25] Ask ChatGPT about DEFINITIONAL CONVENTIONS before proving — the zero-extension/junk-value trap
### [2026-06-25] Use ChatGPT as a MATHEMATICAL LITERATURE SEARCH tool — ask "is there a proof of theorem X that uses only tools Y?"
### [2026-06-26] Graduate from ASSESSMENT → DETAIL → CODE across multi-round campaigns
