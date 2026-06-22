Load the Lean 4 formalization playbook, project context, and the hard-won working tactics.

1. Read `~/.openclaw/workspace/formalization-playbook.md` — the full playbook (design → implementation → acceptance). It now includes:
   - §2.6 anti-patterns (incl. classify-without-source, banking re-wrappers/trivially-true/native_decide)
   - §2.7 **multi-agent orchestration** (the "分头打" parallel-codex discipline)
   - §3.2 codex/agent output review (4 extra steps)
   - §3.3 **三方对抗审查** (three-way adversarial faithfulness audit: self+codex+ChatGPT cross-check; catalogue of failure modes that pass mechanical-green — VACUOUS conditional theorems via unsatisfiable hypotheses, too-strong predicates, contract-field/parameter-coupling bound inflation; `#print axioms` can't detect an unsatisfiable premise)
   - §1.3 **文档纪律 (爸爸 2026-06-22 点名)** — 要写文档就**只写一个专档 = 项目根的 `UNDERSTANDING.md`**,跑 Lean 时在关键节点(收口残差/里程碑/换方向/发现旧理解错/session 边界)**就地维护它**;**绝不撒** `DOCTRINE_*`/`HANDOFF_*` 指路文/`*_INVENTORY`/`*_LEDGER`/带日期快照的并行文件(写完即烂、互相矛盾、反过来骗人);**默认不信任**任何文档说的"还欠什么/已完成什么",一律 grep sorry / import 闭包 / `#print axioms` 代码现算。记忆 `feedback_no_unmaintained_index`、`project_doty_clock_doctrine_stale`。

2. If in a Lean project directory (has `lakefile.lean` or `lakefile.toml`):
   - Read `UNDERSTANDING.md` if it exists — **this is the ONE maintained doc; if you write/update anything this session, update it in place, don't spawn a new doctrine/handoff/snapshot file (§1.3)**
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

5. **Maintain a CHECKLIST view — the "列清单挨个磨, 挨个钩" discipline (Xiang 2026-06-17, reaffirmed as 老规矩 2026-06-21).**
   This is the STANDING DEFAULT for any multi-sorry / multi-axiom / multi-atom Lean campaign — not a
   special mode and not something to wait for Xiang to request. The MOMENT a campaign has more than ~2
   open gaps (sorries, custom axioms, named seams, sub-lemmas), make the board FIRST, then grind. "老规矩,
   列清单挨个磨, 挨个钩" = enumerate the finite list, grind each item, flip its box the moment it verifies.
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
   - **Report the board EVERY turn going forward** — by default, proactively, from the moment the board
     exists (老规矩; no need to wait to be asked). Cite the file; show which slot moved. Clearing one atom
     = flipping one box ⬜/🟡 → ✅. Mirror it into the in-session TaskList (one task per OPEN atom) so the
     spinner reflects the board.
   - **`✅` is reserved for genuinely-discharged, sorry-free, axiom-clean.** A conditional wrapper that
     carries a satisfiable hyp is `🟡`, never `✅` — honest-labeling (§3.3) applies to the board too. When
     a survival "core" is sorry-free but carries an abstract escape, mark `✅ (escape-audit pending)`, don't
     silently upgrade to done.
   - **Distance = the atom inventory, never a time estimate.** When asked "how far", answer with
     done/partial/open counts + what each open atom needs — not "几天/一晚/multi-week" (banned, and each atom
     tends to expand 1–2 sub-layers when pushed, so even the count is a lower bound). See `/automode`
     fingerprint rules + memory `project_exactmajority_phase_board` for the canonical example.

6. **Derive structure from primitives — eliminate well-formedness / compatibility fields (Cauchy-rigidity ch13 campaign, 2026-06).**
   A structure FIELD is either *definitional data* or a *derivable theorem in disguise*. Audit every
   "well-formedness" / "closed-surface" / "compatibility" field: can it be proved from the geometric/primitive
   fields (+ one honest geometric completeness hypothesis)? It usually can — leaving it as a field is an
   un-eliminated assumption hiding in the object.
   - **Combinatorial/topological structure is downstream of geometry.** The closed-surface skeleton of a
     convex polytope — each edge in exactly two facets, each vertex link a single cycle, consistent face
     orientation, sphere-ness (connected + Euler=2), rotation-faithfulness, degree≥3 — are all CONSEQUENCES
     of convexity + "the listed faces are exactly the facets of conv(vertices)" (a `SimplicialFacetComplete`
     predicate). They were carried as structure fields/axioms; all got DERIVED as theorems.
   - **Redundant data + a compatibility axiom ⇒ eliminable.** If you store a quantity that the primitives
     already determine (a face's outward `normal`) AND an axiom that it is compatible (`0 < det3 … normal`),
     DEFINE the quantity from the primitive (`normal := cross` of the cyclic-vertex-order edges) and the axiom
     collapses to a triviality (`det3 (u) (v) (cross u v) = ‖cross u v‖² > 0`, nondegeneracy from
     strict-support + ≥4 vertices). The orientation then lives ENTIRELY in the cyclic vertex order — pure
     definitional data, no "orientation axiom".
   - **The definition-vs-theorem-vs-assumption test (Xiang 2026-06).** If a field is definitional, accept it
     and do NOT call it an "irreducible assumption you couldn't eliminate" — that framing is *incoherent*
     (不通). If it is a theorem, prove it from primitives. There is no third "irreducible field" category: a
     field is data you define, or a consequence you derive. The mirror argument ("orientation can't come from
     *unoriented* data") justifies the cyclic order as definitional INPUT — it does NOT justify keeping a
     separate normal + compatibility axiom.
   - **Mathlib-gap reformulations (reach for elementary, not the missing heavy machinery).** Mathlib lacks
     normal cones / extreme rays / a polytope face lattice / ∂(convex)≅Sⁿ⁻¹. Don't build them; reformulate:
     · *edge of a full-dim 3-polytope lies in exactly two facets* → **shadow-slope**: pick coords transverse
       to the edge, min/max slope over the other vertices give the two supporting facets (no cones).
     · *vertex link / facets-at-a-vertex is connected (single cycle)* → **potential descent**: generic linear
       functional X; every non-X-min exposed vertex has a strictly-lower X adjacent vertex via the one
       geometric lemma `local_tangent_cone` (Q ⊆ apex + cone of the two incident edge directions); finite
       descent ⇒ connected (no Jordan / ∂≅S²). Build only `local_tangent_cone`; the rest is det2 + finite order.
     · transport 3D↔2D via an explicit **chart** (`τ : ηᗮ ≃ₗ E2`) + a **homogenization** identity
       `L s − L p = h(s)·(ℓ(ρ s) − C)`, not by transporting `IsExposed` across an affine map (Mathlib has no
       affine-image lemma for `IsExposed`).
   - **Faithfulness grep — the cheap §3.3 catch for "derive field F".** A proof of `F_of_complete (X) (h) : …`
     that does `exact X.F` is a circular cheat that passes lake-green + `#print axioms` clean. After any
     "derive a field" task, `grep` the proof body for the field name (`\.opposite_side_unique` etc.); it MUST
     be empty. Caught codex doing exactly this once; force geometry-only proofs and grep to confirm.
   - **To actually REMOVE a field (not just prove it redundant): split + re-parametrize + reconstruct.** Split
     the structure into `Core` (primitive data only) + the predicate; re-parametrize the derivations onto
     `Core` (binder change `X : Full → C : Core` — proofs transfer because they never used the field); the
     user-facing object = `Core` + the genuine geometric primitive (`SimplicialConvexBody := Core + complete`);
     build the full object via `toClosed` that FILLS the now-derived fields from the derivations. Preserve the
     public name with `abbrev Full := Closed` / `Closed extends Core` so downstream is untouched; verify with a
     FULL `lake build` + `#print axioms` clean-3 on the headline applied to the reconstructed witness.
   - Net result of one such campaign: the user-facing convex-polytope object's only inputs became geometry
     data (`pos`, `tri`) + convex well-formedness (`tri_inj`, supporting/strict halfspace) + `complete`;
     everything combinatorial/topological is a clean-3 theorem. ~57k lines (ch13+ch35), 0 sorry, 0 native.

7. **The adversarial-discharge loop — turning a "all-conditional, no real theorem" repo into genuine landed reductions (Shen trilogy, 2026-06).**
   The failure this beats: a whole formalization that is `#print axioms`-clean with zero `sorry` and yet
   proves NOTHING — every headline is a conditional theorem carrying a pile of hypotheses that LOOK
   satisfiable but are never discharged. **Mechanical-green ≠ real content.** The fix is a per-brick loop
   that makes the default reflex *delegate → hostile-audit → land only what survives*, and it is portable to
   any Lean project where a strong grinder (codex) produces discharges you must trust.

   - **The loop (one brick at a time).** codex grinds a discharge → an INDEPENDENT, read-only, HOSTILE opus
     auditor whose default is *"this is fake — break it"* → commit ONLY on a clean verdict. The author agent
     AND the orchestrator (you) share a blind spot; neither counts as review. Spawn a separate adversary.

   - **The verdict taxonomy — name the failure modes that pass mechanical-green, so the auditor has a checklist.**
     · **GENUINE-NET-REDUCTION** — the ONLY commit-worthy verdict: the headline's transitively-carried free
       analytic hypotheses STRICTLY decreased, the removed hypothesis is genuinely DERIVED (proof read, not a
       re-posit), and the conclusion type is byte-identical. Commit.
     · **NO-OP** — internal rewiring that stops *consuming* a field but does NOT *delete* it from the structure
       + constructors → the construction obligation is unchanged. Zero headline progress dressed as progress.
       Do not commit it as a reduction; take the deletion it set up.
     · **RENAME-CARRY** — the field is "removed" but relocated/renamed into a sub-structure that is itself
       carried free → frontier unchanged.
     · **FANOUT-INCREASE** — removing 1 field ADDS ≥1 new free fields → a net increase disguised as a reduction.
     · **VACUITY** — the conclusion was silently WEAKENED (e.g. a solution-ness field dropped while the headline
       still claims "a solution") → the theorem now asserts less. Reject hard.
     · **FRAGMENT-mislabel** — a real but PARTIAL brick (one term of an inequality, one leg of the bound)
       reported by codex as "proved the main theorem"; the headline does not consume it. Keep the brick, reject
       the label, re-attack the real core.
     · **UNSATISFIABLE-FRONTIER / TOO-STRONG** — a carried hypothesis (a parameter condition, a ∀-field) has NO
       producer / no witness anywhere → never dischargeable, the theorem is vacuously true. `#print axioms`
       cannot detect this.

   - **How the auditor actually breaks it (the mechanics).**
     · **Net-free-hypothesis count** — `git show HEAD:file` (before) vs the workdir (after); count the headline's
       transitively-carried hypotheses both ways. Strictly fewer, or it is not a reduction. Diff the structure
       and the constructors, not just the headline signature.
     · **Read the proof term, never the codex report** — codex overstates ("proved X") when it proved one term
       of X. Read the actual conclusion type + TRACE how the hard step is established vs assumed-away. Verify the
       discharged field is a genuine THEOREM `…_of_classical (hsol) : …`, not a re-posited structure field.
     · **Satisfiability witness** — every carried hypothesis needs a producer/`Nonempty` instance or a concrete
       satisfying-parameter witness. `grep` for any producer of the carried Prop; none ⇒ relocated-undischargeable.
     · **Faithfulness grep** — a "derive field F" proof that secretly does `exact X.F` is circular and passes
       lake-green; `grep` the proof body for the field name, it MUST be empty.

   - **Root-build EVERY commit, not per-paper (the gate that catches drift).** A file dropped from the headline's
     import closure still ships in the repo; a fatal error in it (duplicate decl, broken proof) surfaces ONLY at
     the full `lake build <RootLib>`, never at a per-module `.Statements` build. Per-module verification silently
     let `main` drift to non-root-building for several commits. Land behind a **fresh-checkout-of-origin/main +
     full root build** clean-3 gate. For speed, keep a persistent warm build clone (a dedicated remote tmux:
     `git fetch && git reset --hard origin/main && lake build <Root>`, record the exit code) for iteration, and
     reserve a COLD fresh clone for the final "this paper is DONE" gate (a warm clone can mask a stale-olean bug).

   - **Honest labeling is the bar above any milestone.** conditional is conditional, fragment is fragment. Land
     the genuine brick with a commit message stating EXACTLY what is proved vs what is still carried (name the
     open atoms). Never let "axiom-clean" mean "done". When codex overstates, the visible signal is the audit
     verdict + the corrected accounting, not the green build.

   - **The payoff.** This converts a hopeless "everything's conditional" repo into a FINITE frontier of named
     hard cores — each headline reduces to its genuine analytical main theorem (e.g. a C³ Schauder bootstrap, a
     logistic-absorption energy inequality, a spectral-agreement frontier), which you then grind one at a time.
     The turning point is not speed; it is blocking the fakes so the real reductions are trustworthy. Pair with
     §3.3 (the failure-mode catalogue), the CHECKLIST board (§5), and `/fable` (Fable-as-master: design the
     route + dispose of deliverables, delegate the grind).

8. Report: "Playbook + tactics loaded. [N] sorry found. [If a CHECKLIST exists: k/N atoms ✅.] Ready for Lean work."

$ARGUMENTS is an optional project path. If provided, cd there first.
