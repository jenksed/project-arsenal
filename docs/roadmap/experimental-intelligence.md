# Experimental Intelligence Roadmap Note

Status: proposed cross-cutting roadmap refinement

This note captures a durable lesson from a real browser-performance investigation: an agent can have strong code-search and code-generation ability yet still fail badly when it lacks a discriminating observation. Plausible source-level reasoning is not causal evidence.

The durable Arsenal lesson is not a browser-specific optimization rule. It is a general engineering rule:

> **Diagnosis requires discrimination.**
>
> Do not mutate a system merely because a theory is plausible. First obtain a signal and design an observation capable of distinguishing that theory from credible alternatives. When the necessary observability does not exist, build the probe before building the fix.

A companion rule follows:

> **Preserve falsification.**
>
> A rejected hypothesis is reusable engineering knowledge when the experiment, environment, prediction, and observation that rejected it are preserved.

## Why this belongs in Arsenal

The existing Diagnose method already points in the right direction: establish a red-capable feedback loop, rank falsifiable hypotheses, probe one variable at a time, and measure performance regressions against a baseline. The missing architectural step is to make the resulting investigation state durable, typed, and composable across the Knowledge Plane, Reality Budget, Flight Recorder, Bench, and Development Packs.

The intended shape is **experimental intelligence**: project intelligence about what can be observed, which hypotheses remain live, which experiments discriminate among them, what evidence supports or falsifies them, what observer effects apply, and which product invariants a remediation must preserve.

This is not a new general-purpose reasoning engine and should not duplicate Diagnose. Diagnose remains the behavioral method. Experimental Intelligence supplies typed knowledge and evidence contracts that make the method inspectable and reusable.

## Cross-cutting additions

### 1. Diagnostic Signal

A diagnosis should name the signal capable of detecting the reported failure before causal theory is allowed to harden into implementation.

Candidate fields:

- symptom / claim under investigation;
- measurement or observation surface;
- red-capable baseline;
- environment and relevant hardware/runtime properties;
- repeatability / reproduction rate;
- measurement limitations;
- observer intrusiveness;
- fidelity to the claimed production behavior.

If no adequate signal exists, building the smallest useful probe is a valid first-class outcome.

### 2. Hypothesis / Prediction / Experiment graph

The Knowledge Plane should be able to represent an active investigation without flattening it into conversational prose.

Candidate entities and relationships:

- Hypothesis;
- Prediction;
- Diagnostic Signal;
- Experiment;
- Observation;
- Intervention;
- Invariant;
- Evidence;
- Reconsideration Trigger;
- `predicts`;
- `tested_by`;
- `observed_as`;
- `supports`;
- `challenges`;
- `falsifies`;
- `confounded_by`;
- `preserves`.

A useful compiled context should distinguish:

- hypotheses that are merely plausible;
- hypotheses with a testable prediction;
- experiments not yet run;
- observations with declared fidelity;
- supported hypotheses;
- falsified hypotheses;
- unresolved confounders;
- protected invariants for remediation.

### 3. Causal Evidence Receipt

For performance, reliability, and other hard-to-localize failures, a successful intervention should be able to carry evidence stronger than “the code changed and the tests passed.”

A minimal receipt should record:

- symptom and signal;
- baseline/control observation;
- intervention;
- observed effect;
- reversal/replay where practical;
- environment and observer conditions;
- confounders;
- claim scope;
- remaining unknowns;
- product/behavior invariants preserved.

An A/B/A' or equivalent reversible experiment is preferred when the system permits it. The receipt must not imply causal certainty when the experiment only establishes correlation.

### 4. Observer effect as evidence metadata

Reality Budget currently models how much reality and authority are required to establish a claim. It should also represent when the observation mechanism itself perturbs the phenomenon.

Examples include:

- debugger attachment changing timing;
- browser developer tools altering rendering/performance behavior;
- headless browser execution differing from headed GPU/compositor behavior;
- tracing/profiling overhead;
- synthetic environments omitting relevant refresh-rate, DPI, HDR, GPU, network, or concurrency properties.

Candidate evidence properties:

- `observer_intrusiveness`;
- `known_perturbations`;
- `environment_fidelity`;
- `hardware_relevance`;
- `measurement_overhead`.

The selector should be able to explain when a cheaper observation surface cannot establish the requested claim.

### 5. Preserve negative evidence

Failed theories should not disappear from context once a bug is fixed.

