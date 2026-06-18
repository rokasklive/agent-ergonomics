# The Five Domains of Agent Ergonomics

The domains are the *coverage map* for the review — they make sure you look at the
whole system, not just the part that's easy to see. Each domain solves a distinct
problem, and each maps onto one row of the Ergonomics Scorecard. The laws explain
*why* something is wrong; the domains make sure you *checked everywhere*.

A note on the relationship between domains and laws: the laws cut across all five
domains. A single finding usually lives in one domain (where the artifact is) but
is explained by one or more laws (why it bites). Score by domain; explain by law.

## Table of contents

- [D1 — Tool & Interface Design](#d1--tool--interface-design)
- [D2 — Information Architecture](#d2--information-architecture)
- [D3 — Workflow & Coordination](#d3--workflow--coordination)
- [D4 — Economic Design & Resource Management](#d4--economic-design--resource-management)
- [D5 — Evaluation & Measurement](#d5--evaluation--measurement)
- [Scoring guidance](#scoring-guidance)

---

## D1 — Tool & Interface Design

**What it governs.** The *active* interface — what the agent calls and what comes
back. Tools, CLIs, APIs, protocols, structured outputs: how they're described, how
they return results, how they signal errors, how they relate to each other.

**Why it's a domain.** Agents consume tools as sequences of structured text, not as
visual surfaces. Description quality, output shape, and error signaling directly
set success rates, token cost, and failure modes — independent of model quality.
"Agents don't see your tools; they see their descriptions."

**Diagnostic questions.**
- Are interfaces **explicit** — defaults, units, side effects, ordering, auth scope
  all stated rather than assumed? (L2)
- Are descriptions **authoritative and current** — do they match actual behavior,
  and are they versioned so they can't drift silently? (L2)
- Are **errors actionable** — do they say what failed, why, and what to do next, or
  just that something went wrong? (L2)
- Is **discoverability** strong — can the agent find the right tool/flag without
  trial and error, with unambiguous hierarchical names? (L2)
- Is **state visible** — distinct, labeled responses for *every* state the system
  can emit (success, empty / zero-result, error, partial / truncated, warning,
  unauthorized), rather than one ambiguous shape? Enumerate them and confirm the
  agent can distinguish each; don't assume the obvious states cover it. (L2)
- Is the **response minimal by default** — high-signal fields, verbose behind a
  flag — or does every call dump everything? (L1)
- Is the **tool surface consolidated** — flexible tools over one-per-endpoint
  sprawl that confuses selection past ~5–10 tools? (L4)

**Primary laws:** L2 (the dominant one here), L1, L4.

---

## D2 — Information Architecture

**What it governs.** The *passive* knowledge and state available at decision time —
what the agent knows, remembers, and can reference. Memory systems, context-window
management, skills, documentation, discovery manifests, structured state files.

**Why it's a domain.** The context window is a finite, shared resource. Without
deliberate architecture for what information exists and how it's reached, agents
drown in noise or starve for signal. "Context is the scarce resource."

**Diagnostic questions.**
- Is information **easy to locate** when needed — or does the agent have to hold it
  all in context just in case? (L1)
- Is state **externalized** into durable, inspectable structures (state files,
  progress logs) rather than living implicitly in conversation history that drifts?
  (L1, L3)
- Is **progressive disclosure** used — names/summaries first, detail on demand,
  layered loading — so the agent pays only for what it uses? (L1)
- Is **context locality** preserved — is the information an agent needs for a step
  near at hand, or scattered so it must reassemble it across turns? (L1)
- Is **knowledge fragmented** across sources that must be cross-referenced (each
  reference a boundary crossing), or coherent at the point of use? (L3)
- Is presentation **decoupled from payload** where two audiences (human + agent)
  share a surface, so each gets what it needs? (L2)
- Does **memory drift** — is stale state allowed to be trusted as fresh? (L3)

**Primary laws:** L1 (dominant), L3, L2.

---

## D3 — Workflow & Coordination

**What it governs.** How a single agent structures work across turns, and how
multiple agents coordinate, delegate, share context, and recover from failure.
Orchestration patterns, retries, circuit breakers, degradation, context isolation,
handoffs.

**Why it's a domain.** Multi-agent systems fail at high rates and most failures are
coordination defects, not model limits. Adding agents can make a system *worse*.
"Coordination is a first-class architectural concern."

**Diagnostic questions.**
- How many **handoffs** occur, and is each one necessary? Every handoff sheds
  fidelity and spends budget. (L3, L1)
- Where does information **cross boundaries**, and is it **validated** at the
  crossing — or trusted blindly? (L3, L2)
- Is the topology **closed-loop** (outputs re-grounded against intent) or an open
  **pipeline** (catastrophically vulnerable to propagated faults)? (L3)
- Is **responsibility clear** — is authority structurally enforced (hard routing,
  gates), or does the system average expert and non-expert views? (L4)
- Are **feedback loops** present — does the system detect and correct drift, or run
  open-loop until it fails? (L3)
- Are **retries controlled** — exponential backoff *and* circuit breaker *and*
  budget ceiling — or a bare retry loop that can storm? (L4)
- Is **context isolated** between agents/subagents to contain contamination rather
  than let it propagate? (L3)

**Primary laws:** L3 (dominant — this is its home domain), L4, L1, L2.

---

## D4 — Economic Design & Resource Management

**What it governs.** The cost models and resource-allocation strategies that shape
agent behavior — the token budget as a first-class design constraint. Where cost
concentrates, what scales poorly, how pricing incentivizes or punishes the
behavior you want.

**Why it's a domain.** Every design decision has an economic dimension that
compounds at scale. Tool output silently consumes the majority of tokens; pricing
models can penalize the very exploration you want from agents. "Cost is a design
constraint, not a backend metric."

**Diagnostic questions.**
- What **consumes tokens** — have you located the real cost driver (usually tool
  output, ~67.6%) rather than optimizing the cheap part (system prompts, ~3.4%)?
  (L1)
- What **drives cost** per *successful* task — including retries, idle time, and
  coordination overhead, not just the happy-path call? (L4)
- What **scales poorly** — where does marginal cost of the Nth component exceed the
  (N-1)th (multi-agent at ~10–15× single-agent)? (L4)
- Are **resources treated as constraints** — is there a budget ceiling, bounded
  rather than unbounded spend, model routing (cheap workers / capable
  orchestrator)? (L1, L5)
- Does **pricing fight the agent** — does usage-based metering penalize the
  explore/fork/discard behavior agents do best, where bounded pricing would free
  it? (L1)

**Primary laws:** L1, L4, L5. (Confidence in this domain's evidence base is the
weakest of the five — much rests on single sources. Score and recommend, but flag
where claims are not independently validated.)

---

## D5 — Evaluation & Measurement

**What it governs.** How anyone *knows* whether an interface, protocol, or workflow
actually works. Metrics, graders, traces, reliability measures, regression
detection. The domain that turns ergonomics from opinion into engineering.

**Why it's a domain.** Without evaluation, every ergonomics claim is an untestable
opinion — including the ones in your own report. Standard software metrics
(uptime, latency, error rate) are structurally blind to agent-specific failures.
"The evaluation suite is the deliverable."

**Diagnostic questions.**
- Is success **measurable** — is there a defined task/outcome and a grader, or just
  vibes? (L5)
- Are **traces observable** — can you see every tool call, response, and reasoning
  step (trajectory), or only the final outcome? Outcome-only metrics miss ~55.6%
  of contamination. (L3, L2)
- Are **failures diagnosable** — trace trees not flat logs, observability that
  spans agent boundaries? (L2, L3)
- Are **regressions detectable** — is there a maintained eval set, calibration
  cadence for any LLM-as-judge, reliability measured as pass^k (consistency) not
  just pass@k (luck)? (L5)
- Is evaluation **built before** the agent it grades, treated as the primary
  artifact? (L5)

**Primary laws:** L5, L3, L2. (Like D4, strong framework but lacks
ergonomics-specific standardized benchmarks — no shared composite score exists
yet.)

---

## Scoring guidance

Each domain gets a **score 1–10**, a **justification**, and a **confidence**
(High / Medium / Low) reflecting how much evidence you actually had — a system you
could probe deeply earns higher confidence than one you only read about.

Anchor the 1–10 scale so scores mean something across reviews:

- **9–10 — Exemplary.** Actively engineered for agents. Minimal, explicit,
  closed-loop, measured. Few findings, and those are refinements. You'd cite it as
  a model.
- **7–8 — Strong.** Sound fundamentals, a few real but non-blocking issues. Agents
  operate effectively; targeted fixes would sharpen it.
- **5–6 — Mixed.** Works, but friction is real and recurring. Some Medium/High
  findings rooted in foundation laws. Agents succeed but pay for it in tokens,
  retries, or ambiguity.
- **3–4 — Weak.** Structural problems that degrade agent effectiveness materially.
  Multiple High findings; at least one likely Critical.
- **1–2 — Hostile.** The architecture actively fights the agent. Critical findings
  dominate; agents fail or burn resources regardless of model quality.

Calibrate to *agent* operation, not human convenience: a CLI a human loves can
score low for agents if its output is unparseable, its errors are silent, or its
state is invisible. Reserve the extremes — most real systems land 4–8. Don't
inflate to be kind; the value of the review is in honest, well-explained scores.
A low score with a clear causal explanation and a fix is a *gift*, not an insult.

If you couldn't assess a domain (no visibility into, say, the eval setup), say so
and set confidence Low rather than guessing a number — an honest gap is more useful
than a fabricated score.
