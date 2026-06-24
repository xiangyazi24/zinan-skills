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
