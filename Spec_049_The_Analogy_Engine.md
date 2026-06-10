Mike,

A great example of a semantic service WunderOS provides to agents trying to understand the world. “What is the dollar of Mexico? Peso.”

# Spec 049: The Analogy Engine — Typed, Holey Inference and Cross-Context Transfer

**Date:** 2026-05-22

**Status:** RFC — committed design. One upstream gate: §1 (category-layer override). Everything downstream of §1 is committed, not exploratory.

**Scope:** A typed analogical-inference operator for the cognitive substrate, wired into Gleamalog as an evaluable predicate; its numeric core expressed as semiring kernels on Herzschlag; and intra-tenant cross-agent-context transfer built on it. Theory layer included as ratifiable foundation.

**Code name:** "Analogium".

**Informs:** implementation tickets filed via `/to-issues` from §7.

**ADR cross-refs:** `uor-compatibility.md` (the decline this RFC overrides), ADR-015 + addendum (Gleamalog parent — extended here), ADR-034 (Gleamalog scale-out), ADR-039 (Herzschlag systolic — the execution target), ADR-046 (Grafiklog — the semiring-kernel surface; this RFC is its validating consumer), ADR-029 (colored e-graphs / tenant isolation — governs §4), ADR-035 (IM backend federation — home of the §4.4 future), ADR-022/023 (OWL2-VSA — predicate semantics), ADR-013 (kāraka↔Pentad — the C slot is the context boundary), `05_query_execution.md` (three-engine model).

**Memory anchors:** `feedback_tier1_is_physics`, `feedback_vsa_navigate_classical_extract`, `feedback_no_silver_bullets`, `feedback_design_for_mature_system`, `project_auth_model`, `feedback_decouple_extraction_from_retrieval`, `feedback_scoreboard_mandate`.

---

## 0. Summary

Wb has `analogy` as a declared tier-1 axiom (`zig/tier1_axioms.zon`, `:analogy(A,B,C)`) with **no evaluator**. The relation name exists; the inference operator does not.

This RFC builds it. Four threads:

1. **Theory (§2)** — analogy is not similarity. Similarity is a *coupling* (untyped, complete, hole-free; optimal transport). Analogy is a *typed relation with a free variable*; filling the variable is a **Kan extension**. VSA's `bind`/`unbind` is a one-shot approximation of one. And the engine's numeric core is, at every layer, **semiring computation** — one primitive, iterated generalized matmul, under three semirings.
2. **Mechanism (§3)** — the Analogy Engine: a multi-sample VSA Kan-estimator, exposed as the Gleamalog built-in predicate `analogy/3`, returning answers *with confidence*. Gleamalog's existing provenance layer is extended to a confidence semiring so graded analogy facts compose soundly with crisp `:if` derivations.
3. **Execution (§3.8)** — the numeric core runs as semiring kernels on the **Herzschlag** systolic engine — some as raw systolic primitives, some via **Grafiklog**, the graph-analytics layer atop it. The Herzschlag/Grafiklog stack is THE execution target, not a someday-backend. The engine and both op surfaces co-evolve (Neurath posture); the Analogy Engine is the second validating consumer ADR-046 names as its ratification precondition.
4. **Application (§4)** — intra-tenant cross-agent-context transfer. Agent A's learned structure becomes analogically available to agent B in the same tenant. The transferred payload may be the *morphism*, the *objects*, or both. Inter-tenant federation is named as future (§4.4), not built.

The whole RFC reverses one prior decision — `uor-compatibility.md` declined the category-theory layer as "elegance without operational BSC surface." §1 identifies the operational surface and requests the override.

---

## 1. ADR conflict callout — REQUIRES KENDALL OVERRIDE

`docs/architecture/uor-compatibility.md` records a deliberate decline:

> Formal-math namespaces (cohomology, homology, **monoidal, operad**, linear, region) — non-substrate concerns; category-theoretic elegance without practical BSC operational surface.

That decline was **correct for what it decided**: it rejected *importing UOR's `monoidal/` + `operad/` namespace vocabulary*. This RFC does **not** import UOR's namespace. It builds a Wb-native operator.

But the decline's stated rationale — "no operational BSC surface" — no longer holds. The operational surface is now named: **typed, holey analogical inference** (§3) and **cross-context transfer** (§4). The category layer stops being decorative the moment `analogy/3` has an evaluator, because:

