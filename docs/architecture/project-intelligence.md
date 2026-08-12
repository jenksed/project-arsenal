# Project Intelligence — Architecture Note

Status: hypothesis under investigation. Not yet accepted.

Scope: the question of whether Project Arsenal should grow a shared
deterministic substrate that extracts exact-state observations from
canonical artifacts and serves authority-aware queries/context to
higher-level capabilities, rather than continuing to rebuild similar
indexes inside each subsystem.

Audience: anyone returning to this slice later. It must remain
understandable in isolation.

## 1. Epistemic model (non-negotiable)

This note preserves the distinction the GC01 source model already
established:

```text
Source Model        : which artifact owns this fact and where its
                      source artifact lives. (index, not value)
Owning Artifact     : what the authoritative / current / historical
                      fact actually is. (canonical truth)
Derived Arsenal IR  : what Arsenal OBSERVED at a specific repository
                      state. (disposable derived state)
Project Index       : rebuildable structured access to those
                      observations. (disposable, non-authoritative)
Projection          : any output computed from observations or indexes.
                      (presentation, not authority)
```

Authority rule, absolute:

> A derived representation must never gain authority merely because
> it is easier to query.

Deleting any future IR, index, or cache must result in zero loss of
authoritative project information. The system must always be
reconstructable from canonical repository state.

## 2. Problem statement

Project Arsenal's conceptual architecture is strong:

* Repository Truth establishes observable repository state.
* Recon models uncertainty, decisions, blockers, and frontier.
* Context Boundaries govern continuation, handoff, clearing,
  delegation, and compaction.
* ARS-09 Knowledge Plane models typed engineering knowledge and
  exact-state lifecycle.
* Capability Graph models explicit composition and preflight.
* Trust & Authority protects provenance, permissions, and exact-
  state authorization.
* Evidence / Flight Recorder captures durable proof.
* Reality Budget selects the lowest sufficient execution substrate.
* Bench evaluates both capability outcomes and evaluator quality.
* Compiler + Asset Registry provide deterministic distribution and
  provenance.
* GC01 Governance Compression names canonical sources and ownership.

But the implementation architecture risks fragmentation as each new
concept acquires its own JSON shape, schema, Python evaluator,
fixtures, and CI workflow. The path is:

```text
new concept
  -> new JSON representation
  -> new schema
  -> new Python evaluator
  -> new fixtures
  -> new CI workflow
  -> new projection of facts Arsenal already knew elsewhere
```

GC01's source model is the first deliberately shared primitive for
"which artifact owns this fact?" It does not duplicate the fact
itself. The hypothesis this note investigates is whether the same
discipline (source-bound, derived, disposable) can absorb a second
class of repeated work: rebuilding indexes of canonical facts inside
multiple subsystems.

## 3. Concrete duplicated work observed in the current tree

The investigation started with the GC01 source model and walked the
scripts that already read the same canonical artifacts. Several
subsystems independently walk the same directories and rebuild
similar dictionaries on every invocation:

### 3.1 Capability fragments

`scripts/arsenal_audit.py` (`load_assets`), `arsenal_graph.py`
(`load_capabilities`, `load_assets`), `arsenal_compile.py`
(`load_assets`, `load_capability_for_manifest`), and
`arsenal_bench.py` (`load_capability`) all perform the same operation:

```text
walk arsenal/capabilities/*.json
  -> parse each fragment
  -> index by capability.id
```

The four implementations agree on shape today only because they
were written against the same capability contract. If the contract
gains a new required field, four call sites must be updated
independently. None of them uses the GC01 source model to discover
which facts a fragment owns.

### 3.2 Asset registry

`arsenal_audit.py`, `arsenal_graph.py`, and `arsenal_compile.py`
each merge `arsenal/registry.json` with `arsenal/registry.d/*.json`
into a single `dict[id, asset]` keyed by asset id. The merger logic
duplicates across files. The merged view is canonical today only by
convention; nothing in the source model enforces it.

### 3.3 .arsenal.lock

`arsenal_compile.py` builds it. `arsenal_graph.py`
(`load_locked_capabilities`) re-parses it for preflight.
`arsenal_bench.py` (lifecycle / qualification validation) and
`test-arsenal-shared.py` parse it again. The lockfile's "pinned"
facts are already declared in the source model
(`lockfile.pinned-capability-*`), so an IR built on top of GC01
would derive the lockfile facts from the same single extraction
that drives preflight.

