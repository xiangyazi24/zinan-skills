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

   **Stop-and-audit when an assembly keeps spawning "one more producer" (the never-bottoming-out fingerprint):**
   - When a conditional assembly keeps reducing to "just one more residual / tail / producer" round after
     round (≥3–4 rounds) WITHOUT ever bottoming out at its base hypotheses, that pattern is the FINGERPRINT
     of a STRUCTURAL mis-framing — NOT genuine difficulty. STOP grinding the next layer and trigger the §3.3
     adversarial audit (self + codex + ChatGPT, in parallel). Grinding "one more layer" each round is a
     disguised 退堂鼓; the user should not have to be the one to call "something is wrong, go audit".
   - Audit checklist for "why won't it close" (assume there IS a structural error; find it):
     · **Redundant producer** — is the "new" hypothesis ALREADY discharged by a result you closed elsewhere,
       but the wrapper re-demands it as a fresh hypothesis instead of CONSUMING the closed theorem? Wire the
       closed result; don't re-prove.
     · **Wrong gate / predicate shape** — is an interface built on a RAW/STATIC gate (or a predicate you
       already discredited) while the closed route uses the STOPPED-PROCESS / monotone / killed-kernel shape?
       A raw/static gate re-demands the same tail FOREVER. Make every sibling interface use the same
       stopped/monotone object the closed route uses.
     · **Double-counting** — does the final wrapper consume both a closed result AND a staged path that re-does
       the same probabilistic work (e.g. driving the same quantity to its target twice)? Keep one route.
     · **Over-decomposition** — compare your stage structure to the SOURCE PROOF's actual structure. If the
       formalization has more stages / entry-gates than the paper's argument needs, the extra entry-tails are
       ARTIFACTS; collapse them. The correct repair is usually ONE-STEP and COLLAPSING (consume closed result
       + swap to the stopped gate + drop the double-count), not additive. Only if the audit finds a GENUINELY
       new tail (not an artifact) is it a real producer — then name it precisely and discharge it.
   - **Killed-object hygiene** — when you eliminate a false/discredited object (a non-monotone "floor", an
     unsatisfiable invariant, a too-strong predicate, a static cover that should be a stopped/killed kernel),
     grep ALL its uses and purge every one. It will survive in a SIBLING definition / gate / interface and
     re-spawn the exact obstruction you thought you killed. Replacing it on one route is not enough — the
     stale sibling keeps the bug alive and the assembly "forever rediscovers" the same producer.

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

   **NEVER ask Xiang to adjudicate math / Lean-technical / correctness questions — resolve them yourself (Xiang 2026-06-19, after an over-escalated protocol-encoding bug: "我不拍板, 我不懂, 你不要问我数学. 你有 paper, 有 python 可以编程验证, 有 chatgpt pro 可以问").**
   - Xiang does **not** adjudicate mathematics or proof-engineering. The instant you catch yourself writing
     "should we fix this? / which route? / is this a bug? / your call" to him about a technical/math point —
     STOP. That is the banned over-escalation. You have three tools that settle it without him:
     · the **source/paper** = ground truth on faithfulness, definitions, statements (`pdftotext` + grep the
       actual lemma/definition; the paper decides whether the encoding is a bug, not Xiang);
     · **python / the build** = empirical fact (write a script, run the scoped build, measure the ripple —
       facts, not opinions);
     · **ChatGPT Pro** (repo-connected) = cross-check on the hard fork (`ask-gpt.py`).
   - **Litmus: if the paper or a build can answer it, it is NOT a question for Xiang.** "Is the Lean update
     `min` when the paper says `max`?" → read the paper, it's a bug, fix it, rebuild, report the RESULT. Decide
     → verify → execute → report the outcome; never hand him the question.
   - The ONLY calls that are genuinely Xiang's: **method-downgrade** (relaxing/weakening the *problem* — see
     `/automode` "method-flexibility is Xiang's call"), authorship/signature, external-facing/destructive ops,
     frozen-paper edits. A faithfulness fix the source unambiguously dictates is **correctness, not a
     downgrade** — so you just do it. Don't confuse "this changes a core definition / ripples widely" with
     "this needs Xiang"; ripple is measured by a build, not by his opinion.

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
