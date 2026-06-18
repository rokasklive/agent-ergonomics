# Design Procedure — Full Detail

This is the step-by-step depth behind the nine-stage overview in SKILL.md. Read the
relevant stage before you design it; pull a single stage when auditing. Each stage ends
with the exact **output** it contributes to the design document.

Before the stages, the eight **design inputs** in full — what to extract and *why each
matters for a specific decision*. An input you can't fill is itself a finding.

---

## Design inputs

### 1. System Purpose Statement
**Extract:** the system's reason for existing — what user-observable outcomes it produces,
for whom, under what conditions. Not the technical spec or capability list. Concrete enough
that you can answer "would a 20% improvement in metric X be user-observable?" with
confidence.
**Why:** metric-to-purpose alignment is impossible without a defined purpose. If you can't
state the purpose, you can't tell a proxy from an outcome, and every metric decision in
Stage B is groundless. Getting this wrong is the root of Goodhart's Law optimization.

### 2. Current Metrics Inventory
**Extract:** every metric tracked — what it measures, how it's computed, who owns it,
whether it drives optimization decisions (vs. merely being reported), and any documented
purpose-alignment rationale.
**Why:** Stage B operates on this list. Without it you can't find proxy gaps, redundancy, or
structural blindness.

### 3. Contamination Baseline
**Extract:** has contamination been measured or observed? If measured: prevalence and
distribution across the three types; current detection infrastructure (outcome-only,
trajectory-sampled, or full-trajectory). If unmeasured: does the system have multi-agent
boundaries that make contamination plausible?
**Why:** Stage E depends on knowing the contamination surface. Any measured rate makes
outcome-only evaluation already invalid (the empirical baseline is 55.6% invisible to
outcome-only metrics). Never measured in a multi-agent system → critical gap, design
detection from scratch.

### 4. Gold-Set Size, Age, and Calibration History
**Extract:** current size (traces), age of oldest examples, last calibration date, last
Cohen's kappa, judge family relative to system family, whether calibration is same- or
different-family, and whether the set includes examples from the last 90 days of production.
**Why:** drives calibration cadence (F) and rotation schedule (G). Below 200 traces, older
than 90 days, same-family-calibrated, or missing recent production failures → remediate
before results can be trusted. These are hard constraints, not suggestions.

### 5. Cost Data Structure
**Extract:** how cost is measured (per-call/token/session/task), whether retries and idle
time are included, whether tool-output token distribution has been analyzed, and whether the
pricing model (bounded vs. usage-based) has been evaluated against exploration patterns. If
cost-per-successful-task isn't computed, extract raw data sufficient to compute it.
**Why:** Stage H depends on whether the current metric aligns with task success or
incentivizes counterproductive behavior. Only cost-per-call tracked → compute
cost-per-successful-task to reveal hidden retry/idle cost. Token distribution (67.6%
tool-output empirical anchor) shows whether optimization effort is misallocated.

### 6. Reliability Requirements
**Extract:** is the system production-critical (failure is user-observable), safety-critical
(failure can cause harm), capability-exploratory (measuring ceiling), or research-internal?
Does a single failure among k attempts matter, or only the best attempt?
**Why:** directly determines Pass@k vs Pass^k (Stage C, DR1–DR2). Using Pass@k for production
readiness is the #1 reliability-measurement mistake.

### 7. Judge Model Configuration
**Extract:** the model family used as LLM judge (if any), the family used by the system, any
overlap, and the calibration infrastructure (human gold-set availability, 200+ trace minimum,
kappa-computation capability).
**Why:** the different-family constraint is non-negotiable (DR4). Shared family → evaluation
is self-referential, scores inflate 12–18% with no external validity. Reject before any eval
runs.

### 8. Scale Projections
**Extract:** current component count (agents, tools, services), planned 6–12 month growth,
whether scaling is linear (more instances, same architecture) or architectural (new
coordination patterns), and whether compounding-friction test infrastructure exists.
**Why:** Stage I must bracket current and planned scale to catch non-linear degradation before
deploy. Inflection anchors: ~5–10 tools, ~3–8 agents. Imminent scaling with no friction test →
architecture review is a hard gate before approval.

---

## Stage A — Purpose Definition

Document the system's actual purpose before examining any metric:

1. State the reason for existing — user-observable outcomes, for whom, under what conditions.
2. Distinguish purpose from capabilities. Purpose = "what must be true for the system to
   succeed"; capabilities = "what it can do."
3. Identify what failure looks like — what outcomes show the system is not achieving purpose.
4. Define the minimum acceptable quality threshold for each purpose dimension.
5. Document disagreements: if stakeholders disagree on what the system is for, alignment is
   impossible until resolved — surface it rather than papering over it.

