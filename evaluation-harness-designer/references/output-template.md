# Output Template — Evaluation Harness Design Document

Use this structure for a **full design**. For an **audit**, don't emit all thirteen sections —
emit the findings (which stages are broken, tied to decision rules and failure modes), the
corrected design for those stages, and a prioritized remediation order, leading with
validity-breaking issues. (Audit shape is at the bottom.)

Each section below carries one line of guidance and a short worked fragment so the artifact is
consistent and downstream-usable. Fill with the system's real specifics; don't pad. If a section
is genuinely N/A (e.g. scale testing for a frozen single-agent tool), say so in one line with the
reason and move on.

```markdown
# Evaluation Harness Design: [System Name]

## 1. System Purpose Statement
[What the system exists to accomplish, for whom, in user-observable terms. Success criteria,
failure definitions, minimum quality thresholds per dimension, documented stakeholder
disagreements. Must be concrete enough to support the metric-to-purpose audit.]

> e.g. *Purpose: resolve customer billing disputes correctly and to the customer's satisfaction
> without human escalation. Success: dispute closed with a correct resolution the customer
> accepts. Failure: wrong resolution, or correct resolution the customer rejects, or unnecessary
> escalation. Min quality: zero incorrect refunds; ≤5% escalation. Disagreement: support lead
> counts "fast closure" as success; finance counts "correct charge" — unresolved, flagged.*

## 2. Metric Inventory with Purpose Alignment
[Table: metric | what it measures | how computed | owner | drives decisions? | 20%-gaming test
result | classification (Aligned / Proxy / Misaligned) | proxy gap | recommendation.]
[Then: structural-blindness inventory and purpose-dimension gaps.]

| Metric | Drives decisions? | Gaming test | Class | Gap | Action |
|--------|:---:|---|---|:---:|---|
| Resolution rate | Yes | Closing disputes wrong-but-fast would spike it, users would notice | Proxy | ~35% | Supplement with trajectory correctness |
| CSAT survey | Yes | Hard to game without real satisfaction | Aligned | <20% | Keep |
| Avg handle time | Yes | Cutting time by skipping verification invisible to this metric | Misaligned | >50% | Replace / demote |

**Structural blindness:** no metric sees *wrong-but-accepted* resolutions (customer accepts an
incorrect refund). **Gaps:** "correctness of charge" has no aligned metric.

## 3. Reliability Metric Specification
[Selected metric (Pass^k / Pass@k / dual), k, justification grounded in context, what it
reveals and hides.]

> e.g. *Pass^k, k=5. Production-critical: a single wrong billing resolution is user harm, so
> consistency across all attempts is the requirement — best-of-k would credit luck. k=5 balances
> confidence against eval cost. Reveals per-attempt consistency; hides ceiling capability (use a
> separate Pass@k research track if needed).*

## 4. Harness Architecture Design
[Scope and boundaries; trajectory analysis spec (full or sampled + rate + budget); the dual
metric (outcome success rate + trajectory quality rate); data-collection spec; trigger mode;
token budget.]

## 5. Contamination Monitoring Architecture
[Three-type detection spec (semantic drift / behavioral detours / combined disruption);
instrumentation; baseline plan; per-type thresholds; dashboard structure; classification rules
keeping contamination-positive trajectories separate from clean successes/failures. Sensitivity
target: >70% detection on injected contamination (DR11).]

## 6. LLM Judge Calibration Specification
[Judge family with explicit different-family verification; gold-set spec (200–500 traces,
diversity, coverage); cadence (monthly critical / quarterly stable); kappa threshold 0.7;
calibration workflow; judge versioning with history. If same-family is unavoidable: mark all
evals provisional, document expected 12–18% inflation, state the correction path.]

## 7. Gold-Set Rotation Protocol
[Rotation schedule (quarterly min); stale-example criteria; production-failure ingestion
pipeline; rotation procedure with before/after composition; versioning; coverage tracking
(flag <50%).]

## 8. Cost Metric Definition
[Cost-per-successful-task formula with all components (retries, idle time, coordination
overhead); cost-per-call role (anomaly detection only); token-distribution method; format
economics; idle-time threshold (20%); incentive analysis per metric.]

> e.g. *Primary: cost-per-successful-task = (Σ tokens·price over all attempts + idle + coord) /
> successful tasks. At $0.05/call, 80% success with retries → ~$0.62/task (8× per-call). Track
> cost-per-call only for anomaly alerts.*

## 9. Scale Test Plan
[Current-scale baseline; simulation points (2×/5×/10×); methodology; degradation-metric
definitions (slope per metric); super-linear detection; scale-approval gate; repetition
schedule.]

## 10. Decision Rule Traceability
[Each major decision → the DR that drove it, with a one-line justification.]

| Decision | DR | Justification |
|---|---|---|
| Pass^k chosen | DR1 | Production-critical |
| Different-family judge required | DR4 | Avoid 12–18% self-referential inflation |
| ≥20% trajectory sampling | DR10 | Multi-agent boundaries present |

## 11. Assumption Register
[Assumptions deferred to other skills: statement | owning skill | placeholder | verification
requirement.]

| Assumption | Owner | Placeholder | Verify |
|---|---|---|---|
| cost-per-successful-task computable at daily granularity | Implementation Playbooks (AF-8) | assume yes | confirm infra feasibility before implementation |

## 12. Risk Register
[Proxy-gap risks from Misaligned metrics that couldn't be replaced; contamination detection that
is aspirational (human-in-the-loop today); calibration risks from gold-set limits; scale-test
limits from simulation fidelity; the uncharacterized isolation/fidelity trade-off curve where
relevant.]

## 13. Calibration Schedule
[Timeline of recurring maintenance: monthly calibration (if critical); quarterly gold-set
rotation; quarterly metric-to-purpose audit; event-driven recalibration triggers; scale-test
repetition at scaling milestones.]
```

---

## Audit output shape

When auditing an existing evaluation rather than designing one:

```markdown
# Evaluation Harness Audit: [System Name]

## Verdict
[Can the current numbers be trusted, partially trusted, or not at all? One paragraph. Lead with
validity-breaking issues — a same-family judge, outcome-only blindness, or an uncalibrated judge
presented as fact means current numbers cannot be trusted, period.]

## Findings
[Per finding: **[Severity]** Title — what's broken, the decision rule it violates (DRn), the
failure mode it invites (Fn), the mechanism (why it manufactures false confidence), and the
specific evidence from their setup. Order by validity impact, not ease of fix.]

## Corrected Design
[For each broken stage, the corrected specification — the relevant Output Template section,
filled for their system.]

## Remediation Order
[Prioritized: validity-breaking first (so numbers become trustworthy), then refinement. Note
what they can keep using in the meantime, and what they must stop trusting now.]
```
