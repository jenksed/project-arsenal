# Project Intelligence — Architecture Shootout

Status: experimental specification. No implementation is committed
by this document.

Scope: a deliberate empirical decision point comparing the current
specialized-subsystems architecture against a shared-substrate
prototype driven by the GC01 source model. The shootout must
produce verdict-equivalent (or stricter) safety behavior on every
existing adversarial case, or it fails the experiment.

See `docs/architecture/project-intelligence.md` for the hypothesis
this shootout tests.

## 1. The decision the shootout answers

```text
Should Project Arsenal adopt a shared deterministic Project
Intelligence substrate that extracts source-bound observations from
canonical artifacts, indexes them rebuildably, and serves
authority-aware queries to existing subsystems?

Or should Arsenal retain the current specialized-subsystems
architecture and limit shared work to helper libraries?
```

A single false `AUTHORIZED` or any weakening of an existing
safety verdict disqualifies the challenger. Correctness dominates
every other metric.

## 2. Control vs challenger designs

### Control

The current specialized subsystems:

* `scripts/arsenal_knowledge.py` (`compile_context`,
  `evaluate_authority`) — Knowledge Plane snapshot ingestion and
  authority adjudication.
* `scripts/arsenal_graph.py` (`load_capabilities`, `load_assets`,
  `load_locked_capabilities`, preflight, graph traversal).
* `scripts/arsenal_compile.py` (asset and capability loaders;
  export-plan resolution; lockfile emission).
* `scripts/arsenal_audit.py` (registry + capabilities integrity).
* `scripts/arsenal_bench.py` (capability and qualification
  evaluation).

The control rebuilds its indexes per call.

### Challenger

A prototype Project Intelligence extractor + index that:

1. Reads `arsenal/source-model.json` for fact-source assignment.
2. Opens each owning artifact deterministically and extracts the
   fact at the locator.
3. Stores observations in a disposable index keyed by
   `(fact_id, source.artifact_id, source.repository_sha,
   source.content_sha256)`.
4. Serves a query API that returns observations plus their full
   source provenance.
5. Re-derives every result on demand; never lets the index become
   authoritative.

The prototype must be a thin layer on top of the GC01 source model.
It must not modify canonical artifacts, the source model, or the
loader/validator contracts.

## 3. Tracer scenarios

Each tracer is a specific safety-critical or reuse-critical
question. The challenger must answer it correctly, with provenance,
on the current commit.

### Tracer 1 — KFT-0 authority determination

Current path:

```text
Knowledge Snapshot
  + Python authority adjudicator
    -> AUTHORIZED / NOT_AUTHORIZED / BLOCKED / STALE / UNKNOWN
```

Challenger path:

```text
GC01 source model
  -> extractor opens snapshot fixture (already historical)
  -> extractor reads exact plan / owner / scope / digest facts
  -> IR observations
  -> authority query (same rules as control)
    -> AUTHORIZED / NOT_AUTHORIZED / BLOCKED / STALE / UNKNOWN
```

Adversarial cases the challenger must pass:

* KFT-0 fixture: P1-S02 query returns `NOT_AUTHORIZED` against
  `arsenal/knowledge/fixtures/kft-0-kiln.json`.
* Wrong owner cannot authorize.
* Wrong scope cannot authorize.
* Plan drift invalidates authorization.
* Absent authority evidence returns `UNKNOWN`.
* Stale repository state cannot authorize.

A single false `AUTHORIZED` disqualifies the challenger.

### Tracer 2 — "What should I work on next?"

Current path:

```text
Repository Truth
  + Recon
```

Challenger path:

```text
GC01 source model
  -> extractor opens capability fragments + lockfile + open
     frontier indicators in roadmap / Field Trial docs
  -> IR observations
  -> frontier query
```

The challenger must surface at minimum:

* canonical capability lifecycle states that are not `stable`;
* the locked vs canonical divergence;
* any field-trial observation marked `friction` or `unknown`.

This tracer is a correctness+reuse test, not a safety-critical one.
The challenger may fail to match every nuance of the manual frontier
calculation and still pass this tracer, provided it never fabricates
a frontier that contradicts canonical state.

