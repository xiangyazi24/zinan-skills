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

## Learned Tactics (Self-Improvement)

<!--
  此 section 由 /self-improve 自动维护，请勿手动编辑此区域内的内容。
  - header 以上 = 手写正文，/self-improve 绝不修改；header 以下 = 自动维护区。
  - 每条经验通过三道筛：可泛化、非重复、经过验证。超过 30 条触发压缩合并。
-->

### [2026-06-22] Your OWN "wiring vs hard theorem" verdict is a claim — TEST it, never assert it
The adversarial-audit discipline (#7) targets the GRINDER's deliverables. It applies just as hard to the
ORCHESTRATOR's own strategic judgments. When a residual is ambiguous between "just wiring of landed pieces"
and "a genuine hard analytic theorem (Gronwall / circularity / multi-session)", do NOT assert either label —
dispatch a subagent that ATTEMPTS the wiring with an explicit charge: "DECISIVE VERDICT: wiring, or
estimate-blocked? Land it if wiring, name the precise lemma if not." Asserting the difficulty level instead
of testing produces overclaims in BOTH directions: "it closes" (when it silently carries a hypothesis) AND
over-correcting to "multi-session hard" (when a kernel/index trick makes it wiring).
**Why:** This session I asserted "regularity closes" (false — the headline carried an uninstantiated
`UniformBootstrapStep` field), then over-corrected to "genuine multi-session PDE theorem" (also false — a
Duhamel kernel separation e^{−λ(t−τ)}=e^{−λt}e^{λτ} reduced ∂ₜ to plain FTC). Each was resolved only by a
decisive attempt-the-wiring test, which both landed the bricks AND gave the true verdict.
**How to apply:** Before committing/reporting any difficulty verdict on a residual you have NOT attempted —
stop, spawn one decisive test that tries to close it and reports "wiring vs precise-residual". And read the
headline theorem's CARRIED hypotheses before any "unconditional" claim — a conditional theorem builds green
and is axiom-clean.

### [2026-06-22] CAS-verify algebraic identities before encoding in Lean
When a proof step reduces to an algebraic identity (`linear_combination` / `ring` over symbolic terms),
verify it in a CAS first: treat all function values as free symbols, compute cofactors, confirm the
combination is exactly zero. Only then transcribe to Lean — it closes first try. Companion step: Lean's
`ring` treats syntactically different index expressions (e.g. `f (m+(k+1))` vs `f (m+k+1)`) as distinct
atoms, so canonicalize all indices with `simp only [show ... by ring]` before the `linear_combination`.
**Why:** Guessing cofactors and iterating in the slow build loop (minutes/cycle, opaque errors) wastes
time on something a CAS settles in seconds.
**How to apply:** Any `linear_combination` or `ring` goal involving multi-variable polynomial identities
or indexed sequences.

### [2026-06-22] Prove the multiplied form first, cancel the factor separately
When a proof step requires dividing out a factor, split into two layers: (1) prove the unconditional
multiplied identity `factor · residual = 0` over a general `CommRing` — no nonvanishing hypothesis
needed, pure `linear_combination`/`ring`; (2) cancel the factor separately in a domain using
`mul_eq_zero.mp` + the nonvanishing lemma. This isolates the "factor ≠ 0" obligation to one place and
keeps the algebraic core reusable. Watch for indices where the factor genuinely vanishes (boundary
cases) — those need separate, usually simpler, discharge.
**Why:** Baking cancellation into the main identity drags a nonvanishing hypothesis through the entire
proof and makes `linear_combination` fail over a general ring.
**How to apply:** Any inductive or recurrence identity whose step is naturally a ratio (identity mod a
common factor).

### [2026-06-22] Borrow the repo's OWN probability/analytic library before proving a base brick from scratch
A mature formalization repo accretes its own concentration/analytic toolbox (Chernoff, Azuma, Janson,
Bennett, Freedman, MGF-tail, first-passage). When an assembly bottoms out at a probabilistic/analytic base
fact, `rg` the repo's `Probability/` (or equivalent) dir FIRST — the brick is usually already proven there,
often under a project-specific name (`*.syncFail_le`, `seam_atRiskTail_of_entry`, …). Reaching straight for a
from-scratch proof or declaring a "Mathlib gap" is the slow default and frequently re-derives what the repo
already owns. Grep name-variants of the inequality SHAPE, not just the exact lemma name.
**Why:** Framed several base seam/drift facts as fresh obligations when the repo's own `ClockBudgets`,
`BennettLemma`, `FreedmanBound`, MGF modules already supplied most of them. Xiang's correction:
"概率的东西不要全部从正，借用…先调研再动手." This generalizes verify-before-claim from "is it open?" to
"did WE already close it?" — pairs with memory `feedback_grep_repo_before_gap`.
**How to apply:** The moment a base analytic/probabilistic inequality appears in an assembly, before writing
any proof or citing Mathlib: `rg` the repo's probability dir for the inequality shape + plausible name
variants. Only build from scratch (or escalate to Mathlib/ChatGPT) after that survey comes up empty.

### [2026-06-22] De-axiomatizing: audit the hypotheses the axiom let you SKIP, first
Removing a custom axiom that directly concludes a goal which a long *conditional* theorem chain also
concludes means you must now discharge that chain's full hypothesis bundle. The prime vacuity suspects are
the hypotheses that ONLY the axiom-path let you skip — carried unused through every level of the chain and
never discharged anywhere in the existing tree. Before grinding any discharge or building integration on top,
locate where the axiom plugs in, enumerate the hypotheses it short-circuits, and verify each has a
producer/satisfiability witness.
**Why:** A regularity/continuity hypothesis about a composite — e.g. `Continuous (indicator ∘ trajectory)`
where the indicator is a step/threshold and the trajectory is a continuous ODE solution that actually crosses
the threshold — can be flatly UNSATISFIABLE (step ∘ continuous-crossing = Heaviside jump), yet the headline
still builds green and `#print axioms`-clean because the axiom bypasses that whole sub-chain. Caught only by
checking that the existing "complete" headline reaches its conclusion *via the axiom*, never via the
hypothesis. Faithful fix was a continuous (Bernstein-polynomial) realization of the indicator, threaded
additively at the use-site so the chain's other (abstract) uses were untouched.
**How to apply:** First move of any de-axiomatization — map the axiom's plug-in point, list the conditional
chain's hypotheses it skips, satisfiability-audit each (find the producer/witness) BEFORE committing effort.
A "Continuous (step/snap/threshold ∘ continuous-flow)" obligation is unsatisfiable wherever the flow crosses;
realize the discontinuous object as a continuous polynomial. Pairs with §3.3 + §7 UNSATISFIABLE-FRONTIER.

### [2026-06-22] `subst h` can eliminate the WRONG variable — use `rw [h]` when you need both names
When `h : a = b` and BOTH sides are local variables, `subst h` removes one of them everywhere (often
not the one you expect). If a lemma you call later still references the eliminated variable, you get
"unknown identifier". When you only want to specialize the goal's occurrences and keep both variable
names in scope for subsequent lemma applications, `rw [h]` on the goal instead of `subst h`.
**Why:** A case-split equality branch followed by `subst` silently dropped a variable that the very next
lemma application still needed, producing a confusing "unknown identifier" far from the real cause.
**How to apply:** In case-split branches that handle an equality between two in-scope variables where you
still apply lemmas mentioning both afterward — prefer `rw [h]` over `subst h`.

### [2026-06-22] Overlapping rewrite lemmas misfire — state the goal already-canonical
When normalizing function-argument indices for a `ring`-family closer via `simp only [show _ = _ by ring,
…]`, if one rule's LHS is a SUBTERM of another's (e.g. `x+(y+1)` occurs inside `x+(y+1)+1`), simp rewrites
the inner match first and the outer rule then never fires, leaving a non-canonical atom that breaks the
closer even though the math is exact. Fix: write the lemma's GOAL with already-canonical arguments (don't
ask simp to derive them) and apply the index rewrites only to the hypotheses; or remove the subterm
overlap from the rewrite set. Sibling tip: library lemmas arrive with un-normalized arguments too — run
the same canonicalization `at that_hypothesis ⊢`, not just on the goal.
**Why:** A combination a CAS had confirmed exact still failed because one residual atom stayed `x+y+1+1`
instead of `x+y+2` after an overlapping rewrite set.
**How to apply:** Any proof that simp-normalizes shifted indices before `ring`/`linear_combination`,
especially when an index appears both as `t` and `t+1`.

### [2026-06-22] Clear integer-division-by-a-constant before `ring`
`ring`/`linear_combination` cannot reason about integer division (`Int.ediv`, e.g. `n*(n-1)/2`). For an
exponent/polynomial identity whose terms contain division by a fixed literal `k` (with the dividend always
divisible), prove `k * LHS = k * RHS` instead: rewrite each divided term via its banked doubled-form lemma
(`k * (x/k) = x'`, e.g. a `2*f = fTwice` lemma) so the goal is division-free, close with `ring`, then recover
the original by `mul_left_cancel₀ (by norm_num : (k:ℤ) ≠ 0)`.
**Why:** A polynomial exponent identity a CAS confirmed exact still failed `ring` purely because its terms
carried `/2` integer division.
**How to apply:** Any `ring`/`linear_combination` goal containing `_ / k` (literal `k`) — scale by `k` to
clear the division, prove the polynomial form, cancel the factor.

### [2026-06-22] Build a faithful model to DISCOVER the proof mechanism, not just verify truth
When a finite coefficient identity is numerically verified-true but resists assembly, mirror EVERY definition
in an exact-arithmetic script and probe it for the proof STRUCTURE — test candidate slicings (per-index,
telescoping, region/wall correspondences) and term-level index maps — rather than only re-confirming the
total. An off-by-a-constant discrepancy in a naive index map is a clue, not a failure: the constant pins the
CORRECT map (a sign or shift guessed wrong). The model exposes the actual bijection/cancellation the Lean
proof must encode, and turns a diffuse "irreducible" wall into ring/omega-fact foundation lemmas + one
structured core.
**Why:** A long campaign stalled isolating "one more lemma"; probing a faithful model revealed the real
mechanism, and an off-by-constant in the naive correspondence directly corrected the index shift.
**How to apply:** Any hard finite identity where you don't yet see the proof's combinatorial mechanism —
build the exact model, probe slicings + term maps, let the numbers (including constant offsets) dictate the
correspondence before writing Lean. Pairs with the stop-and-audit discipline.

### [2026-06-23] Trace the proof DAG for hidden circularity in "proven-modulo-X" reduction chains
When a goal is reduced through several theorems each "proven modulo the next residual", verify the chain is
not CIRCULAR before trusting "one sorry remains". Theorem A may be proved using B, and B (or a lemma it
calls) back through A — so the genuine content is an undischarged sorry hidden in the cycle and the "single
residual" is illusory (closing it would require itself). A docstring claiming "non-circular / breaks the
circularity" is a CLAIM, not a fact: confirm it by reading the actual proof's dependency on the residual.
**Why:** Several "discharge" theorems each cited the next; one's proof secretly routed back through the
shared identity, so multiple "proven" results all rested on one undischarged core that the comments
mislabeled as independently closed — wasted effort attacking the wrong (circular) form.
**How to apply:** Before reporting "reduces to one sorry", trace each `proven-modulo-X` link down to a
genuine base (axioms / proven leaves); if any citations form a cycle, the real residual is the cycle's true
independent identity, and only a proof that avoids every member of the cycle counts. Pairs with
verify-before-claim + `#print axioms`.

### [2026-06-23] Map ALL existing whole-goal reductions before building a NEW route
"Borrow the repo's library" applies at the ROUTE level, not just the lemma level. Before constructing a fresh
reduction of a major goal, `rg` for every theorem that concludes or reduces that goal and inspect each one's
open residual. A more-complete route often already exists that reduces the goal FURTHER than the new one you
are about to build (which would then need extra glue to connect back), and its residual is usually the true
single core. A redundant parallel route is wasted effort dressed as progress.
**Why:** Spent a large effort building a fresh multi-variable reduction route to a goal, then found the repo
already had a more-complete route reducing it to the very same single residual — the new route was redundant
and additionally needed specialization wiring the existing route did not.
**How to apply:** At the first thought of "I'll build a route to close X" — first `rg` the repo for theorems
mentioning X and its known intermediate forms, list their sorry residuals, and adopt/extend the most-reduced
existing route instead of starting a parallel one.

