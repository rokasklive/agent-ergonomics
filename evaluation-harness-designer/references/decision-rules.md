# Decision Rules & Failure Modes

The eighteen decision rules (DR1–DR18) are the operational gates that turn the eight design
habits into concrete yes/no checks at specific stages. Each has a trigger and the failure mode
it prevents. Below the rules: the five laws in full (as evaluation constraints) and the eight
failure modes they guard against.

Read this when you make a gated decision or run an audit. When a rule says **REJECT**, you don't
refuse to help — you explain why the configuration manufactures false confidence, then design the
version that doesn't. If the better option genuinely isn't available, mark results **provisional**,
document the distortion, and specify the correction path.

---

## Metric selection

| # | Rule | Stage / prevents |
|---|------|------------------|
| **DR1** | IF context is production-critical OR safety-critical → select Pass^k, not Pass@k. | C / reporting luck as reliability. |
| **DR2** | IF context is capability-exploratory OR research-internal → Pass@k is acceptable but must be labeled "capability ceiling," not "reliability." | C / conflating ceiling with consistency. |
| **DR3** | IF a metric's proxy gap >50% (gaming it +20% would NOT be user-observable) → flag Misaligned; design a replacement or supplementation before deploying the eval. | B / F1 Goodhart's Law. |
| **DR4** | IF judge family == system family → **REJECT**. Evaluations are provisional until different-family calibration exists. | F / F2 judge drift, self-referential eval. |

## Calibration

| # | Rule | Stage / prevents |
|---|------|------------------|
| **DR5** | IF production-critical OR safety-critical → calibration cadence must be monthly. | F / F2 drift beyond 30 days for critical systems. |
| **DR6** | IF stable AND lower-criticality → quarterly calibration is acceptable. | F / over-calibration wasting human-review budget. |
| **DR7** | IF Cohen's kappa <0.7 → flag judge uncalibrated, mark subsequent evals provisional, trigger recalibration. | F / trusting a judge whose agreement with humans fell below threshold. |
| **DR8** | IF judge family changes (version update, provider switch) → recalibrate regardless of schedule. | model-change event / F2 drift from family-internal distribution shift. |

## Contamination detection

| # | Rule | Stage / prevents |
|---|------|------------------|
| **DR9** | IF the system has 2+ agents OR agent-to-agent communication → contamination monitoring (all three types) is mandatory. | D / F4 outcome-only blindness to 55.6% contamination. |
| **DR10** | IF the design uses outcome-only metrics → **REJECT** for any system with agent boundaries. Trajectory analysis (≥20% sampling) is required. | D / F4 outcome-only blindness. |
| **DR11** | IF contamination detection rate on injected contamination <70% → the harness is insufficiently sensitive (leaking >30% into "clean"). Increase sampling or improve instrumentation. | E / deploying a harness that systematically misses contamination. |

## Cost economics

| # | Rule | Stage / prevents |
|---|------|------------------|
| **DR12** | IF cost-per-call is the primary optimization metric → replace with cost-per-successful-task as primary; demote cost-per-call to anomaly detection only. | H / optimizing call count at the expense of task success. |
| **DR13** | IF cost-per-call declines while cost-per-successful-task rises → flag "incentive misalignment": the system is cutting calls at the expense of outcomes. | ongoing cost monitoring / F5 cost-per-call optimization worsening real economics. |
| **DR14** | IF tool-output format tokens exceed 50% of total token consumption → run format economics analysis (TOON, CSV, compressed JSON). | H / misallocated effort on system prompts (3.4%) while tool-output bloat (67.6% anchor) is ignored. |
| **DR15** | IF idle time exceeds 20% of total cost → flag "hidden cost dominance"; audit for async patterns, speculative execution, timeout tuning. | H / idle time silently dominating cost at scale. |

## Scale testing & gold-set freshness

| # | Rule | Stage / prevents |
|---|------|------------------|
| **DR16** | IF planned scale exceeds current scale by 2× or more → compounding-friction scale test is mandatory before scale approval. | I / deploying at scale without validating degradation is linear/manageable. |
| **DR17** | IF any metric degrades super-linearly at or below the planned scale point → **REJECT** scale-up; coordination architecture needs redesign. | I / F6 compounding friction causing catastrophic degradation at scale. |
| **DR18** | IF gold-set age exceeds 90 days OR coverage of recent (90-day) production failures drops below 50% → trigger gold-set rotation. | G / F3 eval-set rot producing misleading improvement signals. |

