# Decision Rules (DR1–DR16)

> Operational gates, not suggestions. Each turns a build habit into a yes/no check at a specific
> stage of the procedure, and each names the failure mode (or law) it prevents — **lead with that
> reasoning when you apply it, don't just cite the number.**
>
> The decision points behind them: **D1** trace-infrastructure depth, **D2** compaction safety
> boundary, **D3** boundary-validation rigor. The failure modes they guard: **F1** safety-instruction
> loss, **F2** incorrect ordering, **F3** incomplete boundary validation, **F4** disclosure without
> escape hatches, **F5** ephemeral-without-promote, **F6** coordination without fault injection, **F7**
> trace without reasoning, **F8** over-validation.

## Dependency ordering — safety before optimization, prerequisites before dependents

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR1** | Playbook includes ANY optimization step (compaction, summarization, model routing) → safety-instruction immunity markers (PS-2) must be implemented **and verified FIRST**, earlier in the graph. The reverse order is the documented data-loss path. | Any optimization step present. | F1, F5-criterion |
| **DR2** | Playbook includes cross-agent trace → **reasoning capture (C1.RC3) precedes cross-agent trace correlation (C1.RC4)**. Trace without reasoning can't reconstruct *why*, so contamination detection — its primary value — isn't realized. | Cross-agent trace steps present. | F7 |
| **DR3** | Playbook includes default-vs-verbose interface setup → **escape-hatch discoverability is verified before disclosure is declared complete**. An undiscoverable hatch is no hatch. | Progressive-disclosure / lazy-loading steps present. | F4 |
| **DR4** | Playbook includes any boundary-creation step → the **boundary map from the design template must be complete before any validation is implemented**. You can't validate a boundary set you haven't enumerated. | Any boundary creation. | F3 |
| **DR5** | Playbook includes multi-agent coordination → **failure handling (dimension 7) is defined before communication topology (dimension 2)**. Failure modes determine which topology is safe; topology-first bakes in an unsafe assumption. | Coordination setup present. | F6, L4 |

## Compaction safety — the 83.5% threshold and what may override it

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR6** | Implementing compaction → threshold is **83.5%, not 100%**, UNLESS the system operates entirely within single-session context limits. 100% is the documented catastrophic point. | Context-management / compaction steps. | F1 |
| **DR7** | A non-standard compaction threshold is proposed → justification **must** include: (a) safety-critical content size, (b) remaining headroom, (c) historical context-utilization patterns, (d) red-team test result at the proposed threshold. No justification → revert to 83.5%. | DR6 override attempted. | F1, L1 |
| **DR8** | Implementing summarization → immune-tagged content must survive **verbatim** — not paraphrased, not compressed. Paraphrase is silent loss the agent can't detect (L2). | Conversation/history summarization. | F1, L2 |
| **DR9** | Implementing model routing for cost savings → **safety-critical task classification must exist and routing rules must respect it BEFORE routing is deployed**. Routing a safety-critical task to a weaker tier is a silent safety downgrade. | Model selection / tiered routing. | F1 |

## Boundary-validation depth — graduated, never zero on a cross-agent boundary

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR10** | Boundary is agent↔agent AND system is production → deploy **full validation: schema + semantic + contamination monitor + circuit breaker**. This is the boundary type contamination compounds across (L3). | Production agent↔agent boundary. | F3 |
| **DR11** | Boundary is agent↔tool AND volume exceeds the validation latency budget → deploy **statistical validation with a defined sampling rate** — never zero. Full validation that dominates cost is its own failure (F8); zero validation is worse. | High-volume tool boundary. | F8, F3 |
| **DR12** | A boundary is proposed as **trust-based (no validation)** → permitted ONLY if the system is single-agent with no information-crossing boundaries. **Trust-based validation is forbidden in any multi-agent system** (C8.FS1). | Any boundary in a multi-agent context. | F3 |

## Artifact storage — promote path before ephemeral, retention before deployment

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR13** | Implementing ephemeral-first storage (PS-7) → the **promote-to-durable path must exist and be tested BEFORE ephemeral storage is deployed**. Ephemeral-without-promote is a data-loss mechanism, not a minimal-viable subset. | Agent artifact generation / exploration infra. | F5 |
| **DR14** | Implementing storage tiering → durable artifacts must have **explicit retention policies** AND ephemeral artifacts must have **explicit auto-expiry**. Untiered "temporary" storage either bloats forever or expires the wrong thing. | Any storage infra steps. | F5, L1 |

## Progressive disclosure — abstraction with a discoverable way back

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR15** | Implementing progressive disclosure → **each abstraction layer documents what it hides and how to access the hidden detail**. Abstraction without an escape hatch violates III-1 and starves the agent. | Disclosure / lazy loading / skill loading. | F4, III-1 |
| **DR16** | Implementing lazy tool loading → **tool descriptions must be complete enough for tool selection without loading the full schema**. Name-only descriptions violate L2 — the agent selects blind. | Lazy loading / skills architecture. | F4, L2 |

---

**When a rule says "block," "reorder," "revert," or "forbidden":** you're not refusing to help. You're
declining to sequence an optimization ahead of the safety step it depends on, to call an unvalidated
boundary done, or to ship a threshold that the corpus shows is unsafe. In every case: name the
obligation (or its absence), name the ordering constraint or threshold, and — for content crossing the
ownership boundary (WHAT-to-contain, whether-compliant, which-metric) — name the downstream skill that
owns it and the placeholder assumption you'll use until it's verified.