### Tracer 3 — Context compilation

Current path:

```text
explicit seed entity ids
  -> snapshot.relationships closure
  -> selected knowledge entries
  -> compiled context bundle
```

Challenger path:

```text
GC01 source model
  -> extractor opens canonical knowledge artifacts (snapshot
     schema, knowledge fixtures, capability fragments)
  -> IR observations including relationship edges
  -> closure over IR observations
  -> compiled context bundle
```

The challenger's output for the same query must:

* select the same seed entities;
* expand to the same closure (within the same relationship kinds:
  `supported-by`, `challenged-by`, `governed-by`);
* include the same source provenance for every selected entity;
* record the same excluded count.

A divergence is acceptable only if the challenger's bundle is
strictly safer (e.g. it surfaces an additional contradiction the
control missed). Any silent loss of provenance fails the tracer.

### Tracer 4 — Feature delivery preflight

Current path:

```text
Capability Graph + Gap Preflight
  -> READY / CAPABILITY_GAP / AUTHORITY_GAP / QUALIFICATION_GAP /
     UNKNOWN
```

Challenger path:

```text
GC01 source model
  -> extractor reads capability.identity, capability.lifecycle,
     capability.evaluation, capability.authority, lockfile.pinned-*
  -> IR observations
  -> same rule evaluation as preflight
```

The challenger must reproduce every existing preflight adversarial
case from `scripts/test-arsenal-graph.py`:

* unknown capability;
* dependency not yet available;
* invalid semantic version;
* invalid lifecycle minimum;
* invalid evaluation minimum;
* unsafe remote-cloud authority profile;
* lockfile qualification divergence.

Any verdict weaker than the control fails the tracer.

### Tracer 5 — Governance status

Current path:

```text
hand-maintained status in docs/roadmap/*.md
  + narrative PR-body state claims
```

Challenger path:

```text
GC01 source model
  -> extractor reads governance.artifact-classification.project-arsenal,
     governance.source-assignment.project-arsenal, lifecycle/evaluation
     states
  -> derived status projection
```

The challenger does not need to reproduce the prose. It must
produce:

* the exact artifact count and fact count the source model
  contains;
* the current lifecycle state of each canonical capability;
* the current qualification state of each registered receipt;
* a deterministic digest of the status projection.

A divergence is acceptable only if the challenger reveals a
stale-status defect in the manual surface. The challenger must
NOT silently invent a status the source model does not support.

### Tracer 6 — Incremental verification

Current path:

```text
broad/full evaluation
  -> all suites run unconditionally
```

Challenger path:

```text
changed paths
  -> dependency closure (content-addressed)
  -> affected-only suites
  -> mandatory safe fallback to full rebuild when provenance is
     unclear
```

This tracer is permitted to be partially failing in v0; its goal
is to measure whether incremental computation is honest. The
tracer fails if:

* the affected-only path silently skips a suite whose canonical
  inputs changed;
* the cold rebuild diverges from the affected-only rebuild on any
  fact;
* the safe fallback is slower than the unconditional full rebuild
  without a measured win.

## 4. Measurements

For each tracer, record:

### Correctness

* verdict equivalence (`PASS` / `FAIL` / `STRICTER`);
* false positives (none allowed);
* false negatives (must be enumerated and justified);
* provenance quality (every result references its source);
* explanation quality (the challenger explains its verdict with
  pointers to specific facts).

### Performance

* cold latency (clean repo, no cache);
* warm latency (cached);
* files opened;
* bytes parsed;
* JSON documents parsed;
* graph scans;
* repeated lookup reconstruction sites;
* cache hit rate when content-addressed caching is active.

### Context efficiency

* tokens emitted into compiled-context bundles;
* irrelevant-context share;
* repeated repository reads;
* rediscovery operations avoided.

### Engineering complexity

* implementation LOC for the challenger;
* manually synchronized representations remaining;
* source locations edited for one semantic change;
* duplicate rules;
* duplicate schemas;
* duplicate fixtures.

### Maintainability

* deterministic rebuildability (does deleting the IR and
  rebuilding from canonical state produce the same digest?);
