# Anti-Patterns of Agent Ergonomics

A catalog of recurring, evidence-backed design mistakes. Use it as a fast
recall layer during Phase 2 — scan the list against the system and see what fires.
But do not stop at "this matches anti-pattern N." An anti-pattern is a *symptom
with a name*; your job is still to trace it to its root law and explain the
mechanism for *this* system. The catalog tells you where to look; the laws tell you
why it hurts.

Each entry gives: the harm, the root law(s), the diagnostic tell, and the
direction of the fix. Severity reflects how badly it degrades agent effectiveness
when present, weighted by evidence strength.

| #  | Anti-pattern | Severity | Root laws |
|----|--------------|----------|-----------|
| 1  | Wrap every API endpoint as a tool | Critical | L4 ← L1, L2 |
| 2  | Return excessive irrelevant data | Critical | L1 |
| 3  | Silent interface changes (no CUJ test) | Critical | L2 |
| 4  | Outcome-only monitoring | Critical | L2, L3 |
| 5  | Add agents without expertise enforcement | High | L4 |
| 6  | Single retry loop without circuit breaker | High | L4 |
| 7  | Optimize prompts, ignore tool-output bloat | High | L1 |
| 8  | Usage-based pricing for agent-first products | Medium | L1, L4 |
| 9  | No human gold-set for LLM-as-judge | Medium | L5 |
| 10 | Treat chat as the whole UX | Medium | L2 |
| 11 | Ignore context invalidation / state drift | Medium | L3 |
| 12 | Consensus-seeking with uneven expertise | Medium | L4 ← L3 |

---

### 1. Wrap every API endpoint as a tool — *Critical*
**Harm.** Tool count climbs, schemas crowd the context window, and overlapping
tools create selection ambiguity; agents tend to lose accuracy somewhere past
~5–10 tools — treat that number as a model- and design-dependent heuristic, not a
hard cliff, and frame any "shrink the surface" recommendation as a tradeoff.
**Root.** L4 surface (it compounds), L1 + L2 underneath (each schema costs budget
and adds an interface to trust).
**Tell.** One tool per endpoint; dozens of narrow tools; near-duplicate tools.
**Fix.** Consolidate into fewer flexible tools; opt-in toolsets; split into
subagents past the threshold instead of piling tools on one agent.

### 2. Return excessive irrelevant data — *Critical*
**Harm.** Every unused field is reasoning capacity spent for no return; verbose
output literally makes the agent less capable. Tool responses are ~67.6% of all
tokens.
**Root.** L1.
**Tell.** Responses with 10+ fields where 3–4 carry the signal; full objects where
a reference or summary would do; no verbose/terse distinction.
**Fix.** Minimal default schema, verbose behind a flag; truncate large content;
token-budget pagination; keep bulky data out of context (execution environment).

### 3. Silent interface changes without CUJ testing — *Critical*
**Harm.** A renamed or reworded tool changes agent behavior with no 4xx/5xx and no
schema break. The agent can't detect it; prior-version memory becomes actively
misleading.
**Root.** L2.
**Tell.** Rolling-release tools; unpinned versions; descriptions edited without a
behavioral test; no critical-user-journey suite.
**Fix.** Version and pin interfaces (`@v1.2.0`); treat description regressions as
breaking changes; CUJ tests that exercise agent behavior, not just schema validity.

### 4. Outcome-only monitoring — *Critical*
**Harm.** Outcome checks miss ~55.6% of contamination — silent semantic corruption
and recovered detours never show up. The system fails in ways monitoring can't see.
**Root.** L2 (degradation invisible from inside) + L3 (it happens at boundaries).
**Tell.** Logs show inputs and final outputs but not the trajectory; no per-step
trace; success measured only by end state.
**Fix.** Trace-level observability (every call, response, reasoning step); trace
trees not flat logs; measure outcome *and* trajectory.

### 5. Add agents without structural expertise enforcement — *High*
**Harm.** Teams underperform their best member by 8–38% because they average expert
and non-expert views — even when told who the expert is.
**Root.** L4 (integrative compromise grows with team size).
**Tell.** Multi-agent designs added for "more perspectives"; consensus/voting with
no authority weighting; expertise asserted only in prompts.
**Fix.** Structural enforcement — hard routing, authority gates, weighted voting —
not prompted expertise. Use the smallest team that does the job.

