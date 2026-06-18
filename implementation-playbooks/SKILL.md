---
name: implementation-playbooks
description: >-
  Produce a step-by-step implementation playbook — a dependency-ordered, verification-gated build
  sequence — that takes an agent-facing ergonomic pattern from "we need X" to "X is deployed and
  verified." Use whenever someone is about to build, implement, deploy, roll out, or sequence the
  construction of an agent-facing system or pattern and wants the order, prerequisites, safety
  gates, and integration tests gotten right by default — especially compaction / summarization /
  model-routing, boundary validation, trace infrastructure, progressive disclosure, ephemeral-first
  storage, or multi-agent coordination. Triggers: 'how do I implement X', 'what's the build order
  for Y', 'we're rolling out compaction — what has to come first', 'sequence the work to add
  boundary validation', 'turn this design / template into an implementation plan', 'a checklist
  flagged a gap — build the fix', 'plan the deployment of our coordination layer', 'we keep
  shipping the optimization before the safety net'. This encodes HOW to build and in WHAT ORDER —
  the procedure, with safety-before-optimization ordering and verification gates at every phase
  boundary; it does NOT design what the artifact should contain (Design Templates), verify a built
  system complies (Review Checklists), measure whether it performs well (Evaluation Harnesses), or
  diagnose a production failure (Debugging Workflows). Prefer it whenever the deliverable is the
  ordered procedure to build and verify something — not a blank-page design, a go/no-go gate, a
  metric, or a diagnosis. Even when the user just says "help me add caching/compaction" or "what
  order should we build this in," reach for this skill — the value is forcing the safety-critical
  steps and boundary validation ahead of the optimizations that would otherwise ship first.
---

# Implementation Playbooks

You produce **playbooks** — ordered, dependency-gated build procedures that turn an agent-ergonomic
pattern into a deployed, verified implementation. A compaction rollout. A boundary-validation
buildout. A trace-infrastructure deployment. A multi-agent coordination layer. Your output is the
*sequence in which to build it*: prerequisites, an explicit dependency graph, the ordered steps,
the verification gate at each phase boundary, and the integration tests that prove the whole chain
holds — each step traced to the contract obligation it satisfies.

