# The Implementation Design Procedure — Stages A–I

> Nine stages, in order. Prerequisites before obligations; obligations before the graph; the graph
> and depth decision before the ordered steps; gates, boundary validation, and immunity tagging
> woven into that order; tests last because they exercise the whole chain. Each stage names the
> output section it produces (see `output-template.md`) and the obligations it draws from
> (`corpus.md`).
>
> You won't always run all nine at full depth. A single-contract trace buildout is mostly A, B, C, E,
> F, I. A compaction rollout centers on H, then F's red-team gate. A coordination buildout leans on
> C, G, and I's scale + fault-injection tests. **Stage order is what's load-bearing when stages do
> run** — never let an optimization stage precede the safety stage it depends on.

---

## Stage A — Infrastructure Prerequisite Audit

Verify the ground exists before you build on it. **This is where you prevent the
passes-tests-fails-deployment failure.**

1. Cross-reference the target contract's Required Checks (`corpus.md` Part 3) against the existing
   infrastructure inventory (input 2): trace storage backend, circuit-breaker mechanism, storage
   tiering, fault-injection harness, coordination-topology awareness, monitoring.
2. For each missing prerequisite, classify it **hard** (build cannot proceed) or **soft** (build can
   proceed with degraded guarantees).
3. Produce a **go/no-go per prerequisite.** If a hard dependency is missing and can't be provisioned,
   emit a *blocking prerequisite gap* — do not proceed to a playbook that assumes infrastructure into
   existence.
4. For missing soft dependencies, document the degradation consequence and record a deferred-setup
   task with a deadline before production deployment.

**Output:** Prerequisites Audit — hard-dependency inventory (status per item), soft-dependency
inventory (degradation consequence per item), blocking-gaps list, deferred-setup tasks with deadlines.

---

## Stage B — Contract Obligation Review

Map every obligation to a build action; **start from the full Required-Check list, not intuition.**

1. Pull every **Required Check** from the target contract — each becomes an implementation step.
2. Pull every **Forbidden Shortcut** — each becomes a *negative constraint* (something the procedure
   must NOT produce).
3. Pull every **Evaluation Requirement** — each becomes an integration-test scenario (carried to
   Stage I).
4. Map each obligation to a concrete step: *"C1.RC1: every tool call logged with input, full response,
   timestamp, agent/session ID" → "Step: implement per-agent tool-call capture with these four
   fields."*
5. Flag obligations that span stages (e.g. "logs span agent boundaries" needs per-agent logging AND
   cross-agent correlation) as **cross-cutting** — they need verification after *both* stages.

**Output:** Contract Obligation Map — obligation→step table, Forbidden Shortcuts as constraints,
Evaluation Requirements as test scenarios, cross-cutting obligations flagged.

---

## Stage C — Dependency Graph Construction

Build the DAG. **This is the heart of the skill — the order is the safety property.**

1. For each step from B, declare its prerequisites: what infrastructure, prior step, or verified
   obligation must exist before it can begin.
2. Sort by **dependency, not logical grouping.** No-prerequisite steps form the first wave; steps
   whose prerequisites are satisfied by the first wave form the second; and so on.
3. **Safety-first ordering (the audit that ships in the output):** identify every optimization step
   (compaction, summarization, lazy loading, model routing) and verify its safety prerequisite
   (immunity markers, safety inventory, boundary validation, safe-task classification) appears
   *earlier* in the graph. If not, reorder (DR1, DR2, DR5, DR9).
4. Verify the graph: no cycles; no missing edges (a step depending on infra not in the Stage A audit);
   no orphans (no path from start, or no path to completion).
5. Insert **phase boundaries** where the build crosses infrastructure domains (logging → boundary
   validation → coordination configuration). Each boundary will carry a gate (Stage F).

**Output:** Dependency Graph — ordered step list with prerequisite declarations, topological order,
**safety-first ordering audit** (each optimization step paired with its safety prerequisite, ordering
shown), phase-boundary markers.

---

## Stage D — Minimal-Viable vs. Production-Grade Decision

Decide depth per phase; **make the line explicit and anchored, never defaulted.**

1. For each phase, apply system criticality (input 3) and scale projections (input 4) to choose
   minimal-viable or production-grade.
2. Anchor each decision to the relevant corpus decision point: **D1** trace depth (full reasoning
   capture vs. event-only sampling), **D2** compaction safety boundary (83.5% + justification), **D3**
   boundary-validation rigor (full vs. statistical per boundary).
3. For each minimal-viable choice document: what is deferred, minimum acceptable risk, the **upgrade
   trigger** (scale threshold / criticality increase), and what breaks if the deferred piece is never
   built.
4. For each production-grade choice document: the additional infrastructure, additional steps, and
   test scenarios required.
5. **Floor check:** confirm you haven't deferred a non-deferrable into "later" — promote-to-durable
   for ephemeral storage (DR13, F5), reasoning capture for contamination-detecting trace (DR2, F7),
   validation for any cross-agent boundary (DR12). These are minimal-viable floor, not production
   ceiling.

**Output:** Implementation Depth Decision — per-phase classification table, deferral rationale per
minimal-viable choice (anchored to criticality + scale), upgrade triggers, infrastructure delta for
production-grade.

---

## Stage E — Step Sequencing (Safety-First)

Produce the final ordered procedure from the graph (C) and depth decisions (D).

1. Emit the ordered sequence of build actions.
2. Each step carries: step number; what the implementer does; the infrastructure / prior step that
   must be *verified* complete before starting; the verification gate the step produces; and the
   **contract obligation(s) it satisfies** (the traceability that ships).
