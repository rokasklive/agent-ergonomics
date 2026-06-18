# Output Template — The Implementation-Playbook Artifact

> A full playbook artifact has these fourteen sections. **Emit only the ones the build calls for** —
> a single-tool trace buildout needs 1, 2, 3, 5, 8, 9; a compaction rollout needs 1, 3, 5, 6, 8, 9; a
> coordination buildout needs 1, 2, 3, 4, 5, 7, 8, 9, plus 11–14 as applicable. **Don't pad:** a
> section the build doesn't touch gets one line ("N/A — single-agent, no inter-agent boundaries").
>
> The artifact is executable: a team walks it top to bottom, stopping at each verification gate until
> it passes. It is downstream-connected — it builds *to* a [[Design Templates]] artifact, its gates
> reference [[Review Checklists]] items, and its integration tests feed [[Evaluation Harnesses]].
> **Keep step numbers and obligation references intact** so the chain survives.
>
> Each step, when you write it, carries four things: a **number**, an **action** (what the implementer
> does), a **prerequisite** (what must be *verified* complete before it starts), and a **contract
> obligation reference** (e.g. "C1.RC2"). Safety-critical steps additionally carry their place in the
> safety-first ordering audit.

The sections, with what each carries and which stage produces it:

## 1. Prerequisites Audit — *(Stage A)*
Hard-dependency inventory (status per item: present / missing-provisionable / missing-blocking);
soft-dependency inventory (degradation consequence per item); blocking-gaps list; deferred-setup tasks
with deadlines before production. **Plus the ownership-boundary statement**: what this playbook builds,
and — explicitly — what it does not (the WHAT it builds *to* → Design Templates; the checks it's
verified *by* → Review Checklists; the metrics it's measured *by* → Evaluation Harnesses).

> **Worked fragment**
> - **Hard:** trace storage backend — *missing, provisionable* (deferred-setup S1, deadline before
>   prod). Circuit-breaker mechanism — *present*.
> - **Soft:** fault-injection harness — *missing*; consequence: PS-11 drill (Stage I) can't run until
>   built; flagged.
> - **Blocking gap:** none. (If a hard dependency were missing-blocking → emit the gap, do not proceed.)

## 2. Contract Obligation Map — *(Stage B)*
Obligation→step table (every Required Check mapped to a build step); Forbidden Shortcuts listed as
negative constraints the procedure must not produce; Evaluation Requirements listed as the test
scenarios carried to §9; cross-cutting obligations flagged (need verification after >1 stage).

## 3. Dependency Graph — *(Stage C)*
Ordered step list with prerequisite declarations; topological order; phase-boundary markers. **Plus
the safety-first ordering audit** — the single most important artifact in the playbook:

> **Safety-first ordering audit (worked fragment)**
> | Optimization step | Safety prerequisite | Earlier in graph? |
> |---|---|---|
> | S12 enable compaction at 83.5% | S4 safety-content inventory; S5 immunity tagging | ✓ (S4,S5 < S12) |
> | S14 model-routing to cheaper tier | S6 safe-task classification | ✓ (S6 < S14) |
>
> If any row is ✗, the playbook is not shippable — reorder (DR1, DR9).

## 4. Implementation Depth Decision — *(Stage D)*
Per-phase classification table (minimal-viable / production-grade); deferral rationale per
minimal-viable choice anchored to system criticality + scale (D1/D2/D3); **upgrade trigger** per
deferred component (the scale threshold or criticality increase that forces production-grade);
infrastructure delta for production-grade. Include the floor check: confirm no non-deferrable
(promote-to-durable, reasoning capture, cross-agent validation) was deferred.

## 5. Implementation Procedure — *(Stage E)*
The ordered step list each team member executes. Per step: number; action; verified-complete
prerequisite; the verification gate it produces; the contract obligation(s) it satisfies. Inline depth
decisions for multi-implementation steps. Interface-creating steps carry the description-engineering
sub-step (empty/error/success distinguishable; selectable without full schema; errors carry fix
instructions).

> **Worked fragment (one step)**
> - **S5 — Tag safety-critical content with immunity markers.** *Prereq (verified):* S4 safety-content
>   inventory complete and signed off. *Action:* tag each inventory item per-statement; immunity =
>   {compaction, summarization, routing-downgrade}. *Gate produced:* G3 (post-tag, pre-optimization).
>   *Satisfies:* C5.RC2, PS-2. *Ordering:* must precede S12, S14 (DR1).

## 6. Safety Immunity Specification — *(Stage H)*
Content→immunity mapping; immunity scope per item (compaction / summarization / routing); ordering
verification against every optimization step; post-compaction verification procedure (immune content
preserved *verbatim* and acted on); compaction threshold (83.5%) with DR7 justification if
non-standard.

## 7. Boundary Validation Integration — *(Stage G)*
Per-boundary validation table; circuit-breaker config per boundary (threshold / timeout / reset /
degradation trigger); graceful-degradation sequence per boundary (full → reduced → LLM-only → static);
contamination-monitoring spec per agent↔agent boundary. Graduated depth per DR10–DR12 — never zero on
a cross-agent boundary.

> **Worked fragment (one boundary)**
> - **B-3 worker→orchestrator handoff.** Crosses: structured finding (JSON). Validation: schema +
>   semantic plausibility + contamination monitor. *Breaker:* 3 schema failures / 60s opens; reset
>   120s. *Degradation:* full → orchestrator re-requests → flag for human. *Monitoring:* semantic-drift
>   signature, ≥20% sampled. *Depth:* full (production agent↔agent — DR10).

## 8. Verification Gates — *(Stage F)*
Gate-by-gate spec: what to verify (traced to obligations), how (concrete test/inspection/measurement),
pass/fail criteria, consequence of failure (block / degrade / defer-with-risk). Red-team specs for
safety-critical gates. **The final gate is the boundary completeness audit**: enumerate every boundary
from the design template's map, confirm each has a validation step, inject contamination at an
allegedly-unvalidated boundary to confirm the gap would be caught.

## 9. Integration Test Scenarios — *(Stage I)*
Per-obligation test cases (injection method, expected detection, pass/fail); dependency-chain test;
red-team safety test (insert safety instruction → trigger optimization → verify survival);
boundary-validation test matrix; **scale-test procedure** at 2×/5×/10× with thresholds (success rate,
contamination rate, cost per successful task, coordination overhead — super-linear degradation = fail)
plus latency/cost to catch over-validation; escape-hatch discoverability test.

## 10. Deferred Implementation Registry — *(Stage D deferrals)*
Every item deferred from minimal-viable to production-grade, with: trigger condition for upgrade,
current risk level, and owner for the upgrade decision. This is what stops "later" becoming "never"
(F5).

## 11. Cross-Contract Conflict Resolution — *(Stage B cross-reference)*
Where multiple contracts impose conflicting obligations, surface the conflict and give resolution
guidance. The canonical one: **C1 (trace completeness, full logging) vs. C6 (progressive disclosure,
minimal defaults)** — resolved by logging the full response *out-of-band to storage*, sending the agent
the minimal default, and reconstructing from storage on demand. The dependency graph shows these as
parallel, non-conflicting concerns when logging is out-of-band. A playbook that puts full responses in
the agent's context (bypassing C6) or ignores C6 ("trace is higher priority") fails.

## 12. Rollback Plan — *(IV-1 / C2)*
For each irreversible step (schema change, protocol deployment, coordination-pattern change): the
rollback procedure and the pre-change baseline preservation (capture the baseline *before* the change —
C2.RC1). A live-interface change with no captured baseline is an un-evaluable, un-reversible change.

## 13. Implementation Cost Estimate — *(scale projections + depth)*
Token-budget impact per component (L1 — necessary vs. deferrable context); infrastructure cost
projections (storage, processing, latency); maintenance-overhead estimate. Grounds the minimal-viable
vs. production-grade decision in economics, not just risk.

## 14. Dependency on Other Playbooks — *(Stage C cross-playbook)*
Cross-reference to prerequisite and co-requisite playbooks, with explicit assumptions about what they
produce (e.g. "assumes the trace playbook has deployed reasoning capture before this coordination
playbook's contamination monitors rely on it — verify before executing Stage G"). Flag each assumption
for verification before declaring this playbook complete.
