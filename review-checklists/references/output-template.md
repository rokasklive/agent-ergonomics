# Output Template — The Checklist Artifact

> A full review produces these sections. A targeted review (a single description-change gate)
> emits a focused subset — usually 1–6 plus 8–9. **Don't pad:** if a section is genuinely N/A
> (e.g. Red-Team Calibration Log for an internal prototype), say so in one line and move on.
>
> The artifact is downstream-usable: tracked action items feed *Decision Records*, escalations
> feed *Design Templates* / *Implementation Playbooks* / *Debugging Workflows*. Keep item IDs
> and contract references intact so the chain survives.

The sections, with what each carries and which stage produces it:

## 1. Review Metadata — *(Stage A)*
Review trigger, date, reviewer identity, system under review, system criticality classification
(safety-critical / production-operational / internal-tooling / research-prototype).

## 2. Contract Scope — *(Stages A, D)*
Each of the 8 contracts marked included or excluded, with trigger→contract rationale and depth
level. Exclusions need a one-line reason.

> **Worked fragment**
> - **C2 Interface Change Auditability — INCLUDED (full).** Trigger is a tool description change (DR13).
> - **C6 Progressive Disclosure — INCLUDED (lighter).** Description changes affect tool-selection sufficiency.
> - **C3 Coordination Architecture — EXCLUDED.** No agent added, topology unchanged; the 7 dimensions are unaffected.
> - **C1 Trace Chain — DEPENDENCY CHECK only.** C2 tested evidence requires traces; lightweight verification that trace infra exists.

## 3. Scope & Depth Declaration — *(Stage D)*
Depth (full / scored / binary), scope model (triggered / rotated / full-sweep), gate vs. audit,
and contracts explicitly excluded with rationale.

## 4. Checklist Items — *(Stage B)*
Per contract, per obligation: verification question, pass criteria, fail criteria, obligation
reference. Every item must be discriminable (a failing variant is describable).

> **Worked fragment** — *C2 RC1/RC2*
> | Item | Question | Pass | Fail | Ref |
> |---|---|---|---|---|
> | C2.1 | Are pre-change and post-change agent-behavior baselines compared via CUJ testing? | CUJ run on this change shows no regression vs. the pre-change baseline. | No CUJ run for this change, or CUJ shows regression not classified as breaking. | C2 RC1, RC2 |

## 5. Evidence Standards — *(Stage C)*
Per item: assigned tier (documented / tested / monitored), what the reviewer must inspect, and
the justification for the tier (contract class × system criticality).

## 6. Completed Checklist — *(Stage E)*
Per item: **pass / fail / conditional-pass**, evidence citation (doc name, test run date, metric
value), reviewer notes. Evidence must match the assigned tier; an uncited pass is a fatigue
signal, not a pass.

## 7. Evidence Quality Assessment — *(Stage F)*
Per-item evidence-sufficiency verdict; failure classification (design / evidence / operational
gap); root-cause clusters; fatigue signals; production-incident cross-reference.

## 8. Gate Decision — *(Stage G)*
Per-contract verdict; overall verdict; escalation path per failure (which downstream skill owns
resolution); deadline/condition for each conditional-pass; and the mandatory **scope-limitation
statement**.

> **Worked fragment**
> > **Overall: CONDITIONAL-PASS — deployment blocked.**
> > - **C2: FAIL.** No CUJ test results for this description change (C2 RC2, tested evidence required, DR8). A renamed parameter is a breaking change until proven otherwise (L2). Escalate CUJ pipeline integration → *Implementation Playbooks*. Resolve before deploy.
> > - **C6: PASS.** Default/verbose formats documented; error responses include fix instructions.
> > - *Scope limitation:* This review covered C2 and C6. It did not verify C1, C3, C4, C5, C7, or C8. This is a contract-compliance verification, not a system-performance evaluation, and it does not certify that the CUJ tests, once run, are themselves correct or complete.

## 9. Version Metadata — *(Stage G)*
Contract versions used for derivation (e.g. v0.2, 2026-05-30), checklist application date,
evidence-review date, checklist version identifier. Enables staleness detection (DR17).

## 10. Checklist Iteration Notes — *(Stage H)*
Stale-item flags (>90 days unused), missing-coverage gaps (obligations not yet encoded),
regeneration action items, items that missed production failures.

## 11. Cadence & Rotation Plan — *(Stage I)*
Next review date per contract, cadence adjustments (tightened/loosened with reason), the
rotation schedule for scheduled health checks, and confirmation of per-change gate assignment
for C2 and C4.

## 12. Tracked Action Items — *(Stages F, G)*
Every conditional-pass and fail item with: owner, downstream skill for resolution, deadline,
and blocking status (blocks deployment / advisory).

## 13. Red-Team Calibration Log — *(Hook H2, Stage H; required for safety-critical systems)*
Last red-team injection date, injected violations, detection results, and items modified because
they missed an injected violation. This is how C4's tested-evidence floor stays honest — a green
C4 checklist that has never survived an injection test is exactly the False-Confidence failure (F2).

## 14. Fatigue Monitoring Data — *(Stages F, I)*
Checklist application time, items per review, pass-rate trend over the last 3 reviews, and
production-incident rate vs. checklist pass rate. A 100% pass rate co-occurring with continued
incidents is the signature of F1 (fatigue) or F2 (false confidence) — surface it here.

---

## Targeted vs. full output

- **Pre-deployment gate** (e.g. description change): Sections 1–6, 8, 9, 12. Lead with the gate
  decision; the team needs the block/ship answer first.
- **Scheduled health check**: Sections 1–3 (showing the rotation), 6–11. Emphasize what rotated
  in and what's deferred.
- **Onboarding / first review**: all 14 at reduced depth — the goal is a gap map, so Sections 7,
  10, and 12 (what's missing, what to build) carry the weight.
- **Safety-critical system**: Section 13 is required regardless of trigger.
