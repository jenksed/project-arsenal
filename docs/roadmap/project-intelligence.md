# Project Intelligence — Roadmap Redirection

Status: redirecting slice. No implementation is committed by this
document.

Scope: redirect the existing ARS-NN roadmap so the newly identified
Project Intelligence hypothesis can be investigated deliberately
without destabilizing working contracts or prematurely committing
to an unproven rewrite.

Companion documents:

* `docs/architecture/project-intelligence.md` — hypothesis, scope,
  authority semantics, alternatives, risks, success and kill
  criteria.
* `docs/architecture/architecture-shootout.md` — the experiment
  that must prove or falsify the hypothesis.

## 1. Why a redirection, not a rewrite

Project Arsenal's conceptual architecture is already strong
(Repository Truth, Recon, Context Boundaries, Knowledge Plane,
Capability Graph, Trust & Authority, Evidence, Reality Budget,
Bench, Compiler, GC01 governance). The risk is that the
*implementation* architecture fragments as each new concept gets
its own JSON shape, schema, Python evaluator, fixtures, and CI
workflow.

GC01's source model is the first deliberately shared primitive for
"which artifact owns which fact?". It does not duplicate the fact
itself. The same discipline (source-bound, derived, disposable)
might absorb a second class of repeated work: rebuilding indexes of
canonical facts inside multiple subsystems. The hypothesis must be
tested, not assumed.

## 2. The experimental sequence

The proposed Project Intelligence slice is an **experiment**. It
must prove or falsify the hypothesis before any of its outputs
become operational truth.

```text
GC01                          (ACCEPTED — current head)
  fact-source registry
       |
       v
PI-00  minimal IR tracer      (EXPERIMENT)
       extracts ~8-12 representative facts from canonical artifacts
       via the source model; rebuildable; provenance-bearing;
       no consumer yet
       |
       v
PI-01  disposable project     (EXPERIMENT)
       intelligence index
       rebuildable local index of IR observations; SQLite only if
       evidence shows it helps; otherwise plain JSON cache
       |
       v
PI-02  architecture shootout  (EXPERIMENT)
       runs the six tracer scenarios; passes or kills the
       hypothesis
       |
       v
ARCHITECTURAL DECISION
       |
       +--> adopt shared Project Intelligence substrate (PI-03..)
       |
       +--> retain specialized architecture (Strategy A)
```

This sequence is **provisional**. Names and number ranges are
adapted to Project Arsenal's existing ARS-NN convention but are not
yet permanent slice names. PI-00, PI-01, PI-02 are
*experimental work packages*, not accepted roadmap slices.

## 3. Reclassification of existing work

The existing roadmap items under
`docs/roadmap/capability-system.md` and
`docs/roadmap/post-pr-24-deferred-architecture.md` are classified as
follows. The classification is the change; the original ordering
remains valid until PI-02 disposes.

### 3.1 KEEP — still correct regardless of substrate

These items remain valid whether the substrate experiment passes
or is killed.

* ARS-09 Knowledge Plane — its typed entities and relationships
  remain authoritative. The IR may feed it observations but does
  not replace it.
* ARS-08 Trust & Authority Plane — its authority predicates
  remain authoritative. The IR may be queried about canonical
  state but does not redefine authority.
* Capability Graph (ARS-04) and Capability Gap Preflight —
  capability composition and preflight semantics remain
  authoritative.
* Reality Budget (ARS-05) and execution substrate selection —
  selection logic remains authoritative.
* Flight Recorder (ARS-07) and Bench (ARS-02) — provenance and
  empirical evaluation remain authoritative.
* GC01 governance source model — the fact-source registry is the
  foundation on which the IR would be built.
* GC00 architecture doctrine — ownership/state-role boundary
  guarantees 1–7 remain in force.

### 3.2 MOVE EARLIER — needed to test or enable Project Intelligence

These items must precede PI-02 because the shootout depends on
them.

* GC01 source-model coverage of the load-bearing artifacts
  (already at ACCEPTED coverage for the 30 artifacts and 43 facts
  on the current head). The shootout's tracer 1 (KFT-0 authority)
  and tracer 4 (feature delivery preflight) need every fact they
  observe to be in the source model. A future slice must extend
  coverage to any artifact the shootout needs but the current
  model omits; that extension is a small, GC01-shaped change.
* A canonical reference for the IR's "disposable, rebuildable,
  non-authoritative" status — the IR itself, plus the index file
  path and `.gitignore` entry, must be declared in the
  architecture doctrine before PI-01 ships.

### 3.3 MOVE LATER — would create expensive representations before the substrate question is answered

These items remain in the roadmap but should not begin before PI-02.

* Track A item 3 — Structured Decision Records + commit-role
  vocabulary. This slice is independent of Project Intelligence
  and may proceed in parallel.
* Track A item 4 — First generated governance-status projection.
  This is a *projection*, not a substrate. Its implementation
  should consume the IR if PI-02 passes; if PI-02 is killed, it
  remains implementable from canonical artifacts directly.
* Track A item 5 — Lifecycle separation artifact. Unchanged.
* Track A items 6–8 — Stop-condition taxonomy, consistency lint,
  generated review summary. Unchanged.