### [2026-06-23] A cancellation/uniqueness engine on a SCALED difference needs the scale invariant under the engine's transport
A "difference satisfies a homogeneous recurrence/operator ⇒ difference is 0" engine has a precondition: the
difference must satisfy the EXACT operator. Before applying it to `D = A − c·B`, check that the factor `c` is
INVARIANT under the operator's transport (if the operator shifts a variable, `c` must not depend on that
variable). If `c` transforms with a nontrivial quasi-period, `c·B` does NOT satisfy the same operator as `B`,
the engine does not apply, and the residual lemma "`c·B` satisfies the operator" you would state is FALSE —
an unfillable sorry that still builds green.
**Why:** Built a scaffold whose residual asserted a scaled object satisfied a homogeneous functional
equation; the scale factor was variable-dependent with a quasi-period that twisted the equation, making the
residual unprovable — caught only by computing the factor's transform under the step operator.
**How to apply:** Before invoking any zero-from-homogeneous-FE / cancellation engine on `A − c·B`, compute
how `c` transforms under the engine's step operator; apply only if `c` is invariant, else the scaled-FE
residual is false. Pairs with §3.3 (vacuous/false residuals that pass mechanical-green).

### [2026-06-23] Machine-extracted `linear_combination` cofactors must be INTEGER over a general ring
A CAS Gröbner / ideal-membership lift over ℚ returns cofactors that may carry rational denominators —
introduced whenever the reduction divides by a generator's leading coefficient (or eliminates a variable
via a relation with a non-unit leading term). Such fractional cofactors are INVALID for `linear_combination`
over a general `CommRing` (no division). Before transcribing, assert every coefficient is an integer; if not,
either re-extract with a tracked Buchberger / monomial order that keeps the relevant leading coefficients
units (often a single S-polynomial suffices to land an integer certificate), or multiply the whole identity
by the lcm of denominators and carry an explicit `(d : R) ≠ 0` hypothesis — the latter only when the ring
genuinely has that element invertible (e.g. a fixed prime in char-0 work).
**Why:** A lex Gröbner lift produced cofactors with a constant denominator (from a generator whose leading
coefficient was that constant); they verified the identity over ℚ but could not be encoded over a general
ring. A different lift order yielded integer cofactors needing no extra hypothesis.
**How to apply:** Any time you extract `linear_combination` cofactors from a CAS for a goal over a general
commutative ring — check integrality first; prefer an integer-producing lift over clearing denominators with
an invertibility hypothesis. Pairs with the CAS-verify-before-encode entry.

### [2026-06-23] Validate a large `linear_combination` MECHANISM before the final cofactors land
A big `linear_combination` proof carries two independent risks: (i) the surrounding mechanism — the `rw`
expansions, the `norm := simp only [...]` lemma set, index canonicalization, and `ring1` actually handling
a several-hundred-term certificate — and (ii) the exact cofactors. De-risk (i) FIRST with whatever verified
cofactors you already have (even a scaled, non-final, or hypothesis-carrying set), proving a throwaway
placeholder lemma; a clean compile proves the mechanism works at that term count, after which substituting
the final cofactors is a trivial swap. For certificates of that size, auto-generate the cofactor expressions
from the CAS objects (a small translator emitting Lean syntax) rather than hand-transcribing.
**Why:** The cofactor mechanism (definition unfolds + `norm` simp expansion of compound constants + `ring1`
over a large term count) was the real uncertainty; validating it once with a scaled placeholder set meant the
final clean cofactors compiled on the first try, and the translator removed transcription error from a
hundreds-of-terms expression.
**How to apply:** When a `linear_combination`/`ring` encoding is large and its cofactors arrive separately
(CAS, collaborator, a later round) — encode and compile a placeholder-cofactor version to lock the mechanism,
auto-generate the Lean from the CAS, then substitute the final cofactors.

### [2026-06-23] Unconditional `∀`-hypothesis with only an `_of_wf` discharge: try the subclass-unconditional route before declaring vacuity OR restructuring
A carried `∀ a b, P a b` hypothesis can look "too strong / probably vacuous" when the repo only proves the
underlying step-bound as `_of_wf` (needs full well-formedness). Before concluding the hypothesis is false, or
rebuilding the consumer to thread well-formedness through it, check whether `P` is unconditionally true for
STRUCTURAL reasons: the failure mode that forced `_of_wf` — typically a degenerate / error-track / saturation
branch — often cannot fire for the sub-population `P` actually constrains, and a subclass-specific lemma
(`_of_<role>` / `_of_<predicate>`) may already prove the needed brick UNCONDITIONALLY for that subclass.
Prefer the subclass lemma over the general `_of_wf`.
**Why:** Burned effort weighing "carried unconditional hyp is too strong (vacuity)" vs "restructure to thread
well-formedness" — when the hyp was simply unconditionally TRUE: the well-formedness in the general lemma only
served to rule out an error-jump that one structural class can never take, and subclass `…_le_succ_of_<role>`
lemmas already existed proving the per-step bound with no well-formedness needed.
**How to apply:** When a base bound you need unconditionally (or to discharge a carried `∀`-hyp) exists only
as `_of_wf`, `rg` for a subclass variant (`_of_<role>`, `_of_<predicate>`) and check whether the `_of_wf`
necessity is just an error/degenerate branch the relevant inputs avoid. This settles the unconditional-vs-vacuous
fork with no restructuring. Complement of the de-axiomatization hypothesis-audit (that hunts UNSATISFIABLE
carried hyps; this CONFIRMS a suspicious one is sound-unconditional).

### [2026-06-23] A carried hypothesis can BE the conclusion — signature-diff a "conditional result" against its OUTPUT type's fields
A conditional theorem (or a delegated producer's deliverable) can fake non-circularity by carrying, as a
hypothesis, the exact ∀-quantified bound that is a FIELD of its own output structure: assume the bound to
build the intermediate object that needs it, then re-package the same bound as the output's field. This
passes a clean `#print axioms` and the author reports "non-circular / genuinely derived." Catch it by a
mechanical signature diff, never by trusting the prose.
**Why:** On a fixed-point/bootstrap core, the cheapest fake is to carry the output's own domination/bound
field as an input; an axiom check cannot see it, and the author's self-assessment of non-circularity is
unreliable. (Distinct from DAG-tracing a multi-step reduction chain — this is a single-signature field check.)
**How to apply:** For any conditional deliverable, read the signature and ask: is any carried hypothesis
definitionally a field of the conclusion type — especially a ∀-over-the-full-domain bound matching the
output structure's own field? If yes ⇒ circular, reject. The honest form carries a strictly weaker LOCAL
input (a short-range / one-step extension) and emits the global statement as derived OUTPUT.

### [2026-06-23] Re-dispatch a rejected proof task with the EXACT faked anti-pattern named and forbidden
After rejecting a deliverable for a specific defect (carrying the conclusion, an over-strong/unsatisfiable
hypothesis, re-wrapping an already-landed result), never re-brief with a generic "do it correctly." Name the
precise mechanism that was faked, forbid it as a hard rule with the exact shape to avoid, and get an
independent route/design audit first so the correct structure is fixed before the author starts.
**Why:** An author that produced a specific fake once reproduces it unless the brief explicitly bans that
precise mechanism; a generic re-brief re-invites the same shortcut. Verified: a named anti-pattern rule in
the re-dispatch brief stopped the second attempt from repeating a circularity the first attempt committed.
**How to apply:** On re-dispatch after a rejection: (1) route-audit the correct structure, (2) write the
forbidden pattern verbatim into the brief ("X must be derived OUTPUT, never a carried hypothesis"), (3) on
return, scrutinize that exact point first before anything else.

### [2026-06-23] Anti-诈尸: before calling an obligation "open" — or re-proving it — grep the repo for an existing discharger
In a long-lived proof repo there are usually MANY parallel assembly attempts of the same headline
(`*_v2..v7`, `*_assembled`, `*_final`, `*_discharged`, `*_unconditional`, `*_reduced`, `*_seedclosed`,
`*_witness`, `*_of_<fact>`). An obligation you believe is open is often ALREADY discharged in a file you
never opened. Treating it as fresh work — re-proving it, or listing it in a "remaining scope" — is 诈尸
(resurrecting closed work) and burns whole sessions. NEVER state the open set from memory, a prior
conversation summary, a HANDOFF, or a DOCTRINE: those go stale and resurrect closed obligations. Recompute
the open set from CURRENT code every single time you're about to claim something is unfinished or start a proof.
**Why:** Repeatedly in one day I declared obligations "remaining/open" and began re-proving them when proven
`_discharged`/`_unconditional` theorems already existed, and parroted a stale summary's residual list as the
frontier. User, after the Nth time: "我反反复复叫你检查…你老是给我忘." The repo had ~50 headline variants; the
true status was only knowable by grepping + axiom-checking, not from any doc.
**How to apply:** Before claiming open OR writing one line of proof: `rg "theorem.*(<obligation>|_discharged|_unconditional|_reduced|witness|of_<fact>)"` and READ the hits. Verify with `#print axioms`, but mind the trap
— a CONDITIONAL theorem is `[propext, Classical.choice, Quot.sound]`-clean while STILL carrying the obligation
as a hypothesis; **axiom-clean ≠ supplied**. To know it's truly handled, find a SUPPLIER (a theorem whose
RETURN TYPE *is* the obligation / constructs it), not a CONSUMER (one that takes it as a hyp and is clean
trivially). And the standing order: the MOMENT you find it's already proven (诈尸 caught), WIRE the existing
theorem in — do NOT re-prove — THEN update UNDERSTANDING.md. (Companion to memories
`feedback_check_before_build_not_after`, `feedback_no_unmaintained_index`, `feedback_lake_env_vs_build_gate`.)

### [2026-06-23] Repeated correction of your OWN brief's premise on one sub-problem ⇒ escalate to an independent reasoner
When a delegated producer corrects not just its task verdict but a PREMISE in your brief (an asserted scaling factor, a claimed "this reduces to X", an equivalence between two objects), accept it — your brief is a hypothesis the producer tests. But when this happens TWICE+ on the SAME sub-problem, each fix exposing your framing was wrong, that is the signal you are past your reliable understanding of that specific mathematical point. STOP re-dispatching your own (likely still-wrong) framing; get an INDEPENDENT strong-reasoner second opinion (a repo-connected math LLM) on the route/design fork BEFORE the next attempt.
**Why:** On one hard analytic crux, two successive briefs each asserted a route premise (a smallness factor; an equivalence between two continuity objects) that the producer correctly refuted; a third self-framed dispatch would have repeated the error. The reliable move was an independent design audit, not another self-authored brief.
**How to apply:** Track brief-premise corrections per sub-problem. ≥2 ⇒ your intuition there is unreliable — consult an independent reasoner on route soundness before re-dispatching. Complements automode's two-way audit (which says distrust your brief); this gives the concrete escalation trigger.

### [2026-06-23] An UNSATISFIABLE carried continuity/regularity hyp ⇒ the MODEL is over-engineered — simplify the object, don't satisfy the hyp
A conditional chain can carry a continuity/regularity hypothesis that is FALSE for the actual object (continuity-on-a-closed-domain of a zero-extension that jumps at the boundary; joint continuity of a map with a built-in convention discontinuity). Such a hyp makes the headline VACUOUS. The trap is to then build MORE machinery to discharge the unsatisfiable hyp. Usually the hyp is unsatisfiable because the OBJECT/model demanding it is over-engineered; the fix is a SIMPLER object that never needs it — and often the awkward convention that breaks the hyp (a zero special-value at an endpoint) makes the simpler object's obligation TRIVIAL there.
**Why:** A function-space fixed-point model required a map continuous on a closed domain, but the map jumped at one endpoint (a kernel convention S(0)=0), so the continuity hyp was unsatisfiable and the whole chain vacuous. The fix was not to repair the continuity but to DROP the model: a direct per-point/per-slice domination needed no map-continuity, and the same endpoint convention made the endpoint case trivial (the value was 0 there).
**How to apply:** When you catch an unsatisfiable carried continuity/regularity hyp (§3.3), first ask "what weaker object lets me drop this hyp entirely?" before building anything to satisfy it. A function-space fixed point is often replaceable by a direct/inductive argument needing only pointwise/per-slice facts.

### [2026-06-23] Sorry-stub an expensive dependency to fast-iterate the code that USES it
When new code depends on a lemma whose proof is expensive to elaborate (a heavy `ring`/`decide`/`simp` or a giant `linear_combination` that takes many minutes), do NOT re-pay that cost on every edit while debugging the dependent code. Temporarily replace the expensive lemma's body with `sorry` (keep its EXACT signature), build the dependent code against the stub — it compiles in seconds — and fix the integration/name-resolution bugs there. Restore the real proof (ideally in its own file) once the dependent logic is green.
**Why:** Bugs in code layered on top of a slow-to-elaborate lemma are cheap to find when that lemma is stubbed; paying a 30–60 min build per fix-attempt destroys the debug loop, and the heavy proof tells you nothing about the downstream wiring.
**How to apply:** Heavy lemma + dependent code you're actively editing → stub the lemma as `sorry` (same signature), build, fix the downstream fast, then swap the real proof back. Complements "heavy proofs → separate file": the stub verifies the downstream cheaply; the separate file caches the real olean.