---

## The five laws — as evaluation constraints

### L5 — Ergonomic Dominance *(load-bearing for this skill)*
The surrounding system — tool design, coordination, information structures, economic
constraints — governs real-world outcomes more than the model does. The harness defines what the
system optimizes for; *the evaluation suite IS the deliverable* (Principle 12). An excellent model
under a broken harness optimizes broken behavior.
**Consequence:** begin every design with purpose definition before any metric. Version, calibrate,
and treat metrics with production-infrastructure rigor. Design the harness first.
**Violation symptom:** harness designed after the system, metrics chosen for measurability, harness
treated as a QA checkpoint rather than the defining constraint.

### L1 — Bounded Attention
Effective capacity is bounded by the finite context window; every token consumed reduces reasoning
capacity. Evaluation itself consumes tokens — LLM-judge, trajectory analysis, contamination
detection. A harness that costs more than it saves is net-negative.
**Consequence:** prefer deterministic graders; sample trajectories (e.g. 20%) rather than evaluate
exhaustively; track the harness's own token consumption as a cost metric; when eval cost exceeds a
threshold, reduce sampling rather than eliminate trajectory analysis.
**Violation symptom:** harness token cost untracked, unbudgeted, never compared to task-execution
cost; exhaustive 100% trajectory evaluation where sampling would suffice.

### L4 — Compounding Friction
Inefficiencies compound non-linearly with scale; components past a threshold degrade rather than
improve outcomes. A harness validated only at current scale is silent on degradation just past it.
**Consequence:** every harness includes a compounding-friction scale test (2×/5×/10×) measuring
success, contamination, cost, and latency, computing degradation slopes. Super-linear degradation →
architecture review before scale approval. Repeat at each scaling milestone.
**Violation symptom:** no scale-testing component; performance measured only at current scale; one
scaling event away from undetected catastrophic degradation.

### L3 — Boundary Entropy
Information fidelity necessarily decreases crossing any boundary (agent↔agent, agent↔tool, context
window, representation); errors and drift compound at each, and the loss is invisible locally.
Outcome-only metrics are structurally blind to it — the 55.6% contamination rate is invisible
*because* boundary entropy operates silently.
**Consequence:** trajectory-level analysis covering all boundaries; contamination monitoring of the
three types as independent metrics; reject outcome-only designs as structurally insufficient for any
multi-boundary system.
**Violation symptom:** aggregate success rates with no per-boundary degradation measurement;
contamination neither measured nor monitored; "the system succeeded" accepted without verifying
fidelity across boundaries.

### L2 — Interface Ground Truth
Agents (and operators) have no access to reality beyond interfaces; every signal the harness emits
is treated as authoritative truth. A harness that reports misleading signals — same-family
inflation, green outcome-only dashboards, stale gold-sets presenting as current — becomes false
ground truth. A broken harness is more dangerous than no harness: it manufactures false confidence.
**Consequence:** qualify every harness output with calibration status, gold-set age, judge family,
and contamination coverage. Mark uncalibrated or stale evaluations provisional. Maintain traceability
from output to calibration evidence.
**Violation symptom:** scores presented as bare numbers — no calibration date, no judge-family
disclosure, no gold-set age, no contamination status. Users can't tell quality from infrastructure
decay.

---

## The eight failure modes

### F1 — Goodhart's Law Optimization
**Symptom:** metrics improve while outcomes degrade or stay flat; teams celebrate improvements users
don't notice; effort moves dashboard numbers, not user experience.
**Cause:** metrics chosen for measurability, not purpose; the eval accreted from what was easy to
instrument; no metric-to-purpose audit (Stage B skipped).
**Prevention:** begin with purpose (A); audit every metric Aligned/Proxy/Misaligned (B); replace or
supplement Misaligned before deploy; repeat quarterly.
**Laws:** L5; Invariant II-2 (eval criteria must match purpose).