### 6. Single retry loop without circuit breaker — *High*
**Harm.** Agents retry aggressively; one failing dependency triggers retry storms
across the fleet. Step-repetition loops have burned $40+ on a single query.
**Root.** L4.
**Tell.** Retry with backoff but no circuit breaker, no budget ceiling, no
de-duplication, no step cap.
**Fix.** Three-layer defense: exponential backoff + circuit breaker
(CLOSED→OPEN→HALF_OPEN) + hard budget/step ceiling.

### 7. Optimize system prompts while ignoring tool-output bloat — *High*
**Harm.** Effort goes to the ~3.4% (system prompts) while the ~67.6% (tool output)
goes untouched — a large misallocation of optimization effort.
**Root.** L1.
**Tell.** Heavy prompt engineering next to raw, verbose, unoptimized tool
responses.
**Fix.** Profile token consumption first; optimize the dominant driver — output
format, schema minimization, pagination — before polishing prompts.

### 8. Usage-based pricing for agent-first products — *Medium*
**Harm.** Agents explore by nature (create, fork, test, discard). Per-unit metering
penalizes exactly that behavior, suppressing the value you want.
**Root.** L1 (finite resource framing) + L4 (compounds at scale).
**Tell.** Cost scales linearly per operation with no bounded tier; create/destroy
is expensive.
**Fix.** Bounded pricing / budget ceilings; make create and destroy nearly free;
disposability by default.

### 9. No human gold-set for LLM-as-judge — *Medium*
**Harm.** LLM judges drift over 60–90 days; same-family judges over-reward. The eval
that was meant to validate decisions becomes misleading noise.
**Root.** L5 (the measurement layer is itself the deliverable, and it's degrading).
**Tell.** LLM-as-judge with no maintained human gold-set, no calibration cadence,
vague rubrics, judge from the same model family.
**Fix.** Maintain a 200–500 trace human gold-set; calibrate monthly (e.g. Cohen's
kappa); concrete rubrics; cross-family judges.

### 10. Treat chat as the whole UX — *Medium*
**Harm.** Agentic interfaces need more than a chatbox — goal, control, work, and
accountability surfaces. Missing the control/work/accountability surfaces leaves
agents (and their overseers) without the means to steer, observe, and audit.
**Root.** L2 (state and control that exist only implicitly are invisible).
**Tell.** A single conversational surface; no permission/autonomy controls, no
progress visibility, no audit/undo/rationale.
**Fix.** Add the missing surfaces; expose control and state explicitly. (Note: this
one shades into human-facing UX — apply it only insofar as the missing surfaces
also degrade *agent* operation, e.g. the agent can't tell what it's allowed to do.)

### 11. Ignore context invalidation and state drift — *Medium*
**Harm.** Long-running state goes stale; summarization drift corrupts session
memory; prior-version memory misleads under tool evolution. The "3 AM bug" lives in
summarization drift, not long-term storage.
**Root.** L3 (session and turn boundaries shed fidelity).
**Tell.** Implicit memory assumed fresh; repeated summarization with no fidelity
check; no version-awareness in stored state.
**Fix.** Structured state files over implicit memory; validate/refresh at session
boundaries; treat each summarization as lossy and check it.

### 12. Consensus-seeking with uneven expertise — *Medium*
**Harm.** When expertise is unevenly distributed, averaging/compromise is a bug:
expert judgment gets diluted with non-expert noise, and the gap grows with team
size.
**Root.** L4, via L3 (expertise signal lost at the discussion boundary).
**Tell.** Peer-critique or majority-vote across agents of clearly different
competence; minority-but-correct views collapsing.
**Fix.** Route to the competent agent; gate authority; weight by demonstrated
expertise rather than seeking agreement.

---

## Using the catalog without over-fitting

- **It's a recall aid, not a rubric.** Firing one of these is a lead to
  investigate, not a finding to report verbatim. Many real systems exhibit a
  *variant* the catalog doesn't name exactly — describe what you actually see.
- **Absence isn't health.** A system can dodge all twelve and still be hostile to
  agents in some way unique to its domain. The laws are the complete test; the
  anti-patterns are the common cases.
- **Severity here is a prior, not a verdict.** "Critical" means *usually* severe.
  Down-rate it if the context makes it minor (e.g. excessive data on a tool the
  agent calls once per session), and up-rate a "Medium" if it's load-bearing in
  this system. Judge impact in context, then explain the judgment.
