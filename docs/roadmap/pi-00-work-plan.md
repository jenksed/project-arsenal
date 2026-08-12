# PI-00 — Minimal IR Tracer (Work Plan)

Status: work plan. NOT AUTHORIZED.

This document prepares the implementation package that can
subsequently be explicitly authorized as PI-00. It does not grant
that authorization. If Project Arsenal's current governance
requires an owner-issued exact-state authorization record, this
boundary must be preserved at merge time.

Companion documents:

* `docs/architecture/project-intelligence.md` — hypothesis, scope,
  authority semantics, alternatives, risks, success and kill
  criteria.
* `docs/architecture/architecture-shootout.md` — the experiment
  PI-00 feeds into.
* `docs/roadmap/project-intelligence.md` — the redirection that
  introduced PI-00 / PI-01 / PI-02.

## 1. Work identity

PI-00 — minimal IR tracer.

## 2. Objective

Prove that GC01's fact-source assignments drive deterministic
extraction of a small heterogeneous set of exact-state facts from
canonical artifacts, while preserving source provenance, without
introducing a second source of truth and without transferring
authority to the derived representation.

PI-00 is not:

* a complete Project Intelligence implementation;
* a database;
* a context engine;
* a rules engine;
* a graph database;
* an event ledger;
* a replacement for ARS-09;
* a replacement for Capability Graph;
* a general repository parser.

## 3. Base state

The PI-00 implementation will be designed against the post-GC01
exact head. At this document's preparation time that head is the
GC01 surgical repair commit on the `agent/governance-artifact-roles`
branch.

The PI-00 work plan itself is committed on
`agent/project-intelligence-redirection` derived from that GC01
base.

## 4. Governing architecture

* `docs/architecture/project-intelligence.md`
* `docs/architecture/architecture-shootout.md`
* `docs/roadmap/project-intelligence.md`
* `arsenal/source-model.json` (GC01 foundation; the IR extractor
  reads this as the single source of fact-source assignment).

## 5. Allowed implementation scope

* one small extractor/runtime module
  (`scripts/pi_ir.py` or equivalent);
* one dedicated characterization/adversarial test module
  (`scripts/test-pi-ir.py` or equivalent);
* minimal test fixtures only if necessary for adversarial cases
  not already covered by the existing suites;
* this work plan document;
* small additions to `arsenal/source-model.json` ONLY if a PI-00
  tracer fact requires an artifact whose ownership is not yet
  declared in GC01 (must be authorized separately; not in PI-00
  default scope).

PI-00 must NOT add, modify, or remove:

* SQLite (or any database);
* any production consumer of the IR;
* the Knowledge Plane implementation;
* the Capability Gap Preflight implementation;
* the Compiler implementation;
* the Bench implementation;
* the Repository Truth implementation;
* any governance projection;
* a generalized IR schema file committed to the repository;
* an event ledger;
* a rule engine;
* a generalized source-model widening.

## 6. Tracer facts

The tracer selects approximately 8–12 existing fact identities
already registered through GC01. The selection maximizes
architectural information across:

* **source shape** (direct file vs. artifact family pattern);
* **state role** (`normative`, `derived`, `historical` where
  available);
* **ownership area** (multiple Arsenal concerns);
* **value shape** (scalar, enum, object, list, digest/identity,
  binding/reference).

The canonical fact IDs below are taken from the current
`arsenal/source-model.json` and are NOT invented for convenience.

| # | fact_id                           | owner_artifact                       | value shape      | state_role   |
|---|-----------------------------------|--------------------------------------|------------------|--------------|
| 1 | `protocol.vocabulary`             | `arsenal.protocol`                   | (Python module)  | normative    |
| 2 | `governance.vocabulary`           | `arsenal.governance-vocabulary`      | (Python module)  | normative    |
| 3 | `capability.identity`             | `arsenal.capability-fragments`       | string           | normative    |
| 4 | `capability.current-lifecycle`    | `arsenal.capability-fragments`       | enum             | normative    |
| 5 | `capability.current-evaluation`   | `arsenal.capability-fragments`       | enum             | normative    |
| 6 | `capability.authority`            | `arsenal.capability-fragments`       | object           | normative    |
| 7 | `asset.identity`                  | `arsenal.registry`                   | list/object      | normative    |
| 8 | `distribution.supported-targets`  | `arsenal.distribution.compiler.targets` | list         | normative    |
| 9 | `distribution.compiler.export-plan` | `arsenal.distribution.compiler.export-plan` | object   | normative    |
| 10| `lockfile.pinned-capability-lifecycle` | `arsenal.competence-lockfile`    | object           | normative    |
| 11| `qualification-receipt.qualification-verdict` | `arsenal.bench.distribution-qualification-receipt` | enum/object | historical    |
| 12| `distribution.skill-snapshot`     | `arsenal.distribution.skill`         | path-pattern     | derived      |

