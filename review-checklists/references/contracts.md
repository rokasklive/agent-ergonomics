# The Eight Design Contracts — Checklist Source Material

> This is the data backbone of the skill. Every checklist item you write must trace to a
> numbered obligation in this file. The mechanical rule: **each Required Check becomes a
> checklist item; each Forbidden Shortcut becomes a verification question** ("is there
> evidence this shortcut was avoided?").
>
> The contracts are operational encodings of the corpus laws and invariants. They specify
> what a system must *preserve, prove, check, or avoid* — not how to design or implement it.
> Current corpus version for these contracts: **v0.2 (frozen 2026-05-30)**. Record this as
> the derivation baseline in every checklist's Version Metadata.

## How to read this file

For each contract you get:
- **Encodes / Domain / Criticality** — what invariant or law it operationalizes, which of the
  five ergonomic domains it covers, and how critical it is.
- **Evidence class** — *behavioral* (must observe actual agent behavior → tested/monitored
  evidence floor), *structural* (design documentation can suffice at stable scale), or
  *economic* (requires measurement). This drives evidence-standard assignment (see
  `decision-rules.md`, DR8–DR12).
- **In scope when** — the review triggers that pull this contract into a checklist.
- **Required Checks (RC#)** — translate each into a pass/fail verification question.
- **Forbidden Shortcuts (FS#)** — translate each into "is there evidence this was avoided?"
- **Required Evidence** — what the reviewer must actually inspect.

Reference items by ID, e.g. *"C2 RC2"* or *"C4 FS1"*, so traceability from item to contract
survives into the output.

## Contract dependency rule (read before selecting scope)

**C1 (Trace Chain Completeness) is logically prior to C2, C4, and C5.** You cannot
independently validate CUJ test evidence (C2), safety-instruction survival (C4), or
trajectory-level evaluation (C5) without traces. If a review includes C2/C4/C5 at the
*tested* evidence level but C1 has never been verified, you must either run a lightweight
C1 check first or downgrade those contracts to **conditional-pass** pending trace
infrastructure. Skipping this is Failure F6 (Missing Contract Dependencies).

---

## C1 — Trace Chain Completeness

- **Encodes:** Invariant I-1 (every agent decision traceable to inputs) · **Domain:** all 5 ·
  **Criticality:** Critical · **Evidence class:** behavioral/structural (the *existence* of
  traces is structural; *that they reconstruct a decision* is behavioral)
- **In scope when:** compaction policy change; onboarding; any full-system review; whenever
  C2/C4/C5 are in scope at tested evidence (dependency check). Apply C1 **first** in
  full-system and onboarding reviews.

**Required Checks**
- **RC1** Every tool call is logged with input parameters, full response, timestamp, agent session ID.
- **RC2** Every reasoning step (CoT, intermediate states) is captured where available.
- **RC3** Logs span agent boundaries — not per-agent silos.
- **RC4** Trace reconstruction is possible: given a decision, you can answer "what inputs led to this?"
- **RC5** Traces are reviewed at a regular cadence (weekly minimum for production).

**Forbidden Shortcuts**
- **FS1** Logging only final outcomes without intermediate steps.
- **FS2** Per-agent logging without cross-agent trace correlation.
- **FS3** Logging that drops reasoning steps under memory pressure.
- **FS4** Sampling traces below a rate that preserves detectability of rare failures.

**Required Evidence:** a sample trace reconstructing a non-trivial decision end-to-end;
trace format + retention policy doc; evidence of regular review (schedule, recent notes).
**Pattern seed:** Trace Review Cadence (PS-8) — verify traces are *reviewed*, not merely *collected*.

---

## C2 — Interface Change Auditability

- **Encodes:** Invariant IV-1 · **Domain:** Tool & Interface Design (primary), Information
  Architecture · **Criticality:** Critical · **Evidence class:** behavioral
- **In scope when:** any change to a tool description, error-message format, output schema,
  naming convention, or protocol/Agent-Card metadata. Per-change **gate** model (DR4):
  C2 cannot be deferred to a scheduled audit.

**Required Checks**
- **RC1** Pre-change agent-behavior baseline exists and is documented.
- **RC2** Description changes are tested with actual agents (CUJ testing), not just schema validation.
- **RC3** Tool versions are pinned in production (`@v1.2.0`, not rolling-release).
- **RC4** Description changes that cause agent regression are classified as breaking changes.
- **RC5** Hierarchical naming convention prevents semantic collisions across tools.

**Forbidden Shortcuts**
- **FS1** Deploying description changes without CUJ testing because "the schema didn't change."
- **FS2** Rolling-release versioning for production agent tools.
- **FS3** Renaming tools without recognizing it as a breaking change.
- **FS4** Testing only the tool implementation, not agent behavior with the tool.

**Required Evidence:** CUJ test results comparing pre- and post-change agent behavior;
version-pinning config; breaking-change classification criteria; rollback procedure.
**Highest-leverage item in the whole set:** RC1/RC2 — "Are pre- and post-change agent behavior
baselines compared?" The 47-call loop bug and 20+ point accuracy swings were both
description-change failures this item catches. **Pattern seed:** CUJ Change Gate (PS-4).

---

## C3 — Coordination Architecture Design

- **Encodes:** Invariant IV-2 · **Domain:** Workflow & Coordination (primary), Tool & Interface ·
  **Criticality:** Critical · **Evidence class:** structural (documented evidence acceptable
  at *stable* scale per DR9; tested at/after scaling events)
- **In scope when:** new agent added; tool count crosses the single-agent threshold (~5–10);
  agent count crosses the multi-agent mesh threshold (~3–8); scheduled audit (quarterly).
  Scheduled-audit model (DR6), not per-change gate.

**Required Checks**
- **RC1** The 7 configurable coordination dimensions are explicitly defined: agent endpoints,
  communication topology, authority distribution, synchronization regime, aggregation rules,
  termination conditions, failure handling.
- **RC2** The coordination configuration has a documented failure signature.
- **RC3** Scale testing validates that adding components does not degrade outcomes.
- **RC4** Structural expertise enforcement exists where expertise is unevenly distributed.
- **RC5** Circuit breakers exist at all agent-to-agent and agent-to-tool boundaries.

**Forbidden Shortcuts**
- **FS1** Adding agents to an undesigned coordination architecture.
- **FS2** Consensus-seeking as default without explicit expertise weighting.
- **FS3** Assuming prompted expertise is sufficient without structural enforcement.
- **FS4** Linear pipeline architectures without fault-propagation testing.

**Required Evidence:** documented architecture covering all 7 dimensions; failure-signature
characterization; scale-test results bracketing current and planned size; circuit-breaker
config + test evidence. **Pattern seed:** Coordination Dimension Checklist (PS-6) — the 7
items map directly to the 7 dimensions.

---

## C4 — Safety-Critical Information Preservation

- **Encodes:** Invariant II-1 · **Domain:** all 5 (anywhere optimization touches safety) ·
  **Criticality:** Critical · **Evidence class:** behavioral — **tested evidence floor for
  every C4 item; documented claims are never sufficient.**
- **In scope when:** any change to compaction thresholds, summarization algorithms, model
  routing rules, or context-isolation policies. Per-change **gate** model (DR5).

**Required Checks**
- **RC1** Safety-critical information is explicitly tagged and tracked.
- **RC2** Compaction/summarization preserves all safety-tagged information.
- **RC3** Model routing never sends safety-critical tasks below a defined capability floor.
- **RC4** Optimization paths are tested with safety-critical information present.
- **RC5** Human-in-the-loop gates are mandatory regardless of optimization path.

**Forbidden Shortcuts**
- **FS1** Compaction at 100% context (drops safety headroom; verified threshold is ~83.5%).
- **FS2** Routing safety-critical tasks to cheaper/weaker models.
- **FS3** Disabling safety checks under load or cost pressure.
- **FS4** Summarization that drops safety constraints without explicit review.

**Required Evidence:** inventory of safety-critical information; compaction test results
showing instructions survive; routing rules documenting capability floors; recent
optimization-path safety-regression test results. Reference precedent: Claude Code's ~83.5%
compaction threshold with CLAUDE.md immunity; the deleted-inbox-emails failure. The
verification is a **red-team injection** (Hook H2): inject a safety instruction, run full
optimization, confirm it survived.

---

## C5 — Evaluation Validity

- **Encodes:** Invariant II-2 · **Domain:** Evaluation & Measurement (primary), constrains all ·
  **Criticality:** Critical · **Evidence class:** behavioral, **time-windowed** — calibration
  and rotation are ongoing, so items must check recency, not just existence.
- **In scope when:** model routing change; evaluation-system change; scheduled C5 audit
  (monthly for critical, quarterly for stable).

**Required Checks**
- **RC1** Metrics aligned with actual purpose (cost per successful task, not per call; Pass^k, not Pass@k).
- **RC2** Human gold-set exists (200–500 traces min) and is calibrated monthly (Cohen's kappa).
- **RC3** LLM-as-Judge uses a different model family than the system being evaluated.
- **RC4** Trajectory-level analysis is mandatory — outcome-only metrics are insufficient.
- **RC5** Eval set is refreshed regularly to prevent rot.

**Forbidden Shortcuts**
- **FS1** Outcome-only metrics without trajectory validation.
- **FS2** Same-family LLM judges (e.g., Claude evaluating Claude).
- **FS3** No human gold-set, or gold-set older than 90 days without recalibration.
- **FS4** Pass@k as the reliability metric for production readiness.
- **FS5** Vague rubrics ("is the answer correct?") without criteria.

**Required Evidence:** documented metric-to-purpose alignment; gold-set size/recency/kappa;
judge model spec showing different-family constraint (actual model IDs, not just the claim);
trace-review cadence; eval-set refresh date. **Phrase items as recency questions:** "Is the
gold-set calibrated within 90 days?" not "Was the gold-set calibrated?" **Pattern seeds:**
Gold-Set Rotation Protocol (PS-18), Metric-to-Purpose Alignment Audit (PS-5).

---

## C6 — Progressive Disclosure With Escape Hatches

- **Encodes:** Invariant III-1 · **Domain:** Tool & Interface Design, Information Architecture ·
  **Criticality:** High · **Evidence class:** structural (documented acceptable in low-risk
  systems; tested when agents have been observed failing)
- **In scope when:** tool description change (alongside C2); new tool added; agents exhibit
  context bloat or tool-selection errors.

**Required Checks**
- **RC1** Default responses return minimal information (3–4 fields).
- **RC2** Verbose/full response is available via explicit flag or parameter.
- **RC3** Error responses include fix instructions, not just error codes.
- **RC4** Abstractions document what they hide and how to access it.
- **RC5** Tool descriptions are complete enough for tool selection without loading full schemas.

**Forbidden Shortcuts**
- **FS1** Default responses with insufficient information for common tasks.
- **FS2** Abstractions that permanently hide detail with no access path.
- **FS3** Error responses returning only codes without diagnostic info.
- **FS4** Tool descriptions so minimal agents cannot distinguish similar tools.

**Required Evidence:** interface docs showing default vs. verbose formats; escape-hatch
mechanism docs; sample error response with fix instructions; abstraction docs listing hidden
detail + access path. **Pattern seed:** Progressive Disclosure Audit (PS-12).

---

## C7 — Cost Per Successful Outcome

- **Encodes:** Invariant II-2 (economic) · **Domain:** Economic Design (primary), Evaluation ·
  **Criticality:** High · **Evidence class:** economic (requires measurement)
- **In scope when:** model routing change; cost-optimization initiative; scheduled economic audit.

**Required Checks**
- **RC1** Cost is tracked per successful task, including retries, idle time, coordination overhead.
- **RC2** Pricing model (bounded vs. usage-based) is evaluated against agent exploration patterns.
- **RC3** Token distribution is measured (tool responses, system prompts, history, retrieved context).
- **RC4** Format economics are calculated ($/year per tool at current scale).

**Forbidden Shortcuts**
- **FS1** Tracking cost per call and assuming it correlates with cost per successful task.
- **FS2** Optimizing system prompts (~3.4% of tokens) while ignoring tool-output bloat (~67.6%).
- **FS3** Usage-based pricing without measuring its effect on agent exploration.
- **FS4** Ignoring idle-time costs in cost-per-task calculation.

**Required Evidence:** cost-per-successful-task dashboard/report; token-distribution analysis;
format-choice economic analysis (e.g. JSON vs. TOON vs. CSV at current scale); pricing-model
evaluation. **Note:** C7 verifies that *economic analysis exists* — it does **not** prescribe
a format. "Switch to TOON" is design guidance (escalate to *Design Templates*), not a
checklist item.

---

## C8 — Boundary Validation

- **Encodes:** Law L3 (Boundary Entropy), operationalized · **Domain:** all 5 ·
  **Criticality:** Critical · **Evidence class:** behavioral/structural (the boundary map is
  structural; validation effectiveness is behavioral)
- **In scope when:** new agent, new tool, or new service boundary added; scheduled audit
  (quarterly for production). Per-change gate when the change *adds* a boundary; audit model
  for ongoing boundary health (DR6/DR14).

**Required Checks**
- **RC1** All system boundaries are explicitly identified and documented (a boundary map exists).
- **RC2** Validation exists at each agent-to-agent boundary (format, schema, semantic checks).
- **RC3** Circuit breakers exist at all inter-service boundaries.
- **RC4** Graceful degradation paths are defined (full → reduced → LLM-only → static).
- **RC5** Contamination-detection monitors are in place at agent-to-agent boundaries.

**Forbidden Shortcuts**
- **FS1** Trusting upstream agent output without validation.
- **FS2** Forwarding tool results across agent boundaries without schema/semantic check.
- **FS3** Single retry loop without circuit breaker at any boundary.
- **FS4** No graceful degradation path — system is either fully operational or fully down.

**Required Evidence:** boundary map; per-boundary-type validation docs; circuit-breaker config
+ test results; graceful-degradation drill results; contamination-monitoring baseline +
alert thresholds. The 55.6% invisible-contamination finding is the direct consequence of
unvalidated agent-to-agent boundaries. **Pattern seed:** Boundary Map + Validation Gate (PS-3)
— RC1 (the map) must pass before RC2–RC5 can be verified.

---

## Trigger → contract scope map (Stage A)

| Review trigger | Contracts in scope | Notes |
|---|---|---|
| Tool description change | **C2 (full)** + **C6 (lighter)** | DR13. Exclude others unless tool count or topology also changed. |
| New tool addition | C2, C6, **C8** | C8 because a new tool adds a boundary. |
| New agent addition | **C3 (full)** + **C8 (full)** | DR14. Add a scale test if an inflection point is crossed. |
| Compaction policy change | **C1**, **C4** | DR5. C1 because C4 verification needs traces. |
| Model routing change | C4, C5, C7 | Routing touches safety floors, eval validity, and cost. |
| Scheduled health check | **Rotate** across cycle | DR15. Never all 8 at full depth in one session. |
| Onboarding / first review | All 8 at **reduced depth** | DR16. Purpose is gap identification, not gating. |

## Domain coverage (L5 reminder)

A checklist that covers only C2/C6 (tool design) is blind to coordination, safety, evaluation,
and cost — all of which dominate outcomes per L5 (Ergonomic Dominance). The five domains and
their contracts: **Tool & Interface** (C2, C6) · **Workflow & Coordination** (C3, C8) ·
**Information Architecture** (C4, C6) · **Economic** (C7) · **Evaluation & Measurement**
(C1, C5). When a *full-system* review is in scope, confirm every domain is represented before
finalizing.
