# Output Template — The Design-Template Artifact

> A full template artifact has these twelve sections. **Emit only the ones the domain calls for** —
> a boundary-validation template needs sections 1, 3, 9, 10, 11; a coordination template needs 1, 2,
> 3, 4, 5, 9–12; a tool-interface template needs 1, 6, 7, 9–12. **Don't pad:** a section the domain
> doesn't touch gets one line ("N/A — single-agent system, no inter-agent boundaries").
>
> The artifact is downstream-usable: its fields feed [[Review Checklists]] (each field's verifying
> item), [[Implementation Playbooks]] (the target shape to build to), and [[Decision Records]] (the
> reversibility fields). **Keep field IDs and contract references intact** so the chain survives.
>
> Each field, when you write it, carries four things: a **prompt** (one sentence), a **citation**
> (the corpus obligation it encodes, e.g. "C3.d"), a **strictness tier** (Mandatory /
> Evidence-Required / Advisory), and — for complex or evidence-required fields — an **example of a
> correct answer**.

The sections, with what each carries and which stage produces it:

## 1. Domain Scope & Contract Mapping — *(Stage A)*
Target artifact domain; contracts encoded with clause references; relevant invariants; a table mapping
each contract obligation → template field. **Plus the ownership-boundary statement**: what this template
covers and — explicitly — what it does not, with the WHAT/HOW line drawn (design here; build →
Implementation Playbooks; verify → Review Checklists). If the domain is thin-evidence, the [PRELIMINARY]
mark and why (DR12).

> **Worked fragment**
> - **Domain:** coordination architecture (orchestrator-worker, 5→8 agents).
> - **Encodes:** C3 (all 7 dimensions), C8 (agent↔agent + agent↔orchestrator boundaries), II-3
>   (topology reversibility), IV-2 (scale projection).
> - **Does NOT cover:** how to *configure* the orchestrator runtime → [[Implementation Playbooks]];
>   whether the built system performs → [[Evaluation Harnesses]]. [ASSUMPTION: checklist item C3.g
>   exists for structural expertise enforcement — verify before deploy.]

## 2. Coordination Dimensions — *(Stage B, C3)*
Seven mandatory sub-sections, one per configurable dimension, each with completion guidance: **Agent
Endpoints** (C3.a), **Communication Topology** (C3.b), **Authority Distribution** (C3.c — structural,
not prompted, expertise enforcement), **Synchronization Regime** (C3.d), **Aggregation Rules** (C3.e),
**Termination Conditions** (C3.f), **Failure Handling** (C3.g). Plus a **failure-signature field**
characterizing the chosen topology's known failure mode. Synchronization and termination are the most
commonly omitted — never let them silently drop (F3).

## 3. Boundary Map — *(Stage B, C8 / PS-3)*
**Structured enumeration**, not prose. One row per boundary with: boundary ID, what crosses
(type/format), classification (format/semantic/protocol/state), validation mechanism (C8.c),
circuit-breaker threshold *(evidence-required — C8.d)*, graceful degradation path *(evidence-required —
C8.e; full → reduced → LLM-only → static)*, contamination monitoring config *(evidence-required — C8.f)*. Exhaustive: every
agent↔agent, agent↔tool, session↔session crossing named.

> **Worked fragment (one boundary)**
> - **B-3 worker→orchestrator result handoff.** Crosses: structured finding (JSON). Class: semantic.
>   Validation: schema check + semantic plausibility check. *Circuit breaker [Evidence-Required]:* "3
>   consecutive schema failures within 60s opens the breaker; tested 2026-05-15 with synthetic
>   malformed payloads, opened at threshold." *Degradation [Evidence-Required]:* full → orchestrator
>   re-requests → flag for human; tested 2026-05-15 — orchestrator correctly re-requested after
>   injected failure on B-3 and degraded output correctly flagged. *Monitoring [Evidence-Required]:*
>   contamination rate on this boundary, sampled ≥20%.

## 4. Reversibility Analysis — *(Stage D, II-3 / PS-15)*
Per architectural decision, **inline with the decision** (DR4/DR6): evidence tier (T1–T4), evidence
source citation, reversal-cost estimate (low/medium/high/prohibitive + justification),
optionality-preserving alternatives evaluated, trigger conditions for reconsideration, next
re-evaluation date. Decisions on T3/T4 evidence MUST display inline.

## 5. Scale Projection — *(Stage D, IV-2 / L4)*
Current component counts (agents, tools, boundaries); projected counts at 2× and 5×; known inflection
points for the domain (~5–10 tools, ~3–8 agents); scale-test result placeholders *(evidence-required —
must hold a test result or be flagged unfilled)*; scaling strategy (add agents? optimize existing?
redesign coordination?). Reject "designed to scale" — require a test result at projected scale.

## 6. Progressive Disclosure Specification — *(Stage D, C6 / III-1 / PS-12)*
Per interface: default output format (target 3–4 fields, C6.a); verbose-flag spec (C6.b); escape-hatch
mechanism (C6.c); error response structure — error code + human-readable description + fix instruction
+ retry guidance (C6.d, L2); what is hidden at each level and how to reach it (C6.e).

## 7. Tool Interface Specification — *(Stage B, C2 / C6)*
Agent-facing, designed for agent consumers. Fields for: tool description format (C2.a); default vs.
verbose output (C2.b); error format (C2.c, per 6); **CUJ test *plan* spec** (WHICH workflows, WHICH
pre/post baselines — not HOW to run them, C2.d); **versioning strategy spec** (WHICH of Pin/Scope/Test
— not HOW to configure it, C2.e); schema documentation (C2.f). Push all HOW content to
[[Implementation Playbooks]] / [[Evaluation Harnesses]] (F6).

## 8. Evaluation System Design — *(Stage B, C5)*
Fields for: metric-to-purpose alignment, each metric traced to a system purpose (C5.a, anti-Goodhart);
gold-set spec — size, recency, calibration (C5.b); judge model spec — different-family constraint
(C5.c); calibration cadence (C5.d); eval-set rotation plan (C5.e). The *methodology* for these metrics
and calibration → [[Evaluation Harnesses]]; this section specifies only what the design must contain.

## 9. Checklist Traceability Matrix — *(Stages F, H)*
Table mapping every field → its verifying [[Review Checklists]] item. Bidirectional coverage scores.
Gap flags: **[UNVERIFIABLE]** (field with no checklist item), **[UNTEMPLATED CHECK]** (checklist item
with no field). Checklist version reference. Target ≥0.9 both ways; below 0.7 is divergence (F4).

## 10. Strictness Tier Assignment — *(Stage G)*
Every field classified Mandatory / Evidence-Required / Advisory. Evidence-validity criteria for
evidence-required fields (what counts; the explicit reject-list). Skip-justification format for advisory
fields. Guidance annotations where designer expertise is below the floor (DR11).

## 11. Template Metadata & Versioning — *(Stage I)*
Template version (semantic); corpus artifact versions encoded (laws/invariants/contracts/pattern-seeds
v0.2); checklist version; generation date; staleness-check date; the staleness rule (DR13); designer
expertise floor; expected completion time; ownership-boundary statement; downstream-brief references.

## 12. Cross-Reference Map — *(Stages F, I)*
Table of downstream owners this template references: [[Review Checklists]] (verifying items),
[[Implementation Playbooks]] (build procedures), [[Decision Records]] (evidence-tagged decision format),
[[Evaluation Harnesses]] (eval methodology), [[Principle-to-Implementation Guides]] (the template-guided
design as a teaching example), [[Debugging Workflows]] (uses the boundary map + failure signatures to
trace a production failure).