### F2 — LLM Judge Drift
**Symptom:** scores change without system changes; same-family judges inflate 12–18%; human review
disagrees with automated eval more over time; reported quality stable while real quality drifts.
**Cause:** judges decay over 60–90 days; no calibration, too-infrequent calibration, or same-family
self-reference (DR4, DR5–DR8 violated).
**Prevention:** different-family gate (DR4); monthly cadence for critical (DR5), quarterly for stable
(DR6); track kappa trend; version judges; flag uncalibrated as provisional.
**Laws:** L2; Contract C5 (evaluation validity).

### F3 — Eval-Set Rot
**Symptom:** eval scores improve while production incidents rise; the set holds only "solved" failure
modes; new failure types pass because they're not in the set; gold-set static while system evolves.
**Cause:** gold-set never rotated; no production-failure ingestion; no coverage tracking (Stage G,
DR18 violated).
**Prevention:** rotation protocol (G) — quarterly, stale detection, ingestion pipeline, versioning,
coverage tracking; flag age >90 days or coverage <50% (DR18).
**Laws:** L3; C5.

### F4 — Outcome-Only Metric Blindness
**Symptom:** 95%+ success dashboards while users report degraded quality; contamination propagates
undetected; the system "succeeds" while producing subtly wrong outputs via wrong reasoning;
monitoring green while the system silently degrades.
**Cause:** evaluation measures only final outcomes; 55.6% of contamination produces correct-looking
outcomes through incorrect reasoning; structurally blind to lucky-correct process failures (DR9–DR10
violated).
**Prevention:** dual-metric structure (D); mandatory trajectory analysis (full for safety-critical,
≥20% sampling for production); contamination monitoring of all three types (E); reject outcome-only
designs (DR10).
**Laws:** L3; Invariant I-1 (traceability); Anti-Pattern 4 (outcome-only monitoring).

### F5 — Cost Metric Misalignment
**Symptom:** cost-per-call falls while cost-per-successful-task rises; team celebrates declining API
cost while completion rate degrades; cutting retries reduces per-call cost but raises failure rate
and total cost per outcome.
**Cause:** cost-per-call is the primary optimization metric; retries, idle time, coordination
overhead invisible; token distribution unanalyzed, so effort targets system prompts (3.4%) while
tool-output bloat (67.6%) is ignored (DR12–DR15 violated).
**Prevention:** cost-per-successful-task primary (DR12); cost-per-call for anomaly detection only;
token-distribution measurement and format economics; include idle time.
**Laws:** L1; C7 (cost per successful outcome).

### F6 — Scale Blindness
**Symptom:** system performs well at current scale; scaling approved on current-scale metrics;
post-scale performance degrades non-linearly; the harness never tested beyond current scale.
**Cause:** no compounding-friction scale test; scaling assumed linear when known non-linear (L4);
scale testing omitted (DR16–DR17 violated).
**Prevention:** scale test as core component (I); 2×/5×/10×; degradation slopes for success,
contamination, cost, latency; reject scale-up on super-linear degradation at/below planned scale
(DR17).
**Laws:** L4; Invariant IV-2 (system scale must not outpace coordination design).

### F7 — Calibration as Afterthought
**Symptom:** eval deployed without human gold-set calibration; judge operates with no baseline;
scores trusted as objective; when calibration finally happens months later, scores shift
significantly — revealing the prior eval was uninformative.
**Cause:** calibration treated as optional polish; deploy pressure overrode the calibrate-first
requirement; gold-set creation deferred to a sprint that never came (DR5–DR7 violated).
**Prevention:** human gold-set (200–500) required before LLM-judge evals are trusted; flag all
uncalibrated evals provisional and bar them from being presented as quality metrics; schedule
calibration at design time; make it a deployment gate.
**Laws:** L2; C5.

### F8 — Contamination Monitoring as Future Enhancement
**Symptom:** design acknowledges contamination is possible but defers monitoring to a later phase;
success-rate dashboards ship without contamination detection; months later 55.6% propagates
invisibly while the dashboard reports green.
**Cause:** contamination monitoring treated as nice-to-have, not a structural validity requirement;
outcome metrics (easy) prioritized over trajectory analysis (hard), the hard component deferred
indefinitely (DR9–DR11 violated).
**Prevention:** make contamination monitoring mandatory for multi-agent systems (DR9); design it
simultaneously with success-rate monitoring; if it can't be measured initially, deploy detection in
parallel and flag results "contamination-unmonitored" until operational.
**Laws:** L3; Invariant I-1 (traceability).