* inspectability;
* explainability;
* ease of migration;
* failure isolation;
* dependency burden.

### CI efficiency

* evaluations avoided safely;
* affected-only execution coverage;
* full fallback cost;
* incremental cache effectiveness.

### Durability

* continuation quality after clearing context;
* ability to reconstruct previous project state from canonical
  artifacts alone;
* resistance to stale narrative claims;
* quality of exact-state handoff across sessions.

## 5. Disqualification conditions (mandatory)

The challenger fails the shootout and the hypothesis is rejected
if any of the following occurs:

1. Any false `AUTHORIZED`, false `READY`, false `QUALIFIED`, or
   false `APPROVED` verdict where the control produces the correct
   rejection.
2. Any safety verdict corpus (KFT-0 NOT_AUTHORIZED, preflight
   AUTHORITY_GAP, qualification QUALIFIED, etc.) becomes weaker
   than the specialized implementation.
3. Any IR query result without a complete source provenance.
4. Any IR query result that contradicts the canonical artifact it
   claims to observe.
5. Deleting the IR loses authoritative project information.
6. Cold rebuild cost exceeds the documented budget (initial
   budget: a 10x multiple of a no-op Python import).
7. Index state silently diverges from canonical state on a real
   branch where the divergence is not surfaced by the rebuild.
8. Maintaining extractors exceeds the duplication they replace
   (LOC delta positive with no reuse gain).
9. The challenger becomes a god object that owns canonical
   artifacts, knowledge entities, authority, capability composition,
   evidence, or execution selection.

## 6. Graduating conditions (mandatory)

The challenger graduates and Project Intelligence is accepted only
if all of the following are evidenced:

1. At least two existing subsystems (KFT-0 authority, Capability
   Gap Preflight, or Knowledge compile-context) demonstrably
   consume IR observations and stop duplicating the extraction.
2. No canonical authority moves into the IR.
3. Deterministic full rebuild succeeds; deleting and recreating
   the IR is documented as a no-op procedure.
4. Exact-source provenance survives every transformation.
5. The existing safety verdict corpus remains verdict-equivalent
   or stricter across all adversarial cases.
6. Repeated parsing sites measurably decrease (quantify with a
   simple grep-based metric).
7. Context rediscovery measurably decreases on representative
   Knowledge Plane compile-context calls.
8. Manually synchronized representations decrease (the
   `docs/roadmap/*.md` "now / next" tables move toward derived
   projections over time).
9. Explanation quality on Authority / Readiness verdicts is
   unchanged or improves.
10. Cold rebuild cost remains under the documented budget.
11. Tracer 6 (incremental verification) either passes cleanly or
    is honestly characterized as "premature" with a kill date.

If any condition fails, the experiment is killed and the
specialized architecture is retained (Strategy A).

## 7. Tracer evidence format

For each tracer, the shootout MUST produce:

* the exact commit SHA tested;
* the canonical inputs read;
* the exact control verdict with citation to the source line that
  produced it;
* the exact challenger verdict with citation to the source line
  that produced it;
* a structured PASS / FAIL / STRICTER comparison;
* the measurement table for that tracer.

The evidence belongs in `docs/architecture/shootout-evidence/`
once the shootout is executed. Until then, this document is the
specification only.

## 8. Out of scope

The shootout does NOT decide:

* whether to commit a particular IR schema (decided by the slice
  that uses it, if it survives);
* whether to adopt SQLite specifically (Strategy C may use any
  local disposable index; the substrate is the IR, not the
  database);
* whether PI-00, PI-01, PI-02 become permanent slice names;
* whether any existing roadmap item should be reordered beyond
  the reordering already captured in
  `docs/roadmap/capability-system.md`.

## 9. Authority to run the shootout

The shootout is implementation work. It must be authorized the same
way any new slice is authorized:

* by the program rules in
  `docs/roadmap/capability-system.md`;
* by the ownership boundaries in
  `engineering/doctrine/ARCHITECTURE.md`;
* by the kill criteria in
  `docs/architecture/project-intelligence.md`.

This document does not grant that authority. It describes the
experiment that must pass for the substrate to be adopted.
