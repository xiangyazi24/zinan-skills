# lean skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/lean.md


## Learned Tactics (Self-Improvement)
_(titles only; read full file for details)_
### [2026-06-22] Your OWN "wiring vs hard theorem" verdict is a claim — TEST it, never assert it
### [2026-06-22] CAS-verify algebraic identities before encoding in Lean
### [2026-06-22] Prove the multiplied form first, cancel the factor separately
### [2026-06-22] Borrow the repo's OWN probability/analytic library before proving a base brick from scratch
### [2026-06-22] De-axiomatizing: audit the hypotheses the axiom let you SKIP, first
### [2026-06-22] `subst h` can eliminate the WRONG variable — use `rw [h]` when you need both names
### [2026-06-22] Overlapping rewrite lemmas misfire — state the goal already-canonical
### [2026-06-22] Clear integer-division-by-a-constant before `ring`
### [2026-06-22] Build a faithful model to DISCOVER the proof mechanism, not just verify truth
### [2026-06-23] Trace the proof DAG for hidden circularity in "proven-modulo-X" reduction chains
### [2026-06-23] Map ALL existing whole-goal reductions before building a NEW route
### [2026-06-23] A cancellation/uniqueness engine on a SCALED difference needs the scale invariant under the engine's transport
### [2026-06-23] Machine-extracted `linear_combination` cofactors must be INTEGER over a general ring
### [2026-06-23] Validate a large `linear_combination` MECHANISM before the final cofactors land
### [2026-06-23] Unconditional `∀`-hypothesis with only an `_of_wf` discharge: try the subclass-unconditional route before declaring vacuity OR restructuring
### [2026-06-23] A carried hypothesis can BE the conclusion — signature-diff a "conditional result" against its OUTPUT type's fields
### [2026-06-23] Re-dispatch a rejected proof task with the EXACT faked anti-pattern named and forbidden
### [2026-06-23] Anti-诈尸: before calling an obligation "open" — or re-proving it — grep the repo for an existing discharger
### [2026-06-23] Repeated correction of your OWN brief's premise on one sub-problem ⇒ escalate to an independent reasoner
### [2026-06-23] An UNSATISFIABLE carried continuity/regularity hyp ⇒ the MODEL is over-engineered — simplify the object, don't satisfy the hyp
### [2026-06-23] Sorry-stub an expensive dependency to fast-iterate the code that USES it
### [2026-06-23] `simp only [bridge_lemma]` BEFORE full simp, when norm_cast would push the cast and kill the match
### [2026-06-23] `linear_combination` "ring failed" — check the hypothesis ORIENTATION before the algebra
### [2026-06-23] A uniform ∀-index field can be UNSATISFIABLE on a parallel assembly that weakens the per-index postcondition — gate it to the sub-band that needs it
### [2026-06-23] Check the BASE invariant isn't STRONGER than the headline needs — pointwise/uniform-sup objects are often self-inflicted
### [2026-06-23] Re-home a needed def/lemma OUT of an un-banked or discredited module — don't import the bad file for it
### [2026-06-23] nlinarith times out on a high-degree goal — `ring`-expand the power FIRST, then it's hint-linear
### [2026-06-23] A lemma that only PROPAGATES a bound should be parametric in the bound — generalize once, don't edit every layer
### [2026-06-23] Push the slack to the stage that's actually hard — pick the intermediate constant that makes the resisting step TRIVIAL
### [2026-06-23] Lean module-system migration: public / @[expose] / no-private-in-public
### [2026-06-23] Escape an import-layer inversion by RELOCATING the goal above its proof materials
### [2026-06-23] Harvesting LLM-generated Lean: classify clean-vs-sorried per decl, land only sorry-free; expect tactic-SCOPE bugs
### [2026-06-24] Bridge an instance-coherence diamond with convert + congr + Subsingleton.elim — don't change file-wide instances
### [2026-06-23] An off-by-a-CONSTANT in a "should-close" reindex means you're on a PRE-TRANSFORMED object
### [2026-06-24] "Reaching state X triggers setup Y" — verify the TRANSITION PATH invokes Y, not just that X's init does Y
### [2026-06-24] A `whnf`/`isDefEq` heartbeat timeout on a heavy definition is NOT a maxHeartbeats problem — block the unfolding
### [2026-06-24] A carried "X satisfies identity E" hypothesis is VACUOUS if E's operator silently drops a term the real X has
### [2026-06-24] Build on a disk-full host in tmpfs + pull dependencies from the package cache, not a cold recompile
### [2026-06-24] Identities over a square-zero / dual-number extension: convert the `MulOpposite.op _ • _` from `snd_mul` before `ring`
### [2026-06-24] Wire the downstream chain FROM the open frontier sorry, then axiom-check it propagates
### [2026-06-24] A continuous-time tracking/tube invariant is true only on the window where the flow CONTRACTS — and if the target is rewritten each cycle, index the NEXT target; sample-window and target co-move
### [2026-06-24] Large starting error in a relaxation/Grönwall tracking bound: use the FULL Duhamel form (mass contracts the start), not the simplified "hold" corollary (which demands start ≤ radius)
### [2026-06-24] Lower-bound an integral via an interior sub-interval + nonneg extension — when the integrand's favorable pointwise bound degrades near the endpoints
### [2026-06-24] Reconstruct the univariate slice EXPLICITLY from coefficients, don't fight the multivariate eval/map API
### [2026-06-24] `#print axioms` clean is NECESSARY, not SUFFICIENT, for "unconditional" — it is SILENT on carried hypotheses
### [2026-06-24] A "hard floor / needs new machinery" obligation: REDUCE-and-bank iteratively, don't attempt to close it
### [2026-06-24] A side-condition that's FALSE at the stated generality must be THREADED from the call site — and re-routing to a "proven general" lemma won't drop it
### [2026-06-24] Before a multi-brick CAMPAIGN on a "hard remaining piece", grep the conclusion's head symbol repo-wide — the bottleneck is often already proved (trivially)
### [2026-06-24] Before building wiring around a "newly isolated" sorry, check if it's an EXISTING sorry restated
### [2026-06-24] A record-constructor shortcut that ties two distinct parameters creates an impossible constraint when either can vanish independently
### [2026-06-24] A discharge lemma in a DEAD file proves nothing for the headline — trace the import chain FROM the headline before claiming "discharged"
### [2026-06-24] A predicate excluding only the EXTREME CORNER of a bad set silently admits the bad INTERIOR — witness the admitted region before trusting a carried surface
### [2026-06-24] A prefix-sum budget (monotone increasing by the step) CANNOT also converge to zero — the unweighted error blows up after the depth exhausts
### [2026-06-24] Search for the SOURCE PAPER's proof method as a repo MODULE before grinding any route
### [2026-06-24] A packaged algebraic structure's type-class gate ≠ its raw coordinate formulas' gate — check the FORMULA's generality
### [2026-06-25] When an error-accounting framework counts DESIGNED/CORRECT behavior as an error event, the bound is mathematically false — fix the error taxonomy, not the proof
### [2026-06-25] A piecewise-defined object + a global-regularity consumer = UNSATISFIABLE hypothesis — build windowed consumer variants
### [2026-06-25] An all-cycle interface feeding an eventual conclusion: add a PARALLEL shifted chain with warm-up, don't modify the original
### [2026-06-25] A described-but-unimplemented plan may have been TRIED and ABANDONED — check the history before declaring it "ready"
### [2026-06-25] `ring` cannot see through `Polynomial.C` — lift scalar relations via `congrArg C` then distribute with `map_*` simp lemmas
### [2026-06-25] Trace what the downstream consumer ACTUALLY accesses in the upstream's output before building a heavyweight upstream — most fields may be unused, enabling a far simpler alternative
### [2026-06-25] Never iterate full builds to debug a tactic in a heavy downstream file — sorry-stub the file, test the tactic in a MINIMAL import, THEN full-build once
### [2026-06-25] After a route wins, DELETE the abandoned-route files and orphan experiments — a proof repo is not an archive of every attempt
### [2026-06-25] A wrapper that hardcodes a parameter the engine leaves free creates an artificial universal-quantifier requirement — relax the hardcode to eliminate the hypothesis
### [2026-06-25] `ring` cannot close identities where the atoms are algebraically dependent — diagnose dependence BEFORE building a clearing campaign
### [2026-06-25] Universal-ring transport: prove a result over a domain, then map to the general ring
### [2026-06-25] Grep your OWN repo for working Mathlib call patterns before iterating on type mismatches
### [2026-06-25] A formal-series parameter encoding a MONOMIAL exponent cannot express a non-monomial scalar specialization — define a new object
### [2026-06-25] When stuck on a deep identity, search the LITERATURE for alternative proofs — not just the source paper's own method
### [2026-06-25] A `∀ x` field in a packaging structure may be UNSATISFIABLE — inline for the specific `x`
### [2026-06-25] Bypass a hard type-class field by taking its consequences as explicit hypotheses
### [2026-06-25] Grep ALL consumers before weakening a structure type — not just the direct file
### [2026-06-25] Build-gate a sub-agent's uncommitted work BEFORE committing — stash/pop + commit is NOT a verified delivery
### [2026-06-25] Induction base case fails at an offset sample point — check for a zero-offset formulation
### [2026-06-25] After 2+ failed API name guesses, delegate to a fresh-context agent — don't iterate blind
### [2026-06-27] A structure field NEVER REFERENCED in the proof chain is a dead field — grep before satisfying
### [2026-06-27] A zero-coupling coordinate decouples from the multi-coordinate tracking problem — exploit to break circularity
### [2026-06-27] Assembling ContDiffAt from HasDerivAt layers: use `contDiffOn_succ_of_fderivWithin` + `ContDiffOn.smulRight` + the `toSpanSingleton` bridge
### [2026-06-27] Counterexample-test your own auxiliary lemmas BEFORE grinding their Lean proof
### [2026-06-27] Numerically pre-verify quantitative conditions before writing the Lean proof — a 10-line Python check can save hours
### [2026-06-27] A "clean-signature wrapper" that instantiates a proved theorem's abstract parameters from concrete inputs is NEW PROOF WORK — scope it separately from refactoring
### [2026-06-27] For gcd/lcm identities, use the UNIVERSAL PROPERTY (dvd_antisymm + dvd_gcd) — don't hunt named lemmas
### [2026-06-27] Verify composition-level support/boundedness conditions NUMERICALLY before building infrastructure on them
### [2026-06-27] Renaming a file requires FOUR synchronized layers — filename, namespace, qualified refs, and docs
### [2026-06-27] `lake env lean` type-checks but does NOT write oleans — use `lake build` when downstream files need the artifact
### [2026-06-27] Mathlib API search → ChatGPT (repo connector), never subagent blind grep
### [2026-06-27] A coefficient-reindexing relation does NOT commute with order-dependent constructions — test it before building on it
### [2026-06-27] When kernel can't evaluate a large numerical bound, overapproximate with algebraic powers to stay in small-number arithmetic
### [2026-06-27] Use `IsCoprime.pow_left_iff` instead of rewriting `x^n → x*...*x` + `of_mul_left`
### [2026-06-27] An axiom/sorry stub must include TYPECLASS INSTANCES the proof consumer needs — not just the Prop
### [2026-06-27] A green incremental build with cached olean can mask pre-existing errors — delete the cache after upstream changes
### [2026-06-27] Split a heartbeat-heavy theorem into private helpers for independent budget isolation
### [2026-06-27] Expand the bridge-error term ALGEBRAICALLY before building telescope infrastructure
### [2026-06-27] For small-modulus proofs (mod 4/8), substitute-and-ring beats ZMod cast
### [2026-06-27] `ring` canonical form kills `rw` in calc chains — rewrite at hypotheses instead
### [2026-06-27] Use `set` to abstract big number literals before calling `ring`/`simp` — prevents maxRecDepth explosion
### [2026-06-27] When the kernel must verify a large exact computation, precompute the answer externally and verify equality — "tell, don't compute"
### [2026-06-27] A globally-quantified pipeline interface can be blocked by an analytical boundary singularity — bypass with a cutoff route
### [2026-06-27] Decompose a monolithic sorry into typed per-case stubs — make each independently attackable
### [2026-06-27] A coordinate shear that reindexes one summable family may FAIL on another whose term-coordinates depend on a different index
### [2026-06-27] Audit infrastructure sorry counts before estimating a sorry's difficulty — assembly of zero-sorry components is categorically easier than missing theory
### [2026-06-28] `set x := expr/k` makes `k*x` opaque to `rw`/`exact` — use `conv` or avoid `set` for divided expressions
### [2026-06-28] An existential bound `∃ C, f(N) ≤ C*g(N)/N` is VACUOUS for convergence if C grows with N — audit the witness
### [2026-06-28] BddAbove on a noncompact domain via left-zero / compact-mid / explicit-tail
### [2026-06-28] `ContinuousOn.congr` takes `Set.EqOn g f s` — the argument is REVERSED from `f = g`
### [2026-06-28] A `classical`-defined function is opaque to simp even on concrete arguments — use proven reduction lemmas instead
### [2026-06-28] `apply_ite` distributes any function through if-then-else — use two phases for deeply nested ifs to avoid maxRecDepth
### [2026-06-28] Numerically verify LLM-proposed proof intermediates BEFORE building infrastructure on them
### [2026-06-28] `sorry` inside `exact f ... (by sorry) ... argN` suppresses type-checking of ALL later arguments — treat sorry-passing proofs as UNVERIFIED
### [2026-06-28] `let` bindings in a theorem's argument type conflict with `set` variables — use `show` to force the outer name
### [2026-06-28] Bound a tsum via `abs_tsum ≤ tsum_abs ≤ tsum_majorant` — write the helper once
### [2026-06-28] Zero-radius placeholders in a multi-field bundle skeleton must be SATISFIABLE per field — not just per type
### [2026-06-28] Lift a blocking sorry to a constructor PARAMETER to unlock downstream sorry
### [2026-06-28] A documented "numerically FALSE" blocking claim may itself be false — independently re-verify before trusting
### [2026-06-28] A coefficient identity verified in total but not pointwise per index requires involution/telescope — not bijection