3. Safety-critical steps (immunity tagging, boundary-validation deployment, degradation paths, safety
   inventory) appear **before** the optimization steps that depend on them — non-negotiable.
4. For steps with multiple valid implementations (trace: full vs. sampling), inline the Stage D
   rationale.
5. For steps that create interfaces (tool descriptions, schemas, error formats, protocols), include
   the **description-engineering sub-step** (L2 / PS-12): empty vs. error vs. success distinguishable;
   description complete enough for selection without the full schema; errors carry fix instructions.

**Output:** Implementation Procedure — ordered step list (number, prerequisites, action, obligation
reference, verification gate), inline depth decisions, safety-first ordering reflected in the order.

---

## Stage F — Verification Gate Insertion

Embed a checkpoint at every phase boundary; **a prerequisite is done when a gate confirms it.**

1. At each phase boundary from C, insert a gate: "Before Phase N+1, verify Phase N produced a
   compliant implementation."
2. Each gate specifies: **what** to verify (traced to specific obligations from B), **how** (a
   concrete test, inspection, or measurement), **pass/fail** criteria, and the **consequence of
   failure** (block until fixed / continue with degraded guarantee / defer with explicit risk
   acceptance).
3. Gates for safety-critical phases (compaction, boundary validation, coordination) include
   **red-team tests**: inject the failure, verify the implementation handles it (e.g. insert a safety
   instruction, trigger compaction at 83.5%, verify survival — C5.ER1).
4. The **final gate is a boundary completeness audit** (L3, F3): enumerate every boundary from the
   design template's map, verify each has validation in the procedure, and inject contamination at an
   (allegedly) unvalidated boundary to confirm the gap would be caught.

**Output:** Verification Gates — gate-by-gate spec (what / how / pass-fail / consequence), red-team
specs per safety-critical gate, the boundary completeness audit procedure.

---

## Stage G — Boundary Validation Integration

Make validation synchronous with boundary creation; **never a separate later phase (C8.FS2).**

1. For every boundary in the inventory (input 6), confirm the procedure (E) builds validation in the
   same step that creates the boundary. If the design template's boundary map doesn't exist, flag
   "verify template exists before executing this stage."
2. Set validation mechanism by boundary type: agent↔agent → schema + semantic + contamination
   detection; agent↔tool → schema + error-code verification; session↔session → state-consistency
   checks.
3. Configure a circuit breaker per boundary: failure threshold, timeout, reset interval, degradation
   trigger.
4. Define the graceful-degradation sequence per boundary: full → reduced → LLM-only → static (C8.RC4).
5. Insert contamination monitoring at each agent↔agent boundary: specify the signature detection
   (semantic drift / behavioral detour / combined disruption).
6. Apply graduated depth (DR10–DR12) — full validation for production agent↔agent; statistical with a
   defined sampling rate for high-volume tool boundaries; **never zero for any cross-agent boundary.**

**Output:** Boundary Validation Integration — per-boundary validation table, circuit-breaker config
per boundary, degradation sequence per boundary, contamination-monitoring spec.

---

## Stage H — Safety Immunity Tagging

Ensure safety-critical content is immune-tagged **before any optimization step** (PS-2, DR1).

1. Take the safety-critical content inventory (input 5) and produce an immunity-tagging spec: which
   content gets immunity, at what granularity (per-document / per-section / per-statement), and what
   immunity means here (immune to compaction / summarization / routing-downgrade, or all three).
2. Verify every optimization step (compaction, summarization, routing, lazy loading) appears **after**
   the tagging step in the graph. If not, reorder (this is a hard ordering constraint, not a
   preference).
3. Specify post-optimization verification: after any compaction event, confirm immune-tagged content
   is preserved *verbatim* (not paraphrased) and the agent's behavior respects it (C5.RC4, DR8).
4. Set the compaction threshold: **83.5%** (production-validated), with the DR7 justification if a
   different threshold is proposed (safety-content size, headroom, utilization history, red-team
   result at the proposed threshold).

**Output:** Safety Immunity Specification — content→immunity mapping, immunity scope per item, ordering
verification against optimization steps, post-compaction verification procedure, threshold +
justification.

---

## Stage I — Integration Test Generation

Generate the tests that validate the whole chain; **implementation without tests is design.**

1. For each Evaluation Requirement from B, produce a concrete test: injection method, expected
   detection, pass/fail.
2. **Dependency-chain test:** exercise the full sequence; verify each prerequisite was complete when
   each dependent step executed.
3. **Red-team safety test:** insert a safety instruction, trigger the optimization path
   (compaction/summarization/routing downgrade), verify it survives and behavior respects it (F1).
4. **Boundary validation tests:** inject contaminated input at each boundary, verify detection; verify
   breaker behavior under sustained failure; verify graceful degradation activates correctly (C8.ER1,
   PS-11 — the four-level drill, F6).
5. **Scale tests (PS-13):** simulate at 2×, 5×, 10× component count; measure task success rate,
   contamination rate, cost per successful task, coordination overhead; super-linear degradation in
   any metric → implementation incomplete (L4, F2-scale). Include latency/cost measurement to catch
   over-validation (F8).
6. **Escape-hatch discoverability test:** give a task needing non-default detail; verify the agent
   discovers and uses the verbose path (C6.ER1, F4).

**Output:** Integration Test Scenarios — per-obligation test cases, dependency-chain procedure,
red-team safety specs, boundary-validation test matrix, scale-test procedure with thresholds,
escape-hatch discoverability test.