Notes on diversity:

* `protocol.vocabulary` and `governance.vocabulary` exercise the
  case where the "value" lives in a Python module rather than a
  JSON file. PI-00 must either record a deterministic
  representation of that value (e.g. the sorted set of vocabulary
  tokens) or record a "non-extractable" pointer. If non-extractable
  for any reason, the tracer may substitute a fact with a JSON
  representation (see §10 stop conditions).
* `asset.identity` exercises a fact whose value is a list/object
  rather than a scalar.
* `distribution.skill-snapshot` is owned by an artifact registered
  with a `path_pattern`, exercising the artifact-family case.
* `qualification-receipt.qualification-verdict` is owned by a
  `historical` artifact, exercising the non-normative state role.

## 7. Minimal representation

PI-00 may use a small internal Python representation sufficient
for the tracer. It should conceptually retain at least:

```
fact_id
subject or binding identity (where applicable)
observed value (canonical, not a copy)
owning artifact identity (matches source model)
source path (matches source model)
source locator (matches source model `locator`)
repository SHA (single canonical hash)
source content digest (sha256 of the owning artifact's bytes)
ownership classification (matches source model)
state_role (matches source model)
```

A field that is not actually needed to prove the tracer
hypothesis must be challenged before it is added. The
representation is internal; PI-00 does not commit a generalized
IR schema file.

## 8. Deterministic extraction

Extraction proceeds conceptually as:

```
fact_id
  ↓
source-model lookup (owner_artifact, path / path_pattern, locator)
  ↓
canonical source resolution (single owning artifact)
  ↓
deterministic value extraction (no model; no semantic search;
  no embeddings; no heuristic prose interpretation)
  ↓
source-bound observation
```

If a selected fact cannot be extracted deterministically, the
implementation must either:

* select another tracer fact;
* record that fact class as a PI-00 limitation requiring later
  design (e.g. vocabulary-in-Python-module extraction may require
  a separate "module-fingerprint" mechanism that is out of scope
  for PI-00).

It must NOT hide nondeterminism.

## 9. Repository-state binding

Every PI-00 observation must be unambiguously bound to:

```
repository SHA
source artifact identity
source path
source locator
source content digest
```

PI-00 MUST reuse Arsenal's existing digest utilities in
`scripts/arsenal_io.py` rather than create a new hashing
convention.

## 10. Acceptance criteria

PI-00 passes when every one of the following is evidenced:

1. the source model identifies each fact's owner (one test per
   fact);
2. the extractor opens the actual canonical source file (one
   test per fact);
3. exact value extraction succeeds for every selected fact (one
   test per fact);
4. the source locator survives the extraction (one test per
   fact);
5. the repository SHA survives the extraction (one test);
6. the source content digest survives the extraction (one test
   per fact);
7. two runs against the same commit produce observationally
   identical digests (deterministic rebuild, one test);
8. deleting the IR and rebuilding from canonical state produces
   the same digest (disposable rebuild, one test);
9. an in-memory mutation of an observation cannot override
   canonical truth on a fresh extraction (contradiction
   protection, one test);
10. no consumer in the repository depends on PI-00 (a
    structural check: PI-00 imports nothing from production
    modules and production modules import nothing from PI-00);
11. existing Arsenal safety/evidence suites remain green
    (`arsenal_source_validate.py`, `test-arsenal-governance.py`,
    `test-arsenal-shared.py`, `arsenal_audit.py`,
    `arsenal_graph.py validate`, `arsenal_compile.py validate`,
    `arsenal_bench.py validate`, and the negative test suites);