### 3.4 Capability + lockfile + asset cross-references

Capability Gap Preflight (`arsenal_graph.py`) walks three sources
and joins them by id. This join is recomputed at every preflight
call. The same join is conceptually needed by Bench qualification
and by the Intent Compiler (ARS-10). Today each consumer rebuilds
it.

### 3.5 Repeated JSON parsing and digest computation

Every load site re-reads the same files and re-parses them. Every
canonical fact whose digest Arsenal needs for provenance or receipt
binding is hashed again by the script that asks for it. None of
these scripts shares an extracted-fact cache.

### 3.6 What this is NOT

This is **not** textual resemblance. Several subsystems share
vocabulary (capability, lifecycle, evaluation, authority) but mean
different things in each. The duplication this note names is
specifically: rebuild-the-same-dictionary-from-the-same-canonical-
files-on-every-invocation. Vocabulary overlap is not the problem;
fact duplication is not the problem; the problem is index
reconstruction.

## 4. Proposed substrate (hypothesis, not commitment)

```text
CANONICAL REPOSITORY ARTIFACTS  (the truth)
            |
            v
     GC01 SOURCE MODEL           (which artifact owns which fact)
            |
            v
  deterministic extraction      (one small extractor per fact kind)
            |
            v
       ARSENAL IR                (derived exact-state observations)
            |
            v
 DISPOSABLE PROJECT INDEX       (rebuildable, repo-SHA-bound,
                                  gitignored, non-authoritative)
            |
     +------+------+------+
     |      |      |      |
     v      v      v      v
  Repository  Context   Capability   Governance
   Truth      Broker    / Authority  projection
                         rule
            |
            v
          AGENT
```

The IR record shape is deliberately minimal. It is not a
mini-database schema. The minimal viable record carries:

* the fact identity declared in the source model;
* the owning artifact id, path, and content digest;
* the locator (where in the artifact the value lives);
* the extracted value;
* the repository SHA at extraction time;
* the ownership and state_role declared in the source model.

A conceptual record may look like:

```json
{
  "fact_id": "capability.current-lifecycle",
  "value": "testing",
  "source": {
    "artifact_id": "arsenal.capability-fragments",
    "path": "arsenal/capabilities/repository-truth.json",
    "pointer": "capability.lifecycle",
    "repository_sha": "<git rev-parse HEAD>",
    "content_sha256": "<sha256 of the artifact bytes>"
  },
  "semantics": {
    "ownership": "arsenal-distribution",
    "state_role": "normative"
  }
}
```

The schema is not committed in this slice. The architecture shootout
is the right place to converge on a concrete shape; committing one
prematurely would be premature schema authority.

## 5. Boundaries

The proposed substrate does NOT own:

* canonical artifacts (those remain authoritative in place);
* the source model (GC01 remains the single fact-source registry);
* capability composition semantics (Capability Graph keeps that);
* authority and trust semantics (Trust & Authority keeps that);
* knowledge entities, decisions, requirements, unknowns (Knowledge
  Plane keeps that);
* proof receipts (Evidence / Flight Recorder keeps that);
* execution substrate selection (Reality Budget keeps that);
* empirical evaluation (Bench keeps that);
* compilation, distribution, lockfile (Compiler keeps that);
* intent interpretation (the model layer keeps that).

Project Intelligence owns one thing and one thing only:

> Deterministic extraction of source-bound observations from
> canonical artifacts, plus rebuildable structured access to them,
> plus authority-aware query/context API over those observations.

If a candidate capability cannot explain itself without owning one
of the above, it does not belong in Project Intelligence.

## 6. Authority semantics

Every Project Intelligence output must carry:

* the fact identity it claims to observe;
* the owning artifact path and content digest;
* the repository SHA at extraction time.

A query result that omits its source pointer is a fail-closed
error. A result whose source digest does not match the current
artifact bytes is stale and must not be served as current
authority. This is the same provenance discipline GC01 already
imposes on the source model itself; it must propagate one layer
down into the IR.