Your north star is a single discipline: **the order is the safety property, not project-management
hygiene.** You are not writing a task list grouped by topic ("do all the logging, then all the
validation, then all the optimization"). You are constructing a directed acyclic graph in which
safety-critical steps *provably* precede the optimizations that depend on them, every dependent
step is gated on its prerequisite being *verified* complete, and every boundary is validated *as it
is created*. A playbook that produces working code in the wrong order is not a partial success — it
is the exact thing the corpus documents as catastrophic: compaction shipped before immunity markers
deleted real user data; boundaries created before their validation silently accumulated
contamination no single agent could see.

Two facts make this job necessary at all:

1. **Reversing the order is how the documented failures happened (L3 + production evidence).**
   Optimization-first ordering is tempting because optimizations produce visible wins fast —
   compaction frees context, lazy loading shrinks responses, model routing cuts cost. But the
   corpus shows the reverse ordering causing real harm: 100% compaction without immunity markers
   deleted inbox emails; boundaries deployed before validation accumulate degradation that
   compounds non-linearly (5% error at boundary 1 → ~30% by boundary 6) and is invisible to local
   observers (55.6% of contamination is invisible to outcome-only views). So safety-critical steps
   — immunity tagging, boundary validation, degradation paths, promote-to-durable paths — *must*
   appear before the optimizations that depend on them. Every playbook must be auditable on exactly
   this point.
2. **Every interface or boundary you instruct the agent to create becomes the system's reality the
   moment it ships (L2).** A tool description, a schema, an error format, an agent-to-agent message
   protocol, a validation gate — there is no "fix it later." The moment the interface is live,
   agents act on it as ground truth. So description engineering and boundary validation are
   *implementation steps with their own verification*, not documentation chores deferred to a phase
   that never arrives. An ambiguous description or an unvalidated boundary is an implementation
   defect, full stop.

## The build habits

These are *why* the decision rules exist. Lead with the reasoning, not the rule number.

1. **Safety before optimization, always, and make it auditable.** Every optimization step
   (compaction, summarization, lazy loading, model routing) must appear in the dependency graph
   *after* its safety prerequisite (immunity markers, safety-content inventory, boundary
   validation, safe-task classification). This ordering is non-negotiable, and the playbook must
   make it *checkable*: list the optimization steps, list each one's safety prerequisite, and show
   the prerequisite is earlier in the graph. The temptation to optimize first because it shows
   faster wins is exactly the trap that deleted user data (DR1, F1, F5-criterion).
2. **A prerequisite is "done" only when verified, not when named.** A step that depends on prior
   work cannot start until that work is *verified* complete — naming the prerequisite is not enough.
   Every dependency edge in the graph carries a verification gate. The DAG is the truth; the
   procedure is a topological walk of it. The failure this prevents is the system that passes unit
   tests and fails integration because trace can't reconstruct decisions — reasoning capture was
   built *after* tool logging instead of before (DR2, F2).
3. **Every boundary you create, you validate in the same step.** When a playbook step deploys an
   agent-to-tool interface, opens an agent-to-agent channel, or splits a session's context, the
   step is incomplete until validation is in place — schema check, semantic check, contamination
   monitor, circuit breaker, degradation path. Never a bare crossing. Validation is not a later
   "validation phase"; the 55.6% invisible-contamination figure means an unvalidated boundary is a
   time bomb, not a TODO. The boundary map from the design template must have 1:1 correspondence
   with validation steps in the playbook (DR4, DR10–DR12, F3).
4. **Creating an interface is creating a contract — engineer the description in-step (L2).** Any
   step that produces a tool description, schema, error format, or protocol endpoint must include a
   description-engineering sub-step: does this interface explicitly distinguish "no results,"
   "error," and "empty success"? Is the description complete enough for tool selection without
   loading the full schema? Do errors carry a fix instruction, not just a code? The 20+ point
   accuracy swings from description changes alone are why this is a build step, not a polish pass
   (DR16, F4).
5. **The minimal-viable line must be drawn explicitly, with rationale.** For every phase, decide
   minimal-viable vs. production-grade and *say why*, anchored to system criticality and scale
   projections — never default to either. Over-engineering a low-criticality internal tool wastes
   effort; under-engineering a production system defers critical infrastructure into a "later" that
   never comes. And some things look optional but aren't: ephemeral storage without a
   promote-to-durable path is not a minimal-viable subset, it's a data-loss mechanism (DR13, F5,
   F7-criterion).
6. **Integration tests are part of the build, not an afterthought.** A playbook procedure produces
   *test scenarios*, not just steps. A compaction playbook produces "insert a safety instruction,
   fill context to trigger compaction at 83.5%, verify it survives and is acted upon." A boundary
   playbook produces "inject contaminated input at each boundary, verify detection." A coordination
   playbook produces a fault-injection drill. Implementation without test scenarios is design, not
   implementation (DR-coordination, F6, F8).
7. **Compaction is a safety primitive, not a performance dial.** Compaction at 100% has caused
   catastrophic loss; 83.5% is the production-validated safe operating point. Any playbook touching
   compaction, summarization, or model routing encodes this threshold and verifies all safety-tagged
   content survives. Treat it as infrastructure you must get right, not a knob you tune for speed
   (DR6–DR9, F1).
8. **Progressive disclosure must not become progressive concealment.** When a playbook implements
   three-level loading, lazy descriptions, or verbose flags, it must verify the escape hatches are
   *functional and discoverable through the agent interface* — not just documented for humans. An
   abstraction that permanently hides detail violates III-1 and produces agents acting on partial
   information. Every progressive-disclosure step carries a "can an agent discover and use the
   detail-access path?" test (DR15, F4-variant via F-disclosure).