- "composition legal iff types match" is the monoidal-category law — it *is* the type discipline that bounds which `analogy` calls are well-formed (§2.4, §3.6);
- "fill the analogy hole canonically" is a Kan extension — it *is* the operator (§2.5, §3.2);
- the graded/crisp seam is an enriched-category fact — it *is* the confidence semiring (§3.4).

**Decision requested:** ratify that the category layer is load-bearing *for the analogy surface specifically*. This does not reopen monoidal/operad import, does not add a `Category` type to the codebase (§5), and does not touch sealed tier-1 physics (§3.6). It commits to using the *laws* with the *names written down in this document*.

A second relationship, *not* a gate but worth recording here: Herzschlag (ADR-039) and Grafiklog (ADR-046) were ratified 2026-05-17 and **built concurrently — Monday–Wednesday of this RFC's own week** — days before this spec. They are not a settled foundation; they are wet paint. ADR-046 is RATIFIED **non-binding**, explicitly pending "Herzschlag op surface lands enough to validate the layer boundary" — and that boundary is itself only days old. The Analogy Engine's numeric core (§3.8) is a first-class, semiring-shaped Grafiklog/Herzschlag consumer across three semirings — it is among the *first* real workloads to exercise that boundary. Building this engine advances ADR-046 toward binding; it joins the same construction wave rather than waiting on it.

Until §1's category-layer override is signed off, §2–§7 are a complete design but not a green light. That is the only gate.

---

## 2. Theory layer

*Epistemic status.* The categorical statements in this section are structural correspondences and design intuitions — they motivate the engine; they are not theorems. Where Phase 0 promotes this RFC to an ADR, any §2 claim ratified as *binding* must first be restated theorem-grade or explicitly hedged ("by analogy with FdVect," "informally," …). The load-bearing claims are the operational ones in §3, not the categorical gloss.

### 2.1 The fuzzy/firm duality

Wb already runs two faces of one structure:

- **Fuzzy** — hypervectors, superposition, Hamming similarity, graded resonance. Answers *how similar?* The WaxCloud, the Honeycomb bank.
- **Firm** — typed Pentad roles, the fingerprint index, the ULID envelope, the tier-1 axiom enum, the CSR integer graph. Answers *which entity? what type?*

`feedback_vsa_navigate_classical_extract` ("VSA navigates, classical extracts") is the operational name for this seam. The `analog` tier-1 axiom (`:analog(String)`, "dense→binary bridge") is the seam named in the ontology itself.

The correct meta-frame is **enriched category theory**: the *firm* part is a category (objects, typed morphisms, composition); the *fuzzy* part is the **enrichment** — hom-objects are not sets, they are vectors/clouds graded by similarity. A category enriched over a similarity space *is* "firm structure with fuzzy hom-sets."

### 2.2 VSA is already a compact-closed category

Wb need not build this; it is latent in BSC:

| Categorical structure | VSA primitive |
|---|---|
| monoidal tensor ⊗ | `bind` (XOR + PathHD rotation) |
| compact-closed cup/cap (the "yank") | `unbind` |
| biproduct / superposition | `bundle` |
| symmetric monoidal unit + commutativity | XOR group identity + commutativity |

The XOR group — commutative, associative, involutive — is a symmetric monoidal structure. Wb uses these *laws* today (`algebra_router.zig` and friends) without ever naming the category. This RFC names it; it adds no code for it.

### 2.3 DisCoCat as the wiring discipline

DisCoCat — Categorical Compositional Distributional semantics (Coecke, Sadrzadeh, Clark 2010, arXiv:1003.4394) — is the exact "fuzzy bits + firm bits" object: meaning *vectors* (fuzzy) wired by a *grammar category* (firm). Wb's analogue: meaning vectors wired by the **predicate category** (tier-1 + tenant typed relations). The firm category supplies *what may compose with what*; the fuzzy vectors supply *the content that flows through the wiring*.

### 2.4 Similarity ≠ analogy

The distinction this RFC turns on:

| | **Similarity** (coupling) | **Analogy** (typed projection) |
|---|---|---|
| shape | symmetric-ish, complete | asymmetric, partial |
| typing | one scalar distortion; relation-types collapsed | typed relation preserved as a first-class morphism |
| hole | none — a full transport plan | a free variable; the answer |
| operator | Sinkhorn OT / Gromov-Wasserstein (`gw_kernel.zig`) | `bind(C, unbind(B,A))` — and its honest form, a Kan extension |
| Wb engine family | resonance / OT kernels | ZRP-family (typed, multi-hop, variables) |