**Output — System Purpose Statement:** purpose definition, success criteria, failure
definitions, quality thresholds, known stakeholder disagreements.

---

## Stage B — Metric Selection & Purpose-Alignment Audit

Inventory and audit every current or proposed metric against the purpose statement:

1. List every metric — what it measures, how computed, who owns it, whether it drives
   optimization decisions.
2. For each, run the purpose-alignment test: *"If we improved this metric 20% by gaming it,
   without improving actual outcomes, would users notice?"*
3. Classify:
   - **Aligned** — improvement drives user-observable improvement; proxy gap <20%.
   - **Proxy** — improvement may or may not drive improvement; proxy gap 20–50%.
   - **Misaligned** — improvement does not drive improvement; proxy gap >50%.
4. Flag all Misaligned for replacement; flag all Proxy for supplementation with trajectory
   analysis or human calibration (DR3).
5. Identify structural blindness — important failure modes no current metric captures.
6. Document remaining gaps where a purpose dimension has no aligned metric.

**Output — Metric Inventory with Purpose Alignment:** metric list with classification,
proxy-gap notes per metric, replacement/supplementation recommendations, structural-blindness
inventory, documented purpose-dimension gaps.

---

## Stage C — Reliability Metric Selection

1. Determine evaluation context (Input 6): production-critical, safety-critical,
   capability-exploratory, research-internal.
2. Select:
   - **Pass^k (all-k-must-pass)** — production-critical and safety-critical, where
     inconsistency *is* failure. Every one of k attempts must succeed.
   - **Pass@k (best-of-k)** — capability-exploratory and research, measuring ceiling. Accepts
     the best of k. Label it "capability ceiling," never "reliability" (DR2).
   - **Dual reporting** — when both matter, report both, explicitly distinguished. Never
     conflate.
3. Select k for desired statistical power, balanced against evaluation cost.
4. Document the rationale: why this metric, why this k, what it reveals and what it hides.
5. If Pass@k is selected for a production system, flag it as a design risk requiring
   documented justification (DR1).

**Output — Reliability Metric Specification:** selected metric, k, justification grounded in
context, documented limitations.

---

## Stage D — Harness Architecture

1. Define scope — which components, task types, and boundaries are in scope.
2. Specify trajectory analysis — full-trajectory for safety-critical; sampling (≥20%) for
   cost-sensitive production; with explicit budget allocation (DR10).
3. Design the **dual-metric structure** — outcome success rate (% of tasks that succeeded)
   *plus* trajectory quality rate (% of trajectories uncontaminated and logically sound).
4. Design data collection — what traces, at what granularity, across which boundaries, with
   what retention.
5. Specify the evaluation trigger — continuous (every task), periodic (batch at cadence), or
   event-driven (deploy / model change / anomaly).
6. Define the evaluation budget — token allocation for LLM-judge, trajectory analysis, human
   review — as a design constraint (L1).

**Output — Harness Architecture Design:** scope, trajectory spec (full or sampled + rate),
dual-metric structure, data-collection spec, trigger mode, budget allocation.

---

## Stage E — Contamination Monitoring

1. Define the three types to monitor:
   - **Semantic drift** — meaning of information shifts across a boundary.
   - **Behavioral detours** — agent takes unnecessary actions that propagate contamination.
   - **Combined disruption** — both compound.
2. Design trace-level instrumentation per type — what reasoning patterns and
   boundary-crossing signatures indicate contamination and should trigger alerts.
3. Establish baseline rates via initial trajectory analysis. If contamination can't be
   measured initially, document the critical gap and deploy detection *simultaneously* with
   the harness (DR9).
4. Set detection thresholds per type. Sensitivity check: detection on injected contamination
   must exceed 70%, or the harness is leaking >30% into "clean" classifications (DR11).
5. Define the dashboard — per-type metrics, cross-boundary fidelity scores, alert thresholds,
   review cadence.
6. Specify classification: contamination-positive trajectories are classified *separately*
   from clean successes and clean failures — never folded into the success rate.

**Output — Contamination Monitoring Architecture:** three-type detection spec, instrumentation
design, baseline plan, thresholds, dashboard structure, classification rules.

---

## Stage F — LLM Judge Calibration

1. Enforce the different-family constraint (DR4): verify judge family ≠ system family. If
   same-family is the only option, flag all evaluations provisional and document the expected
   12–18% inflation.
2. Specify the human gold-set: ≥200 traces for lower-criticality, ≥500 for production- and
   safety-critical. Cover diverse task types, failure modes, and difficulty levels.
3. Define cadence: monthly for production/safety-critical (DR5); quarterly for stable,
   lower-criticality (DR6); event-driven on model-family change, gold-set rotation, or
   anomaly (DR8).