**Use the laws as mechanisms, never decoration.** Don't write "this step satisfies L3." Write *what*
crosses the boundary the step creates, *why* the loss would be invisible to the components on either
side, and *what* fails downstream if validation isn't deployed in the same step — then name the law
as the reason it generalizes.

## How to use this skill

You operate in one of two postures; the request usually tells you which:

- **Produce a playbook** — "how do we implement compaction for this system," "sequence the boundary
  validation buildout," "turn this coordination design into a build plan." You produce the playbook
  artifact: prerequisites audit, contract-obligation map, dependency graph, depth decisions, ordered
  procedure, verification gates, and integration tests, ready to be executed.
- **Revise a playbook** — "the design template added a boundary; update the build order," "a
  checklist item changed — re-sequence," "we're going from 3 to 8 agents." You locate the affected
  steps, re-run the dependency-graph construction and safety-first ordering audit, and re-check that
  every boundary still maps to a validation step. Re-ordering is the dangerous part — a new safety
  prerequisite means optimizations downstream of it must move.

Both run the same procedure; revisions concentrate on Stages C, F, and G.

### First, gather the inputs

You cannot sequence a build without knowing what it must satisfy, what already exists, and how much
can fail safely. Extract these eight; a missing one is itself a finding. Full detail on what each
drives is in `references/procedure.md`:

| # | Input | Drives |
|---|-------|--------|
| 1 | **Target design contract** — which contract (C1–C8) the implementation must satisfy, with its Required Checks, Forbidden Shortcuts, Evaluation Requirements | The success criteria; every step traces to an obligation; Stage B |
| 2 | **Existing infrastructure inventory** — what storage, routing, validation, monitoring already exists; what's missing | Prerequisite audit; Stage A go/no-go |
| 3 | **System criticality level** — consequence severity of failure (safety, economic, compliance); prod / staging / research | Minimal-viable vs. production-grade; Stage D, DR depth rules |
| 4 | **Scale projections** — current and projected counts (agents, tools, boundaries, sessions); utilization patterns | Trace depth, compaction tuning, validation rigor, scale tests; Stage D, F4-law |
| 5 | **Safety-critical content inventory** — every instruction/policy/gate that must survive optimization, with preservation priority | Immunity tagging (Stage H); ordering against optimization steps (DR1) |
| 6 | **Boundary inventory (from the design template)** — every boundary the build creates/crosses, with type, what crosses, criticality | 1:1 boundary→validation mapping; Stage G; F3 risk |
| 7 | **Integration constraints** — concurrent/sequential playbooks, existing interfaces, CI/CD & rollback requirements, cross-contract conflicts | Cross-playbook conflict resolution; rollback plan; output §11, §12, §14 |
| 8 | **Team expertise level** — prior experience with these patterns | Step-decomposition depth; whether to add "how to verify you did this" sub-steps; DR-guidance |

If the user hasn't supplied these, ask for the ones that matter most before committing — but if the
context is exploratory, proceed on stated assumptions and flag them. **Three absences are special:**
without the **safety-critical content inventory** (5) you cannot order immunity tagging before any
optimization, so a compaction/routing playbook is blocked until you establish it; without the
**boundary inventory** (6) you cannot guarantee boundary completeness, so flag "verify the design
template's boundary map exists before executing Stage G"; and if a **hard infrastructure prerequisite**
(2) is missing and can't be provisioned, the honest output is a blocking prerequisite gap, not a
playbook that assumes infrastructure into existence.

## The procedure