### [2026-06-23] `simp only [bridge_lemma]` BEFORE full simp, when norm_cast would push the cast and kill the match
A rewrite that bridges two forms of a function across a cast (e.g. `f (↑n) = g n` — the nat-vs-int versions) only fires while the cast is in the shape the bridge matches (`↑(expr)`). Full `simp`/`simpa` includes norm_cast-style lemmas that PUSH the cast inward first (`↑(a*b+c)` → `a*↑b + …`), after which the bridge no longer matches and you get "Type mismatch: after simplification, term h has type … but is expected to have type …" (one side in `f (↑compound)`, the other in the pushed-in form). Stage it: `simp only [bridge_lemma]` (plus the unfoldings you need) FIRST — `simp only` does not auto-push casts — then run the full `simpa`/`simp` to close.
**Why:** The bridge rewrite and norm_cast race; default simp lets the cast-push win, silently breaking a rewrite that would otherwise unify the two sides.
**How to apply:** `simpa [...] using h` fails with the hypothesis and goal differing only in cast placement (one has `↑(compound)`, the other the pushed-in version) → split into `simp only [bridge]` then `simpa [...]`.

### [2026-06-23] `linear_combination` "ring failed" — check the hypothesis ORIENTATION before the algebra
When `linear_combination c * h` fails with "ring expressions not equal", before suspecting the coefficient or the identity, check whether `h` came back from `simp` ORIENTED backwards (`0 = X` instead of `X = 0`). A flipped orientation flips the sign of the coefficient you need (`h` ↔ `-h` / `h.symm`), and the give-away is that the leftover term is exactly a doubling (`2·X`) of your target.
**Why:** `simp` normalizes an equality to a canonical side that may be the reverse of what your coefficient assumes; the failure reads like a math error but is just a sign, and re-deriving the whole identity to "fix" it wastes a long build.
**How to apply:** `linear_combination` ring-fails and the residual is `2·(your target)` → flip the coefficient sign (`h` → `-h`, or feed `h.symm`).

### [2026-06-23] A uniform ∀-index field can be UNSATISFIABLE on a parallel assembly that weakens the per-index postcondition — gate it to the sub-band that needs it
When you add a single uniform `∀ i, (work i).Post c → <strong fact about c>` field and plan to "supply it internally" from a per-index postcondition (e.g. an exact-value invariant), VERIFY that postcondition is actually delivered for EVERY index — not just the representative ones you checked. A second, parallel construction of the same record (a "strengthened" / "drained" / alternate assembly) often guarantees only a WEAKER postcondition for some indices, making the uniform field unsatisfiable there — and it still type-checks and is `#print axioms`-clean. If the strong fact is only NEEDED on a sub-band of indices, GATE the predicate as an implication `InBand i → StrongPred c`: off-band the complement is empty (discharge by `simp`), on-band it reduces to the strong supplier. Consumers that only USE the field on-band are unaffected, so no downstream signature changes.
**Why:** I supplied a uniform field from a per-index "exact" postcondition that the main assembly delivers for all indices, but a parallel strengthened assembly delivered only a weaker postcondition for a subset of indices — the field was genuinely unsatisfiable there. The real need for the strong fact was confined to a small sub-band, and the downstream producer already case-split on exactly that band and ignored the field off it (independent confirmation the gate is faithful).
**How to apply:** Before supplying any uniform ∀-index field internally, enumerate how EACH parallel constructor of the record establishes the per-index precondition; if any constructor weakens it for some indices, gate the field's predicate to the sub-band where it is both needed and available. Pairs with §3.3 (too-strong / unsatisfiable-frontier) and verify-before-claim.

### [2026-06-23] Check the BASE invariant isn't STRONGER than the headline needs — pointwise/uniform-sup objects are often self-inflicted
When a formalization's hardest seam is a BASE invariant (the object a bootstrap/ladder starts from), check whether it is strictly STRONGER than the headline theorem actually requires. A uniform-over-a-parameter COORDINATEWISE bound (sup over t of each mode/component, demanded to lie in a summable/Sobolev sequence space) is strictly stronger than the corresponding NORM bound (sup over t of the norm), because sup-of-sum ≠ sum-of-sup — a trajectory can stay in a bounded ball while visiting different components at different parameter values, so the coordinatewise sup can fail to lie in the space at all. Proving the stronger object manufactures hard seams the theorem never needed (and a per-component ladder then multiplies them).
**Why:** A whole architecture (a coordinatewise envelope + a per-component regularity ladder) ground for dozens of producer-rounds on a base seam that an independent strong-reasoner identified as strictly stronger than the needed uniform NORM bound; rebuilding on the standard norm/energy route — which proves exactly the headline object — converged far faster and was more faithful to the source paper. The norm route then needed only a PER-SLICE (fixed-parameter) regularity, not the uniform-sup envelope, and that per-slice fact was already landed.
**How to apply:** Before grinding a hard base invariant, ask "is this strictly stronger than the headline?" — especially any sup-over-a-parameter of a pointwise/per-mode quantity (vs the sup of the norm). If stronger, switch the main route to the norm/integral object and keep the coordinatewise/pointwise object as optional one-shot post-processing, never the base. Complements the unsatisfiable-hyp/over-engineered-model entry (that catches FALSE carries; this catches over-STRONG-but-true ones).

### [2026-06-23] Re-home a needed def/lemma OUT of an un-banked or discredited module — don't import the bad file for it
When the symbol you need lives in a module that is un-banked, vacuous, or otherwise discredited (its other
contents are unsatisfiable/wrong, or its compiled artifact was never produced), do NOT `import` that module to
reuse the one good piece — importing drags in the vacuity/staleness, and a missing build artifact hard-breaks
you. COPY the needed definition into a clean, verified file and make THAT the canonical home; downstream imports
the clean file.
**Why:** A needed structure definition lived in an un-banked scaffolding module whose siblings were a known-
vacuous residual; importing it failed (no olean) and would have re-introduced the vacuity. Copying the def into
a fresh verified file and re-pointing the proof there cleanly closed the producer (and let the bad module die).
**How to apply:** Before importing a module just to reuse one symbol, check the module's health (banked/built?
siblings sound?). If not, re-home the symbol into a clean file. Pairs with killed-object hygiene: re-home the
good, purge the bad.

### [2026-06-23] nlinarith times out on a high-degree goal — `ring`-expand the power FIRST, then it's hint-linear
When `nlinarith` hits a `(deterministic) timeout at whnf/isDefEq` (heartbeats) on a goal containing a
power of a sum (`(1 + M/3)^3 ≥ c·M`, `(1+x)^k`), the cost is `nlinarith` internally expanding the
polynomial. Pre-expand it yourself: `have hexp : (1 + M/3)^3 = 1 + M + M^2/3 + M^3/27 := by ring;
rw [hexp] at h`, then feed `nlinarith` the monomial bounds you already have (`M^2 ≥ B`, `B·M ≤ M^3`) so
the remaining goal is LINEAR in those hints. Separately, a genuinely long single proof (big calc + several
nlinarith) can exceed the 200k default — wrap it with `set_option maxHeartbeats N in` (placed BEFORE the
docstring, not between docstring and theorem).
**Why:** A cubic-vs-linear inequality timed out repeatedly; expanding the cube with `ring` and supplying
the `M^3 ≥ …` bound as a hint made `nlinarith` close instantly, and the whole long lemma needed a heartbeat
bump to elaborate at all.
**How to apply:** `nlinarith`/`polyrith` heartbeat-timeout on a goal with `(expr)^n` → `ring`-expand the
power to monomials first and pass monomial-degree bounds as hints; reserve `maxHeartbeats` bumps for proofs
that are simply long, not for masking a tactic that should be restructured.

### [2026-06-23] A lemma that only PROPAGATES a bound should be parametric in the bound — generalize once, don't edit every layer
When a constant (an exponent, an ε, a budget) is a pure pass-through threaded unchanged through a chain of
propagation lemmas (a union bound `∑ ≤ n·max`, a monotone transport, a telescoping wrapper), do NOT
in-place-edit or duplicate the constant at every layer to retarget it. Find the lemma whose proof merely
COPIES the bound (it never uses the value) and generalize it to a parameter (`{B : …}`); then every caller
passes whatever value it needs. One edit replaces a multi-layer constant-threading slog, and the lemma
becomes reusable at all values.
**Why:** Retargeting an exponent through a reset-overshoot chain, the bottom union-bound lemma hardcoded the
specific value in its hypothesis AND conclusion but its proof only summed copies of it; abstracting it to a
generic bound parameter let both the old and new values flow through with zero further edits.
**How to apply:** Before editing a constant across N layers, check each layer's proof — any layer that
doesn't USE the value (just carries it) should be parametric. Generalize that layer to a parameter rather
than threading the new literal through all N. Pairs with "map existing routes" and the no-duplication reflex.

### [2026-06-23] Push the slack to the stage that's actually hard — pick the intermediate constant that makes the resisting step TRIVIAL
In a multi-stage estimate (numeric core → propagation → final fit) you often have freedom in an intermediate
constant (a target exponent, a chosen rate). Different choices shift WHERE the difficulty lands. Identify the
stage that genuinely resists automation (a `nlinarith` needing a sub-linear/log bound, a missing Mathlib
lemma) and choose the constant that makes THAT stage trivial — even if it costs a slightly fancier bound in a
stage that's already mechanical (e.g. a cube instead of a square absorption). Optimize for "no hard residual
anywhere", not for the tightest single constant.
**Why:** One target made the numeric core clean but forced the final fit into a hard log-of-n bound; bumping
the target constant made the fit a one-line `nlinarith` at the cost of a marginally harder (but still
mechanical) polynomial absorption upstream — net far less work and no fragile step.
**How to apply:** When an intermediate constant is free and one downstream step won't close, recompute what
constant would make that step trivial and propagate it, rather than grinding the hard step at the original
constant. The binding constraint is "which step has no clean proof", not "which constant is tightest".

