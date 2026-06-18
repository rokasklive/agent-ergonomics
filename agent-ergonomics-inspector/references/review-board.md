# Review Board — Adversarial Mode Guidance

Review Board is the skill's high-confidence operating mode. It exists to **reduce
false positives, challenge assumptions, and improve review reliability** — at a
real cost in tokens and time. Use it when the stakes justify the spend: a 1.0
launch gate, a contested review, a system where acting on a wrong finding is
expensive. For everyday work, Full Review is the right default; Review Board is the
mode you reach for when "probably right" isn't good enough.

It does **not** change the framework. The five laws, five domains, scoring anchors,
finding structure, and review philosophy are exactly those in `SKILL.md`,
`references/laws.md`, `references/domains.md`, and `references/anti-patterns.md`.
Review Board changes the *process* that produces a report, not the *criteria* the
report is judged against.

## Why a board, and why it's shaped this way

Three results from the agent-ergonomics corpus drive the design:

- **Closed loops beat pipelines.** Closed-loop architectures — where an output is
  re-grounded against intent — neutralize 40%+ of injected faults that open
  pipelines pass straight through (L3). A lone Full Review is an open pipeline: its
  findings exit unchecked. The board closes the loop by sending the review back
  through an adversary and an arbiter who re-reads the target.
- **Verification layers pay.** Adding an explicit self-review/verification step
  against success criteria has measurably improved multi-agent task success. The
  adversarial pass is that verification layer aimed at the *review itself*.
- **Seek truth, not compromise.** Averaging an expert and a non-expert view
  underperforms the better view by 8–38%, and the gap grows with participants
  (integrative compromise, L4 ← L3). So the board is **not** a vote and **not** a
  split-the-difference. The arbiter holds structural authority and rules on the
  evidence — it may side entirely with the primary, entirely with the adversary, or
  with neither. Consensus is not the goal; the correct call is.

**The honest tradeoff (state it in the report when relevant).** The board adds its
own boundaries — primary → adversarial → arbiter — and each crossing spends context
budget (L1) and is itself a place fidelity can drop (L3). That is the price of
higher confidence. The mitigation is that the arbiter re-grounds against the target
system rather than trusting the chain, which is what keeps the added boundaries from
simply compounding error. Don't run a board where a Full Review would do; the cost
is only justified when confidence is the objective.

## The three roles

Run all three roles even when you are a single agent playing them in sequence. The
discipline is in the **separation**: each role has a different job, a different
input, and a different success condition, and you must genuinely switch stance
between them rather than rubber-stamping your own prior output.

### 1. Primary Inspector

- **Job:** Perform a normal **Full Review** of the target — Phases 1–3, the standard
  output, every section. Nothing about this step is different from default mode.
- **Input:** The target system only.
- **Success condition:** A thorough, well-evidenced Full Review with findings in the
  standard block structure. Number the findings (F1, F2, …) so later stages can
  reference them.
- **Stance:** Build the strongest honest review you can. Do not pre-soften findings
  in anticipation of the adversary — that defeats the point. Let the adversary do
  its job on a real review.

### 2. Adversarial Reviewer

- **Job:** Review **the review**, not the system. Try to *falsify* each finding.
- **Input:** The Primary Inspector's review. The adversary does **not** independently
  re-review the target from scratch — it may consult the target to check a specific
  claim, but its product is a critique of the review, not a second review.
- **Success condition:** Every non-trivial finding has been pressure-tested, and the
  weak ones are exposed. A board where the adversary found nothing to challenge
  either had a perfect primary review (rare) or a lazy adversary (more likely).
- **Stance:** Skeptical, specific, fair. You are not contrarian for its own sake —
  you are trying to break findings that deserve to break, and you concede the ones
  that hold. Attack the reasoning, not the inspector.

**Falsification checklist — run each finding against these:**

- **Weak or missing evidence.** Is the claim grounded in something concrete in the
  target (a specific tool, field, error, handoff, format), or is it asserted? Would
  the evidence survive someone actually opening the file?
- **Overclaim / overgeneralization.** Does the finding state as fact what is really a
  hypothesis about agent behavior? Is a directional effect dressed up as a measured
  one? Are literature numbers (the ~5–10 tool figure, 67.6% output share) presented
  as hard limits rather than heuristics?
- **Unsupported recommendation.** Does the fix follow from the diagnosis, or is it a
  preferred pattern bolted on? Would the fix actually remove the root cause, or just
  the symptom?
- **Inflated severity.** Is a Critical/High actually load-bearing for *agent*
  operation, or is it a rare edge case, a once-per-session cost, or a human-UX nit
  ranked too high? Conversely, flag anything *under*-rated.