12. generated outputs (`.arsenal.lock`, `distribution/`,
    qualification receipts, generated manifests) remain
    unchanged unless explicitly expected.

## 11. Stop conditions

PI-00 is killed (and the experimental lane reports the kill
explicitly) if any of the following is observed:

* authoritative values must be duplicated manually to make the
  tracer pass;
* exact provenance cannot be retained through every
  transformation;
* nondeterministic extraction is required to make a tracer fact
  extractable;
* the design requires a generalized IR schema before the tracer
  is finished;
* database persistence becomes necessary merely to finish PI-00
  (PI-01's job);
* implementation starts absorbing authority, knowledge semantics,
  capability composition, evidence semantics, or execution
  selection;
* an existing safety contract must be weakened for the tracer to
  pass;
* the IR cannot be deleted and recreated from canonical state in
  under the documented time budget;
* a tracer fact cannot be extracted because its owning artifact
  is not registered in GC01 and the gap is hidden rather than
  declared.

## 12. Verification requirements

Before declaring PI-00 complete, run the following and record
exact outputs:

```
python3 scripts/arsenal_source_validate.py
python3 scripts/test-arsenal-governance.py
python3 scripts/test-arsenal-shared.py
python3 scripts/arsenal_audit.py
python3 scripts/arsenal_graph.py validate
python3 scripts/arsenal_compile.py validate
python3 scripts/arsenal_compile.py verify
python3 scripts/arsenal_bench.py validate
python3 scripts/test-arsenal-bench.py
python3 scripts/test-arsenal-graph.py
python3 scripts/test-arsenal-knowledge.py
python3 scripts/test-arsenal-observe.py
python3 scripts/test-arsenal-substrate.py
python3 scripts/test-arsenal-trust.py
python3 scripts/test-arsenal-dagger.py

git diff --check
```

Confirm `.arsenal.lock` and `distribution/` byte-identical across
the PI-00 characterization run.

## 13. Authorization state

PI-00 is **NOT AUTHORIZED** at the time this work plan is
written.

This document prepares the implementation package. It does not
authorize implementation. The next owner decision must either:

* explicitly authorize PI-00 (with the owner identity, the
  authorization record's exact digest, and the binding to the
  exact commit SHA that the authorization was issued against); or
* amend the experimental sequence; or
* kill the experiment entirely.

The PR description that introduces PI-00 must restate this
authorization state explicitly and refuse to mark the PR as
ready-for-review until the owner decision is recorded.

## 14. PI-02 prerequisite matrix

The architecture shootout's six tracers depend on GC01 source-
model coverage at varying levels. The matrix below records what
each tracer needs, what is currently registered, and what is
missing.

### Tracer 1 — KFT-0 authority determination

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| snapshot identity / digest                         | partial     | `knowledge.snapshot-fixture-kft-0` → fixture JSON               | snapshot-internal structure not declared |
| source records (each source's owner, scope, base SHA, digest) | no  | (inside snapshot JSON)                                            | declare as GC01 facts                    |
| knowledge entities (each entity, its source_refs)  | no          | (inside snapshot JSON)                                            | declare as GC01 facts                    |
| queries (each implementation-authority query)      | no          | (inside snapshot JSON)                                            | declare as GC01 facts                    |
| authorization records (owner/scope/digest binding) | no          | (inside snapshot JSON)                                            | declare as GC01 facts                    |
| evidence bindings (subject -> Evidence)             | no          | (inside snapshot JSON)                                            | declare as GC01 facts                    |

**Coverage gap.** Tracer 1 cannot be answered by GC01-driven
extraction alone on the current head. A prerequisite package
(`PI-PREQ-01: Extend source-model coverage for Architecture
Shootout tracers`) must declare the snapshot-internal fact
classes as GC01 facts before the tracer can run.

### Tracer 2 — "What should I work on next?"

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| capability lifecycle / evaluation states           | yes         | `capability.current-lifecycle`, `capability.current-evaluation` | none                                     |
| lockfile pin vs. canonical divergence              | yes         | `lockfile.pinned-capability-*`                                   | none                                     |
| qualification state of receipts                    | yes         | `qualification-receipt.qualification-verdict`                     | none                                     |
| Field Trial / roadmap frontier                     | no          | (narrative markdown)                                              | out of scope; tracer may skip this axis  |

**Coverage COMPLETE for the capability / qualification axis.**
Field Trial frontier is narrative and is intentionally outside
the tracer scope.

### Tracer 3 — Context compilation

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| snapshot structure (sources, knowledge, queries, relationships) | no  | (inside snapshot JSON)                                            | declare as GC01 facts                    |
| typed relationship edges (supported-by, challenged-by, governed-by) | no | (inside snapshot JSON)                                     | declare as GC01 facts                    |
| query seeds                                        | no          | (inside snapshot JSON)                                            | declare as GC01 facts                    |

**Coverage gap.** Same prerequisite as Tracer 1
(`PI-PREQ-01`).

### Tracer 4 — Feature delivery preflight

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| capability identity                                | yes         | `capability.identity`                                              | none                                     |
| capability current lifecycle                       | yes         | `capability.current-lifecycle`                                     | none                                     |
| capability current evaluation                      | yes         | `capability.current-evaluation`                                    | none                                     |
| capability authority                               | yes         | `capability.authority`                                             | none                                     |
| capability execution                               | yes         | `capability.execution`                                             | none                                     |
| capability evidence                                | yes         | `capability.evidence`                                              | none                                     |
| asset identity                                     | yes         | `asset.identity`                                                   | none                                     |
| lockfile pin (lifecycle / evaluation / identity / target-export) | yes | `lockfile.pinned-capability-*`                              | none                                     |
| enabled targets                                    | yes         | `project.enabled-targets`                                          | none                                     |
| supported targets                                  | yes         | `distribution.supported-targets`                                   | none                                     |
| compiler export plan                               | yes         | `distribution.compiler.export-plan`                                | none                                     |
| capability current target-export                   | yes         | `capability.current-target-export`                                 | none                                     |

**Coverage COMPLETE.** Tracer 4 can be answered by GC01-driven
extraction on the current head.

### Tracer 5 — Governance status

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| governance artifact-classification (project-arsenal) | yes      | `governance.artifact-classification.project-arsenal`               | none                                     |
| governance source-assignment (project-arsenal)    | yes         | `governance.source-assignment.project-arsenal`                     | none                                     |
| per-capability lifecycle / evaluation              | yes         | `capability.current-*`                                             | none                                     |
| per-receipt qualification verdict                  | yes         | `qualification-receipt.qualification-verdict`                      | none                                     |
| per-fact digest                                    | yes         | (computed at observation time from canonical artifact bytes)      | none                                     |

**Coverage COMPLETE.**

### Tracer 6 — Incremental verification

| required fact class                                | registered? | current owner / source                                            | missing work                            |
|----------------------------------------------------|-------------|-------------------------------------------------------------------|------------------------------------------|
| dependency graph between extractors                | no          | (none)                                                             | PI-08; out of scope for PI-02 prerequisite |

**Coverage absent.** Tracer 6 is permitted to be partially
failing in v0; the architecture shootout records this.

### Prerequisite summary

| Tracer | GC01 coverage     | prerequisite before PI-02 |
|--------|-------------------|---------------------------|
| 1      | gap (snapshot internal) | `PI-PREQ-01`           |
| 2      | mostly complete   | none (Field Trial narrative out of scope) |
| 3      | gap (snapshot internal) | `PI-PREQ-01`           |
| 4      | complete          | none                      |
| 5      | complete          | none                      |
| 6      | absent            | PI-08 (out of scope)      |

`PI-PREQ-01` is defined as a narrow future package:

> `PI-PREQ-01 — Extend source-model coverage for Architecture
> Shootout tracers`
>
> Objective: declare the Knowledge Snapshot internal fact classes
> (sources, knowledge entities, queries, relationships,
> authorization records, evidence bindings) as GC01 facts so
> tracers 1 and 3 can be answered by GC01-driven extraction.
> Scope: additions to `arsenal/source-model.json` + new artifact
> entries for the snapshot-internal schemas (already declared as
> `protocol.knowledge-snapshot-schema`) and any required
> locators; small updates to characterization tests.
> Authorization: must be authorized separately from PI-00;
> PI-PREQ-01 is NOT in PI-00 default scope.

PI-00 does NOT include PI-PREQ-01. PI-00 should be runnable
against the current GC01 head using only the §6 tracer facts.

## 14. Successor slices (conditional)

These are NOT committed by this work plan. They remain candidate
follow-ons only if PI-02 passes the architecture shootout:

* PI-01 — disposable project intelligence index;
* PI-02 — architecture shootout;
* PI-03 — shared exact-state query API;
* PI-04 — Context Broker tracer;
* PI-05 — Repository Truth integration;
* PI-06 — Governance projection integration;
* PI-07 — Capability / authority rule tracer;
* PI-08 — content-addressed dependency graph;
* PI-09 — affected-only evaluation;
* PI-10 — event / observation ledger where justified.

## 15. PI-PREQ-01 — source-model coverage prerequisite (separate package)

PI-02 tracer 1 (KFT-0 authority) and tracer 3 (context
compilation) require source-model coverage of the Knowledge
Snapshot internal structure. The current GC01 source model
registers the snapshot's existence (`knowledge.snapshot-fixture-kft-0`)
but not the typed fact classes inside the snapshot (sources,
knowledge entities, queries, relationships, authorization
records, evidence bindings).

PI-PREQ-01 is the narrow future package that declares these
classes as GC01 facts. PI-PREQ-01 is **NOT** part of PI-00. It
is identified here so the prerequisite is concrete, not buried.

PI-PREQ-01 should declare, at minimum:

* one source record per canonical source the snapshot references
  (Kiln repository URL, plan path, plan digest, governing
  documents, Evidence references);
* one knowledge-entity record per typed entity the snapshot
  contains (Decision, Invariant, Unknown, NegativeKnowledge,
  CompetenceExpectation, ReconsiderationTrigger);
* one query record per implementation-authority query the
  snapshot defines;
* one authorization-record record per authorization the snapshot
  contains;
* one evidence-binding record per Evidence the snapshot binds.

PI-PREQ-01 must be authorized separately from PI-00.

## 16. Roadmap interactions

This subsection records how the experimental lane interacts with
existing roadmap items. No reordering beyond what
`docs/roadmap/project-intelligence.md` records.

| item                                    | classification            |
|-----------------------------------------|---------------------------|
| GC02 and later Governance Compression   | blocked by PI decision;    |
|                                         | GC02 must ask "genuinely   |
|                                         | canonical or derivable?"   |
|                                         | before creating new        |
|                                         | governance surfaces        |
| ARS-09 Knowledge Plane                  | independent of PI;         |
|                                         | potential PI consumer      |
|                                         | after PI-02 (compile-      |
|                                         | context becomes IR-backed) |
| ARS-10 Intent Compiler                  | independent of PI;         |
|                                         | potential PI consumer      |
|                                         | after PI-02 (rule layer    |
|                                         | may consume IR)            |
| ARS-11 Adversarial Verification         | independent of PI          |
| ARS-12 Controlled Capability Evolution  | independent of PI          |
| ARS-13 (Agent Behavioral CI)            | independent of PI          |
| Field Trials (FT-0 ... FT-5)            | independent of PI          |
| Track B Consumer Reliability            | independent of PI          |
| Capability Graph                        | potential PI consumer      |
|                                         | after PI-02 (preflight     |
|                                         | becomes IR-backed)         |
| Compiler                                | potential PI consumer      |
|                                         | after PI-02 (verification  |
|                                         | of capability state reads  |
|                                         | the same IR observation)   |
| Bench                                   | independent of PI          |
| Trust & Authority                       | independent of PI; the IR  |
|                                         | does NOT redefine authority |
| Evidence / Flight Recorder              | independent of PI; the IR  |
|                                         | does NOT redefine evidence |
| Reality Budget                          | independent of PI          |
| Repository Truth                        | independent of PI          |
| Recon                                   | independent of PI          |

The Project Intelligence experiment runs alongside useful work
where possible. It does not freeze Arsenal development.
