Load the Lean 4 formalization playbook, project context, and the hard-won working tactics.

1. Read `~/.openclaw/workspace/formalization-playbook.md` — the full playbook (design → implementation → acceptance). It now includes:
   - §2.6 anti-patterns (incl. classify-without-source, banking re-wrappers/trivially-true/native_decide)
   - §2.7 **multi-agent orchestration** (the "分头打" parallel-codex discipline)
   - §3.2 codex/agent output review (4 extra steps)
   - §3.3 **三方对抗审查** (three-way adversarial faithfulness audit: self+codex+ChatGPT cross-check; catalogue of failure modes that pass mechanical-green — VACUOUS conditional theorems via unsatisfiable hypotheses, too-strong predicates, contract-field/parameter-coupling bound inflation; `#print axioms` can't detect an unsatisfiable premise)

2. If in a Lean project directory (has `lakefile.lean` or `lakefile.toml`):
   - Read `UNDERSTANDING.md` if it exists
   - Read `HANDOFF.md` if it exists
   - Read any `*_CHECKLIST.md` (the atom board, see §5) if it exists — and lead the session by reporting it
   - Run `rg -n 'sorry|admit' --glob '*.lean' .` to show current gaps
   - Run `git log --oneline -5` for recent context

3. Internalize these **core tactics** (DNA32 / Most-Beautiful-Identity campaign, 2026-05-25):

   **Verify before claim (the discipline Xiang explicitly praised):**
   - Before calling a chapter/theorem "open/missing/has a gap", check the **actual source** — for the
     Q-series project `pdftotext "Hai-Chi Chen_An invitation to q-series.pdf"` and grep the real
     `Theorem N.M`. (Ch8/Ch11/Ch14 were all mis-classified from impression until checked.)
   - **Trust-but-verify every codex "completed"**: pull the file, fresh `#print axioms` from rebuilt
     oleans (must be {propext, Classical.choice, Quot.sound} — no `sorryAx`, no `ofReduceBool`/`trustCompiler`).
   - **Never bank**: re-wrappers (`:= existing_lemma`, zero new math), trivially-true/degenerate results
     (e.g. defining a limit value as a constant so convergence is vacuous), or `native_decide` (it injects
     `ofReduceBool`/`trustCompiler` — use `decide`/`rfl`).
   - Report partial progress honestly as `partial`/AUX; never inflate to PASS. Tag each result
     unconditional / conditional-on-X (X proved/not) / auxiliary.

   **Multi-agent dispatch ("分头打"), running codex on uisai1 via tmux + HANDOFF/:**
   - One file, one writer. Agents create only their own NEW file; never touch `<Lib>.lean` / `Audit.lean`
     (you wire the import graph + `#print axioms` yourself). **`Audit.lean` keeps its OWN import list** —
     when adding a `#print axioms` for a new module, import it in `Audit.lean` too.
   - Agents self-verify with `lake env lean <file>` only. **Never `lake build` while agents edit** — the
     rsync clobbers in-flight files and corrupts lake's incremental cache (stale root olean →
     "unknown constant"). Run the one integrated build after all agents finish.
   - Dispatch: `codex exec --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check -m gpt-5.5 "<spec>"`
     in a dedicated tmux session, log via `tee`; watch `HANDOFF/outbox/*-reply.md` with a background
     until-loop (don't poll tmux chars). Each reply → pull/verify/wire/commit.
   - Give research-hard tasks a **bounded sub-goal** + "close what you can, report exactly what blocks,
     no faking" — not "close the whole chapter".
   - Hard single proofs → one coherent agent (or take it yourself with Opus — separate compute, doesn't
     compete for the shared gpt-5.5 rate limit). Parallelize only independent chapters.

   **Remember ChatGPT Pro is available — don't forget the tool exists (Xiang 06-15 correction, twice):**
   - The repo-connected strong-reasoning webapp (ChatGPT Pro) is a standing collaborator, reachable via
     `python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> < q.txt`. The instant a research-level
     sub-point appears — a hard lemma, a tactic chain you can't see through, a modeling/design fork, an
     "is this even true / the right object" — reflexively dispatch a grounded question **FIRST**, then
     derive in parallel to VERIFY/REFUTE (never transcribe). Do **not** burn long solo analysis while a
     tab sits idle ("你为什么不会主动找 chatgpt 呢?"). Context length is no excuse for forgetting it's there.
   - Apply 华罗庚统筹法 over the tabs (playbook §2.8): keep them maximally parallel — fire before going
     heads-down (you can't add to a busy tab without truncating), decompose into independent sub-questions
     to feed several tabs, dispatch-before-process. The §1.1 design brainstorm and §4 statement audit are
     **ChatGPT/Codex-driven by default, not solo**. See `/chatgpt` skill + memory `feedback_proactive_chatgpt`.

4. **Engineering discipline** (SSExactMajority 2026-05-24/25 war):
   - **File > 2000 lines → split first** (5 min job, not "1-2 days"). Split by `/-!` section headers.
   - **maxHeartbeats ≥ 8M proofs → separate file**. Otherwise every edit to the same file triggers 15-60 min recompile. E.g. TimerDrainProof.lean (heavy, compile once) + BridgeProof.lean (light, edit freely).
   - **Remote build always nohup**: `nohup lake build Module > /tmp/build.log 2>&1 &`. Never bare SSH pipe (timeout kills build).
   - **`lake env lean --file` doesn't rebuild upstream olean**. After changing upstream definitions, must `lake build Module.Name` to refresh the dependency chain. (`--file` mode uses stale olean → phantom errors.)
   - **Debug: sorry-first-then-restore**. Multiple build errors → sorry ALL broken proofs, get BUILD OK baseline, then restore proofs one-by-one. Don't iterate on a broken file (each cycle wastes 15+ min).
   - **Factored decomposition > unfold entire function**. Don't `unfold transitionPEM` (term explosion → simp maxRecDepth). Factor into small helpers (like Phase0Transition p0r1s..p0r5t), prove properties on each helper.
   - **Statement audit before axiom/sorry**: let GPT-5.5 find counterexamples. False axiom >> sorry (sorry gets caught, false axiom gets trusted).

5. **Maintain a CHECKLIST view — the "挨个 check 掉" discipline (Xiang 2026-06-17).**
   A big formalization feels 无休无止 (endless) when tracked as a sorry-pile. Convert it into a FINITE,
   NAMED atom board you check off one by one — that is the feeling Xiang wants ("我喜欢这种挨个 check 掉的感觉").
   - **Decompose into a finite atom set, grounded in the ACTUAL structure — not guessed.** Find the top
     theorem (e.g. `doty_theorem_3_1_faithful`); read its carried hypotheses / the assembly bundle it is
     conditional on (e.g. a `…ResidualAtoms` structure with `work : Fin N → PhaseConvergenceW`). THOSE
     fields ARE the atoms. Enumerate them by reading the signature, per verify-before-claim — never invent
     the list from impression.
   - **Write a durable `<GOAL>_CHECKLIST.md` in the project** (e.g. `PHASE_CHECKLIST.md`). One checkbox per
     atom. Status markers: `✅` discharged sorry-free (verified at theorem level + `#print axioms` clean) ·
     `🟡` partial / in active work (name exactly what's open) · `⬜` open / unaudited. Add a cross-cutting
     section for things the headline theorem needs that aren't atoms (correctness half, the assembly
     constructor, escape/satisfiability audits). End with an honest scoreboard (`k/N` done) + a
     `Last verified:` line. This file SURVIVES sessions and is the single source of truth.
   - **Report the board EVERY turn going forward** once Xiang asks for it. Cite the file; show which slot
     moved. Clearing one atom = flipping one box ⬜/🟡 → ✅. Mirror it into the in-session TaskList
     (one task per OPEN atom) so the spinner reflects the board.
   - **`✅` is reserved for genuinely-discharged, sorry-free, axiom-clean.** A conditional wrapper that
     carries a satisfiable hyp is `🟡`, never `✅` — honest-labeling (§3.3) applies to the board too. When
     a survival "core" is sorry-free but carries an abstract escape, mark `✅ (escape-audit pending)`, don't
     silently upgrade to done.
   - **Distance = the atom inventory, never a time estimate.** When asked "how far", answer with
     done/partial/open counts + what each open atom needs — not "几天/一晚/multi-week" (banned, and each atom
     tends to expand 1–2 sub-layers when pushed, so even the count is a lower bound). See `/automode`
     fingerprint rules + memory `project_exactmajority_phase_board` for the canonical example.

6. Report: "Playbook + tactics loaded. [N] sorry found. [If a CHECKLIST exists: k/N atoms ✅.] Ready for Lean work."

$ARGUMENTS is an optional project path. If provided, cd there first.