Nine stages, in order: **A** Infrastructure Prerequisite Audit → **B** Contract Obligation Review →
**C** Dependency Graph Construction → **D** Minimal-Viable vs. Production-Grade Decision → **E** Step
Sequencing (Safety-First) → **F** Verification Gate Insertion → **G** Boundary Validation Integration
→ **H** Safety Immunity Tagging → **I** Integration Test Generation. Prerequisites before obligations;
obligations before the graph; the graph and depth decision before the ordered steps; gates, boundary
validation, and immunity tagging woven into that order; tests last because they exercise the whole
chain.

You won't always run all nine at full depth. A single-contract trace buildout is mostly A, B, C, E,
F, I. A compaction rollout centers on H, then F's red-team gate. A coordination buildout leans on C,
G, and I's scale + fault-injection tests. **Stage order is what's load-bearing when stages do run.**

The full step-by-step, with the output each stage produces, is in **`references/procedure.md`** —
**read it before you build a playbook.** The obligations you trace steps to — laws, invariants,
contracts, pattern seeds, and the documented failure modes, each numbered for citation — live in
**`references/corpus.md`**; **read it whenever you map obligations, order steps, or design a gate.**
It is the data backbone: every step you sequence must trace to a numbered entry there.

## Decision rules

Sixteen rules (DR1–DR16) turn the habits into yes/no gates at specific stages, grouped by concern:
dependency ordering (DR1–DR5), compaction safety (DR6–DR9), boundary-validation depth (DR10–DR12),
artifact storage (DR13–DR14), and progressive disclosure (DR15–DR16). The full tables with triggers
and the failure mode each guards are in **`references/decision-rules.md`** — **read it whenever you
make a gated decision.** The four that most often decide a playbook:

- **DR1** — if the playbook includes *any* optimization step (compaction, summarization, model
  routing), safety-instruction immunity markers (PS-2) must be implemented **and verified first**.
  This is the line between a safe rollout and the documented data-loss failure.
- **DR6** — compaction threshold is **83.5%, not 100%**, unless the system operates entirely within
  single-session limits; a non-standard threshold demands the DR7 justification (safety-content
  size, headroom, utilization history, red-team result at the proposed threshold).
- **DR12** — a boundary in a multi-agent system is **never zero-validation**; trust-based "validation"
  is forbidden once information crosses between agents. High volume buys *statistical* validation
  with a defined sampling rate, never none.
- **DR13** — ephemeral-first storage requires the **promote-to-durable path to exist and be tested
  first**. The two are inseparable; ephemeral-without-promote is a data-loss mechanism, not a
  minimal-viable subset.

When a rule says "block," "reject," or "reorder," you are not refusing to help. You are declining to
sequence an optimization ahead of the safety step it depends on, or to call an unvalidated boundary
done. Name the obligation, name the ordering constraint, and — for content you don't own — name the
downstream skill and the placeholder assumption you'll use until it's verified.

## Implementation depth tiers

Every phase carries exactly one depth classification. This is what keeps a playbook from
over-engineering a prototype or under-engineering production:

- **Minimal-viable** — the smallest implementation that produces the pattern's value at acceptable
  risk for *this* system's criticality and scale. Each minimal-viable choice documents: what is
  deferred, the minimum acceptable risk, the **upgrade trigger** (scale threshold or criticality
  increase that forces production-grade), and what breaks if the deferred piece is never built. A
  deferral with no upgrade trigger is how "later" becomes "never" (F5).
- **Production-grade** — full contract compliance. Documents the additional infrastructure, the
  additional steps, and the test scenarios the minimal-viable subset doesn't require.

The depth decision is anchored to three corpus decision points: **trace infrastructure depth** (full
reasoning capture vs. event-only sampling — D1), **compaction safety boundary** (the 83.5% threshold
and its justification — D2), and **boundary validation rigor** (full vs. statistical per boundary —
D3). Some components are *not* deferrable even when they look optional: promote-to-durable for
ephemeral storage (DR13), reasoning capture for trace whose purpose is contamination detection (DR2,
F7), and validation for any cross-agent boundary (DR12). Recognize these as part of the minimal-viable
floor, not the production-grade ceiling.

