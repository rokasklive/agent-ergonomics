# The Five Laws of Agent Ergonomics

These are the explanatory engine of the review. A scorecard tells the user *what*
is wrong; the laws tell them *why* it is wrong and why it will keep being wrong
until the structure changes. Use them as causal mechanisms, not citations. A
finding that ends with "...violates the Law of Bounded Attention" has done no
work. A finding that says "this returns 40+ fields the agent never reads, and
because the context window is a single shared budget across instructions,
history, and reasoning, every unused field is reasoning capacity spent for no
return" has explained the harm.

Three laws are **foundation** laws — irreducible facts about how agents relate to
the world. Two are **derived** laws — emergent consequences that appear at scale.
When you find a problem, trace it down to a foundation law if you can; that is
where the root cause lives.

## Table of contents

- [L1 — Law of Bounded Attention](#l1--law-of-bounded-attention) (foundation, capacity axis)
- [L2 — Law of Interface Ground Truth](#l2--law-of-interface-ground-truth) (foundation, perception axis)
- [L3 — Law of Boundary Entropy](#l3--law-of-boundary-entropy) (foundation, transmission axis)
- [L4 — Law of Compounding Friction](#l4--law-of-compounding-friction) (derived, scale)
- [L5 — Law of Ergonomic Dominance](#l5--law-of-ergonomic-dominance) (derived, leverage)
- [How the laws relate](#how-the-laws-relate)

---

## L1 — Law of Bounded Attention

**Statement.** An agent's effective processing capacity is bounded by its finite
context window. Every unit of information consumed reduces capacity for reasoning;
every unit revealed before it creates value wastes capacity without return.

**Mechanism — why agents specifically suffer.** The context window is working
memory, attention span, and information budget fused into one quantity. It is
shared across *everything*: system instructions, tool schemas, query results,
conversation history, and the agent's own reasoning tokens. Unlike human
attention, which degrades gracefully, the window has a hard edge — cross it and
behavior fails catastrophically, not gradually. This is the thermodynamic
constraint of agent systems: the way the speed of light bounds physics, the
context window bounds agent reasoning. It also has a *temporal* dimension —
information revealed too early burns budget before it pays off; revealed too late
it forces an expensive retrieval. The optimal strategy is progressive disclosure:
information available exactly when needed, not before.

**Diagnostic signals (what to look for).**
- Tool responses with many fields the agent rarely uses; verbose default schemas.
- Everything loaded eagerly — full file trees, entire docs, all tools' full
  schemas — rather than names-first / detail-on-demand.
- No pagination, or pagination by record count instead of token count (a single
  "page" can blow the budget unpredictably).
- Serialization that pays a structural tax (deeply nested JSON where a column
  format or compact notation would carry the same signal for far fewer tokens).
- System prompts and instructions that restate what the model already knows.
- No notion of a budget: no compaction strategy, no headroom reserved, no sense
  of which zone (instructions / tools / history / retrieved context) dominates.

**Measurements that make findings concrete.** ~67.6% of tokens in agent traces
come from tool responses (system prompts are ~3.4% — so prompt-golfing is usually
optimizing the wrong thing); MCP carries ~2.3× the token overhead of an
equivalent CLI; compact serialization saves ~40% over JSON; selective retrieval
can cut ~90% of tokens; production harnesses compact around ~83.5% of the window,
not at 100%.

**What good looks like.** Minimal default responses (a handful of high-signal
fields, verbose mode behind a flag); token-budget pagination; progressive
disclosure at every level (tool names before schemas, summaries before bodies);
a deliberate budget with reserved headroom; data that can stay out of the window
entirely (e.g. flowing through an execution environment) when the agent doesn't
need to read it.

---

## L2 — Law of Interface Ground Truth

**Statement.** Agents have no access to reality beyond what arrives through the
interface. Every description, schema, error message, and payload is treated as
authoritative truth. Implicit context is invisible; missing information is
unknown; degradation at the interface is undetectable from inside it.

**Mechanism — why agents specifically suffer.** Humans navigate through many
channels — we cross-reference sources, glance at physical state, apply common
sense, and feel when something is off. An agent navigates through a single
channel: the token stream. It cannot open the source to verify a tool's
description, cannot check a changelog to notice an interface drifted, cannot look
around a UI to infer state, cannot tell "no output" from "the command failed"
unless that distinction is spelled out. The interface *is* the agent's reality.
When the interface lies — a stale description, a truncated payload, a missing
field — the agent has no way to know, and integrates the false information into
its reasoning as fact. This is architectural, a property of the paradigm, not a
bug to be patched in the model.

**Diagnostic signals (what to look for).**
- Tool/function descriptions that are vague, out of date, or contradict actual
  behavior. The description is the contract; the implementation is invisible.
- Important behavior conveyed only implicitly — defaults, units, side effects,
  ordering, auth scope assumed rather than stated.
- Ambiguous outputs: the same response shape for "empty result," "error," and
  "not authorized." Silent failures. Null where a labeled empty-state belongs.
- Errors that report *that* something failed without saying *what to do* — no fix
  hint, no validation detail, no likely cause.
- Interfaces that can change underneath the agent (rolling-release tools,
  unpinned versions) where a rename or reword silently alters behavior with no
  4xx/5xx, no schema break.
- State the agent is assumed to "just know" but was never told.

**Measurements that make findings concrete.** Rewriting *descriptions alone* —
same model, same implementation — swings task accuracy by 20+ points and task
time by ~40%; 55.6% of multi-agent contamination is invisible to outcome-only
checks because each downstream agent trusts upstream output as ground truth;
production "47-call loop" bugs have been traced to a single misleading
description.

**What good looks like.** Descriptions treated as immutable, versioned contracts
(`@v1.2.0`, never silently rewritten); everything load-bearing stated explicitly;
definitive empty states ("no results found" vs. silence); structured, actionable
errors (what failed, why, how to fix); hierarchical, unambiguous names
(`admin.users.list`, not `list`); critical-user-journey tests so a description
regression is caught as the breaking change it is.

---

## L3 — Law of Boundary Entropy

**Statement.** Information fidelity necessarily decreases when crossing any system
boundary — between agents, between agent and tool, between context windows,
between representations. Errors, contamination, and semantic drift compound at
each boundary, and the loss is invisible to local observers.

**Mechanism — why agents specifically suffer.** Boundaries are where information
degrades in any system, but agent systems take it especially hard because (a)
agents can't detect the degradation (L2), (b) each crossing also spends scarce
attention (L1), and (c) the loss *compounds* — a 5% error at boundary 1 becomes
~10% at boundary 3, ~30% at boundary 6. It applies to *every* transition, not
just agent-to-agent: agent↔tool, turn↔turn, session↔session,
representation↔representation. Each is a degradation point. The corollary —
"uncertainty accumulation" — folds in here: contamination (agent boundary),
expertise dilution (discussion boundary), silent breakage (version boundary),
judge drift (time boundary) are all the same phenomenon wearing different clothes.

**Diagnostic signals (what to look for).**
- Many handoffs: agent → agent → agent, or summarize → pass → summarize again.
  Count the boundaries; each one sheds fidelity.
- Linear pipelines with no validation at the seams (catastrophically vulnerable —
  one bad output propagates downstream as truth).
- Summarization / compaction steps that silently drop information, especially
  safety-relevant information, with no fidelity check.
- Translation layers (format A → format B → format C; protocol A → protocol B)
  where each conversion can lose or distort meaning.
- Long-running memory that drifts (state assumed fresh that has gone stale).
- No closed loop: outputs are never checked back against intent, so drift never
  gets corrected.

**Measurements that make findings concrete.** 55.6% of contamination invisible to
outcome-only metrics (614 paired runs); multi-agent teams underperform their best
member by 8.1–37.6% (synergy gap that *grows* with team size); memory accuracy
degrading 51.84% → 25.05% over time; closed-loop architectures neutralize 40%+ of
injected faults where pipelines do not.

**What good looks like.** Boundary engineering — make boundaries explicit and
validate information as it crosses them; prefer closed-loop / orchestrator-worker
shapes over open pipelines; isolate context to contain contamination rather than
letting it propagate; checkpoint and re-ground against intent; treat each
summarization as a lossy operation that needs a fidelity check, and accept the
isolation/fidelity trade-off deliberately rather than by accident.

---

## L4 — Law of Compounding Friction

**Statement.** In systems of interacting agents and tools, inefficiencies compound
non-linearly with scale. Adding components beyond an optimal threshold degrades
rather than improves outcomes.

**Type.** Derived — it emerges from L1 + L2 + L3. Theoretically you could always
re-derive it, but it earns a name because it captures a counterintuitive result
that people repeatedly get wrong: *more* is frequently *worse*.

**Mechanism — why agents specifically suffer.** Every new component consumes
context budget (L1), introduces a new description/schema to be trusted as ground
truth (L2), and adds boundary crossings whose degradation compounds (L3). The
crossings themselves grow combinatorially — N agents imply O(N²) potential
pairwise interactions. And the degradations *interact*: a degraded output at
boundary 1 is trusted as truth at boundary 2 and degraded further. So the 5th
agent hurts more than the 3rd, the 10th tool more than the 5th, the 100th turn
more than the 50th. Performance against scale follows an inverted-U, not a line.

**Diagnostic signals (what to look for).**
- One-tool-per-endpoint sprawl; dozens of overlapping, narrowly-scoped tools.
- Multi-agent designs that add members hoping for additive gains.
- Retry logic without a circuit breaker or budget ceiling — one failing
  dependency triggers a retry storm across the fleet.
- Consensus/averaging across agents of uneven expertise (dilutes the expert).
- Coordination overhead that grows faster than the work being coordinated.

**Measurements that make findings concrete.** Single agents lose effectiveness
past ~5–10 tools; multi-agent token costs run ~10–15× single-agent (not 2–3×);
synergy gaps grow with team size (p < 0.05); step-repetition loops have burned
$40+ of API credit on one query; coordination drag of ~30–40% is recoverable.

**What good looks like.** Tool consolidation (one flexible tool beats many narrow
ones); opt-in toolsets so only relevant bundles load; the smallest team that does
the job (optimal is smaller than intuition suggests); structural expertise
enforcement — hard routing and authority gates, not "you are the expert" in a
prompt; three-layer retry defense (exponential backoff + circuit breaker + budget
ceiling). Note the open research gap: the *exact* shape of the compounding curve
(inflection points, interaction with model quality) is hypothesized, not
characterized — so flag scaling risk as directional, not precisely quantified.

---

## L5 — Law of Ergonomic Dominance

**Statement.** The quality of the surrounding system — tool design, coordination
architecture, information structures, economic constraints — determines real-world
outcomes more than the quality of the underlying model. The interface layer
dominates the intelligence layer.

**Type.** Derived (from L1 + L2 + L3 + L4). The least empirically nailed-down law
but the most decision-relevant — it tells people where to spend.

**Mechanism — why this follows.** The model can only act on what it receives (L2),
within the attention budget it's allocated (L1), across fidelity-degrading
boundaries (L3), with inefficiencies that compound at scale (L4). Raw intelligence
— parameter count, benchmark score — matters only to the extent the surrounding
system lets it be expressed. So for any given model generation, the variance in
outcomes explained by ergonomic quality tends to exceed the variance explained by
model quality. (This does *not* say models don't matter; it says the system is the
larger lever in deployment.)

**Diagnostic signals — used differently from the others.** L5 is the *framing*
law. You don't hunt for its "violations" so much as use it to set priorities: when
the user is reaching for a bigger model to fix something that is actually a tool,
harness, or coordination problem, that is the L5 finding. It also tells you which
fixes carry the most leverage — the ones the "Strategic Improvements" section
should surface.

**Measurements that make findings concrete.** 79% of multi-agent failures are
coordination, not model (1,642 traces); principled design beats protocol choice
(100% success at $0.074/task vs. 99% at $0.100); the top improvements on harness
benchmarks come from harness changes, not model swaps; "the model is the small
box" in the full agentic architecture; model routing alone moved cost $0.40 →
$0.18 per run.

**What good looks like / how to wield it.** When you write the Final Assessment,
L5 is the lens for "what change would produce the highest ROI": almost always it's
a structural change to the system, not a model upgrade. If you believe L5, you
invest in tool design, harness architecture, and evaluation infrastructure before
chasing the next model. Use it to redirect effort toward the layer that actually
governs the outcome.

---

## How the laws relate

```
            CAPACITY            PERCEPTION           TRANSMISSION
        ┌──────────────┐   ┌──────────────────┐   ┌────────────────┐
        │ L1 Bounded   │   │ L2 Interface     │   │ L3 Boundary    │
        │   Attention  │   │   Ground Truth   │   │   Entropy      │
        └──────┬───────┘   └────────┬─────────┘   └───────┬────────┘
               │                    │                     │
               └──────────┬─────────┴──────────┬──────────┘
                          │                    │
                          ▼                    ▼
                 ┌─────────────────┐  (L2 makes L3's loss
                 │ L4 Compounding  │   undetectable; every
                 │    Friction     │   L3 crossing spends L1)
                 └────────┬────────┘
                          ▼
                 ┌─────────────────┐
                 │ L5 Ergonomic    │  the system surrounds the
                 │    Dominance    │  model on all three axes
                 └─────────────────┘
```

- **L1, L2, L3 are independent axes** — capacity (what the agent can hold),
  perception (what it believes is true), transmission (what survives the trip).
  Most concrete problems sit primarily on one axis; name that axis first.
- **They interact, which is what makes agent systems hard.** L2 is why L3's
  degradation is dangerous (the agent can't see it). Every L3 crossing also spends
  L1's budget. Progressive disclosure answers L1 *and* L2 (you can't un-see what
  you've already read).
- **L4 is the three foundation laws under load.** Reach for it whenever the
  problem is "this gets worse as it grows."
- **L5 is all four, integrated, pointed at the decision-maker.** Reach for it when
  the question is "where should effort go."

When attributing a finding, prefer the deepest applicable law. "Too many tools" is
visible as L4 (it compounds), but the root is L1 (each schema costs budget) and L2
(each adds an ambiguous interface). Naming the foundation law is what turns a
symptom into a root cause.
