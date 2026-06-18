# The Checklist Procedure — Stages A–I

> Nine stages, in order. Scope before items; items before evidence standards; evidence
> standards before application; application before the gate decision; gate before the
> maintenance machinery that keeps the checklist itself honest. Each stage names the output
> section it produces (see `output-template.md`).
>
> You won't always run all nine. A pre-deployment gate for a description change is mostly
> Stages A–E and G. A scheduled health check leans on A (rotation), H, and I. Onboarding runs
> all nine at reduced depth. Use judgment; the order is what matters when stages do run.

---

## Stage A — Contract Selection

Decide which contracts are in scope. **This is where you prevent both fatigue (too many) and
blind spots (too few).**

1. Identify the review trigger — deployment type, scheduled health check, onboarding, or
   red-team exercise. Name the specific artifact that changed and its scope.
2. Map the trigger to affected contracts using the trigger→contract table in `contracts.md`.
3. Apply the **dependency rule**: if C2, C4, or C5 is in scope at tested evidence, confirm C1
   is either already verified or add a lightweight C1 check. Otherwise those contracts can
   only reach conditional-pass (Failure F6).
4. If the trigger is a scheduled health check, select this cycle's contracts by: time since
   last review per contract, production incidents since last review, and risk-proportional
   priority (safety-critical reviewed more often than efficiency). **Rotate — never all 8 at
   full depth in one session (DR15).**
5. Record every one of the 8 as included or excluded, *with a one-line rationale for each
   exclusion*. An unstated exclusion is indistinguishable from an oversight.

**Output:** Contract Scope section.

---

## Stage B — Item Derivation

Mechanically translate each in-scope contract's obligations into checklist items.