### [2026-06-23] Lean module-system migration: public / @[expose] / no-private-in-public
To make a non-`module` file importable by a `module` file (Lean's module system), the ENTIRE import closure must become modules — `module`/non-`module` cannot mix in EITHER direction. Per-file recipe: add `module` at the top; change `import X` → `public import X`; mark every cross-file-used declaration `public` (in a module, non-`public` decls are file-private and invisible downstream); add `@[expose]` to public `def`/`abbrev` whose BODIES are needed downstream for `rfl`/`simp` unfolding (a module exports signatures, not bodies); and PROMOTE any `private def`/`abbrev` that appears in a PUBLIC declaration's STATEMENT to public (a public decl cannot reference a private one). Adjacent attribute blocks must be merged — `@[expose] @[simp]` is a parse error, write `@[expose, simp]`.
**Why:** A module file rejected a non-module import outright ("cannot import non-`module` X from `module`"); migrating then surfaced, in order, hidden non-public exports, def-bodies not unfolding across modules, and private names leaking into public signatures — each a distinct module-visibility rule.
**How to apply:** Bringing legacy/scratch Lean into a module-system project → migrate the whole closure with a scripted transform (the recipe above), build bottom-up with targeted per-file builds, fix the residual public/@[expose]/private-leak cases build-guided, and build any expensive file in the background while debugging the cheap ones.

### [2026-06-23] Escape an import-layer inversion by RELOCATING the goal above its proof materials
When the lemmas needed to prove a goal live ABOVE it in the import DAG (the goal's file cannot import them without a cycle), don't try to thread the proof downward — RELOCATE the goal AND everything downstream of it into a NEW file that sits above both layers, importing the goal's original file AND the proof-material files; then truncate the goal out of its original location. Re-thread any hypotheses the relocated downstream now needs through the moved theorems.
**Why:** An obligation sat at the BOTTOM of the import graph while its discharge materials (wiring lemmas + an independent non-circular fact) sat above it; proving it in place was an import cycle. Moving it plus its consumer chain into a new top file made the discharge both legal and non-circular.
**How to apply:** A sorry/lemma can't reach its proof inputs without an import cycle → create a top file importing both layers, relocate the lemma + its dependents there, truncate the origin, and fix the newly-needed hypothesis threading build-guided (partial-application errors at `.1`/`cases` sites pinpoint each call).

### [2026-06-23] Harvesting LLM-generated Lean: classify clean-vs-sorried per decl, land only sorry-free; expect tactic-SCOPE bugs
A large block of collaborator-generated Lean (an LLM "git-drop" / answer) is ~80% correct but never just
compiles. Two disciplines: (1) PROGRAMMATICALLY classify each decl as sorry-free vs sorried before landing
anything — split on `theorem`/`def` boundaries, grep each block for `sorry` — and commit ONLY the sorry-free
prefix as additive infrastructure (a committed file cannot carry sorry; the sorried structural/hard pieces
stay out until separately closed). (2) Verify-build-fix the clean pieces expecting a small set of predictable
failure classes: tactic SCOPE — `unfold`/`rw`/`simp` applied to the goal but NOT to a hypothesis it must also
rewrite (fix: `unfold … at h ⊢`); and unbridged definitional steps — double negation (`simpa only [not_not]`
when a `¬G` set is really `{¬¬Danger}`), un-unfolded `let`, and coercion mismatches (`.val` vs `↑`).
**Why:** A multi-decl LLM Lean drop had exactly these: one proof used `unfold … ` (goal only) then failed to
`rw` a hypothesis still carrying the folded definition; another produced a `{¬G}` set that needed `not_not`
to become the target `{Danger}`. Classifying first meant the sorry-free skeleton landed green while the
sorried hard cores were cleanly deferred — no broken commit, no wasted verification on the unfinished parts.
**How to apply:** Receiving any sizable Lean from a collaborator/LLM — scan decls for `sorry` first, land the
clean prefix additively, and on each verify-build failure suspect tactic scope (`at h ⊢`) and a missing
definitional bridge (`not_not`/`let`-unfold/coe) before suspecting the math. Pairs with verify-don't-transcribe.

### [2026-06-24] Bridge an instance-coherence diamond with convert + congr + Subsingleton.elim — don't change file-wide instances
When wiring a proof across a boundary where the two sides made DIFFERENT instance choices for the same typeclass (e.g. one side built with `open scoped Classical` → `Classical.propDecidable`, the other with an ambient `[DecidableEq k]`), the structures derived from that instance (the group, and the `•`/`HSMul` built from it) are different TERMS and won't unify — `exact borrowedLemma` fails with a type mismatch whose two sides differ ONLY in the instance argument. Bridge it LOCALLY: `convert borrowedLemma using N` to peel the structure toward the differing instance, then `congr` to expose the bare instance leaf, then `exact Subsingleton.elim _ _` (a `Decidable`/`DecidableEq` instance is a `Subsingleton`, so any two are propositionally equal). Do NOT instead change the file-wide instance to align them (dropping `[DecidableEq k]` for Classical, or adding `[CharZero k]`): that cascades — it breaks downstream theorems' instance synthesis, or triggers a `whnf` elaboration blowup in some dependent theorem.
**Why:** A cross-module wire failed because one side's additive-group instance used Classical decidability and the other used an explicit `[DecidableEq k]`; `convert + congr + Subsingleton.elim` closed it in three lines, whereas every file-wide instance-alignment attempt cascaded (synthesis failures elsewhere, or a `whnf` heartbeat blowup in a downstream theorem).
**How to apply:** `exact`/`rw` of a borrowed lemma fails with a type mismatch whose two sides differ only in a typeclass instance (decidability, or a group/monoid derived from it) → `convert ... using N; congr; exact Subsingleton.elim _ _`. Reach for local bridging before touching the file's `variable` instances.

### [2026-06-23] An off-by-a-CONSTANT in a "should-close" reindex means you're on a PRE-TRANSFORMED object
When a reindex/involution/completion that is provably TRUE refuses to close and the mismatch is a FIXED constant
(off by k, not off by a variable), the cause is usually that the object you are transforming is ALREADY a
folded/reindexed form of the raw data — and that earlier transform shifted the parameter by k. Back up to the
raw/unfolded definition (the members before the fold), where the completion lands exactly.
**Why:** A symmetry-orbit completion on a two-sided/folded object was off by a fixed constant; the fold had
already reindexed one half into the other with a parameter shift, so the completion lemma's index never matched.
Doing the step on the unfolded (raw-member) form made the same completion exact.
**How to apply:** On a constant-offset failure in a reindex/completion, don't hunt for a sign/scale fudge —
check whether the object is a derived/folded form (`unfold` it, or trace which earlier lemma produced it) and
redo the step on the primitive members. Constant offset means wrong (pre-transformed) representative; variable
offset means genuine index-algebra bug.

### [2026-06-24] "Reaching state X triggers setup Y" — verify the TRANSITION PATH invokes Y, not just that X's init does Y
When reasoning about whether reaching a state/phase X causes a side-effect Y (a field reset, a counter refresh, a flag set) that lives in an init/enter routine, do NOT conclude it from reading the init routine alone. A state can be entered by MULTIPLE transition paths, and some advance into X WITHOUT invoking its init — e.g. a bare "increment/advance" dispatch that bypasses the "advance-WITH-init" dispatcher. The init routine in isolation tells you what WOULD happen if invoked, not what a given path actually does. Read the ACTUAL transition that reaches X on the path in question and confirm IT calls the init.
**Why:** I read that an enter-routine resets a counter at state p and concluded a sub-goal was "easy" because the entrant arrives fresh — FALSE: the relevant transition advanced into p via a bare increment that never ran p's init, so the old (possibly degenerate) value was carried in. The reset existed; that path skipped it. An independent reasoner flagged it and reading the transition function confirmed the bypass — a verified-then-overturned claim.
**How to apply:** Any "reaching X establishes invariant/effect Y" step where Y is produced by an init/enter/setup routine → grep EVERY transition that can land in X and confirm it invokes that routine (vs a raw advance that skips it) before relying on Y. The same trap inflates "this entry is satisfiable/fresh" claims. Pairs with verify-before-claim and the unsatisfiable-carried-hyp audits (§3.3).

### [2026-06-24] A `whnf`/`isDefEq` heartbeat timeout on a heavy definition is NOT a maxHeartbeats problem — block the unfolding
When elaboration dies at `(deterministic) timeout at whnf` / `isDefEq`, raising `maxHeartbeats` (even 10×) usually does NOT help: the cost is the elaborator reducing a deeply-nested definition to normal form during unification — structural, not budgetary. Two independent attempts timing out at the SAME line regardless of budget is the fingerprint. Fix by PREVENTING the unfolding: `set x := <heavy term> with hx` (or `generalize`) so the heavy object is an opaque local before the offending tactic; `attribute [local irreducible] <heavyDef>` to forbid unfolding the named culprit; and `show <pinned goal type>` so elaboration matches syntactically instead of falling into a defeq search. Feed any engine/lemma the heavy object's needed properties (HasDerivAt, continuity, a bound) as already-proven hypotheses ABOUT the opaque variable, never the unfolded form.
**Why:** Two separate attempts to differentiate a coefficient of a composite over a deeply-nested algebra both timed out at `whnf`/`isDefEq` on the identical line even at 1,000,000 heartbeats — bumping the budget was the wrong knob; the elaborator kept re-unfolding the nested structure during unification.
**How to apply:** On any `timeout at whnf`/`isDefEq`, read which exact def the message names, opaque-ify it (`set`/`generalize`), mark it `local irreducible`, and `show` the goal type to block the defeq search — BEFORE (or instead of) raising maxHeartbeats. Two independent attempts timing out at the same line at a huge budget = structural; stop bumping the number.

### [2026-06-24] A carried "X satisfies identity E" hypothesis is VACUOUS if E's operator silently drops a term the real X has
A conditional theorem can pass mechanical-green + clean `#print axioms` yet be VACUOUS — not via an obviously-strong hypothesis, but because a carried *identity* hypothesis (`X = SomeOperator(X)`: a Duhamel / fixed-point / recurrence identity) uses an operator that is a SIMPLIFIED/REDUCED version of the one X actually satisfies, missing a term. The real solution (from its actual fixed-point map / construction) realizes the FULLER operator; the simplified-operator identity then forces the dropped term's coefficient to vanish — false for the live regime — so the hypothesis is unsatisfiable and the theorem vacuous. Nothing ever exhibits a witness, so every mechanical check passes.
**Why:** A headline carried `u = D(u)` where `D` was a strict sub-operator (one nonlinear term omitted) of the operator the solution's own fixed-point construction used; the two coincide only when the omitted term's coefficient is 0, so for the live (nonzero) parameter regime the hypothesis was unsatisfiable — vacuous despite a clean axiom print and project docstrings asserting "faithful". Caught only by unfolding the operator's definition and comparing it term-by-term against the fixed point the solution actually comes from.
**How to apply:** For any conditional carrying an "X satisfies equation/identity E" hypothesis, unfold E's OPERATOR definition and compare it term-by-term against what X's real source realizes — its fixed-point map, its construction, or the conclusion's own stated PDE/recurrence (which often carries the full operator as a field). If E's operator is missing a term present in the real object, the identity is unsatisfiable for the live regime ⇒ vacuous. The fix is usually statement-level: use the correct fuller operator, or route through an engine that consumes the genuine content (the regularity/PDE) directly and never demands the wrong identity. Dropped-operator-term variant of the assumed-conclusion + unsatisfiable-composite vacuity entries.

### [2026-06-24] Build on a disk-full host in tmpfs + pull dependencies from the package cache, not a cold recompile
When the canonical build host's disk is too full to hold a fresh build tree (toolchain + dependency artifacts + your own build output), don't abandon that host — build inside a large tmpfs (`/dev/shm`). Recipe: `rsync` your SOURCE only (exclude the build dir + VCS metadata) into a tmpfs work dir; pull the heavy dependency objects from the package manager's prebuilt cache (`lake exe cache get` for Mathlib) instead of recompiling them; build your own modules there. The dependency cache turns a multi-hour cold dep build into a ~15s download, and tmpfs gives space without touching the constrained root disk. The tmpfs copy is non-persistent — re-rsync after a host reboot.
**Why:** A build host's root disk was ~92% full (GBs short of a dependency tree) while its tmpfs had hundreds of GB free; rsync-source-to-tmpfs + cache-get made the build loop work where a root-disk build would have failed on space, and skipped recompiling an unchanged multi-GB dependency.
**How to apply:** Remote/build host disk-constrained → build in tmpfs, rsync source-only, fetch dependencies from the ecosystem's prebuilt cache, compile only your modules; re-sync on reboot. Complements the nohup/remote-build + heavy-proof-in-separate-file discipline.

### [2026-06-24] Identities over a square-zero / dual-number extension: convert the `MulOpposite.op _ • _` from `snd_mul` before `ring`
Proving an equation over a dual-number / square-zero extension (`TrivSqZeroExt R M`, `DualNumber R`) by splitting into first (`fst`) and second (`snd`) components: the `snd` of a product expands (via the `snd_mul` lemma) to a RIGHT-module action `MulOpposite.op a • b`, NOT an ordinary multiplication — so `ring`/`linear_combination` cannot normalize it and reports "ring failed". Rewrite the smul to a product first (`op_smul_eq_mul : MulOpposite.op a • b = b * a`, a `rfl` lemma) plus `smul_zero` for the `op _ • 0` cross-terms, then the closer succeeds. Also reduce the nat-subtraction exponents (`n-1`, via `Nat.add_sub_cancel`) the component split leaves behind.
**Why:** A from-scratch power/eval lemma over a nilpotent extension stalled with "ring failed" on a goal containing `MulOpposite.op x • (…)`; the second component's multiplication is a bimodule right-action, opaque to `ring`, until converted to a plain product.
**How to apply:** Computing in a square-zero / dual-number extension via an `fst`/`snd` (`TrivSqZeroExt.ext`) split → put `op_smul_eq_mul, smul_zero` in the `simp only` set before the closing `ring`/`linear_combination`, and reduce any `n-1` exponents. Distinct from the instance-coherence-diamond entry: that aligns typeclass instances; this converts a right-module smul that the ring normalizer cannot see through.

### [2026-06-24] Wire the downstream chain FROM the open frontier sorry, then axiom-check it propagates
When a proof reduces to ONE open lemma X, don't wait to close X before building the rest. Prove every
downstream consumer up to the headline FROM X's STATEMENT — each compiles and carries `sorryAx` transitively.
Then `#print axioms headline` must show ONLY {core axioms} + `sorryAx` (nothing else: no circular/forbidden
axiom). A green chain + that clean axiom set is a PROOF that X is the sole remaining gap and that closing X
auto-flips the whole headline clean — zero further wiring. It also catches a hidden SECOND bad dependency
early: an extra axiom in the set means the chain is not actually one-lemma-away.
**Why:** A headline was reduced to one hard frontier lemma; proving the entire chain to the top statement from
that lemma's statement + `#print axioms` (showing sorryAx-via-the-frontier-only, no circular axiom) proved the
frontier is the only gap and that closing it propagates — verified and committed while the frontier stayed open.
**How to apply:** Reduced to one open lemma X? Build X's consumers (the wiring to the headline) from X's
statement now; require `#print axioms headline` = core ∪ {sorryAx}, nothing more. Constructive forward-wiring
with an axiom-verified propagation guarantee — the complement to circularity-DETECTION (finds hidden cycles);
this PROVES forward that a single open lemma is the whole remaining distance.

### [2026-06-24] A continuous-time tracking/tube invariant is true only on the window where the flow CONTRACTS — and if the target is rewritten each cycle, index the NEXT target; sample-window and target co-move
A "the state stays within ε of its target" invariant for a relaxation/driven flow (`y' = k·(target − y) + …`) is NOT true uniformly in time. It holds only on the sub-window where the effective rate makes the flow CONTRACT toward the target; on the complementary phase the same envelope GROWS (budget-limited), so asserting the tube there is FALSE — yet the false interface still type-checks and is `#print axioms`-clean. Compounding it: when the system OVERWRITES the target each cycle (it writes a new value into the tracked coordinate), an invariant sampled in a given window must index the target the flow is currently relaxing TOWARD in that window — often the NEXT cycle's value, not the current one. The sample-window and the indexed-target CO-MOVE; pinning either to the wrong choice makes a provably-false producer that no end-to-end check catches until you exhibit the dynamics.
**Why:** A box-free tracking interface asserted current-target tubes over the full read band; on that band the driving envelope was actually growing (budget-limited), and the coordinate was being rewritten to the next cycle's value mid-band — so BOTH the window (should be the strong-contraction sub-window) and the target index (should be next-cycle) were wrong, making the carried producers unsatisfiable. The repair was not to strengthen a bound but to RE-LOCATE the interface: sample on the contracting sub-window and index the next target. Mechanical-green hid it; only writing out the flow's sign-of-rate per phase + which target it relaxes to exposed the falsity.
**How to apply:** For any time-indexed tracking/tube field over a driven or relaxation flow, before banking it write down (1) the SIGN of the effective rate across each phase of the window — the invariant can only live where it contracts; (2) WHICH target the coordinate is relaxing toward in that window — if the system rewrites the coordinate per cycle, the right target may be the next cycle's value. Co-move sample-window and target together to the physically-true sampling point; don't try to satisfy a bound asserted on a phase where the flow diverges or against a stale target. Continuous-time sibling of the sub-band-gating (discrete ∀-index) and pre-transformed-object (constant-offset reindex) vacuity entries.

### [2026-06-24] Large starting error in a relaxation/Grönwall tracking bound: use the FULL Duhamel form (mass contracts the start), not the simplified "hold" corollary (which demands start ≤ radius)
A relaxation/driven flow `y' = k·(target − y)` typically ships TWO bound lemmas: the full Duhamel/Grönwall form `|y(b)−M| ≤ exp(−∫_a^b k)·|y(a)−M| + δ·(1−exp(−∫_a^b k))`, and a convenient "hold" corollary `|y(a)−M| ≤ δ ∧ (target within δ) ⟹ |y(b)−M| ≤ δ`. The hold corollary REQUIRES the starting gap `|y(a)−M|` to already be within the target radius `δ`. When the tracked coordinate is being freshly written/copied, its start gap is the full O(1) swing (`≫ δ`) — so the hold corollary's hypothesis is UNSATISFIABLE and reaching for it produces a vacuous/unprovable obligation. The correct tool is the FULL form: keep the start gap `B` as a large free constant and let the accumulated rate-mass `Λ ≤ ∫_a^b k` contract it (`exp(−Λ)·B → 0` as the mass grows), giving `ρ = exp(−Λ)·B + δ`. Corollary: to lift a single-endpoint mass bound `Λ ≤ ∫_a^m k` to an interval bound for all `t ≥ m`, use mass monotonicity `∫_a^t k = ∫_a^m k + ∫_m^t k ≥ ∫_a^m k ≥ Λ` (the tail is ≥0 since `k ≥ 0`) + `exp` antitone — no per-`t` re-derivation.
**Why:** A write/copy tracking fact (coordinate just overwritten, so its start gap was the full O(1) swing) could NOT be discharged by the "hold" corollary — that corollary needs start ≤ target-radius, which the fresh write violates. Switching to the full Duhamel form, carrying the large start gap as a free `B` and bounding it by `exp(−Λ)·B` via the gate-mass lower bound, made it provable; the interval version then fell out of mass-monotonicity rather than re-applying the engine at every point.
**How to apply:** When a tracking/tube obligation is over a coordinate that was just written/copied/reset (large start error), do NOT use the "hold"/"stays-in-tube" corollary — its `start ≤ radius` premise is unsatisfiable. Use the engine's full `exp(−∫rate)·start + δ·(1−exp)` form with the start as a free constant `B`, and supply a rate-MASS LOWER bound `Λ` so `exp(−Λ)·B` does the contracting. Lift endpoint→interval via integral-mass monotonicity (nonneg-rate tail), not pointwise re-derivation.

### [2026-06-24] Lower-bound an integral via an interior sub-interval + nonneg extension — when the integrand's favorable pointwise bound degrades near the endpoints
To prove `Λ ≤ ∫_a^b f` for a non-negative integrand `f` whose useful pointwise LOWER bound holds only on a strict INTERIOR sub-interval `[a',b'] ⊂ [a,b]` (the estimate degrades toward `a` and `b` — e.g. a periodic/oscillatory factor swings to its unfavorable extreme near the endpoints, so a whole-window pointwise bound is FALSE), do not fight to bound `f` on all of `[a,b]`. Instead: (1) prove the pointwise bound on `[a',b']`; (2) `intervalIntegral.integral_mono_on` against the constant to get `Λ ≤ ∫_{a'}^{b'} f`; (3) extend for FREE by non-negativity — `∫_{a'}^{b'} f ≤ ∫_a^b f` since the two flanking pieces `∫_a^{a'} f, ∫_{b'}^b f ≥ 0` (`integral_add_adjacent_intervals` ×2 + `integral_nonneg`). The sub-interval only needs to be wide enough that `Λ` (which grows via the integrand's other factor) still diverges.
**Why:** A gate-mass integral had to be lower-bounded to make a contraction term vanish, but the gate's pointwise lower bound was only valid on an interior sub-window (near the window edges the oscillatory factor hit the opposite extreme, making any whole-window bound false). An independent re-derivation flagged exactly the same whole-window failure. Restricting the pointwise bound to the interior sub-window and nonneg-extending to the full window made it provable and is strictly cleaner than chasing a global bound.
**How to apply:** Lower-bounding `∫ f` where `f ≥ 0` but the good pointwise estimate is only interior → restrict to the sub-interval where the estimate holds, `integral_mono_on` there, then nonneg-extend (`integral_add_adjacent_intervals` + `integral_nonneg` on the flanks). Mirror of the standard UPPER-bound move (extend the domain to make `∫` bigger); for lower bounds you SHRINK the domain. Pairs with the contraction-window and full-Duhamel-form entries.

### [2026-06-24] Reconstruct the univariate slice EXPLICITLY from coefficients, don't fight the multivariate eval/map API
When you need a fact about a multivariate or structured polynomial restricted to one variable (a bivariate `p(x, Y)` viewed as a poly in `Y`, an object in `R[X][Y]`, etc.), do NOT thread the heavyweight machinery (`map (evalRingHom x)`, the `evalEval`/`map`/degree-preservation lemmas, bivariate `Y` notation). Instead rebuild the univariate slice you actually need EXPLICITLY from its coefficients — using the library's explicit-evaluation lemma (the one giving `evalEval x Y = <closed coefficient expression>`) — as a plain `K[X]` polynomial, reason about THAT (degree, root existence over an alg-closed field, squarefreeness), and bridge back via the same explicit-eval lemma with `linear_combination`/`ring`.
**Why:** Proving root-existence + a derived nonvanishing for a bivariate curve equation, the `map`/`evalEval` route was a tangle of degree-of-mapped-poly and eval-commutation lemmas; constructing the explicit univariate quadratic from its coefficients and applying the alg-closed root-existence lemma to it closed everything mechanically.
**How to apply:** Need degree / a root / squarefreeness of a multivariate poly's univariate slice → find the library lemma giving the slice's explicit coefficient form, rebuild it as an honest univariate polynomial, do the univariate reasoning, bridge with the explicit-eval lemma. Avoid `map (evalRingHom …)` + bivariate-notation gymnastics unless a clean `evalEval = eval ∘ map` lemma already exists. (Complements CAS-verify: there you verify the identity externally; here you pick the lighter Lean representation of the object.)

### [2026-06-24] `#print axioms` clean is NECESSARY, not SUFFICIENT, for "unconditional" — it is SILENT on carried hypotheses
A theorem whose `#print axioms` shows only `[propext, Classical.choice, Quot.sound]` is verified free of `sorry`/`native_decide`/custom-axiom — but that says NOTHING about whether it is UNCONDITIONAL. A conditional theorem `(h₁ : P₁) … (hₙ : Pₙ) : C` is fully axiom-clean while carrying n hypotheses; banking it as "discharged" / "fully proved" / "the residual is closed" conflates two independent properties. `#print axioms` audits the PROOF's foundations; it cannot see the theorem's own binder list.
**Why:** Repeatedly banked "full discharges" of carried obligations on the strength of an axiom-clean `#print axioms`, then a fresh signature-read revealed the supposedly-discharged lemmas still carried substantial hypotheses cascading to an open frontier — three such over-claims in one session, each from treating axiom-clean as unconditional.
**How to apply:** Before reporting any "discharged / fully closed / unconditional" claim, read the theorem's BINDER LIST, not its `#print axioms`. Enumerate every carried `(h : …)`; for each, decide it's a satisfiable standing input (⇒ faithful conditional) or it must be discharged separately. `#print axioms` is one necessary gate (no sorry/custom-axiom); the binder audit is the SEPARATE, required gate for unconditionality. Pairs with the §3.3 faithful-conditional discipline and the dropped-operator/datum-class vacuity entries.

### [2026-06-24] A "hard floor / needs new machinery" obligation: REDUCE-and-bank iteratively, don't attempt to close it
When a sub-obligation is labelled "the genuine hard core / needs new machinery / unformalized / coupling-hard", treat the LABEL as a claim and attack it by ITERATIVE REDUCTION, not a from-scratch proof. Each round: (1) verify-existing FIRST — grep for an already-proven producer of this EXACT shape; (2) land ONE axiom-clean lemma reducing the obligation to a strictly SMALLER named residual (typical shapes: "X over a window" → "X at the entry, step-preserved" → "an assembly of EXISTING quantitative apparatus"); (3) re-aim at the residual. Bank each verified reduction; the "floor" recedes. Deep floors repeatedly dissolve into an already-proven-but-unconsumed result (redundant-producer pattern) or an assembly of existing apparatus — NOT new mathematics.
**Why:** A residual twice-labelled "core-hard / coupling-hard / unformalized" collapsed over three reduce-and-verify rounds: first to an already-proven regime lemma (a near-诈尸 caught only by verify-existing), then to a step-preserved entry predicate, then to an assembly of existing Lyapunov/contraction apparatus — never needing a new proof; each round landed an axiom-clean reduction. Attempting to CLOSE the floor directly (or briefing a delegate to "prove X") would have re-derived machinery that already existed.
**How to apply:** Faced with a deep open fact, do NOT brief yourself or a delegate to "prove X." Brief: "verify-existing FIRST (grep producers of X's shape), then reduce X by ONE verified layer and name the smaller residual; report whether it's CLOSED, REDUCED (name the residual + which existing apparatus it parallels), or genuinely needs new machinery." Re-dispatch on the residual. Pairs with anti-诈尸, "your difficulty verdict is a claim — test it", and "borrow the repo's own library".

### [2026-06-24] A side-condition that's FALSE at the stated generality must be THREADED from the call site — and re-routing to a "proven general" lemma won't drop it
A leftover side-hypothesis `sorry` (characteristic ≠ p, a nonvanishing, a positivity) often can't be closed because the surrounding declaration is stated at a generality where the condition is genuinely FALSE (e.g. it fails in small characteristic). Two consequences: (1) you cannot prove it in place — thread it as a typeclass/argument from the call site where generality narrows, and discharge it trivially there; (2) re-routing to an existing "proven over general k" version does NOT eliminate it unless that target's OWN binder list lacks the same hypothesis.
**Why:** Spent effort hypothesizing a side-condition sorry could be dropped by routing to an already-proven general version; signature-diffing the target showed it carried the identical hypotheses, so the route removed nothing — and the condition was simply false at the stated generality, so it had to come from the narrower instantiation.
**How to apply:** When a side-condition sorry resists, first ask "is this even TRUE at this signature's generality?" If no → thread it from where the type narrows (the concrete instantiation supplies it for free). Before banking "re-route to the proven lemma drops it," diff that lemma's binder list — a route only removes a hypothesis the target doesn't itself demand. Complements "your wiring verdict is a claim — test it" and the binder-audit / faithful-conditional entries.

### [2026-06-24] Before a multi-brick CAMPAIGN on a "hard remaining piece", grep the conclusion's head symbol repo-wide — the bottleneck is often already proved (trivially)
"Borrow the repo's library" (per-lemma) and "map existing routes" (per-route) escalate to the CAMPAIGN level:
before launching a multi-brick effort to build the "hard remaining analytic core", grep the WHOLE repo for the
conclusion's head symbol (the membership/regularity predicate applied to the specific object). The bottleneck
is frequently ALREADY landed unconditionally — and often by a route far cheaper than the deep machinery you
were about to build (e.g. a regularity⟹membership one-liner: "this composite is C^k, and C^k+boundary ⟹ the
weighted-ℓ² membership at that order" dissolves a presumed-hard "general-data composition" into nothing). An
inherited framing — a prior session's note, a stale docstring/board, your own earlier summary — that says "X is
THE bottleneck" is a CLAIM, not a fact; the difficulty itself must be verified, not assumed.
**Why:** A whole campaign (small-data series + planned heavy estimate + collaborator rounds + several dispatched
bricks) chased a composition "bottleneck" that the repo already proved unconditionally via a trivial regularity
route; the cited "open residual" docstring was stale (code had moved past it hours earlier). Three-to-four
over-builds, all preventable by one repo-wide grep of the object before the first brick.
**How to apply:** At the FIRST thought of "I'll build the hard piece X" — grep the conclusion object across the
whole repo (head symbol + the specific argument), read any hit's actual statement (unconditional? what σ/order?),
and treat every "this is the bottleneck" assertion as a claim to falsify by code, never a premise. The cheapest
existing route also tells you the result was never hard. Pairs with #1.3 (don't trust docs), §3.3, and the
per-lemma/per-route borrow rules — this is the same reflex one level up, at the campaign premise.

### [2026-06-24] Before building wiring around a "newly isolated" sorry, check if it's an EXISTING sorry restated
When you isolate what looks like a new frontier residual X and build an elaborate reduction/wiring chain around
it, first check whether X is algebraically equivalent to a sorry Y that ALREADY EXISTS elsewhere in the repo
(via a simple `ring`/`linear_combination`/`congr` step). If X = Y, then X was never a separate gap — the
elaborate isolation work was engineering (valuable for verifying the propagation chain) but doesn't add a new
mathematical target. The resolution is to close Y (which may have different surrounding scaffolding better
suited to the proof), or to recognize that the two sorries are the same one.
**Why:** A "new frontier sorry" was isolated with several commits of non-circular wiring built around it; a
fresh-context agent then closed it in one line by showing it equals an existing sorry via a trivial algebraic
rearrangement. The wiring was real engineering (it verified axiom propagation), but the sorry was never new.
**How to apply:** After isolating a frontier sorry X, before building wiring/dispatch around it, grep the repo
for any other sorry whose STATEMENT differs from X only by ring-level rearrangement (`linear_combination`
test: can you prove X from Y with zero mathematical content?). If yes, they're the same gap — record the
equivalence and focus on whichever formulation has better surrounding scaffolding.

### [2026-06-24] A record-constructor shortcut that ties two distinct parameters creates an impossible constraint when either can vanish independently
When wiring a record/structure by calling its producers, a natural shortcut is to set two parameters to the SAME value when the types happen to unify (e.g. `ζ := η`). This can create an IMPOSSIBLE downstream constraint: if the record's induction or budget field uses both parameters, tying them forces `X ≤ Y * Z` where `Z` can be 0 while `X > 0` — an unsatisfiable inequality that no schedule choice can close. The fix is to keep the parameters DISTINCT and let each have its own concrete schedule, even if the types coincide. A collaborator-audit that reads the budget constraint and checks its satisfiability at regime corners (e.g. `Z = 0`) catches this cheaply.
**Why:** A multi-field assembler tied the "z-read tube radius" and the "recurrence additive defect" to the same parameter (types unified). The resulting budget demanded the U-copy error be bounded by `amp * (current error)`, which is false whenever the current error vanishes while the copy error is positive. Splitting the two parameters and using the original budget shape (`copy + read ≤ amp * err + additive`) made it trivially closable.
**How to apply:** When a record constructor or wiring call shares a parameter across two logically distinct roles (two different producers feeding the same value), check the downstream budget/constraint at the CORNER where either quantity vanishes. If the shared value creates `positive ≤ something * 0`, the sharing is unsound — split the parameters. A single-corner check (`Z = 0`) is the cheap audit. Complements the ∀-index-gating and base-invariant-strength entries (those catch unsatisfiable carried hypotheses; this catches unsatisfiable schedule constraints FROM the wiring shortcut).

### [2026-06-24] A discharge lemma in a DEAD file proves nothing for the headline — trace the import chain FROM the headline before claiming "discharged"
A theorem that correctly discharges a hypothesis has ZERO effect on the public headline if the file containing it is not in the headline's transitive import closure. A side-file theorem builds green, passes `#print axioms`, and even has a producer that "wires" the discharge — but if no `import` chain connects the headline to that file, the headline's carried hypothesis is untouched. This is a strictly stronger failure than an unsatisfiable carried field: the discharge is correct AND satisfiable, yet the headline doesn't consume it.
**Why:** An entire discharge campaign (six correct, axiom-clean per-component lemmas + a producer wiring them) was landed in a side file and repeatedly reported as "the headline's hypothesis is now discharged." The side file was imported by nothing on the headline's live path — the headline still carried the original unsatisfiable hypothesis verbatim. Only when the import closure was explicitly traced did the disconnect surface — after many rounds of false "discharged" claims.
**How to apply:** Before claiming ANY headline hypothesis is discharged, trace the import chain: start at the headline's file, follow its `import` declarations transitively, and confirm the discharge lemma's module appears AND the headline (or its producer) CALLS the lemma. A side-file green build is evidence the LEMMA is correct; it is NOT evidence the HEADLINE consumes it. grep the headline's producer for the lemma name as the final gate. Pairs with §3.3 and the `#print axioms` insufficiency entry.

### [2026-06-24] A predicate excluding only the EXTREME CORNER of a bad set silently admits the bad INTERIOR — witness the admitted region before trusting a carried surface
When a carried hypothesis gates on a predicate like `¬(x = 0)` (excluding the extreme point), any quantitative bound that fails at `x = 1, 2, …` (the interior of the bad set) is UNSATISFIABLE even though the predicate has the "right shape" and excludes the worst case. The predicate looks protective but only cuts off a measure-zero corner; the bound needs the FULL good set (e.g. `x ≥ B`), not just `x ≠ 0`. To catch this: for any carried surface of the form "P(c) → quantitative bound Q(c)", construct a WITNESS configuration in the P-allowed region that VIOLATES Q — if one exists, the carried surface is unsatisfiable and the discharge is vacuous.
**Why:** A carried surface excluded only counter-exactly-0 clocks but the quantitative bound failed at counter 1, 2, … — the interior of the "bad-counter" set. The surface looked protective and passed all mechanical checks, but a counter-1 configuration with potential far exceeding the required bound was a reachable witness refuting it. This happened twice in the same run before the pattern was recognized.
**How to apply:** For every carried "P → Q" surface, ask: is there a point in the P-ALLOWED region where Q fails? Construct the witness (a specific configuration satisfying P but violating Q). If the predicate P is a negation of an exact-value condition (¬(x=0)), the near-extreme interior (x=1) is the first place to check. A surface that only excludes a corner when the bound needs the full complement is unsatisfiable by design. Pairs with the `#print axioms` insufficiency and the conditional-milestone-witnessing entries.

### [2026-06-24] A prefix-sum budget (monotone increasing by the step) CANNOT also converge to zero — the unweighted error blows up after the depth exhausts
When a weighted-bound induction uses a prefix-sum budget `W(j+1) ≥ W(j) + positive_term` (the step ABSORBS each cycle's additive defect by INCREASING W), W is monotone non-decreasing and converges to a finite positive limit. The unweighted error `err ≤ W / k^depth` is controlled ONLY while the depth exponent `k^depth` is large enough to compensate. Once the depth exhausts (e.g. `depth = D − j` hits zero at `j = D`), `err ≤ W · k^{j−D}` AMPLIFIES instead of converging. A "tail budget" (`W → 0`) would fix this, but a tail budget is DECREASING — and the step requires W to INCREASE by a positive term each cycle. These two monotonicity directions are algebraically incompatible (unless every additive term is zero). So the unweighted error CANNOT be proved → 0 from the weighted-bound API alone when the depth is exhausted and the recurrence multiplier exceeds 1. The convergence must come from a SEPARATE direct-dynamics argument (the "Reach premise" pattern).
**Why:** Attempted to prove a per-coordinate tube bound from the weighted-bound induction's prefix budget after the depth ran out; an independent algebraic audit proved the obstruction: prefix step (W increasing) + convergence (W decreasing) ⟹ additive term = 0, proven as a one-line Lean theorem. The fix was recognizing the tube bound as a "Reach premise" — a genuinely separate layer of analysis (ODE dynamics), not a schedule-algebra consequence.
**How to apply:** When a weighted-bound induction's prefix budget exhausts its depth exponent and you need the unweighted error to converge, do NOT try to swap to a tail/decreasing budget while keeping the same step — the monotonicity directions conflict. Either: (a) use a genuinely contractive recurrence (`amp < 1`) so depth grows instead of shrinking; or (b) carry the convergence as a separate hypothesis ("Reach premise") proved by direct dynamics analysis. Check the parallel route's capstone — if it also carries the same hypothesis, your "open atom" is a Reach premise, not a schedule bug.

### [2026-06-24] Search for the SOURCE PAPER's proof method as a repo MODULE before grinding any route
When a formalization is stuck on a hard identity, grep the repo for the **source paper's own proof method**
by name (e.g. the paper's named bijection, its named theorem, its technique keyword). If the paper proves the
identity via "the Hecke bijection", search `*Bijection*.lean`; if via "Sturm bound", search `*Sturm*.lean`.
The repo may already contain a partially-built module encoding that exact method — with all ingredients proven
and only the final assembly missing — while you grind a completely different (and possibly dead) route.
**Why:** A repo contained a 443-line module encoding the source paper's named bijection with ALL mathematical
ingredients proven (0 sorry), but 8 days were spent grinding a different coefficient-level route (which was
ultimately proved impossible by computational probe). The bijection module was found only when a subagent
mentioned it offhand. Searching for the paper's method name in the repo would have found it immediately.
**How to apply:** At session start on a stuck identity, `find . -name '*<PaperMethodName>*'` and
`grep -rln '<method_keyword>' *.lean`. Read any matching module's sorry count + assembly plan before choosing
your attack route. The source paper's proof strategy is the FIRST place to look, not the last.

### [2026-06-24] A packaged algebraic structure's type-class gate ≠ its raw coordinate formulas' gate — check the FORMULA's generality
When a Mathlib algebraic structure (group law, module action, ring instance) requires a strong hypothesis (Field, IsDomain), the underlying raw coordinate FORMULAS it is built from often work over a strictly weaker ring (CommRing). A wall "no group law over this ring" may dissolve if you bypass the packaged structure and call the raw formulas directly. This unlocks evaluation over extension rings (dual numbers, power series, localizations) where the packaged structure doesn't apply.
**Why:** The group law on an algebraic object was field-only in Mathlib, blocking a key construction over a non-field extension ring. Checking the raw coordinate-formula file revealed the formulas were defined over `[CommRing R]` — evaluating them directly over the extension ring bypassed the wall and unlocked the entire construction.
**How to apply:** When blocked by "Mathlib's packaged X doesn't work over ring R," grep the underlying formula file's `variable` line — if the raw definitions use `[CommRing R]` (or weaker), call them directly over R, bypassing the packaged structure's field gate. The packaged structure gates the QUOTIENT/EQUIVALENCE layer; the raw formulas gate only the POLYNOMIAL layer.

### [2026-06-25] When an error-accounting framework counts DESIGNED/CORRECT behavior as an error event, the bound is mathematically false — fix the error taxonomy, not the proof
In a formalization of a multi-phase system (a protocol, an algorithm, a reactive system), the error-accounting framework may decompose the total failure probability into per-phase "seam" error contributions, each bounding `P(unwanted event at this phase transition)`. If the "unwanted event" at some seam IS the system's designed/correct transition (the system is SUPPOSED to advance through this phase), the per-seam error bound is mathematically false (approaching 1, not small) — no proof technique, no surface change, and no parameter tuning can make it true. The fix is at the error TAXONOMY level: redefine what counts as an error for that seam (e.g., use a one-step bridge instead of a multi-step tail, or exclude the designed transition from the error event, or handle that seam's contribution differently in the composition).
**Why:** A multi-phase protocol's seam assembly used a uniform "probability of advancing past phase k" bound for all seams. For most seams, this was the right error event (premature/corrupted advance IS rare). For two seams, the phase advance was the DESIGNED behavior — the bound measured success probability, not failure. Confirmed by parameter computation (the bound ≈ 1, not ≈ 1/n²), multiple independent design consultations, and exhaustive search of proof approaches. Every attempt to prove the bound (concentration tails, timing separation, exact-step narrowing) failed because the mathematical statement was false.
**How to apply:** Before grinding on a per-component error bound that resists all attack vectors, check: is the "error event" for this component actually the system's CORRECT/DESIGNED behavior? If the system is supposed to do X, then P(X happens) is large, and bounding it small is mathematically impossible. The fix: revise the error taxonomy for that component (redefine the event, use a different composition mechanism, or handle the component internally). This is a framework-level fix, not a proof-level one. Pairs with the category-error re-classification entry and the assembly-early/end-to-end entries.

### [2026-06-25] A piecewise-defined object + a global-regularity consumer = UNSATISFIABLE hypothesis — build windowed consumer variants
When a formalization's core object is defined piecewise on a time/parameter interval (`if t ∈ [0,T] then f(t) else 0`), any consumer theorem requiring GLOBAL regularity (`∀ s, HasDerivAt ...` or `Continuous` on all of ℝ) produces an UNSATISFIABLE hypothesis — the function jumps at the piecewise boundary, so the two-sided derivative doesn't exist there. This is a §3.3 vacuity at the TYPE level (the hypothesis compiles, the structure builds green, `#print axioms` is clean, but no inhabitant exists). The fix is NOT to change the piecewise definition (which may be forced by the type system) or to refactor all consumers (massive ripple). Instead, build WINDOWED variants of the load-bearing consumer theorems that take `HasDerivWithinAt` / `ContinuousOn` on the relevant interval, then use those inside the proof body. The output type (which goes to downstream consumers) typically does NOT carry the regularity structure, so the windowed variants compose without cascading.
**Why:** A conditional theorem's last carried hypothesis required global differentiability of a source coefficient family, but the family was defined via `dite` (0 outside the Picard window). The global `HasDerivAt` was genuinely false at the boundary (left-derivative = 0, right-derivative ≠ 0). The fix: three windowed consumer theorems (~100 lines total) taking the windowed regularity structure, used inside the proof body; the output structure was unchanged, so no downstream ripple.
**How to apply:** Whenever a carried hypothesis requires ALL-of-ℝ regularity (`∀ s, HasDerivAt/Continuous/Differentiable`) for a function you know is defined piecewise, check satisfiability at the piecewise boundary BEFORE grinding the producer. If unsatisfiable, build the windowed consumer variant as a new file (mirroring the global proof but using `WithinAt`/`On` + interior-point extraction), not a multi-file refactor. Pairs with "check-existing before a campaign" and "#print axioms clean ≠ unconditional."

### [2026-06-25] An all-cycle interface feeding an eventual conclusion: add a PARALLEL shifted chain with warm-up, don't modify the original
When an intermediate interface demands a property for ALL cycles (`∀ j, P j`) but the final conclusion is EVENTUAL (`∃ T, ∀ t ≥ T, Q t`), and the property `P` is only provable for `j ≥ j₀` (some warm-up number), do NOT try to force the all-cycle interface to hold at early cycles (where it may be genuinely false). Instead add a PARALLEL shifted chain: a shifted variant of the intermediate structure (`∀ j ≥ j₀, P j`) + a shifted version of each downstream consumer, producing the same eventual conclusion. Keep the original all-cycle chain UNCHANGED (trunk-green additive variant). The shifted chain threads through the eventual latch/readout because "eventually" absorbs the warm-up: `∃ T ≥ 2π·j₀, ...`.
**Why:** A tracking induction needed a per-cycle tube bound for all j. The tube was only provable for j ≥ j₀ (after the per-cycle contraction rate made the error small). Attempting to satisfy the all-cycle interface at j=0 was algebraically impossible (the error starts O(1) while the tube radius is O(0.001)). Adding a parallel shifted chain (shifted inputs/results/readout) with warm-up j₀ threaded correctly to the same eventual simulation conclusion, leaving the original all-cycle chain untouched.
**How to apply:** When `∀ j` feeds `∃ T`: check if `P 0` is genuinely provable. If not (warm-up needed), don't fight the base case — write a `...From j₀` variant of the interface + consumers, keeping the original. The shifted chain must thread through every downstream consumer that touches the cycle index; check each for whether it accesses j < j₀. Complements additive-variant/trunk-green (which is about refactors) with the eventual-conclusion warm-up case.

### [2026-06-25] A described-but-unimplemented plan may have been TRIED and ABANDONED — check the history before declaring it "ready"
When you find a multi-step plan described in a docstring (e.g. "M1→M2→M3, mechanical") with all prerequisites proven, do NOT announce it as "the route" until you check: (a) the SAME docstring or nearby comments for a note that the plan was attempted and hit a wall; (b) the project's run log / commit history for prior sessions that tried the same approach. A described plan with proven ingredients does NOT mean the plan is executable — one step may be computationally infeasible (term explosion, Lean timeout) or mathematically blocked at a scale invisible from the description. The docstring often records this in a single sentence ("was left with one open algebraic crux") that is easy to miss if you stop reading at the plan.
**Why:** Found a proven intermediate theorem + a 3-step mechanical plan described in the docstring. Announced "found the shortest route, pure algebra." The user asked "are you sure we haven't tried this?" — and indeed, two lines later in the same file: "which was left with one open algebraic crux." The project's 14-session run log confirmed ALL direct routes had been exhausted. The overclaim wasted credibility and time.
**How to apply:** Before announcing any described plan as viable: (1) read the FULL surrounding docstring/comment block, not just the plan steps; (2) grep the run log and recent commits for keywords from the plan; (3) if the plan was previously attempted, find WHY it was abandoned before re-proposing it. "Prerequisites proven + plan described" is necessary but not sufficient for "plan is executable." Distinct from existing anti-诈尸 (which checks if an obligation is already closed) and route-mapping (which checks if a reduction exists) — this checks if a route was already TRIED and FAILED.

### [2026-06-25] `ring` cannot see through `Polynomial.C` — lift scalar relations via `congrArg C` then distribute with `map_*` simp lemmas
When a CAS-verified polynomial identity involves `C`-wrapped coefficients (`C a₁`, `C (W.b₂ * W.b₆)`, etc.), `ring` in the polynomial ring `R[X]` cannot close because it treats `C a` as an opaque term, not as the scalar `a` it wraps. To prove such identities: (1) lift the base-ring relation to the polynomial ring via `congrArg C hrel` (or `congr_arg (fun t => (C t : R[X])) hrel`), producing a polynomial-level equality; (2) distribute `C` through operations using `simp [map_add, map_sub, map_mul, map_pow]`; (3) THEN `ring` or `linear_combination ... * lifted_hrel` closes.
**Why:** A CAS-verified coefficient identity (involving base-ring relations like `4b₈ = b₂b₆-b₄²`) failed 4 consecutive attempts with `linear_combination X^2 * hbrel` because `hbrel` was an `R`-level equality while the goal was in `R[X]`. The `C`-lift pattern (`congrArg C`) was the missing bridge.
**How to apply:** When `ring` fails on a polynomial-ring goal that should follow from a base-ring hypothesis, check if the hypothesis lives in `R` while the goal lives in `R[X]` (or `R[X][Y]`). If so, lift first (`congrArg C`), distribute (`map_*`), then close.

### [2026-06-25] Trace what the downstream consumer ACTUALLY accesses in the upstream's output before building a heavyweight upstream — most fields may be unused, enabling a far simpler alternative
Before building a complex upstream producer (a multi-field record, a full induction, an all-coordinate invariant), read the downstream consumer's PROOF BODY to see which fields of the upstream's output it actually touches. If the consumer only accesses ONE field out of seven, the full upstream is OVER-POWERED — you can write a far simpler alternative that supplies just that one field, bypassing the entire heavyweight mechanism. This is NOT "do less work because you're lazy" — it's architectural: the heavyweight upstream may have unsatisfiable hypotheses (e.g. an all-coordinate invariant that requires conditions impossible after a warm-up), while the simple single-field alternative avoids those hypotheses entirely.
**Why:** A multi-field tracking induction (7 fields, all-coordinate, requiring depth budget + weighted bound + per-cycle tube for all coordinates) was built to feed a downstream readout theorem. Reading the readout's proof body revealed it accessed ONLY the flag-coordinate z-read field — one out of seven. A direct flag-only producer (one coordinate, no tracking induction, no depth budget) compiled clean-3 and completely bypassed the all-coordinate hhold_slack problem that had blocked the full upstream for hours.
**How to apply:** Before building any multi-field/multi-coordinate/full-induction producer: `grep` the consumer's proof for which fields it actually accesses. If it touches ≤ 2 of N fields, write a direct producer for just those fields instead of the full record. The savings compound: each field of the full upstream may carry its own hypotheses (which become the unsatisfiable "Reach premises"), while the direct producer for the one used field may have trivially-satisfiable hypotheses. Complements assemble-early (which discovers the keystone atom) with field-ACCESS-tracing (which discovers the keystone is the ONLY atom that matters).

### [2026-06-25] Never iterate full builds to debug a tactic in a heavy downstream file — sorry-stub the file, test the tactic in a MINIMAL import, THEN full-build once
When a proof change in file A breaks a downstream proof in heavy file B (which takes N minutes to elaborate), do NOT iterate: edit B → full build (N min) → error → edit B → full build (N min) → error → .... Instead: (1) sorry-stub everything in B EXCEPT the broken proof, (2) create a minimal test file that imports B's module and tests JUST the broken tactic (`#check`, `example`, or a copy of the theorem), (3) iterate the fix in the test file (seconds per attempt), (4) paste the working fix into B, (5) full-build B ONCE. The sorry-stub entry already says this for expensive DEPENDENCIES; this extends it to expensive CONSUMERS of your change — same principle, different direction. The cost function is `(number of tactic attempts) × (build time per attempt)`; stub + test makes the per-attempt cost seconds instead of N minutes.
**Why:** A proof change in one file broke a downstream corollary in a 70-minute-to-elaborate file. Six consecutive full builds (v1→v6) were spent iterating tactic fixes (`simp`, `ext; simp`, `simp [map_zero]`, `ext n; simp [...]`, bare `simp`, `sorry`) — each waiting 70 minutes to discover the next error. The fix (`change ... ; exact qExpansion_zero 1`) was found by a collaborator in 4 minutes. Total wasted: ~7 hours of build time on what was a 4-minute tactic problem.
**How to apply:** When a change breaks a proof in a file that takes >5 minutes to build: (1) identify the exact broken theorem name + its unsolved goal (from the FIRST failed build's error), (2) write a minimal test: `import TheModule; example : <the goal> := by <your tactic attempt>`, (3) iterate there until it closes, (4) paste into the real file, (5) one final full build. Never pay N-minute builds for tactic debugging.

### [2026-06-25] After a route wins, DELETE the abandoned-route files and orphan experiments — a proof repo is not an archive of every attempt
When multiple parallel routes (A, B, C) are explored and route A wins (builds clean, sorry closed, committed), the files created for routes B and C — scaffolding modules, standalone certificate files, scratch experiments (`_Bij.lean`, `_HF81.lean`, standalone `ModularityCertificate.lean`) — must be DELETED or clearly quarantined (moved to a `scratch/` dir excluded from the build). Leaving them in the repo creates a proof forest: the next session (or the next person) encounters multiple contradictory starting points, wastes time re-discovering which route is live, and risks importing a dead file that silently reintroduces stale axioms. The same applies to inline dead code: if a sorry was closed by route A, and route B's conditional theorem in the same file is now redundant, remove or clearly mark route B's code as dead. UNDERSTANDING.md should record which route won and which were abandoned, but the CODE should contain only the live route.
**Why:** A session exploring three parallel routes (kernel axiom, modularity certificate, Winquist clearing) left files from all three in the repo. A standalone certificate file duplicated an inline axiom; orphan experiment files committed alongside the winning route confused the next session's reconnaissance — it took 30+ minutes to determine which path was actually live and which files were dead. The git history preserves everything; the working tree should contain only what builds and matters.
**How to apply:** After committing a route that closes a major sorry/axiom: (1) `git rm` any standalone files created solely for abandoned routes, (2) grep for orphan experiments (`_*.lean`, scratch files not imported by any live module), (3) in files that contain BOTH the winning proof AND abandoned-route scaffolding, remove the dead code or gate it behind a comment block, (4) update UNDERSTANDING.md with "route X won; routes Y, Z abandoned (see git history)." Do this cleanup IN THE SAME commit or the next — not "later." Pairs with §1.3 documentation discipline (one UNDERSTANDING.md, no scattered docs) and killed-object hygiene (grep + purge stale objects).

### [2026-06-25] A wrapper that hardcodes a parameter the engine leaves free creates an artificial universal-quantifier requirement — relax the hardcode to eliminate the hypothesis
When a public-facing theorem hardcodes a parameter (e.g. starting index `J := 0`) that the underlying engine accepts as a free variable, the hardcode silently STRENGTHENS the theorem's hypothesis from "eventual" to "all" (or from "from J" to "from 0"). If the goal only needs the eventual/from-J version, the hardcoded wrapper's hypothesis is UNSATISFIABLE while the engine's relaxed version is trivially satisfied. The fix: write a new wrapper passing the free parameter through, NOT changing the engine. This can eliminate an entire carried hypothesis.
**Why:** A convergence engine accepted an arbitrary starting cycle J, but its public wrapper hardcoded J=0, forcing an "all cycles" hypothesis on the non-halt indicator. The "all cycles" version was unsatisfiable at early cycles (the ODE hadn't converged yet). Writing a new wrapper with J=j₀ (the convergence cycle) turned the unsatisfiable all-cycle hypothesis into a trivially-satisfiable eventual one, eliminating a carried hypothesis from the headline.
**How to apply:** When a carried hypothesis looks "too strong" (demands ALL cycles / ALL time / from time 0), check whether the proof path routes through an engine that accepts a free starting parameter. If so, the strength comes from a WRAPPER's hardcode, not the engine. Write a relaxed wrapper passing the parameter through. The engine doesn't change; only the call site does.

### [2026-06-25] `ring` cannot close identities where the atoms are algebraically dependent — diagnose dependence BEFORE building a clearing campaign
When you clear denominators of rational expressions in a formal-series ring (theta functions, modular forms, algebraic functions on a curve) to get a "polynomial identity" in the cleared atoms, `ring` will FAIL if the atoms satisfy algebraic relations among themselves. `ring` works only in a FREE commutative ring — it cannot see that certain products of atoms relate to others via functional equations or product identities. The cleared identity is true only IN THE QUOTIENT by those relations, not in the free polynomial ring. Diagnosing this BEFORE building the clearing infrastructure saves the campaign from a dead end at the final assembly. The correct closing tactic is a structured algebraic argument that explicitly invokes the inter-atom relations (factorization, `linear_combination` using a known relation, or a quotient-ring `ring`).
**Why:** A full denominator-clearing campaign (25 per-quotient clearings + 9 per-piece assemblies + global skeleton) reached the final polynomial identity and discovered `ring` could not close it — the atoms satisfied algebraic relations making the identity true in the quotient but FALSE in the free ring. The "open algebraic crux" that blocked 14 prior sessions was this algebraic-dependence wall, misdiagnosed as "term explosion."
**How to apply:** Before starting a denominator-clearing campaign: (1) check whether the cleared atoms are algebraically INDEPENDENT or DEPENDENT; (2) if dependent, identify the relations and plan how the final proof invokes them — NOT by `ring`; (3) consider whether factorization (`f * g = 0` where `f = 0` is a known relation) can bypass the polynomial-level identity entirely.

### [2026-06-25] Universal-ring transport: prove a result over a domain, then map to the general ring
When a result (like polynomial divisibility) is true over a general commutative ring but the proof requires domain properties (cancellation, primality, no zero divisors) that the target ring's formal-power-series API doesn't expose, prove it over a UNIVERSAL polynomial ring (e.g. `MvPolynomial (Fin k) ℤ`, which IS a domain), then transport to the general ring via the evaluation homomorphism. Divisibility is preserved under ring maps (`RingHom.map_dvd`), so `p ∣ q` in the domain implies `f(p) ∣ f(q)` in the target. The key requirement: the construction is NATURAL (commutes with base change / `map f`), which Mathlib's formula-level definitions typically are.
**Why:** Needed divisibility by `(X₀-X₁)³` in `MvPowerSeries (Fin 2) R` for general `R`. The proof used domain cancellation (`mul_left_cancel₀`) and unit inversion, both requiring `NoZeroDivisors`. Proving over `ℤ[a₁,...,a₆]` (a domain) and mapping to `R` via coefficient evaluation avoided the missing UFD/coprimality API for `MvPowerSeries`.
**How to apply:** When a proof needs domain properties in a ring whose Mathlib API doesn't provide them, check if the construction is natural/functorial. If so, prove over the universal polynomial ring (always a domain) and transport via `RingHom.map_dvd` / `map_*` naturality.

### [2026-06-25] Grep your OWN repo for working Mathlib call patterns before iterating on type mismatches
When a Mathlib lemma application fails with "type mismatch" or "application type mismatch", don't iterate blindly on `HasDerivAt.comp` / `.scomp` / `comp_hasDerivAt` etc. Instead, `grep -rn 'lemma_name' ShenWork/` to find how your OWN codebase already calls that lemma — including explicit named arguments `(f := ...) (x := ...)`, type annotations, and import paths. The repo's existing call sites have already resolved the typeclass/elaboration issues.
**Why:** Spent 5+ build-fix cycles trying `HasDerivAt.comp`, `HasDerivAt.scomp`, `comp_hasDerivAt`, `deriv.scomp` for the parity chain rule. Each failed with type mismatch. Grepping `WaveTrapProps.lean` revealed the working pattern `deriv_comp_neg (f := g) (x := x)` — a one-line fix. The same pattern repeated with `deriv_comp_const_sub` for the shift symmetry.
**How to apply:** After the FIRST "type mismatch" on a Mathlib lemma, immediately grep the repo for existing uses of that exact lemma name. Copy the working call pattern (named args, explicit types). Don't iterate on alternative lemma names until you've checked the repo's own usage.

### [2026-06-25] A formal-series parameter encoding a MONOMIAL exponent cannot express a non-monomial scalar specialization — define a new object
When a formal power/Laurent series definition parameterizes a variable by an integer exponent (`z : ℤ` meaning `Q^z`), the resulting object `f(a, z)` can only be specialized to MONOMIAL values `Q^0, Q^1, Q^{-1}, ...`. If the proof you're formalizing needs a specialization to a non-monomial scalar (like `z = -1` meaning the scalar minus-one, not the monomial `Q^{-1}`), there is NO integer value of z that gives the right object. This is a TYPE-LEVEL gap, not a missing bridge lemma. The fix: define a new sibling definition (`f_at_minusOne(a)`) that directly encodes the scalar specialization, with its own geometric-branch expansion. Don't try to shoehorn it through the monomial-parameterized version.
**Why:** A formal Appell-Lerch function `m(Q^A, Q^M, Q^z)` parameterized by z∈ℤ could not express the identity's required specialization `m(x, q, -1)` where -1 is the scalar sign. `z=0` gives `Q^0=1`, `z=-1` gives `Q^{-1}`, neither is the scalar -1. Three rounds of analysis confirmed no bridge exists — a new definition was needed.
**How to apply:** When a theorem requires specializing a formal-series parameter to a value that is NOT a monomial in the series variable, check whether the parameter's TYPE (integer exponent) can express it. If not, define the specialized object directly rather than trying to route through the parameterized version.

### [2026-06-25] When stuck on a deep identity, search the LITERATURE for alternative proofs — not just the source paper's own method
When a formalization is stuck on a deep theorem and all routes tried so far fail (the source paper's proof method hits infrastructure walls, algebraic tactics can't close the identity, modular-forms bridges don't exist), search the mathematical literature for ALTERNATIVE PROOFS of the same theorem by different authors. A different proof method may be far more suitable for formal verification — e.g., a bilateral-summation proof avoiding modular forms when the modular-forms approach requires an analytic bridge your formal framework lacks. This is distinct from "search the repo for the paper's proof method" (which looks for existing partial implementations) — this is looking for a DIFFERENT mathematical proof that's better adapted to your tools.
**Why:** A formalization was stuck for 14 sessions because all three routes (Sturm/modular forms, Winquist/ring, kernel statements/involution) hit fundamental walls. A literature search found a 2012 paper by a different author giving a purely bilateral-summation proof of the same theorem, using only tools the repo already had (JTP, geometric series, sign arithmetic). The alternative proof was immediately actionable — 3 of its 6 components were already proved in the repo, and the remaining 3 had clear formalization paths.
**How to apply:** After 3+ failed routes on a deep theorem, search arXiv/MathSciNet for alternative proofs by querying the theorem's NAME + keywords like "new proof", "elementary proof", "bilateral summation", "combinatorial proof". Read the proof method (not just the abstract) to assess whether it maps better to your formal framework. The best alternative proof uses tools you already have, not tools you'd need to build.

### [2026-06-25] A `∀ x` field in a packaging structure may be UNSATISFIABLE — inline for the specific `x`
When a packaging structure has a field quantified over all inhabitants of a type (e.g. `∀ sol, init_property sol`), but the property only holds for SPECIFIC inhabitants (the ones constructed by the supply/existence theorem), the `∀ x` quantifier makes the structure UNCONSTRUCTIBLE. `#print axioms` and `lake build` will never catch this — the structure compiles fine; it just can't be instantiated. The fix: bypass the overly-general structure and inline its pieces for the specific instance, using the per-instance theorem (e.g. `init_property_of_residual` which takes an `InitResidual` predicate on the specific `x`).
**Why:** A packaging structure's `init_zero/init_succ` fields were `∀ sol La, rational_init_eq (tuple sol La 0)`, but `tuple sol La 0` depends on `sol.init_z` which varies across sols — so no single rational `init` value works for ALL sols. The existing code had per-sol init theorems (`init_zero_of_initResidual`) that proved the identity from a predicate on the specific sol's initial data. Inlining these into the final theorem (bypassing the structure) made the headline unconditional. Multiple ChatGPT audits confirmed the `∀ sol` was genuinely unsatisfiable.
**How to apply:** When instantiating a packaging structure fails with no obvious producer for a `∀ x` field, check whether the property is `x`-DEPENDENT (involves `x.field` at a specific time/index). If so, search for a per-`x` theorem guarded by a construction predicate, and inline the packaging for the specific `x` rather than constructing the general structure. Pairs with §3.3 UNSATISFIABLE-FRONTIER.

### [2026-06-25] Bypass a hard type-class field by taking its consequences as explicit hypotheses
When a Mathlib type class (or your own structure) has N fields and the downstream theorem you need only uses K < N of them, do NOT grind the hard missing field just to build the type-class instance. Instead, define your construction taking the K needed fields as explicit hypotheses (not the type-class wrapper). This compiles, avoids the hard field entirely, and lets you land the downstream result immediately. The hard field can be proved later if/when something genuinely needs it.
**Why:** A `FormalGroup` structure required associativity (a substantial proof involving formal power series substitution) alongside three simpler coefficient properties. The downstream tangent theorem (`[n]'(0) = n`) only used the three coefficient properties — never associativity. Defining the formal `[n]` series directly from the power series + three hypotheses (instead of requiring a `FormalGroup` instance) compiled 0-sorry and gave the tangent result immediately, completely bypassing the associativity proof.
**How to apply:** Before grinding a hard field to instantiate a type class: read the downstream PROOF of the theorem you actually need — does it use the hard field? If not, define the construction with explicit hypotheses for the used fields only. The type-class instance becomes an optional future convenience, not a blocking prerequisite. Similar in spirit to "call the raw formulas instead of the packaged structure's field gate" but at the STRUCTURE-field level rather than the CommRing-vs-Field level.

### [2026-06-25] Grep ALL consumers before weakening a structure type — not just the direct file
Before concluding that a structure field is unused and refactoring to a weaker type, grep the ENTIRE repo for `.fieldname` access patterns across ALL downstream consumers — including transitively-imported files. A field that appears unused in the primary consumer file may be critical in a transitive consumer (e.g., the PDE identity path uses `adot` even though the eigenvalue summability path only uses `envelope`). The almost-correct refactor breaks silently: builds pass on the refactored file but the transitive consumer fails on a now-missing field.
**Why:** Attempted to weaken a time-regularity structure to just its envelope fields after checking one consumer file and seeing only envelope access. A second consumer in a transitive import used the full time-derivative fields for a PDE identity proof. Had the refactor been committed, the build would have broken in the transitive file — not the refactored one.
**How to apply:** Before ANY type-weakening refactor: `grep -rn '.fieldname' ShenWork/` for EVERY field you plan to remove. If any file accesses it (even transitively), the field is live and cannot be removed. Only weaken when grep confirms zero access across the entire tree.