Gromov-Wasserstein minimizes a *single* scalar distortion over intra-space distances — it physically cannot distinguish `causes` from `part_of`. Analogy must. In VSA the relation is **reified**: `r = unbind(B,A)` is itself a vector — a *portable typed transformation*, re-applicable to any third object via `bind(C,r)`. A GW transport plan is *not* portable; it is glued to the two clouds it was solved for. Reified-portable-relation = "typed."

### 2.5 Analogy is a Kan extension

Analogy `A:B::C:?` is an *inference with an unknown*, not a description. Categorically: you have a relation/functor observed on part of its domain, and you want its value at a new point. The universal construction for "extend a partially-known functor canonically" is a **Kan extension** (Mac Lane, *CWM* ch. X — "all concepts are Kan extensions").

VSA's `D = bind(C, unbind(B,A))` is best read as a **one-shot, rank-1 estimator in the spirit of** a left Kan extension — `unbind` recovers the transform from a single observed cell, `bind` extends it to the new input. The Kan extension is the formal ideal the estimator gestures at; this RFC does not claim the operator satisfies its universal property. Observing *many* cells `{(A_i,B_i)}` and bundling the recovered transforms raises the rank of the estimate — a multi-sample estimator (§3.2). The honest (higher-rank, optimization-based) Kan extension is the v2 north star (§5).

The "typed, directional, possibly-partial relation between two structures" object already has a name: a **profunctor** (`C^op × D → V`, enriched). A GW transport plan is, roughly, a profunctor's underlying weighting *with the typing forgotten*. Analogy refuses to forget the typing.

### 2.6 The objects Wb commits to (formal)

- **Predicate category `P`** — objects = entity-types (the `AxiomClass` scheme: Concept/Relation/Attribute/…); morphisms = typed relations (tier-1 atoms + Lexicon-registered tenant predicates). Composition = relation chaining. This is descriptive: `P` is a *lens on* tier-1 + Lexicon, not a new datastructure.
- **A relation as a profunctor** — a tenant predicate `p` is a (graded) profunctor; its known extension is a finite set of observed cells.
- **The analogy operator as a Kan-estimator** — given a profunctor observed on cells `{(A_i,B_i)}` and a query `C`, produce `(D, confidence)` approximating the Kan extension at `C`.

No category-theoretic *types* enter the codebase. The formalism lives in this section and bounds the engine's contract. That is the whole content of §1's "use the laws, write the names down."

### 2.7 The engine is semiring computation

Every numeric layer of the engine reduces to computation over a semiring:

| Engine layer | Semiring | ⊕ | ⊗ |
|---|---|---|---|
| confidence (Module B) | Viterbi | max | × |
| VSA bundle + resolve (Module A) | Hamming | popcount-add | XNOR |
| OT alignment (Module D) | tropical / linear | min / + | + / × |

("Hamming semiring" is shorthand for binary dot-product with popcount accumulation; its formal semiring status is not load-bearing. What *is* load-bearing: the operation is iterated generalized matmul.)

And each layer's operation is the **same primitive**: *iterated generalized matrix multiplication* — a relation / codebook / cost as a (sparse) matrix, `⊗` pairing elements, `⊕` accumulating, iterated to a fixpoint or a fixed depth. Graded Datalog fixpoint, VSA codebook resolution, and Sinkhorn / Gromov-Wasserstein / Floyd-Warshall are one algorithm under three semirings.

That primitive is what a **systolic array** is built for (Kung & Leiserson, 1978) — the heart-pump etymology of "systolic" is exactly what Wb's systolic core is named for: *Herzschlag*, "heartbeat." It is also the GraphBLAS thesis: graph algorithms as semiring linear algebra (Kepner et al., 2016). So the Analogy Engine's numeric core is not bespoke. Its kernels run on the **Herzschlag** systolic core (ADR-039) — directly as systolic primitives, or through the **Grafiklog** graph-analytics layer atop it (ADR-046) where the kernel is graph-analytic. §3.8 makes the Herzschlag/Grafiklog stack the committed execution target.

---

## 3. Mechanism — the Analogy Engine

### 3.1 Operator contract

Primary form, exposed as a Gleamalog built-in:

```
analogy(P, C, D)
```

- `P` — a predicate symbol (a relation in the predicate category).
- `C` — a bound input entity.
- `D` — output logic variable. The engine binds it.
- Semantics: `D` is the analogical completion of `P` at `C`, estimated from `P`'s **current known extension** `{ p(X_i, Y_i) }`. This is the Kan extension of `P` to a new domain element — multi-sample by construction.
- Returns `D` bound, annotated with a **confidence** ∈ (0,1] (§3.4).

Single-pair sugar `analogy(A, B, C, D)` ("as A:B, so C:D") is deferred to v2 (§8) — the predicate-extension form subsumes it and is the Datalog-idiomatic shape.

**Precondition (v1).** `P` must be approximately *functional / low-rank* — each domain element maps to essentially one codomain element under a consistent transform. The planner emits a warning when `P`'s extension shows high relational rank (many Y per X with no consistent offset); v1 still runs but confidence will be low. Higher-rank relations are the v2 honest-Kan case (§5).

### 3.2 v1 estimator — multi-sample VSA Kan-approximation

The estimator is *orchestration* over semiring kernels. The orchestration module (`zig/analogy.zig`, new) sequences the steps; the numeric kernels are Grafiklog semiring ops on Herzschlag (§3.8).

1. Gather `P`'s extension from the HotSlab — observed cells `{(A_i, B_i)}`. (ZRP-family read: exact fingerprint-index scan, per `05_query_execution.md` Engine 2. Not a Honeycomb probe — superposition would smear the cells.)
2. For each cell reify the transform: `r_i = unbind(B_i, A_i)`.
3. Bundle: `r̄ = bundle_i(r_i)` — a **⊕-reduction** in the Hamming semiring. More samples → `r̄` converges → variance falls → the rank-1 estimate sharpens. (`feedback_no_silver_bullets`: more samples is one contributor to quality, not a cure — a genuinely high-rank `P` stays poorly approximated; that is the honest limit, surfaced as low confidence, not hidden.)
4. Apply: `D̂ = bind(C, r̄)`.
5. Resolve: nearest clean codeword to `D̂` — a **codebook matmul** in the Hamming semiring (`codebook ⊗ D̂` under XNOR / popcount-add) followed by `argmax`. Resolution to the Lexicon fingerprint index. ("VSA navigates, classical extracts" — `feedback_vsa_navigate_classical_extract`.)
6. Emit the **raw resonance margin**: the normalized Hamming gap between `D̂`'s nearest and second-nearest codeword. Calibration to a confidence is a separate module (§3.4 / Module A2 / §8) so the calibration method can be swapped without touching the estimator.

Exact binding algebra = Wb's existing `bind`/`unbind` (XOR + PathHD cyclic rotation). Steps 3 and 5 are the hot path and are semiring matmul — §3.8.

### 3.3 Alignment substep

When `C` lives in a *different* structure than `P`'s observed domain (the cross-context case, §4), the cells and `C` are not in a shared frame. Before step 4, run an alignment:

- shared embedding available → Sinkhorn OT — iterated `(+,×)`-semiring matmul with row/column normalization;
- no shared coordinates, only intra-structure distances → Gromov-Wasserstein — matmul over the cost tensor; the BSC-6 escape hatch's Floyd-Warshall fallback is tropical `(min,+)`-semiring closure.

The OT plan fixes the `A↔C` correspondence — *which* element of `P`'s domain `C` plays the role of. This is similarity used as a *substep of* analogy, exactly per §2.4. Same-context calls skip alignment (identity).

`sinkhorn_kernel.zig` / `gw_kernel.zig` are the **seed implementations**. They migrate onto the Grafiklog op surface as that surface lands (§3.8, Neurath posture) — the engine does not maintain a second, parallel OT path.

### 3.4 graded ⋈ crisp — the confidence semiring

Gleamalog is crisp Datalog. `analogy/3` yields *graded* facts. They must compose soundly with crisp `:if` derivations in one proof. This is the hardest design point, and it has a textbook answer.

ADR-015 already gives Gleamalog a **provenance** layer. Provenance and confidence are the *same machinery* under different semirings (Green, Karvounarakis, Tannen — "Provenance Semirings," PODS 2007). Commit:

- Annotate every derived tuple with an element of the **Viterbi semiring** `⟨[0,1], max, ×, 0, 1⟩`: `⊕ = max` (best derivation wins), `⊗ = ×` (confidence multiplies along a proof).
- A crisp fact / crisp `:if` firing carries `1`. Crisp-only derivations carry `1` end-to-end — **existing behaviour is unchanged**, which the test suite must prove (§3.7).
- An `analogy/3` result carries its calibrated confidence `c < 1` (calibration = Module A2; the estimator emits the raw margin, A2 maps margin→confidence). Any rule whose body touches a graded atom carries the `×`-product through to its head.
- **Negation (v1):** stratified negation over a graded atom is the known-hard case. v1 commits to the simple sound rule — *an `analogy`-derived atom may not appear under negation*; the Gleamalog planner rejects such a rule with a clear diagnostic. Revisable when a use case demands it.

This is not a hack bolted onto Datalog — it is the standard semiring-annotated generalization, and Gleamalog already has the provenance plumbing it rides on. The graded fixpoint is itself semiring matmul (relation-as-matrix, join = semiring matmul, iterate); §3.8 names Herzschlag as its execution target for the inner loop, with ADR-034 differential dataflow retained as the standing-view materialization layer.

### 3.5 Materialization — query-time-only

Decision, following the precedent in `uor-compatibility.md` §5A Category 2 (sidestore-vs-Pentad): **analogy results are query-time-only.** The engine returns `(D, confidence)` to the caller; nothing is written to WM.

Rationale (mirrors that precedent): keeps the ingest hot path untouched; avoids two representations of the same inference drifting; `feedback_decouple_extraction_from_retrieval`.

**Revisability criterion:** materialize analogy results as facts-with-confidence in WM when a *named* use case needs them ZRP-queryable downstream (e.g., "show every fact analogically derived from context X in the last hour"). Not in v1.

### 3.6 Wiring — tier-1 stays sealed

`feedback_tier1_is_physics`: tier-1 is closed Wb physics. The `analogy/3` *evaluable inference* must not change it.

- The tier-1 atom `:analogy(A,B,C)` (3 entity args) is unchanged. It remains the **assertable relation** — you may *state* that three things stand in analogy as a fact.
- The **evaluable** `analogy(P,C,D)` (predicate-symbol arg) is a **Gleamalog built-in predicate** — it lives in the evaluator's built-in table alongside comparison built-ins, *not* in `tier1_axioms.zon`. Built-ins are not tier-1 axioms.

No arity change to sealed physics. The two share a name and nothing else; the spec is explicit so no future reader conflates them.

### 3.7 Tests (TEST FIRST — CLAUDE.md)

Every phase ships failing tests first. The engine's required tests:

- **Canonical analogy** — Kanerva "dollar of Mexico": `analogy(currency_of, mexico, D)` ⇒ `peso`.
- **Multi-sample convergence** — confidence and accuracy rise monotonically as `P`'s observed extension grows; a single-cell call is the rank-1 floor.
- **High-rank honesty** — a deliberately high-rank `P` returns *low confidence*, not a confident wrong answer (`feedback_no_silver_bullets`).
- **Confidence calibration** (Module A2 in isolation) — empirical accuracy within tolerance of reported confidence over a labelled set; survives swapping conformal for Platt.
- **Semiring kernel correctness** — each Grafiklog semiring kernel the engine uses (Hamming bundle ⊕-reduce, Hamming codebook matmul, tropical closure) is unit-tested for semiring laws and against a scalar reference, independent of the analogy orchestration.
- **Semiring soundness** — a proof mixing crisp `:if` and graded `analogy` atoms carries the correct `×`-product; a crisp-only proof is bit-identical to pre-RFC Gleamalog output.
- **Negation rejection** — a rule with an `analogy` atom under negation is rejected by the planner with the documented diagnostic.
- **Alignment** — cross-frame `C` routes through GW/Sinkhorn and recovers the right correspondence on a synthetic two-frame fixture.

Perf + accuracy land on a new `analogy` scoreboard (`feedback_scoreboard_mandate`): operator latency, accuracy vs. extension size, confidence calibration error. Where Herzschlag two-backend conformance applies (ADR-039), the kernel tests run both backends.

### 3.8 Execution target — the Herzschlag / Grafiklog stack

The numeric core — the §3.2 hot path (bundle ⊕-reduce, codebook resolve), the §3.3 alignment, and the §3.4 graded fixpoint inner loop — executes on the **Herzschlag systolic engine**. Two layers, kept distinct:

- **Herzschlag** (ADR-039) — the systolic engine itself. Raw systolic primitives: vector ⊕-reduction, generalized matmul.
- **Grafiklog** (ADR-046) — graph analytics *atop* Herzschlag. Higher-level graph operations (closure, traversal, fixpoint) expressed as semiring linear algebra.

The Analogy Engine's kernels split across both. The Hamming-semiring bundle and codebook matmul (§3.2) are raw systolic primitives — Herzschlag-level. The graded Datalog fixpoint (§3.4) and the Floyd-Warshall closure (§3.3) are graph-analytic — Grafiklog-level. Sinkhorn / GW alignment straddles. **A kernel the engine needs may require work in Herzschlag, in Grafiklog, in neither (already present), or in both.** The spec does not pre-assign — that resolves per kernel during implementation.

This is a committed target, not a someday-backend:

- **No SIMD-shadow.** The engine does not maintain a parallel scalar/SIMD numeric path "until the stack is ready." One hull. The kernels run on Herzschlag — directly, or through Grafiklog where the kernel is graph-analytic.
- **Neurath posture.** Herzschlag and Grafiklog were built concurrently in a single three-day window the same week as this RFC — the stack is wet paint, not bedrock; *both* layers' op surfaces are still forming. Where the Analogy Engine needs a kernel a layer does not yet expose, the engine work **extends that layer** — Herzschlag or Grafiklog, whichever is the right home — rather than routing around it. Given how new both are, expect the engine to be a *major* contributor, not a marginal caller. We rebuild the boat while sailing it; we do not build a second boat. The seed kernels (`sinkhorn_kernel.zig`, `gw_kernel.zig`, and the BSC bundle/Hamming primitives) are existing planks, migrated onto the stack, not duplicated beside it.
- **Validating consumer.** ADR-046's ratification precondition is "Herzschlag op surface lands enough to validate the layer boundary" — and that Herzschlag↔Grafiklog boundary, like both engines, is only days old. The Analogy Engine is a real, semiring-shaped workload across three semirings spanning both layers; it is among the first to exercise that boundary in anger, and is precisely the consumer that validates it. Building this engine advances ADR-046 toward binding.
- **Layer split (compute vs coordinate).** Per ADR-039 (Gleam coordination, Zig computation): orchestration and the Gleamalog planner stay Gleam; the semiring kernels are Zig on the Herzschlag/Grafiklog stack. Module C is the boundary.
- **What stays off the stack.** Module A2 (scalar calibration), Module C (planner / control flow), Module E (capability/policy decisions). Not semiring-shaped; they stay where they are.

Phase sequencing (§7) co-orders with *both* op surfaces: a phase whose kernels are not yet present either contributes them — to Herzschlag, to Grafiklog, or both — or is sequenced after the contributing work. Never replaced by a shadow path.

---

## 4. Application — intra-tenant cross-context transfer

### 4.1 Scope

Cross-**agent-context** transfer **within one tenant**. Agent A operates in context X; agent B operates in context Y; both belong to the same tenant. What A's facts encode structurally becomes analogically available to B.

Context = the Pentad **C slot** (ADR-013: C and L are structured user/sys-namespaced records — context is a first-class, addressable thing, not a tag). This is the Wb realization of **institutional memory**: one agent's learning, reusable by another in the same organization.

Transfer is just §3 run across a context boundary: gather `P`'s observed cells from context X, take query `C` from context Y, route through the §3.3 alignment substep, return `(D, confidence)` usable in Y.

### 4.2 The morphism, never the objects

What crosses the context boundary is the reified transform `r̄` (the typed morphism) plus the typed relation identity — **not** the entity vectors of context X. `r̄` is *structure*; the entities are *content*.

Consequence: transfer ships the *shape* of a solution without the data that instantiates it. This is a privacy property and it holds even intra-tenant (context isolation is real isolation). It is also exactly what will make the inter-tenant case defensible later (§4.4).

### 4.3 Governance — colored-context capability gate

ADR-029 (colored e-graphs / tenant isolation) gives Gleamalog per-color isolation. Each context is a color. Cross-context transfer is, by construction, a **color-crossing** operation.