1. For each contract, pull its Required Checks (RC#) and Forbidden Shortcuts (FS#) from
   `contracts.md`.
2. Translate each **Required Check** into a verification *question* — not "the system must have
   X" but "**does** the system have X, and what evidence shows it?"
3. Translate each **Forbidden Shortcut** into "is there evidence this shortcut was avoided?"
4. Define **pass criteria**: what specific evidence would satisfy this item.
5. Define **fail criteria**: what absence of evidence, or what contrary evidence, fails it.
6. **Discriminability test (Hook H1):** describe a system variant that would *fail* this item.
   If you can't construct a failing variant, the item is verifying nothing — reformulate it.
   This is the single most important quality gate in the procedure; an item that always passes
   is worse than no item, because it manufactures confidence (F2).
7. Tag each item with its contract obligation reference (e.g. "C2 RC2"). Items without a
   contract reference are scope creep (F4) — move them to *Design Templates* or drop them.

**Output:** Checklist Items section (question, pass criteria, fail criteria, obligation ref).

---

## Stage C — Evidence Standard Assignment

Assign an evidence tier to each item.

1. Classify the item's contract: **behavioral** (C2, C4, C5 — needs observed agent behavior),
   **structural** (C3, C6 in stable systems — design docs can suffice), or **economic** (C7 —
   needs measurement). See the Evidence class line per contract in `contracts.md`.
2. Classify the system: **safety-critical** (behavioral contracts need tested or monitored),
   **production-operational** (behavioral → tested; structural → documented), **internal-tooling**
   (documented acceptable), or **research-prototype** (documented; many items N/A).
3. Assign the tier: **documented / tested / continuously monitored**, applying DR8–DR12.
4. For each item, specify *what the reviewer must inspect* — not "review the doc" but "verify
   the doc contains X, Y, Z" or "verify the judge model ID is from a different family than the
   system model."
5. Flag any item where the required tier exceeds what the system can currently produce. These
   become **conditional-pass** items with tracked action items (e.g. "no CUJ infrastructure →
   conditional-pass, action: build CUJ testing"). "Compliance cannot be verified" is a finding,
   not a pass.

**Output:** Evidence Standards section.

---

## Stage D — Scope & Depth Finalization

Lock in depth, scope model, and the gate-vs-audit posture.

1. Choose depth by criticality (DR1–DR3): full / scored / binary.
2. Choose scope model by trigger (DR13–DR16): triggered (deployment), rotated (health check),
   full-sweep at reduced depth (onboarding).
3. Decide **gate vs. audit**: does this checklist *block* deployment (must pass to proceed) or
   *inform* without blocking? C2 and C4 changes are gates (DR4/DR5).
4. State contracts explicitly excluded and why.

**Output:** Scope & Depth Declaration section.

---

## Stage E — Checklist Application

Apply the checklist to the system, gathering evidence at the assigned tier.

1. For each item, gather evidence at its tier.
2. **Documented:** retrieve and inspect the doc — verify it addresses the specific obligation,
   not just the general topic.
3. **Tested:** retrieve and inspect test results / CUJ outputs / compaction logs / calibration
   records — verify the tests were run *recently* and against the *current* system state.
4. **Monitored:** retrieve and inspect dashboards, alert history, recent metrics — verify
   monitoring is active and thresholds are configured.
5. Record the evidence reviewed per item (doc name, test run date, dashboard value) so a second
   reviewer can reproduce the verification. **An item marked pass with no evidence citation is
   itself a fatigue signal (F1).**
6. Mark each item **pass** (evidence meets criteria), **fail** (evidence absent or contrary), or
   **conditional-pass** (evidence insufficient but action item tracked).

**Output:** Completed Checklist section.

---

## Stage F — Evidence Review

Critically assess the *quality* of the evidence, not just its presence.

1. For each passed item, verify the assigned tier was actually met — "CUJ testing exists" (a
   claim) does not satisfy a tested standard; CUJ *results* do.
2. For each failed item, classify the failure: **design gap** (the thing doesn't exist),
   **evidence gap** (it exists but evidence is insufficient), or **operational gap** (it existed
   and was verified, but regressed).
3. Look for **root-cause clusters**: do several failures share one cause? (e.g. no trace
   infrastructure → C1, C2, C4, C5 all fail at tested level — one fix, not four.)
4. Cross-reference **production incident history**: are there recent failures that passed
   checklist items should have caught? If so, those items are insufficient (triggers DR18).
5. Flag **fatigue signals**: items passed with no evidence citation, evidence older than 90 days,
   evidence that describes design intent rather than operational state.

**Output:** Evidence Quality Assessment section.

---

## Stage G — Gate Decision

Produce the verdict.

1. **Per contract:** pass (all items pass) / conditional-pass (passes or tracked action items
   for failures) / fail (critical items fail with no mitigation).
2. **Overall:** pass (all selected contracts pass) / conditional-pass (≥1 conditional, none
   fail) / fail (≥1 contract fails).
3. For each conditional-pass or fail, record: the specific obligations not met, the downstream
   skill the finding escalates to, and the deadline or condition for resolution.
4. Include the **explicit scope limitation**: "This review covered contracts [list]. It did not
   verify [list]. This is a contract-compliance verification, not a system-performance
   evaluation, and it does not certify the tests it relies on were correct." This sentence is
   mandatory — it is what stops the checklist from being mistaken for system approval (F7) or
   for the tests themselves (F7).
5. State Version Metadata: contract versions used (v0.2 unless newer), date applied, reviewer,
   evidence-review date.

**Output:** Gate Decision + Version Metadata sections.

---

## Stage H — Iteration Trigger

Decide whether the *checklist* needs updating (it decays — L3 applies to artifacts too).

1. Compare item coverage against current contract versions — are obligations missing because
   items were derived from an older version?
2. Audit recent production failures against current items — would the checklist have caught
   them? If not, new items are needed (DR18).
3. Check item age — items unused 90+ days are flagged for staleness review (DR17).
4. Record any regeneration action items.

**Output:** Checklist Iteration Notes section.

---

## Stage I — Rotation & Cadence Update

Update the forward plan.

1. Recommend the next review date per contract.
2. If this review found multiple violations in a contract previously thought stable, **tighten**
   its cadence.
3. If a contract has had zero violations across 3+ consecutive reviews, consider **extending**
   cadence or reducing depth.
4. Record the rotation plan: which contracts at the next health check, and at what depth.
5. If this was a deployment gate, confirm C2 and C4 are correctly assigned the per-change gate
   model.

**Output:** Cadence & Rotation Plan section.

---

## When the user asks "how do we fix this?"

You will frequently finish a review and be asked to fix what you found. **Do not.** That is the
ownership boundary. State the finding, name the owner (*Design Templates* for design gaps,
*Implementation Playbooks* for implementation violations, *Evaluation Harnesses* for performance
questions, *Debugging Workflows* for diagnosing a detected failure), record it as a
conditional-pass with a tracked action item, and flag that it must be resolved before the next
review of that contract. You verify; you do not redesign. Crossing this line is Failure F4
(Scope Creep) and it quietly turns a falsifiable checklist into an unfalsifiable design opinion.