4. Specify the Cohen's kappa threshold: flag below 0.7 (DR7); track the trend — degrading
   kappa is a leading indicator of drift.
5. Calibration workflow: humans grade gold-set traces → judge grades the same → compute kappa
   → flag if below threshold → recalibrate (prompt adjustment, model update, or gold-set
   expansion).
6. Version judge configurations: each calibration event produces a new judge version with
   history (date, kappa, gold-set version, reviewer attribution).

**Output — LLM Judge Calibration Specification:** judge family (with different-family
verification), gold-set spec, cadence, kappa threshold, workflow, versioning protocol.

---

## Stage G — Gold-Set Rotation

1. Rotation schedule: quarterly minimum; more frequent for rapidly evolving systems.
2. Stale-example criteria: older than 90 days; failure modes no longer seen in production;
   examples where behavior changed enough that the old gold judgment is invalid.
3. Production-failure ingestion pipeline: capture recent (≤90 day) failures, classify by type,
   select representatives, have humans produce gold judgments.
4. Rotation procedure: retire stale → add new production-failure examples → version the set →
   recalibrate the judge against the new set (Stage F) → document before/after composition.
5. Versioning: each rotation = a new version; results reference the version they were
   calibrated against; historical results stay comparable via version-to-version calibration.
6. Coverage tracking: what % of recent (90-day) production failure types are represented? Flag
   below 50% (DR18).

**Output — Gold-Set Rotation Protocol:** schedule, stale criteria, ingestion pipeline design,
rotation procedure, versioning, coverage tracking.

---

## Stage H — Cost Metric Design

1. Define **cost-per-successful-task** as primary, including: token cost for all attempts
   (successful and not), retry costs, idle time (waiting on tools, inference, or human review),
   coordination overhead (agent↔agent comms, orchestration). (DR12)
2. Define cost-per-call as secondary — anomaly detection only, never optimization (DR13).
3. Token distribution: what % goes to tool responses vs. system prompts vs. history vs.
   retrieved context? Compare against the 67.6% tool-output anchor.
4. Format economics: per output format (JSON, TOON, CSV, text), estimate annual cost at current
   scale; flag where a more efficient format yields meaningful savings (DR14).
5. Idle-time component: measure waiting vs. computing; flag idle time over 20% of cost (DR15).
6. Incentive analysis: for each metric, what behavior does it encourage or discourage?

Worked anchor for the case: $0.05/call at 80% task success, with retries, yields a true cost
of ~$0.62/task — 8× the per-call figure. Cost-per-call falling while cost-per-successful-task
rises is the signature of cutting retries at the expense of outcomes.

**Output — Cost Metric Definition:** cost-per-successful-task formula, cost-per-call role,
token-distribution method, format economics, idle-time threshold, incentive analysis.

---

## Stage I — Scale Testing

1. Baseline at current scale: success rate, contamination rate, cost per successful task,
   latency.
2. Simulation points: 2×, 5×, 10× current component count. Approaching known inflection points
   (~5–10 tools, ~3–8 agents) → add intermediate points.
3. Methodology: synthetic load, additional agent/tool instances, or scaled-down task
   complexity representing real workload.
4. Degradation metrics: compute degradation slope (Δmetric / Δcomponent_count) per metric per
   point. Flag **super-linear** degradation (slope increasing at higher scale) for architecture
   review (DR17).
5. Scale-approval gate: if any metric degrades super-linearly at or below planned scale, the
   coordination architecture needs redesign before scaling. The harness must not approve
   scaling past the super-linear point (DR16, DR17).
6. Repetition: friction tests repeat at each major scaling milestone, not once at design time.

**Output — Scale Test Plan:** current-scale baseline, simulation points, methodology,
degradation-metric definitions, scale-approval gate criteria, repetition schedule.

---

## Pattern seeds these stages encode

- **Metric-to-Purpose Alignment Audit** (Stage B) — quarterly cadence for any production eval.
- **Contamination Monitoring Dashboard** (Stage E) — mandatory companion to any success-rate
  dashboard in a multi-agent system.
- **Isolation/Fidelity Calibration** — the experiment design that measures the isolation vs.
  contamination trade-off. The *methodology* is encodable now; the trade-off **curve is
  empirically uncharacterized** (corpus Open Questions 1, 10) — design the experiment, flag the
  missing target values as a risk.
- **Tool Description Regression Test Suite** — description-only tool changes are breaking changes
  with no 4xx/5xx signal; specify regression tasks and thresholds so behavior drift is caught.
- **Gold-Set Rotation Protocol** (Stage G).
- **Compounding Friction Scale Test** (Stage I).
