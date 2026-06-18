# Corpus Obligations — Playbook Step Source Material

> This is the data backbone of the skill. **Every step you sequence must trace to a numbered entry
> in this file** — a law (L#), an invariant (III-#/IV-#), a contract obligation (C# RC#/FS#/ER#), a
> pattern seed (PS-#), or the failure mode it prevents (F#). A step that traces to nothing is either
> bloat or a misread contract; an obligation here with no step is a gap in the build.
>
> The mechanical rule: **each contract Required Check becomes an implementation step; each Forbidden
> Shortcut becomes a negative constraint (something the procedure must NOT produce); each Evaluation
> Requirement becomes an integration-test scenario (Stage I).**
>
> Current corpus version encoded by this file: **laws v0.2, invariants v0.2, design-contracts v0.2,
> pattern-seeds v0.2 (frozen 2026-05-30)**. Stamp this as the derivation baseline in every playbook's
> metadata. If the corpus has moved past v0.2, the playbook is [STALE] until re-verified.
>
> **Source-strength caveat:** the *patterns* below are production-grounded, but the central
> methodological assumption of this skill — that step-by-step procedural encoding improves
> implementation outcomes — is itself untested in this domain. Mark generated playbooks DRAFT and be
> explicit about which steps rest on production precedent vs. contract-logic inference.

## How to read this file

- **Laws** are design constraints — they tell you *why* an ordering or a gate must exist and what it
  guards.
- **Invariants** impose cross-cutting properties that steps must preserve (context preservation,
  change auditability).
- **Contracts** are the obligation lists — their Required Checks are the raw material for steps,
  their Forbidden Shortcuts for negative constraints, their Evaluation Requirements for tests.
- **Pattern seeds** are reusable implementation moves with when-to-use / when-NOT conditions.
- **Failure modes** are the documented ways playbooks go wrong — each names its prevention.

Reference entries by ID — *"C1.RC2"*, *"L3"*, *"DR1"*, *"PS-2"*, *"F1"* — so traceability survives
into the playbook's step-to-obligation map and safety-first ordering audit.

---

## Part 1 — The Five Laws (build constraints)

### L1 — Bounded Attention *(the thermodynamic constraint)*
An agent's effective capacity is bounded by its finite context window; every unit of information
consumed reduces reasoning capacity, and information revealed before it creates value is pure waste.
**Build consequence:** every step that adds to the agent's context budget must be justified. Each
playbook carries a context-budget verification step — after a progressive-disclosure or trace step,
verify token consumption per phase matches the intended budget. Distinguish *necessary* context
(safety instructions, immune-tagged content, trace header fields) from *deferrable* context (verbose
detail behind escape hatches, lazily-loaded schemas). **Violation symptom:** an implementation that
passes contract checks but consumes 3× the expected token budget per task; traces whose call metadata
(timestamps, session IDs) costs more context than the responses they trace; over-validation at 20+
boundaries dominating cost (F8). Anchor: 67.6% of tokens come from tool responses.

### L2 — Interface Ground Truth *(the reality constraint)*
Agents have no access to reality beyond their interfaces; every description, schema, error message,
and payload is treated as authoritative. Implicit context is invisible; missing information is
unknown; degradation at the interface is undetectable from within it. **Build consequence:** when a
step creates or modifies an interface (tool description, schema, error format, agent-to-agent
protocol), that artifact *is* the system's reality the moment it ships — description engineering is
an implementation step, not a doc chore. Every such step verifies: (a) "no results," "error," and
"empty success" explicitly distinguishable; (b) description complete enough for tool selection
without loading the full schema; (c) errors carry a fix instruction, not just a code; (d) missing
fields, rate limits, auth scopes stated. A "no" is a blocking defect. **Violation symptom:** agents
mis-selecting between two tools with near-identical default descriptions but different behavior;
treating "no output" as "successful empty result." Anchor: 20+ point accuracy swings from description
changes alone.

### L3 — Boundary Entropy *(the degradation constraint; Very High confidence)*
Information fidelity necessarily decreases at every boundary crossing — agent↔agent, agent↔tool,
context↔context, representation↔representation. Errors, contamination, and semantic drift compound at
each, and the loss is invisible to local observers. **Build consequence:** every step that creates a
boundary creates a site of degradation; boundary validation must be implemented *synchronously* with
the boundary — circuit breakers, schema checks, semantic validation, contamination monitors built in
the same step, never a later "validation phase." The final verification gate is a boundary
completeness audit: enumerate every boundary created, confirm each has validation. **Violation
symptom:** system correct at 3 components, degrades at 7; contamination propagates A→B→C undetected;
breakers at tool boundaries but not agent↔agent; boundary map has 12 entries, implementation
validates 8 — the missing 4 are the failure sites (F3). Anchors: 55.6% of contamination invisible to
outcome metrics; 5% error at boundary 1 compounds to ~30% by boundary 6.

### L4 — Compounding Friction *(the scale constraint; Very High confidence)*
Inefficiencies compound non-linearly with scale; adding components past an optimal threshold degrades
outcomes, because each adds boundary crossings (L3), consumes attention (L1), and creates a new
interface to be trusted (L2). **Build consequence:** coordination playbooks must encode the scale
dependency — an implementation that works for 3 agents (orchestrator-worker, simple breakers) won't
auto-work at 8 (closed-loop, zonal breakers, full validation). Include scale-test steps at 2×, 5×,
10× current component count; if any metric degrades super-linearly the implementation is incomplete.
**Violation symptom:** adding a 4th agent collapses a 3-agent system not from misconfiguration but
because the architecture wasn't built for 4; the failure surfaces in a production incident because no
scale gate was encoded. Anchors: inflection ~5–10 tools per agent, ~3–8 agents for mesh coordination;
integrative compromise 8–38%.

### L5 — Ergonomic Dominance *(the system-dominance constraint)*
The quality of the surrounding system — tool design, coordination, information structure, economics —
determines real-world outcomes more than the underlying model. The interface layer dominates the
intelligence layer. **Build consequence:** ergonomic patterns are not "nice to have" optimization
layers — implement them as infrastructure: mandatory, verifiable, gated. Each phase carries a "what
breaks if you skip this step" annotation anchored to the specific contract obligation, failure mode,
and empirical finding it prevents. Cutting corners ("the model can handle it") directly degrades
outcomes regardless of model quality. **Violation symptom:** state-of-the-art model, degraded
outcomes; root cause is missing boundary validation, compaction without immunity, disclosure without
escape hatches — the harness was built without playbook guidance.

---

## Part 2 — The Invariants (properties steps must preserve)

### III-1 — Context Preservation (Progressive Disclosure)
Abstractions must preserve access to local context; a layer that hides detail must provide a path
back to it. **Build consequence:** any step implementing progressive disclosure (three-level loading,
lazy descriptions, verbose flags) must verify the escape hatch is functional *and discoverable
through the agent interface* — not just documented for humans. Inspection → test: "can an agent
discover and use the detail-access path?" If not, the step is incomplete regardless of whether the
verbose format is correct (F4-disclosure).

### IV-1 — Interface Changes Must Be Evaluable / Auditable
A change to an agent-facing interface (schema, protocol, coordination pattern) must be made so its
effect can be evaluated and, where consequential, reversed. **Build consequence:** for every
irreversible step (schema change, protocol deployment, coordination-pattern change), the playbook
documents the rollback procedure and pre-change baseline preservation (output §12). A build step that
changes a live interface without a captured baseline produces an un-evaluable change.

---

## Part 3 — The Contracts (obligation lists → steps / constraints / tests)

> Each Required Check (RC#) → an implementation step. Each Forbidden Shortcut (FS#) → a negative
> constraint (the procedure must NOT produce this). Each Evaluation Requirement (ER#) → an
> integration-test scenario (Stage I). Criticality and the system-criticality input drive the
> minimal-viable vs. production-grade depth (Stage D).

### C1 — Trace Completeness *(encodes L2, L3; Domain: observability; Criticality: critical)*
A system must be debuggable from its traces alone — including across agent boundaries.

- **C1.RC1** Every tool call logged with input parameters, full response, timestamp, agent/session ID.
- **C1.RC2** The *full* tool response captured (in storage, not necessarily in the agent's context).
- **C1.RC3** Agent reasoning / intermediate state (CoT) captured, not just call/return.
- **C1.RC4** Logs span agent boundaries — cross-agent correlation so one agent's output→another's
  input is reconstructable with its degradation.
- **C1.FS1** Logging call metadata only (timestamps, IDs) without responses or reasoning.
- **C1.ER1** Inject a known error; verify it is detectable from the traces alone.
- **C1.ER2** Inject cross-agent contamination; verify the reasoning trace makes propagation visible.

*Build note:* RC3 (reasoning) must precede RC4 (cross-agent correlation) — DR2. Trace without
reasoning capture is a partial implementation whose primary value (contamination detection) isn't
realized (F7).

### C2 — Interface Change Auditability *(encodes L2, IV-1; Domain: tools; Criticality: critical)*
Agent-facing interface changes must be safe to make and to reverse.

- **C2.RC1** Pre-change baseline captured before any live interface change.
- **C2.RC2** Critical-user-journey (CUJ) tests run pre- and post-change.
- **C2.RC3** Versioning strategy applied (Pin / Scope / Test).
- **C2.RC4** Rollback procedure exists and is tested for each irreversible change.
- **C2.FS1** Changing a live tool description/schema with no baseline and no rollback.

### C4 / C8 — Boundary Validation *(encodes L3; Domain: coordination/tools; Criticality: critical)*
Every boundary where information crosses components must be validated; an unvalidated boundary
accumulates invisible contamination. *(The brief uses C4 for the build-side obligation and C8 for the
design-side; treat them as the same boundary-validation contract — cite as C8/C4.)*

- **C8.RC1** Every boundary identified and named (agent↔agent, agent↔tool, session↔session).
- **C8.RC2** Validation mechanism per boundary built *in the same step* as the boundary (schema /
  semantic / contamination check).
- **C8.RC3** Circuit-breaker per boundary: failure threshold, timeout, reset interval, degradation
  trigger.
- **C8.RC4** Graceful degradation path per boundary: full → reduced → LLM-only → static.
- **C8.RC5** Contamination monitoring at agent↔agent boundaries (semantic drift / behavioral detour /
  combined disruption signatures).
- **C8.FS1** Treating any cross-agent boundary as trusted/zero-validation (forbidden — DR12).
- **C8.FS2** A single "validation phase" deferred until after boundaries are built.
- **C8.ER1** Inject contaminated information at each boundary; verify detection and breaker behavior.

### C6 — Progressive Disclosure with Escape Hatches *(encodes III-1, L1; Domain: tools; Criticality: high)*
Interfaces default to the minimum useful information and provide a discoverable path to detail.

- **C6.RC1** Default output format minimal (target 3–4 fields).
- **C6.RC2** Verbose/full-detail mode implemented.
- **C6.RC3** Escape-hatch mechanism discoverable *through the agent interface*.
- **C6.RC4** Error response: error code + human-readable description + fix instruction + retry
  guidance (L2).
- **C6.FS1** A "default" response that is actually the verbose response; a verbose flag that reveals
  the same fields.
- **C6.ER1** Give a task needing non-default detail; verify the agent discovers and uses the escape
  hatch to complete it (F4-disclosure).

### C3 — Coordination Architecture *(encodes L3, L4; Domain: coordination; Criticality: critical)*
Multi-agent builds must configure all seven coordination dimensions. *(Referenced when a playbook
builds a coordination layer; the dimension content is owned by Design Templates — the playbook builds
to it.)* Seven dimensions: agent endpoints, communication topology, authority distribution,
synchronization regime, aggregation rules, termination conditions, **failure handling**. *Build note:*
failure handling (dim 7) must be defined before communication topology (dim 2) — failures determine
safe topology (DR5). Configured ≠ tested: the dimensions are a hypothesis until the fault-injection
drill runs (PS-11, F6).

### C5 — Compaction / Optimization Safety *(encodes L1; Domain: context management; Criticality: critical)*
Optimization under context pressure must not drop safety-critical content. *(The brief's C3/C5 safety
criterion — cite as the compaction-safety obligation.)*

- **C5.RC1** Safety-critical content inventory established before any optimization step.
- **C5.RC2** Immunity markers (PS-2) tagged before compaction/summarization/routing implemented.
- **C5.RC3** Compaction threshold 83.5% (not 100%) unless single-session-bounded (DR6).
- **C5.RC4** Post-optimization verification: immune-tagged content survives verbatim and is acted on.
- **C5.FS1** Ordering any optimization step before its safety prerequisite (the documented data-loss
  path — F1).
- **C5.ER1** Insert a safety instruction, fill context to trigger compaction at threshold, verify it
  survives and the agent respects it.

---

## Part 4 — Pattern Seeds (reusable implementation moves)

> Each seed is owned by Implementation Playbooks — the *procedure* is encoded here; the specific
> mechanism (metadata format, breaker library) is implementation-specific and not prescribed beyond
> "it works and is tested." Each seed's *when-NOT* matters as much as its *when* — it's how you avoid
> over-building simple systems.

### PS-2 — Safety Instruction Immunity Marker
**Solves:** during compaction/summarization/routing, safety-critical instructions are
indistinguishable from general ones; uniform optimization drops them (production-documented: deleted
inbox emails from 100% compaction without immunity). **Procedure (Stage H):** inventory safety content
→ tag immunity at the right granularity (per-document / per-section / per-statement) → define what
immunity means (immune to compaction / summarization / routing-downgrade) → verify every optimization
step is *after* tagging in the graph → post-compaction survival test. **Use** for any production build
with compaction, summarization, or model routing and a non-empty safety inventory. **Don't use** for
research systems where safety failure is acceptable, or single-session systems that never hit limits.
**Governs:** DR1, DR6–DR9.

### PS-3 — Boundary Map + Validation Gate
**Solves:** info crosses many boundaries with no validation; degradation invisible (55.6%) and
compounding (L4). **Procedure:** the boundary map comes from the design template (input 6); validation
gates are built in Stage G, one per boundary, synchronous with creation; depth set by DR10–DR12.
**Use** for any build creating multiple agents, agent↔tool boundaries, or context isolation. **Don't
use** for single-agent/single-tool builds with no crossing. **Governs:** DR4, DR10–DR12; guards F3.

### PS-7 — Ephemeral-First + Promote-to-Durable
**Solves:** agent artifacts accumulate without policy; storage bloats, context wasted on noise,
important results indistinguishable from noise. **Procedure:** deploy ephemeral tier → **implement and
test the promote-to-durable API first (DR13)** → define retention policies → enforce a promotion
justification → test auto-expiry AND promotion-survives-expiry. **Use** for builds with artifact
generation, parallel exploration, or long-running harnesses. **Don't use** where all output is
intrinsically valuable, or humans make preservation decisions. **Governs:** DR13–DR14; guards F5.

### PS-11 — Graceful Degradation Drill
**Solves:** multi-agent systems are built for normal operation; failure behavior is undefined and
discovered during incidents. **Procedure (Stage I test):** simulate component failure → verify
degradation activates through the four levels (full → reduced → LLM-only → static) → verify breakers
open under sustained failure → verify recovery on restoration. **Use** for production builds with
dependency chains or boundary breakers. **Don't use** for single-agent systems where full-stop is the
designed response. **Governs:** guards F6.

### PS-12 — Progressive Disclosure Audit
**Solves:** interfaces default to too much (bloat — 67.6% of tokens from tool responses) or too little
(starvation); no systematic check disclosure actually works. **Procedure:** description-engineering
verification in Stage E (distinguish default vs. verbose; empty/error/success distinguishable) + the
escape-hatch discoverability test in Stage I. **Use** for builds creating agent-facing interfaces, or
where C6 is a target. **Don't use** for trivially simple interfaces. **Governs:** DR15–DR16; guards
F4-disclosure.

### PS-13 — Compounding Friction Scale Test
**Solves:** systems work at current scale with no tested behavior at higher scale; scaling assumed
linear when it is non-linear (L4). **Procedure (Stage I test):** simulate at 2×, 5×, 10× component
count; measure task success rate, contamination rate, cost per successful task, coordination overhead;
super-linear degradation in any → implementation incomplete. **Use** for any multi-agent/multi-tool
build with growth projections. **Don't use** for stable builds well below inflection points (~5–10
tools, ~3–8 agents) with no scaling plan. **Governs:** guards F2/F-scale.

---

## Part 5 — Failure Modes (what the prevention is FOR)

> Each names symptom → likely cause → design prevention → relevant law. Recognize them in the
> playbook *and* in your own construction of it.

### F1 — Safety Instruction Loss During Compaction
**Symptom:** agent violates safety constraints under load; correct at low context utilization,
degrades as context fills; instructions present in config but absent from behavior once compaction
fires. **Cause:** compaction ordered before immunity markers; uniform compaction; threshold at 100%.
**Prevention:** PS-2 before compaction in the graph (DR1); safety inventory complete first (C5.RC1);
threshold 83.5% (DR6); post-compaction survival gate (Stage F). **Laws:** L1, L2.

### F2 — Incorrect Dependency Ordering
**Symptom:** works in isolation, fails in integration; trace can't reconstruct decisions because
reasoning capture came after tool logging; boundary validation covers 8 of 12. **Cause:** steps
followed in logical-grouping order, not dependency order; no gates at phase boundaries. **Prevention:**
DAG with explicit prerequisites (Stage C); a gate at every phase boundary (Stage F); the playbook
executable as a topological sort. **Laws:** L3, L4.

### F3 — Incomplete Boundary Validation
**Symptom:** correct at small scale, degrades as components added; contamination propagates
undetected; breakers at some boundaries not all; +2 agents → +3 unvalidated boundaries. **Cause:**
validation treated as a separate phase; new boundaries don't trigger validation; boundary map not
audited against actual boundaries. **Prevention:** validation synchronous with creation (Stage G); 1:1
map→validation; boundary completeness audit (Stage F final gate). **Laws:** L3, L2.

### F4 — Progressive Disclosure Without Escape Hatches
**Symptom:** default minimal + verbose flag exist, but agents can't discover/use the flag; tasks
needing detail fail; detail exists but is unreachable through the interface. **Cause:** focus on the
disclosure mechanism without verifying discoverability; flag documented for humans, not agents; no
test. **Prevention:** DR15 (document what's hidden + how to reach it); escape-hatch discoverability
test (Stage I); fail = incomplete regardless of verbose format. **Laws:** L2, III-1.

### F5 — Ephemeral Storage Without Promote Path
**Symptom:** ephemeral + auto-expiry deployed; important results lost on expiry; loss discovered on
retrieval. **Cause:** promote-to-durable deferred as "production-grade" while ephemeral was "minimal
viable"; deferral not reviewed before declaring done. **Prevention:** DR13 — promote path exists and is
tested first; recognize ephemeral-without-promote as a data-loss mechanism, not a minimal-viable
subset (Stage D floor). **Laws:** L1, L2.

### F6 — Coordination Deployed Without Fault Injection
**Symptom:** 7 dimensions configured, works in normal operation; single-agent failure behavior unknown;
discovered in a production incident. **Cause:** dimensions configured in design but failure handling
never tested against actual faults — a hypothesis, not infrastructure. **Prevention:** PS-11
fault-injection drill in Stage I; verify four-level degradation, breaker opening, recovery; required
before declaring coordination done. **Laws:** L3, L4.

### F7 — Trace That Captures Calls But Not Reasoning
**Symptom:** every call logged, but "why did the agent decide this?" is unanswerable; cross-agent
contamination undetectable; trace can't answer the question it was built for. **Cause:** RC1/RC2
(call logging) satisfied, RC3 (reasoning) deferred as production-grade; symptoms captured, causes not.
**Prevention:** DR2 — reasoning capture precedes cross-agent correlation; Stage D recognizes
trace-without-reasoning as partial (primary value unrealized). **Laws:** L2, L3.

### F8 — Over-Validation Consuming Context Budget
**Symptom:** full-depth validation on every boundary; correct per C8 but at 20+ boundaries validation
overhead dominates cost; agents validate more than they work. **Cause:** max depth applied uniformly;
DR10–DR12 not used as explicit gates; safety chosen over efficiency without recognizing low-criticality
over-validation as waste. **Prevention:** graduated depth per boundary (DR10–DR12, applied per
boundary not globally); latency/cost measurement in Stage I alongside correctness. **Laws:** L1, L4.
