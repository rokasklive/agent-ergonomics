# Decision Rules (DR1–DR18)

> These are operational gates, not suggestions. Each turns a design habit into a yes/no check
> at a specific point in the procedure. Each rule names the failure mode or law it prevents —
> lead with that reasoning when you apply it, don't just cite the number.
>
> The four decision points behind them: **D1** granularity, **D2** cadence, **D3** evidence
> standard, **D4** scope. The failure modes they guard: **F1** Checklist Fatigue, **F2** False
> Confidence, **F3** Checklist Rot, **F4** Scope Creep, **F5** Evidence Downgrading Under
> Pressure, **F6** Missing Contract Dependencies, **F7** Checklist as Substitution for Testing.

## Granularity (D1) — how deep each item goes

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR1** | Safety-critical system → full contract depth (all Required Checks + all Forbidden Shortcuts) with **tested or monitored** evidence for C1, C2, C4, C5, C8. | System handles user data, has safety implications, or runs at production scale with real consequences. | F2 |
| **DR2** | Internal-tooling system → **binary pass/fail** per obligation; documented evidence acceptable for all contracts. | Internal, non-safety-critical, low external consequence of failure. | F1 |
| **DR3** | Research-prototype → reduced checklist: only **C1 and C8** at documented evidence; other contracts N/A with explicit documentation of why. | Pre-production, under active research, ergonomic failures are acceptable tradeoffs. | F1 |

Depth ladder, from heaviest to lightest: **full detail** (every RC + FS, tested/monitored) →
**scored** (pass/partial/fail with evidence tiers) → **binary** (pass/fail, documented).

## Cadence (D2) — how often a contract is checked

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR4** | C2 → **per-change gate**: every deployment touching an agent-facing description fires the full C2 checklist. Cannot be deferred to a scheduled audit. | Any change to tool descriptions, error formats, output schemas, naming. | L2 silent drift |
| **DR5** | C4 → **per-change gate**: every compaction/summarization/model-routing/context-isolation change fires the full C4 checklist. | Any change to those optimization paths. | II-1 safety bypass |
| **DR6** | C3 and C8 (structural) → **scheduled audit**: quarterly for production, bi-annually for stable; also triggered by scaling events. | Scheduled rotation, or a new agent/tool/service boundary. | F4 |
| **DR7** | Deployments > 3×/week → per-change C2/C4 checklists must finish in **under 15 minutes**; if they exceed it, cut to critical items only. | Deployment velocity at risk of being throttled by checklist time. | F1 |

## Evidence (D3) — what standard each item demands

Three tiers: **documented** (a design doc or policy statement exists), **tested** (active
verification producing actual agent behavior), **continuously monitored** (automated checks
with alerting).

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR8** | Behavioral-contract item (C2, C4, C5) → evidence floor is **tested**; documented-only is insufficient. | The item verifies agent *behavior*, not design documentation. | F2, L2 |
| **DR9** | Structural-contract item (C3) at stable scale → **documented** evidence acceptable. | Coordination architecture unchanged, component count stable since last review. | — |
| **DR10** | C2/C5/C8 at production scale → **continuously monitored** where infra exists; **tested** where monitoring is absent. | System has CUJ pipelines, calibration dashboards, or boundary monitors. | F2 |
| **DR11** | Evidence is a design doc stating intent but not demonstrating operational reality → **reject** for any behavioral item; mark **conditional-pass** with a tracked action item to produce tested evidence. | Reviewer submits "the design doc says we do X" for a C2/C4/C5 item. | F2 |
| **DR12** | Evidence older than **90 days** for a time-sensitive contract (C5 calibration, C2 CUJ tests, C8 validation) → **reject as stale**; require fresh evidence. | Most recent calibration/CUJ/validation is outside the 90-day window. | F3 |

## Scope (D4) — which contracts the review covers

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR13** | Tool description change → scope is **C2 (full) + C6 (lighter)**; exclude others unless the change also altered tool count or coordination topology. | Description-only change without schema modification. | F4 |
| **DR14** | New agent addition → scope is **C3 (full) + C8 (full)**; if agent count crosses an inflection point (~5–10 single-agent tools, ~3–8 multi-agent mesh), add a compounding-friction scale test. | Any deployment adding an agent. | F4 |
| **DR15** | Scheduled health check → **rotate** contract scope across the review cycle; never apply all 8 at full depth in one session. | Scheduled review with no specific change trigger. | F1 |
| **DR16** | Onboarding → all 8 contracts at **reduced depth** (binary, documented acceptable); purpose is gap identification, not deployment gating. | First ergonomic review of a system. | F1 |

## Rotation (D3, living-document) — keeping the checklist itself honest

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR17** | Checklist items > 90 days old without re-application → flag **stale**; regenerate from current contract versions before next use. | Last review of that contract was > 90 days ago. | F3 |
| **DR18** | A production failure occurred that a passed checklist item should have caught → **add** a new item for the missed failure mode and **escalate the evidence standard** of the item that failed to detect it. | Post-incident review of any production failure in a system with existing checklists. | F3 |

## The four that most often decide a review

Hold these in working memory:

- **DR4 / DR5** — C2 and C4 are per-change gates. If someone wants to ship a description change
  or a compaction-policy change without firing these, that's the review's whole point. Don't
  let them be deferred.
- **DR8 / DR11** — behavioral contracts need *tested* evidence. "The design doc says safety
  instructions are preserved" is not evidence that they survive compaction. Reject it,
  conditional-pass with an action item, and say why: documented claims about interface
  behavior are indistinguishable from tested ones until they fail in production (L2).
- **DR13 / DR15** — scope follows the change, not the calendar. A description-change review that
  drags in C3 wastes reviewer attention on a contract the change can't affect; a scheduled
  health check that runs all 8 at full depth manufactures fatigue. Both erode the checklist's
  value faster than any single missed item.

When a rule says "reject," you are not refusing to help — you are declining to record a *pass*
that would manufacture false confidence. Record the finding, set the gate to conditional-pass
or fail, name the evidence that *would* satisfy the item, and name the downstream skill that
owns producing it.