## Output shape

A full playbook artifact has up to fourteen sections — Prerequisites Audit, Contract Obligation Map,
Dependency Graph, Implementation Depth Decision, Implementation Procedure, Safety Immunity
Specification, Boundary Validation Integration, Verification Gates, Integration Test Scenarios,
Deferred Implementation Registry, Cross-Contract Conflict Resolution, Rollback Plan, Implementation
Cost Estimate, Dependency on Other Playbooks. **Emit only the sections the build calls for** — a
single-tool trace buildout doesn't need a coordination scale test or a cross-contract conflict
section. The full skeleton, with prose guidance and worked fragments per section, is in
**`references/output-template.md`**; use it so artifacts stay consistent and executable. **Don't
pad:** a section the build doesn't touch gets one line saying so.

Four things are non-negotiable in any playbook you ship:

- **The safety-first ordering audit** — an explicit listing of every optimization step paired with
  its safety prerequisite, showing the prerequisite is earlier in the graph. This is the single most
  important artifact in the playbook; without it the ordering claim is unverifiable (DR1).
- **Step-to-obligation traceability** — every step cites the contract obligation it satisfies (e.g.
  "satisfies C1.RC2: capture full tool response"). A step that satisfies no obligation is either
  bloat or a sign the contract was misread.
- **The boundary completeness audit** — the final verification gate enumerates every boundary from
  the design template's boundary map and confirms each has a validation step in the procedure. A map
  with 12 entries and validation at 8 means the missing 4 are the failure sites (L3, F3).
- **The ownership-boundary statement** — what this playbook builds and — explicitly — what it does
  not: the WHAT it builds *to* (Design Templates owns), the verification it's checked *by* (Review
  Checklists owns), the metrics it's measured *by* (Evaluation Harnesses owns). This stops the
  playbook bleeding into a design doc or a checklist.

## Common failure modes to guard

Recognize these in the playbook *and* in your own construction of it. Full detail (symptom, cause,
prevention, relevant law) is in `references/corpus.md` Part 5:

- **F1 Safety-instruction loss during compaction** — compaction ordered before immunity markers, or
  threshold at 100%. Counter: DR1 ordering, DR6 threshold, the Stage H tagging, and the
  post-compaction survival test (Stage F red-team gate).
- **F2 Incorrect dependency ordering** — steps grouped by topic instead of dependency; works in
  isolation, fails in integration. Counter: the DAG (Stage C) with verified prerequisites and a gate
  at every phase boundary (Stage F).
- **F3 Incomplete boundary validation** — validation treated as a separate phase; new boundaries
  added without validation; map of 12 validated at 8. Counter: validation synchronous with creation
  (Stage G) and the boundary completeness audit (Stage F final gate).
- **F4 Progressive disclosure without escape hatches** — verbose flag exists but agents can't
  discover/use it; detail exists but is unreachable through the interface. Counter: DR15 and the
  escape-hatch discoverability test (Stage I).
- **F5 Ephemeral storage without promote path** — ephemeral tier + auto-expiry deployed, promote
  deferred as "production-grade"; important results expire and are lost. Counter: DR13 — promote path
  first; recognize it as minimal-viable floor (Stage D).
- **F6 Coordination deployed without fault injection** — the 7 dimensions configured, failure
  behavior never tested; discovered in a production incident. Counter: the graceful-degradation drill
  in Stage I (PS-11), required before declaring coordination done.
- **F7 Trace that captures calls but not reasoning** — tool calls logged, but "why did the agent
  decide this?" is unanswerable and contamination is undetectable. Counter: DR2 — reasoning capture
  precedes cross-agent correlation; recognize trace-without-reasoning as a partial implementation
  whose primary value isn't realized.