Negative evidence can prevent repeated dead ends and is especially valuable for future incidents that share a symptom family. Durable project knowledge should preserve what was tested, under what environment, what prediction failed, and which later changes should cause that conclusion to be reconsidered.

This is different from preserving private chain-of-thought. Arsenal needs the explicit engineering artifact: hypothesis, prediction, experiment, observation, and result.

## Capability ownership

This refinement should be distributed across existing roadmap owners rather than becoming an isolated subsystem:

- **Diagnose** — behavioral discipline: signal first, falsifiable hypotheses, one-variable probes, narrow remediation, rerun original feedback loop.
- **ARS-05 Reality Budget** — observation fidelity, observer intrusiveness, environment relevance, and escalation when a cheaper observer cannot prove the claim.
- **ARS-07 Agent Flight Recorder** — record experiment/evidence references and accepted/rejected diagnostic outcomes without collecting private reasoning traces.
- **ARS-09 Knowledge Plane** — typed Diagnostic Signal, Hypothesis, Prediction, Experiment, Observation, negative Evidence, relationships, and reconsideration triggers; compile the relevant investigation subgraph.
- **ARS-11 Adversarial Verification** — deeper counterfactual experiments, failure laboratories, and independent challenge of causal claims.
- **ARS-12 Controlled Capability Evolution** — use recurring diagnostic failures to propose capability revisions, subject to baseline/regression/adversarial evaluation and human promotion.

## Arsenal Bench case

Add a Core engineering-judgment case in which the visible symptom is a performance regression whose root cause lives outside the layer most source inspection initially suggests.

Candidate case: `core-diagnosis-hidden-runtime-cost`.

The fixture should include:

- a visually simple browser application;
- plausible JavaScript/framework/network suspects;
- a hidden rendering/compositor or animation cost;
- at least one misleading but plausible source-level theory;
- a measurement surface that can expose the real runtime layer;
- reversible controls or feature toggles that permit discrimination;
- protected visual/interaction invariants so “remove the feature” is not automatically accepted as a quality fix.

Treatment behavior should reward:

1. naming the missing discriminating signal;
2. establishing a baseline;
3. isolating the runtime layer;
4. inventorying candidate mechanisms;
5. creating reversible experiments;
6. preserving falsified hypotheses as negative evidence;
7. localizing the cause before broad mutation;
8. implementing the narrowest remediation;
9. preserving declared UX/product invariants;
10. causally verifying the result within the declared evidence scope.

Metrics should include:

- correct root-cause localization;
- time/tool calls to first discriminating signal;
- speculative mutation count;
- lines changed before root cause is established;
- number of falsified hypotheses preserved;
- human interventions;
- false-completion rate;
- outcome improvement;
- invariant regressions;
- tokens, cost, wall time, and repair cycles.

This case should not require the model to memorize a browser-performance trivia fact. It should reward the engineering process that can discover an unfamiliar cause.

## Browser / frontend performance Development Pack tracer

Expand the planned Playwright/browser verification lane into a browser/frontend performance tracer capable of proving this architecture against a real domain.

Candidate domain knowledge and tooling:

- JavaScript and framework update paths;
- style calculation, layout, paint, rasterization, compositing, and GPU-process behavior;
- animation and transition inventories;
- filters, backdrop filters, shadows, and layer-promotion candidates;
- headed vs headless behavior;
- refresh rate, DPI, HDR, viewport, GPU, and display capture;
- browser/tab/window isolation;
- reversible runtime CSS/feature toggles;
- CDP or browser-native metrics where appropriate;
- low-intrusion process sampling;
- visual/interaction regression checks;
- A/B/reversal experiment helpers.

The pack should not encode “CSS animation is bad.” Its purpose is to make the browser world observable enough for Diagnose to discriminate among causes.

## Acceptance direction

This refinement is successful when Arsenal can represent and evaluate a hard diagnostic run in which:

- the first plausible theory is wrong;
- normal source inspection is insufficient;
- the agent explicitly recognizes the missing observation;
- the agent builds or selects a probe before broad mutation;
- experiments narrow the causal frontier;
- falsified alternatives remain reusable knowledge;
- observer effects and environment fidelity are explicit;
- remediation preserves declared product invariants;
- completion is backed by causal evidence within a bounded claim scope.

The product promise is not that Arsenal always knows the answer.

It is that Arsenal should help an agent know **what it does not yet know, what observation would reduce that uncertainty, and what evidence is required before changing the system or claiming the problem solved.**