* Track B items — Consumer integration, checkout topology,
  dependency / materialization ownership, local/CI parity,
  upgrade lifecycle. Unchanged.
* ARS-10 Intent Compiler — its rule layer may consume IR
  observations, but its compilation semantics do not depend on
  the IR and may proceed in parallel.
* ARS-11 Adversarial Verification — independent of substrate.
* ARS-12 Controlled Capability Evolution — independent of
  substrate.

### 3.4 REFRAME — same user outcome but should consume shared facts/indexes

These items should be reframed to consume IR observations when PI-02
passes, without changing their user-visible outcome.

* Knowledge Plane compile-context (already partially implemented in
  `scripts/arsenal_knowledge.py`) — the snapshot input is
  hand-built. After PI-02, an IR-backed extractor can produce the
  snapshot deterministically from canonical artifacts; the
  adjudicator stays the same.
* Capability Gap Preflight — the loader pair
  (`load_capabilities` / `load_assets` / `load_locked_capabilities`)
  is duplicated across three subsystems. After PI-02, a single IR
  observation can serve all three.
* Generated governance-status projection — after PI-02, the
  projection reads the IR instead of hand-restating status.
* Compiler verification of capability state — after PI-02, the
  compiler reads the same IR observation the graph reads, so the
  two cannot drift.

### 3.5 SUPERSEDE — architecture would make the existing planned mechanism unnecessary

None at this slice. The architecture shootout might surface
candidates later (e.g. a hand-maintained frontier table that the
IR can derive), but no such supersession is declared now.

### 3.6 EXPERIMENT — requires empirical validation before acceptance

These are the new experimental work packages introduced by this
slice. They are not accepted roadmap slices.

* **PI-00 — minimal IR tracer.** Objective: prove the source-model
  identifies an owner, the extractor opens the owning artifact,
  the exact fact is extracted, source locator / repository SHA /
  artifact digest are retained, the IR is deterministic, the IR
  is disposable, and the canonical repository remains
  authoritative. Inputs: GC01 source model plus approximately
  8–12 representative canonical facts already registered through
  GC01. Outputs: a small Python extractor + an in-memory IR
  representation + a deterministic rebuild test. Constraints: no
  consumer of the IR; no SQLite; no schema commitment. Acceptance:
  pass the PI-00 characterization tests; produce the same
  observation digest across two runs against the same commit.
* **PI-01 — disposable project intelligence index.** Objective:
  build a rebuildable local index of PI-00 observations. Inputs:
  PI-00 outputs plus the architecture shootout's tracer set.
  Outputs: a rebuildable index keyed by fact id; a documented
  rebuild procedure; an entry in `.gitignore`. Constraints: SQLite
  only if evidence shows it helps; otherwise plain JSON cache;
  no remote infrastructure; no daemon; no service. Acceptance: pass
  the rebuild procedure as a no-op; the index can be deleted and
  recreated from canonical state in under the documented time
  budget.
* **PI-02 — architecture shootout.** Objective: run the six tracer
  scenarios in `docs/architecture/architecture-shootout.md` and
  record the evidence. Inputs: PI-00, PI-01, the existing
  specialized subsystems as the control. Outputs: a structured
  PASS / FAIL / STRICTER comparison per tracer; a recommendation
  to adopt or reject. Constraints: no canonical authority may
  move into the IR during the shootout. Acceptance: every
  mandatory graduating condition is evidenced, OR the
  disqualification conditions fire and the experiment is killed.

## 4. What this redirection does NOT do

* It does not implement PI-00, PI-01, or PI-02. Implementation
  requires the same authority as any other slice.
* It does not reorder Track B, the Knowledge Plane, Trust &
  Authority, or any other accepted slice.
* It does not change the GC01 source model or its surgical repair.
* It does not introduce a new ownership layer, state role, or
  materialization mode.
* It does not commit an IR schema.
* It does not introduce a new dependency, database, or service.
* It does not weaken any boundary guarantee 1–7 in
  `engineering/doctrine/ARCHITECTURE.md`.

## 5. North-star alignment

The redirection embodies the philosophy the architecture note
records:

> Do not build the next representation until we know why the
> existing source cannot answer the question.

> Compute project truth once when practical, preserve its
> provenance, and reuse the observation without transferring
> authority.

> Models reason over project state. They do not define project
> state when deterministic evidence exists.

> A shared architecture must earn itself through correctness,
> reuse, context efficiency, and reduced engineering duplication.

The PI-00 / PI-01 / PI-02 sequence is the smallest path to test
whether the hypothesis holds. The specialized architecture is the
fallback. Project Arsenal must be willing to kill its own
experiment.

## 6. References

* `engineering/doctrine/ARCHITECTURE.md` — boundary guarantees.
* `docs/architecture/project-intelligence.md` — hypothesis and
  constraints.
* `docs/architecture/architecture-shootout.md` — the experiment.
* `docs/roadmap/capability-system.md` — canonical ARS-NN ordering
  this redirection preserves for items outside the experiment.
* `docs/roadmap/post-pr-24-deferred-architecture.md` — Track A /
  Track B items preserved under "MOVE LATER" above.