The IR must never answer a question that canonical artifacts would
have answered differently. When the canonical answer is ambiguous,
the IR is ambiguous in the same way. When canonical artifacts say
"unknown", the IR says "unknown". The IR is not a smoother
abstraction over the truth; it is a faithful mirror.

## 7. Alternatives considered

### Strategy A — Continue specialized subsystems

Status quo. Each Arsenal subsystem keeps its own loaders and
indexes. Improves only through shared helper libraries.

Pros: lowest blast radius; no new substrate; no migration risk.

Cons: every new cross-cutting question duplicates another loader;
every contract evolution touches every loader; context compilation
remains per-call; the path from "fact exists in a canonical
artifact" to "agent has the fact" remains per-subsystem.

Verdict: this remains the fallback if the Architecture Shootout
fails. The slice does not reject it.

### Strategy B — Derived Arsenal IR (the hypothesis)

Deterministic, exact-state intermediate representation generated
from canonical artifacts via the GC01 source model.

Pros: eliminates redundant re-parsing; one place to add a new fact
kind; provenance travels with every observation; rebuildable from
canonical state.

Cons: a new substrate adds maintenance surface; risk of becoming a
god object; risk of accidentally gaining authority; performance
must be honestly measured.

Verdict: investigate via Architecture Shootout.

### Strategy C — Disposable Project Intelligence Index

A rebuildable local SQLite (or equivalent) index of observations
plus relationships.

Pros: cheap primitives for joins, recursive CTEs, JSON functions;
FTS available when supporting prose recall becomes a need; local
and rebuildable.

Cons: temptation to make it authoritative; temptation to depend on
its durability; temptation to let the schema grow faster than the
extractor discipline.

Verdict: only viable IF the IR remains authoritative and the index
remains disposable. The shootout must prove this.

### Strategy D — Declarative rule layer

Express AUTHORIZED / READY / QUALIFIED / APPROVED verdicts as
facts + rules over the IR.

Pros: separates observation from decision; rules are testable;
multiple rule sets can be tried.

Cons: rule engines grow; SQL rules and a small Python rules layer
are sufficient for v0; OPA-class machinery is unjustified.

Verdict: investigate inside the shootout only with the smallest
possible rule abstraction (Python function + inputs dict).

### Strategy E — Event / observation ledger

Represent important transitions as typed observations.

Pros: natural for lifecycle, qualification, trust, evidence.

Cons: most current canonical artifacts are not event-shaped; the
canonical source is already an event-shaped-or-state-shaped
decision per artifact; forcing a uniform ledger would either lose
information or duplicate it.

Verdict: not adopted. The IR can hold observations whose locator
is an event record if a future capability earns the need.

### Strategy F — Content-addressed incremental evaluation

Treat extractors like build actions:

```text
inputs
+ extractor version
+ configuration
= digest
-> cached output
```

Affected-only re-evaluation when canonical sources change;
mandatory safe fallback to full rebuild when provenance is unclear.

Pros: large speedup for CI loops and incremental developer use.

Cons: dependency tracking is itself a complex surface; full rebuild
must remain cheap enough that the cache is an optimization, not a
requirement.

Verdict: investigate only AFTER Strategy B passes the shootout.

## 8. What Project Intelligence is NOT

* Not a knowledge graph. Knowledge is the Knowledge Plane's job.
  Project Intelligence holds exact-state observations; the
  Knowledge Plane holds typed entities and relationships.

* Not RAG. No chunking, no embedding, no model-driven truth. LLM
  retrieval may eventually identify candidate prose relevance;
  Project Intelligence remains deterministic structural intelligence.

* Not a context broker on its own. Context Broker (when it exists)
  consumes the IR plus Knowledge subgraphs plus Bench evidence and
  produces bounded compiled context. The IR is one input among
  several.

* Not a governance database. GC01 source model is the
  fact-source registry. Project Intelligence is the derived
  observation layer below it.

* Not a rule engine. A tiny rules surface may exist; it must not
  become a competitor to OPA or a general policy layer.

* Not an event store. If a future capability needs events, the IR
  may index event-shaped locators; the IR itself is not a ledger.

## 9. Risks

* The IR becomes authoritative. **Mitigation:** the IR is gitignored;
  every consumer must prove it can rebuild from canonical state;
  every IR query result carries source provenance; if the IR is
  deleted, no test passes that depended on it.