Commit: transfer is **default-deny**. The engine performs a color-crossing `analogy` call only under an explicit capability grant (`project_auth_model` — capabilities are Wb's agent-to-agent delegation primitive). Same-color (`X = Y`) calls need no grant. The grant names: source context, target context, and the predicate(s) `P` permitted to cross. Absent a grant, a cross-color `analogy` call fails closed with a diagnostic.

### 4.4 Inter-tenant federation — future, named, not built

Cross-**customer** transfer crosses the hard ADR-029 wall. It is **out of scope for this RFC's build** and explicitly *not* ticketed in §7.

When taken up, its home is ADR-035 (IM backend federation) plus a consent protocol. The §4.2 morphism-not-objects property is the load-bearing reason it can be made defensible — a customer ships the *functor*, never its *objects*. Recorded here so the design does not have to be reconstructed.

---

## 5. Non-goals

- **The honest, higher-rank Kan extension.** v1 ships the multi-sample VSA estimator (§3.2). The optimization-based full Kan extension is the v2 north star — staged, not open.
- **Inter-tenant transfer.** Future (§4.4).
- **Relation-hole abduction** — `analogy(?, A, B)`, "find the rule." A different hole (the morphism, not the object). Named as future; not designed here.
- **A `Category` type in code.** The categorical formalism stays in §2. Wb uses the laws (XOR group, OT plans, structure-preserving maps); it does not grow a categorical runtime.
- **A parallel SIMD-shadow numeric path.** §3.8 — one hull. The engine targets Herzschlag via Grafiklog; it does not maintain a separate scalar execution path.
- **Building the substrate.** ADR-039 owns the Herzschlag systolic engine; ADR-046 owns the Grafiklog graph-analytics layer. This RFC is a *consumer*. It may *extend either layer's op surface* where a needed kernel is absent — that is in scope, by design (§3.8) — but it does not build the substrate.
- **Changing tier-1.** §3.6.

---

## 6. ADR / cross-reference table

| Doc | Relation to this RFC |
|---|---|
| `uor-compatibility.md` | Declined the category layer ("no operational BSC surface"). §1 identifies the surface and requests override. **Gating.** |
| ADR-015 + addendum (Gleamalog) | RATIFIED. This RFC **extends** it: the existing provenance layer gains a Viterbi confidence semiring (§3.4); `analogy/3` is a new built-in (§3.6). Amendment-grade change to ADR-015. |
| ADR-034 (Gleamalog scale-out) | DD backend; planner unchanged. Confidence threads through DD-materialized standing views; DD is retained as the standing-view layer over the Herzschlag semiring inner loop (§3.4, §3.8). |
| ADR-039 (Herzschlag systolic) | **The systolic engine — the execution target.** Ratified 2026-05-17, built concurrently with Grafiklog the same week as this RFC. The §3.8 numeric core runs on Herzschlag; raw systolic primitives (bundle ⊕-reduce, codebook matmul) are Herzschlag-level. Gleam-coordination / Zig-computation split (§3.8) and two-backend conformance (§3.7) are inherited from this ADR. |
| ADR-046 (Grafiklog) | **Graph analytics atop Herzschlag.** Days old — built concurrently with Herzschlag. RATIFIED non-binding, pending a layer-boundary-validating consumer. This RFC is that consumer (§1, §3.8). Graph-analytic kernels (graded fixpoint, Floyd-Warshall closure) are Grafiklog-level; the engine expects to contribute to this surface, not just call it. |
| ADR-029 (colored e-graphs / isolation) | Governs §4.3. Contexts are colors; transfer is color-crossing; default-deny. |
| ADR-035 (IM backend federation) | Home of the §4.4 inter-tenant future. Not touched by the build. |
| ADR-022 / ADR-023 (OWL2-VSA) | Predicate semantics the §2.6 predicate category is a lens over. |
| ADR-013 (kāraka↔Pentad) | The C slot is the §4 context boundary. |
| `05_query_execution.md` | Three-engine model. Analogy = ZRP-family (typed, holey); the alignment substep = resonance/OT family. §3.2 step 1 obeys "never probe Honeycomb for cell extraction." |

Per CLAUDE.md Design Discipline: no binding ADR blocks this design. ADR-015 is amended (additive); ADR-039/046 are *consumed* (the engine is a downstream workload, and a validating one for ADR-046); `uor-compatibility.md`'s decline is the one conflict, surfaced in §1 rather than worked around.

---

## 7. Phasing → `/to-issues`

Committed phasing. Each phase is test-first (CLAUDE.md). Phase 1–4 are this RFC's deliverable; Phase 5 is future and **not** ticketed now. Phase ordering co-sequences with *both* the Herzschlag and Grafiklog op surfaces (§3.8) — a phase contributes the semiring kernels it needs, to either layer, or follows the work that does; no shadow path.

- **Phase 0 — gating: promote Spec 049 to a ratified ADR.** A discrete spec→ADR ratification ticket. Resolves §1: explicitly override the category-layer decline in `uor-compatibility.md`. Records the ADR-046 validating-consumer relationship. Until this ticket is closed, Phases 1–4 are fully designed but not green-lit.
- **Phase 1 — Modules A + A2.** Estimator orchestration (`zig/analogy.zig`) + the Hamming-semiring kernels (bundle ⊕-reduce, codebook matmul) as systolic primitives on Herzschlag; calibration module. NIF surface in `nifs.zon`. Tests: canonical analogy, multi-sample convergence, high-rank honesty, calibration, semiring-kernel correctness (both backends). Register the `analogy` scoreboard. Phase 1 may resolve partly as Herzschlag op-surface contributions before Module A analogy code lands.
- **Phase 2 — Modules B + C.** `analogy/3` built-in; Viterbi confidence semiring threaded through the existing provenance layer; graded fixpoint inner loop on the Grafiklog graph-analytics layer; planner rejects analogy-under-negation; confidence threads through ADR-034 DD views. Tests: semiring soundness, crisp-unaffected, negation rejection. New `features` row.
- **Phase 3 — Module D.** Alignment substep — Sinkhorn / GW / Floyd-Warshall migrated onto the Herzschlag/Grafiklog stack. Tests: two-frame correspondence recovery.
- **Phase 4 — Module E.** Cross-context `analogy` API; colored-context capability gate (default-deny); morphism-only transfer payload. Tests: color-crossing denied without grant, morphism-only payload assertion, end-to-end A→B transfer.
- **Phase 5 — future, not ticketed.** Honest higher-rank Kan extension; inter-tenant federation; relation-hole abduction.

---

## 8. Open decisions

- **Codename.** "Analogy Engine" is descriptive. A Wb-style codename is Kendall's call.
- **Confidence calibration method (Module A2).** Lean conformal prediction — Wb already has conformal-prediction work, and a conformal set is the honest "confidence" for a graded fact. Platt scaling is the lighter alternative. Decide at Phase 1.
- **Single-pair sugar `analogy/4` (§3.1).** Ship in v1 or fold into v2. Default: v2 — the predicate-extension form is sufficient and idiomatic.

---

## 9. References

**External**
- Coecke, Sadrzadeh, Clark — "Mathematical Foundations for a Compositional Distributional Model of Meaning," 2010. arXiv:1003.4394. (DisCoCat — §2.3)
- Gentner — "Structure-Mapping: A Theoretical Framework for Analogy," *Cognitive Science* 7(2), 1983. (analogy = relation-preserving map — §2.4)
- Mémoli — "Gromov–Wasserstein Distances and the Metric Approach to Object Matching," 2011. (cross-space structural similarity — §3.3)
- Green, Karvounarakis, Tannen — "Provenance Semirings," PODS 2007. (semiring-annotated Datalog — §3.4)
- Kung, Leiserson — "Systolic Arrays (for VLSI)," 1978. (the systolic primitive — §2.7, §3.8)
- Kepner et al. — "Mathematical Foundations of the GraphBLAS," 2016. (graph algorithms as semiring linear algebra — §2.7)
- Mac Lane — *Categories for the Working Mathematician*, ch. X. (Kan extensions — §2.5)
- Kanerva — hyperdimensional analogy / "the dollar of Mexico." (VSA analogy — §3.2)

**Internal**
- `docs/architecture/uor-compatibility.md`; ADR-015/022/023/029/034/035/039/046, ADR-013; `docs/architecture/05_query_execution.md`; `docs/architecture/specs/roadmap-maths.txt` (WaxCloud / Sinkhorn lineage).


Cheers,
Kendall


k@pentad.ai
Thanks! .. I'll save it as a .md file an

Michael Hogan <mike@bareedge.ai>
8:36 PM (2 hours ago)
to k

Cool .. looks like markdown .. I'll have a peek in VSCode .. 