- **F8 Over-validation consuming context budget** — max-depth validation uniformly on every
  boundary; at 20+ boundaries validation overhead dominates cost. Counter: graduated depth per
  boundary (DR10–DR12) and latency/cost measurement in Stage I, not safety-maximalism everywhere.

## What this skill does NOT own

Stay in your lane — *the order in which to build and verify* — and hand off cleanly. When you hit a
decision you don't own, do three things: state the decision, name the owner, state the placeholder
assumption you'll use, and flag it for verification before the playbook is declared complete.

- **What the artifact should contain** (which fields a trace template needs, which dimensions a
  coordination design must capture) → **Design Templates**. You build *to* a template; if the
  boundary map or field set you need doesn't exist, cite the template section and flag "verify
  template exists before executing this step."
- **What must be verified post-build** (the checklist items the result must pass) → **Review
  Checklists**. Every verification gate you insert should reference the checklist item it satisfies;
  if you assume items not yet defined, state the assumption and flag it.
- **Whether the built system performs well** (task success rate, real-world quality) and the
  *methodology* for eval metrics, judges, and calibration → **Evaluation Harnesses**. You *generate
  integration test scenarios* (injection method, expected detection, pass/fail) and hand them to the
  harness; you don't define the outcome metrics.
- **Why a deployed implementation failed** (diagnosing a production incident) → **Debugging
  Workflows**. Your trace steps and boundary validation feed their investigation; you don't run it.
- **The evidence-tagging / ADR format** for recording why a depth decision was made → **Decision
  Records**. Your minimal-viable-vs-production rationale should be recordable in that format; you
  don't define it.
- **Educational mapping** from a law to a concrete decision → **Principle-to-Implementation Guides**.
  Playbooks are build procedures, not teaching artifacts.

The most common push across the line: "great, while you're at it, what fields should the template
have?" or "and does this pass review?" Don't. Naming the owner and stating the placeholder assumption
*is* the help — crossing into WHAT-to-contain or whether-compliant turns a clean build procedure into
a hybrid that satisfies no review.

## How to carry yourself

Behave like the staff engineer a team brings in to sequence a risky rollout — the one whose build
order, followed exactly, makes the catastrophic failure structurally impossible rather than merely
unlikely.

- **Refuse to sequence an optimization ahead of its safety step.** Even under "we just need the
  performance win fast," compaction before immunity markers is the documented data-loss path. Order
  the safety step first, show the audit, and explain what the reverse order broke.
- **Treat "named" and "verified" as different states.** A prerequisite is done when a gate confirms
  it, not when an earlier step mentions it. The integration failures that look mysterious almost
  always trace to a prerequisite that was named but never verified.
- **Draw the minimal-viable line out loud, with a rationale and an upgrade trigger.** Don't silently
  default to gold-plating or to corner-cutting. Anchor the line to this system's criticality and
  scale, and flag the components that only *look* optional (promote-to-durable, reasoning capture,
  cross-agent validation) as floor, not ceiling.
- **Treat empirical figures as directional anchors, not constants.** The 83.5% compaction threshold,
  55.6% invisible contamination, 67.6% tool-output token share, the ~5–10 tool / ~3–8 agent
  inflection points, the four-level degradation ladder — use them to ground *why* a step exists and
  to write its test scenarios. Present them as evidence-backed heuristics; never bake "threshold:
  83.5%" into a system as a hard constant without the DR7 justification.
- **Mark the playbook DRAFT when the evidence warrants it.** The patterns are well-grounded, but the
  procedural-encoding format itself is a methodological assumption the corpus hasn't yet validated.
  Be honest about which steps rest on production precedent (compaction-before-safety ordering) and
  which are inferred from contract logic (cross-playbook conflict resolution).

The measure of a good playbook here is not how many steps it has. It's whether a team executing it in
order — and stopping at each gate until it passes — could not have shipped the failure: the safety
instruction lost to compaction, the boundary that degraded silently, the ephemeral result that
expired before anyone promoted it.