- **Law misuse.** Is the named law the real mechanism, or decoration? Is a surface
  law (L4/L5) named where a foundation law (L1/L2/L3) is the actual root? Is the law
  invoked as a citation ("violates L1") instead of an explained mechanism?
- **Ignored tradeoff.** Does the finding present a design *choice* (flip a default,
  shrink the tool surface, split agents) as an objective requirement, hiding the
  cost? What does the recommended change make worse?
- **Scope drift.** Has the review wandered into correctness bugs, style, or
  human-UX that don't change how an agent operates the system?
- **Double-counting.** Are several "findings" really one root cause that should
  collapse, inflating the apparent problem count?

For each challenge, state: the finding, the line of attack, and the specific
falsification attempt (what would have to be true for the finding to hold, and
whether it is).

### 3. Evidence Arbiter

- **Job:** Read all three — the **target system**, the **primary review**, and the
  **adversarial critique** — and issue the final, binding report.
- **Input:** All three. The arbiter is the only role that re-reads the target with
  the critique in hand; this is what closes the loop.
- **Success condition:** Every finding has a defensible final status grounded in the
  target, and the final report is something the reader can act on with confidence.
- **Stance:** **Truth-seeking, not compromise-seeking.** Hold the authority. Rule on
  evidence, not on who argued harder. You may:
  - **Confirm** a finding the adversary failed to break;
  - **Reject** a finding whose evidence doesn't survive a look at the target;
  - **Downgrade or upgrade** severity independent of both prior positions;
  - **Elevate** a real issue the primary review missed and the adversary surfaced
    (or that you see on re-reading) — mark it arbiter-introduced;
  - **Introduce nuance** — split a finding, narrow its scope, or add the tradeoff the
    primary review omitted;
  - **Side with neither** when both are wrong.

**Arbitration rubric — classify every finding:**

| Status | When to assign |
|--------|----------------|
| **Confirmed** | The finding holds: evidence is solid, the mechanism is right, the severity is appropriate (or you've corrected it). The adversary's challenge failed or didn't apply. |
| **Partially Confirmed** | The core problem is real but the finding overreached — severity was inflated, scope was too broad, the fix was wrong, or the law was mis-attributed. Keep the true part; record what you trimmed. |
| **Rejected** | The finding does not survive contact with the target: evidence is absent or wrong, it's a non-issue for agent operation, it's a misread of behavior, or it's a tradeoff misframed as a defect. |
| **Uncertain** | Cannot be resolved with available visibility — it depends on runtime/trace data you can't see, or on how an agent would actually behave. Say what evidence would settle it. Do not resolve uncertainty by guessing; an honest Uncertain is a valid result. |

Record, per finding: original severity, the adversarial position, the final status,
the final severity, and the arbiter's reasoning (grounded in the target). The final
scorecard reflects the arbitrated findings — if arbitration confirmed fewer or
less-severe problems in a domain, the score moves accordingly, with the justification
saying so.

## Running the board in one context

The default execution is **sequential role-play in a single context**: do the Full
Review as the Primary Inspector and write it out; then explicitly switch stance to
the Adversarial Reviewer and critique that written review; then switch to the
Evidence Arbiter, re-open the target, and produce the final report. Make the switches
visible in the output (the three input sections — Primary Findings, Adversarial
Challenges, Arbitration Results — are how the reader sees the separation held).

The risk of single-context role-play is that the adversary goes soft on the
inspector's work because it's "your own." Counter it deliberately: when you put on
the adversary hat, your job is to *break* the findings, and a critique that waves
everything through is a failed critique. Treat the prior review as a stranger's.

If the harness makes genuine subagents available and the review is large enough to
warrant it, you may dispatch the Primary Inspector, Adversarial Reviewer, and
Evidence Arbiter as separate agents — true context isolation strengthens the
adversarial separation (L3: isolate to contain contamination). This is an option,
not a requirement; the sequential single-context process is the baseline and is
sufficient for most reviews. Whichever you use, the roles, inputs, and arbitration
rubric are identical.

## Output

The arbiter produces the **Review Board output format** defined in `SKILL.md`:
Executive Summary, Ergonomics Scorecard, Primary Findings, Adversarial Challenges,
Arbitration Results, Architectural Risks, Quick Wins, Strategic Improvements, Final
Assessment — with each arbitrated finding carrying its original severity, the
adversarial position, and the final status. Keep the scorecard and finding logic
identical to Full Review; the board adds layers, it does not fork the framework.
