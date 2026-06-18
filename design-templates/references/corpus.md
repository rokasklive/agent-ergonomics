# Corpus Obligations — Template Field Source Material

> This is the data backbone of the skill. **Every template field you write must trace to a
> numbered entry in this file** — a law (L#), an invariant (II-#/III-#/IV-#), a contract obligation
> (C# RC#/FS#), or a pattern seed (PS-#). A field that traces to nothing is bloat (F1); an
> obligation here with no field is a missing dimension (F3).
>
> The mechanical rule: **each contract Required Check becomes a candidate mandatory or
> evidence-required field; each invariant inspection method becomes a field-completion criterion;
> each pattern seed becomes a set of advisory fields** (mandatory only where it overlaps a
> contract — DR10).
>
> Current corpus version encoded by this file: **laws v0.2, invariants v0.2, design-contracts
> v0.2, pattern-seeds v0.2 (frozen 2026-05-30)**. Stamp this as the derivation baseline in every
> template's Version Metadata. If the corpus has moved past v0.2, the template is [STALE] until
> re-verified (DR13).

## How to read this file

- **Laws** are design constraints — they tell you *why* a field set must exist and what it guards.
- **Invariants** impose cross-cutting properties — they add fields that span sections (reversibility,
  context preservation, scale projection).
- **Contracts** are the obligation lists — their Required Checks are the raw material for mandatory
  and evidence-required fields.
- **Pattern seeds** are reusable design moves — advisory field clusters, with when-to-use / when-not.

Reference entries by ID — *"C3.d"*, *"C8 RC2"*, *"II-3"*, *"PS-15"* — so traceability from field to
obligation survives into the template's Cross-Reference Matrix.

---

## Part 1 — The Five Laws (design constraints)

### L1 — Bounded Attention
An agent's (and a designer's) effective capacity is bounded by a finite context window; every unit
consumed reduces capacity for reasoning, and information revealed before it creates value is pure
waste. **Design consequence:** cap total fields near 40 (mandatory + optional); mark a "Minimum
Viable Section" of core obligation fields to complete first; field prompts are one sentence, not
paragraphs; run a field-elimination pass before publication. **Violation symptom:** a 60+-field
template where the designer fills 30 with substance and 30 with "N/A / standard / TBD."

### L2 — Interface Ground Truth
Agents have no access to reality beyond their interfaces; every description, schema, error message,
and payload is treated as authoritative. Implicit context is invisible; missing information is
unknown. **Design consequence:** fields specifying agent-facing behavior carry *examples of a
correct answer*, not just labels. An "Error Response Format" field guides toward error code +
human-readable description + fix instruction + retry guidance, and prevents conflating output states
(empty vs. error vs. partial success). A template filled with plausible guesses is a template that
lies. **Violation symptom:** an error-handling field reading "Describe how errors are returned,"
yielding a design that says "errors return an error code."

### L3 — Boundary Entropy *(Very High confidence; 20/20)*
Information fidelity necessarily decreases at every boundary crossing — agent↔agent, agent↔tool,
context↔context, representation↔representation. Errors, contamination, and semantic drift compound at
each, and the loss is invisible to local observers (55.6% of contamination is invisible to
outcome-only views). **Design consequence:** every multi-component template carries a mandatory
**Boundary Map** with structured per-boundary fields — (a) boundary ID, (b) what crosses
(type/format), (c) validation mechanism (format/schema/semantic check), (d) circuit-breaker
threshold (failure count / time window), (e) graceful degradation path (full → reduced → LLM-only →
static). Exhaustively enumerated, never prose. **Violation symptom:** the only boundary field is a
text box, "Describe boundary validation strategy."

### L4 — Compounding Friction *(Very High confidence; 19/20)*
Inefficiencies compound non-linearly with scale; adding components past an optimal threshold degrades
outcomes, because each adds boundary crossings (L3), consumes attention (L1), and creates a new
interface to be trusted (L2). **Design consequence:** every architecture template carries
**Scale-Projection** fields — current count, projected at 2× and 5×, known inflection points (~5–10
tools, ~3–8 agents), and scale-test results *(evidence-required)*. Reject "it will scale"; require
"test result at 2× shows task success within 5% of baseline." **Violation symptom:** scale mentioned
only in a prose heading; the design says "designed to scale" with no counts and no test results.

### L5 — Ergonomic Dominance
The quality of the surrounding system — tool design, coordination, information structure, economics —
determines real-world outcomes more than the underlying model. The interface layer dominates the
intelligence layer. **Design consequence:** this law *justifies the whole enterprise.* Treat template
design as architecture-by-forcing-function, not formatting. Justify every field by the failure mode
it prevents, not by documentation completeness. **Violation symptom:** "here's a doc template with a
section per dimension," with no account of why each dimension matters or what it prevents.

---

## Part 2 — The Invariants (cross-cutting field requirements)

### II-3 — Reversibility Preserved
High-uncertainty decisions must preserve a reversal path; decisions made on weak evidence must not
become permanent commitments. **Adds fields, inline with each architectural decision** (not an
appendix — DR4/DR6): evidence tier (T1–T4), evidence source citation, reversal-cost estimate
(low/medium/high/prohibitive, with justification), optionality-preserving alternatives evaluated,
trigger conditions for reconsideration, next re-evaluation date. **Inspection method →
field-completion criterion:** "can you name the conditions under which this decision would be
reversed, and the cost of doing so?" If not, the field is unfilled.

### III-1 — Context Preservation (Progressive Disclosure)
Abstractions must preserve access to local context; a layer that hides detail must provide a path
back to it. **Adds fields to every interface section:** default output format (target 3–4 fields),
verbose-flag specification, escape-hatch mechanism, what is hidden at each level and how to reach it,
structured error response. **Inspection method:** "can an agent get to the detail it needs without
drowning in detail it doesn't?"

### IV-2 — Scale Projection
A design must demonstrate it works not only at N components but at projected scale. **Adds the
scale-projection field cluster** (current/2×/5× counts, inflection points, scale-test results). The
scale-test fields are **evidence-required**, not design-time. **Inspection method:** "is there a test
result at projected scale, or is this a guess?"

---

## Part 3 — The Contracts (obligation lists → mandatory / evidence-required fields)

> Each Required Check (RC#) is a candidate field. Criticality drives strictness (DR7): *critical*
> obligations are mandatory with no skip path; obligations demanding empirical data are
> evidence-required (DR8). Forbidden Shortcuts (FS#) become fields that force the design to show the
> shortcut was avoided.

### C3 — Coordination Architecture *(encodes L3, L4; Domain: coordination; Criticality: critical)*
A multi-agent design must make all seven configurable coordination dimensions explicit — ad-hoc
coordination produces integrative compromise (8–38%). The seven dimensions, each a mandatory field
sub-section with completion guidance:

- **C3.a Agent Endpoints** — every agent named, its role, its expertise level. (Reflect uneven
  expertise here when present — expertise-weighting fields.)
- **C3.b Communication Topology** — orchestrator-worker / mesh / linear pipeline / closed-loop /
  independent ensemble; named, not emergent.
- **C3.c Authority Distribution** — who decides what; how authority is weighted; **structural, not
  prompted** enforcement of expertise.
- **C3.d Synchronization Regime** — sync/async; ordering; how agents wait or proceed. *(Commonly the
  omitted dimension — F3.)*
- **C3.e Aggregation Rules** — how multiple agents' outputs combine; conflict resolution.
- **C3.f Termination Conditions** — when the system stops; success and give-up criteria. Without this
  agents run indefinitely or quit prematurely.
- **C3.g Failure Handling** — what happens when an agent fails; ret/ fallback / escalation.
- **C3 (failure signature)** — a mandatory field characterizing the *known failure signature* of the
  chosen topology (e.g., a linear pipeline propagates one bad result downstream).

**Forbidden Shortcuts:** FS1 free-form coordination prose in place of the seven dimensions; FS2
expertise enforced by prompt rather than structure.

### C8 — Boundary Validation *(encodes L3; Domain: coordination/tools; Criticality: critical)*
Every boundary where information crosses components must be enumerated and validated. Maps directly to
the Boundary Map section.

- **C8.a** Every boundary identified and named (agent↔agent, agent↔tool, session↔session).
- **C8.b** What crosses each boundary specified (information type, format).
- **C8.c** Validation mechanism per boundary (format / schema / semantic check).
- **C8.d** Circuit-breaker threshold per boundary (failure count / time window). *(evidence-required:
  threshold value + test result)*
- **C8.e** Graceful degradation path per boundary (full → reduced → LLM-only → static). *(evidence-required:
  test date + outcome — degradation path must have been exercised and the result recorded)*
- **C8.f** Contamination monitoring configuration (what is monitored, how). *(evidence-required)*

**FS1** treating boundaries as safe by default; **FS2** a single prose paragraph asserting "boundaries
are managed."

### C6 — Progressive Disclosure with Escape Hatches *(encodes III-1, L1; Domain: tools; Criticality: high)*
Interfaces must default to the minimum useful information and provide a path to detail.

- **C6.a** Default output format specified (target 3–4 fields).
- **C6.b** Verbose flag / full-detail mode specified.
- **C6.c** Escape-hatch mechanism (how to reach hidden context).
- **C6.d** Error response structure: error code + human-readable description + fix instruction + retry
  guidance (L2).
- **C6.e** What is hidden at each disclosure level and how to access it.

### C2 — Interface Change Auditability *(encodes L2; Domain: tools; Criticality: critical)*
Agent-facing interface specs must be designed for safe change. *(Template owns the WHAT; the HOW of
running CUJ tests / configuring versioning → Implementation Playbooks / Evaluation Harnesses.)*

- **C2.a** Tool description format (what the description must contain).
- **C2.b** Default vs. verbose output specification.
- **C2.c** Error format (per C6.d).
- **C2.d** CUJ test *plan* specification — which critical workflows, which pre-change and post-change
  baselines to capture (WHAT to test; not HOW to run it).
- **C2.e** Versioning strategy specification — Pin / Scope / Test approach chosen (WHICH approach; not
  HOW to configure it).
- **C2.f** Schema documentation requirements.

### C5 — Evaluation Validity *(encodes II-2; Domain: evaluation; Criticality: high)*
An evaluation-system design must specify what makes its measurements valid. *(Template owns the design
fields; metric methodology and calibration procedure → Evaluation Harnesses.)*

- **C5.a** Metric-to-purpose alignment — each metric traced to a system purpose (anti-Goodhart).
- **C5.b** Gold-set specification — size, recency, calibration plan.
- **C5.c** Judge model specification — different-family constraint stated.
- **C5.d** Calibration cadence.
- **C5.e** Eval-set rotation plan (anti-contamination, anti-drift).

---

## Part 4 — Pattern Seeds (advisory field clusters)

> Advisory by default (DR9). Where a seed overlaps a contract obligation, the contract tier wins and
> the field is mandatory (DR10). Each seed's *when-NOT-to-use* is as important as its *when* — it's
> how you avoid bloat (F1) on simple systems.

### PS-3 — Boundary Map + Validation Gate
**Problem:** info crosses boundaries with no explicit validation; degradation is invisible (55.6%) and
compounds (L4). **Fields it adds:** the per-boundary validation cluster (overlaps C8 → mandatory there).
**Use** for any template with 2+ agents, agent↔tool boundaries, or context isolation. **Don't use** for
single-agent / single-tool systems with no crossing, or where boundary validation is owned elsewhere.
**Downstream:** circuit-breaker / monitoring *implementation* → Implementation Playbooks.

### PS-6 — Coordination Dimension Checklist
**Problem:** multi-agent coordination designed ad hoc → integrative compromise (8–38%). **Fields:** the
seven C3 dimensions as completeness fields (overlaps C3 → mandatory). **Use** for multi-agent deployment
/ scaling / migration from ad-hoc coordination. **Don't use** for single-agent systems, throwaway
prototypes where coordination failure is acceptable, or frozen production coordination. **Downstream:**
verifying each dimension is *adequately addressed* → Review Checklists; configuring a pattern →
Implementation Playbooks.

### PS-12 — Progressive Disclosure Audit *(6+ independent sources)*
**Problem:** interfaces default to too much (bloat) or too little (starvation), with no escape hatch.
**Fields:** the C6 progressive-disclosure cluster (overlaps C6 → mandatory). **Use** for tool-interface /
API / agent-facing output design. **Don't use** for human-operator interfaces, trivially simple
single-field responses, or unlimited-context-budget contexts.

### PS-15 — Reversibility Option Design *(well-motivated, methodologically aspirational)*
**Problem:** architecture decisions made as irreversible commitments when escape-hatched alternatives
exist; reversal cost grows non-linearly with scale (L4). **Fields:** the II-3 reversibility cluster
(evidence tier, reversal cost, alternatives, trigger conditions). **Use** for architecture / protocol /
coordination-pattern / evaluation-methodology decisions, and any decision on Medium-High-or-below
evidence. **Don't use** for Very-High-confidence decisions, negligible-reversal-cost changes (renaming a
tool), or when speed-to-market is the explicit dominant constraint. **Downstream:** the per-decision ADR
format → Decision Records; reversibility *implementation* (flags, dual-running, abstraction layers) →
Implementation Playbooks.