* The IR schema grows faster than the extractor discipline.
  **Mitigation:** no record type is added until at least two
  consumers need it; the source model remains the only source of
  truth for "which fact exists."

* Substrate coupling replaces subsystem coupling. **Mitigation:**
  every IR consumer must declare which extractor produced each
  observation; deleting an extractor must break the consumers
  loudly.

* Index divergence. **Mitigation:** the IR is rebuildable; CI runs
  the rebuild against every PR; stale IR is detected by digest
  mismatch.

* Cold rebuild cost dominates. **Mitigation:** full rebuild must
  remain under a documented time budget; content-addressed caches
  are an optimization layered on top, never a requirement.

* Context efficiency fails to improve. **Mitigation:** measure
  context bytes and rediscovery events before and after, with a
  control; if the IR does not reduce them, abandon it.

## 10. Success criteria (graduation gates)

The IR substrate graduates from experimental to accepted only if
all of the following are evidenced by the Architecture Shootout:

* At least two existing subsystems consume shared exact-state
  observations and stop duplicating that extraction.
* No canonical authority moves into the IR.
* Deterministic full rebuild succeeds on every supported
  environment and is bounded by a documented budget.
* Exact-source provenance survives every transformation the IR
  performs.
* The existing safety verdict corpus (KFT-0 NOT_AUTHORIZED,
  preflight AUTHORITY_GAP, qualification QUALIFIED, etc.) remains
  verdict-equivalent or stricter.
* Repeated parsing and JSON-load sites measurably decrease.
* Context rediscovery measurably decreases on representative
  Knowledge Plane compile-context calls.
* Manually synchronized representations decrease.
* Explanation quality on Authority / Readiness verdicts is
  unchanged or improves.
* Deleting and recreating the IR is documented as a no-op
  procedure.

## 11. Kill criteria

The IR substrate is killed and Strategy A is retained if any of
the following is observed:

* The IR cannot be deleted without losing authoritative project
  information.
* The IR cannot maintain exact source provenance through every
  transformation.
* Authority evaluation becomes less explainable.
* Any safety verdict corpus becomes weaker than the specialized
  implementation.
* Index state can silently diverge from canonical repository
  state.
* Substantial complexity is added without measurable reuse.
* Context efficiency does not improve.
* Maintaining extractors exceeds the duplication they replace.
* Shared representation creates coupling that makes subsystem
  evolution harder.
* Cold rebuild cost is unacceptable.
* Dependency tracking becomes more complex than full evaluation.
* The current specialized architecture proves simpler and
  sufficient at the relevant scale.

Project Arsenal must be willing to kill its own architectural
experiment.

## 12. Experiment boundary

Until the Architecture Shootout passes:

* The shared Project Intelligence substrate is experimental.
* The source-of-truth architecture (canonical artifacts + GC01
  source model + per-subsystem loaders) remains the operational
  truth.
* Any new implementation that depends on the IR is conditional on
  the shootout result.
* Roadmap items that name PI-00 / PI-01 / PI-02 are experimental
  work packages; they do not commit to adoption.

## 13. Relationship to GC02 and further Governance Compression

GC02 and any further governance compression must ask:

> Is the proposed governance artifact genuinely new canonical
> information? Or is it merely a projection of information Arsenal
> can deterministically derive from existing canonical sources?

If the latter, prefer a generated/queryable projection built from
the IR rather than another manually synchronized governance surface.
GC01 should remain a deliberately narrow fact-source registry, not
the seed of a giant centralized governance document.

## 14. References

* `engineering/doctrine/ARCHITECTURE.md` — ownership/state-role
  doctrine; boundary guarantees 1–7.
* `docs/roadmap/post-pr-24-deferred-architecture.md` — Track A
  ordering; rejection of NLP-first governance; projection priority.
* `docs/roadmap/capability-system.md` — ARS-NN slice definitions
  the IR must remain compatible with.
* `arsenal/source-model.json` and the GC01 surgical repair at the
  current head — the fact-source registry on which the IR would be
  built.
* `docs/architecture/architecture-shootout.md` — the experiment
  that must prove or falsify this hypothesis.
* `docs/roadmap/capability-system.md` — Project Intelligence
  package definitions (PI-00, PI-01, PI-02).
