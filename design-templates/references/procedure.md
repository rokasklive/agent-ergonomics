# The Template Design Procedure — Stages A–I

> Nine stages, in order. Contracts before fields; fields before structure; structure before
> strictness; strictness before the traceability audit that keeps the template honest against its
> companion checklist; traceability before publication. Each stage names the output section it
> produces (see `output-template.md`).
>
> You won't always run all nine at full depth. A single-domain boundary-validation template is
> mostly A, B, C, F, G, I. A revision triggered by a checklist change is mostly F and H. Use
> judgment; the order is what matters when stages do run. The obligations you pull from in B, D,
> E, F all live in `corpus.md`.

---

## Stage A — Domain Selection & Contract Identification

Decide what the template covers and which obligations it must encode. **This is where you prevent
both Premature Formalization (F5) and Missing Dimensions (F3).**

1. Name the target artifact domain — coordination architecture, boundary map, tool-interface spec,
   evaluation-system design, progressive-disclosure spec, or reversibility analysis.
2. Identify the contracts whose obligations the template must encode (C3 coordination, C8 boundary,
   C6 progressive disclosure, C2 interface change, C5 evaluation) and the cross-cutting invariants
   (II-3 reversibility, III-1 context preservation, IV-2 scale projection).
3. **Readiness check (F5 gate):** confirm the domain has at least one contract at High confidence and
   a structural law/invariant behind it. If >30% of the fields would rest on Medium-or-below
   evidence, this is a [PRELIMINARY] template with per-field caveats (DR12) — or, if there is no
   solid contract at all, decline and say what evidence would change that.
4. Decide single-domain vs. multi-contract span — this triggers the granularity rules (DR1–DR3):
   modular templates with a cross-dimension interaction checklist for most multi-contract cases; a
   single comprehensive document only for greenfield mesh/closed-loop systems where interactions
   dominate.

**Output:** Domain Scope & Contract Mapping section — the obligation-to-field mapping table, plus an
explicit statement of what this template does NOT cover (ownership-boundary defense).

---

## Stage B — Contract Obligation Extraction

Mechanically pull every obligation; **start from the full list, not intuition** — this is the
defense against F3.

1. For each in-scope contract, pull every Required Check (RC#) from `corpus.md` as a candidate field.
2. Classify each candidate by field type: *design-time* (an architecture choice), *evidence-required*
   (a measurement / test result), or *structural* (an enumeration / identification).
3. Group candidates by template section (all seven C3 dimensions together; all C8 boundary obligations
   together).
4. Merge obligations that overlap across contracts into a single field; cite both contracts.
5. Flag obligations that can't be a design field — a *process* obligation (e.g. "traces reviewed at a
   regular cadence") maps to a checklist item, not a field. Record it as [NOT-TEMPLATABLE → checklist].
6. Count candidate fields; if >40, queue the field-elimination rules (DR15) for Stage C.

**Output:** Contract Obligation Extraction table — obligation, candidate field, field type, merge notes.

---

## Stage C — Field Mapping & Structure Design

Turn candidates into fields a designer can actually complete.

1. Map each obligation to one or more fields with explicit completion guidance.
2. Write field prompts that are (a) one sentence max, (b) cite the clause they encode, (c) carry an
   *example of a correct answer* when the domain is complex or designer expertise is below the floor
   (DR11). The example is what makes a field ritualism-resistant (L2).
3. Group fields into sections matching the contract structure (C3 → seven dimension sub-sections; C8 →
   boundary enumeration + per-boundary details).
4. Choose each field's format: free text (design-time decisions), structured choice (topology /
   pattern selection), or evidence input with required source citation (evidence-required fields).
5. Mark the **Minimum Viable Section** — the core obligation fields to complete before any optional
   ones (L1).
6. **Field-elimination pass:** remove any candidate that can't cite a corpus obligation; if total still
   exceeds ~40, demote the weakest advisory fields to an optional appendix (DR15, F1).

**Output:** Draft template structure — field prompts, section organization, format specs, MVP marker.

---

## Stage D — Invariant Check Encoding

Add the cross-cutting fields the contracts alone don't cover.

1. For each relevant invariant, find the fields that already satisfy it (from B) or add new ones.
2. **II-3 Reversibility:** add evidence tier (T1–T4), reversal cost, optionality assessment, and
   trigger-condition fields to *every architectural-decision* section — inline, not an appendix (DR4).
3. **III-1 Context Preservation:** add progressive-disclosure fields (default output, verbose flag,
   escape hatch, error structure) to *every interface* section.
4. **IV-2 Scale Projection:** add current/2×/5× counts, inflection points, and scale-test fields
   (evidence-required) to *every architecture* template.
5. Where an existing contract field already satisfies an invariant, annotate it ("satisfies II-3 by
   capturing evidence tier") rather than duplicating.

**Output:** Invariant coverage map — which field satisfies each invariant check, plus new cross-cutting
fields.

---

## Stage E — Pattern-Seed Integration

Add the reusable advisory moves — sparingly, to avoid bloat.

1. Identify relevant seeds (PS-3 boundary maps, PS-6 coordination, PS-12 progressive disclosure, PS-15
   reversibility) using each seed's *when-to-use / when-NOT* in `corpus.md`.
2. Add the fields each seed operationalizes. Mark them **advisory by default** (DR9).
3. Where a seed field overlaps a contract obligation, it's already mandatory — keep the higher tier
   (DR10), don't duplicate.
4. Annotate each advisory field with the seed it encodes.

**Output:** Pattern-seed integration table — seed, field additions, strictness tier, overlap notes.

---

## Stage F — Cross-Reference Tagging

Tag every field so the template's completeness is verifiable — and detect divergence from the checklist.

1. Tag each field with its contract clause (e.g. "C3.d"), invariant (e.g. "II-3"), and pattern seed
   (e.g. "PS-15") references.
2. For each field, name the [[Review Checklists]] item that will verify the resulting design satisfies
   it. No such item → flag **[UNVERIFIABLE]**.
3. For each checklist item in the domain, confirm at least one field ensures it's addressed. No field →
   flag **[UNTEMPLATED CHECK]**.
4. Compute bidirectional coverage: (fields with checklist items)/(total fields) and (checklist items
   with fields)/(total items). Target ≥0.9 both ways; below 0.7 is divergence (F4).

**Output:** Cross-Reference Matrix — field→contract, field→invariant, field→checklist; coverage scores;
gap flags.

---

## Stage G — Strictness Assignment

Give every field exactly one tier — this is what makes the template both complete and lean.

1. Classify each field: **Mandatory** (load-bearing contract/invariant obligation; no skip path —
   DR7), **Evidence-Required** (demands empirical data; reject "standard/TBD/it should work" —
   DR8), or **Advisory** (pattern-seed recommendation; skippable with one-line justification — DR9).
2. For evidence-required fields, state what counts as valid evidence: a specific measurement, a source
   citation, or a test result *with parameters*. Provide the rejection list explicitly.
3. Apply the edge-case rules: cross-contract overlap → higher tier wins (DR10); designer below floor →
   guidance annotations on all mandatory/evidence fields (DR11); thin-evidence domain → caveats +
   [PRELIMINARY] (DR12).

**Output:** Strictness Tier Assignment table — field, tier, evidence-validity criteria, skip format.

---

## Stage H — Checklist Traceability Verification

Make the template-checklist link durable, not a one-time snapshot — this is the F4 defense.

1. For each mandatory and evidence-required field, confirm the corresponding checklist item exists and
   is *current* (version-check the checklist).
2. Resolve every [UNVERIFIABLE] field from F: find an item, propose a placeholder item, or demote the
   field to advisory with a note that it can't be verified.
3. Resolve every [UNTEMPLATED CHECK] item from F: add a field, or document why it isn't templatable
   (process obligation, runtime check, owned by another skill).
4. Record the checklist version verified against. When the checklist increments, this template must be
   re-verified (DR13).

**Output:** Checklist Traceability verification report — resolution status for all gaps; checklist
version reference.

---

## Stage I — Versioning & Publication

Make the template safe to reuse and detect when it goes stale.

1. Assign a semantic version: MAJOR (mandatory fields changed — old designs must be re-audited, DR14),
   MINOR (new advisory fields, guidance), PATCH (typos, examples).
2. Record in the header: template version; corpus artifact versions encoded (laws/invariants/
   contracts/pattern-seeds v0.2); checklist version; generation date; staleness-check date.
3. Add the staleness rule: "Current as of corpus v0.2. If any encoded contract, invariant, or pattern
   seed has changed since [date], re-verify before use" (DR13).
4. Document scope (covers / does not cover), designer expertise floor, and expected completion time
   (≈2–4h coordination architecture, ≈1–2h tool spec).
5. Reference the downstream owners: design here, build → [[Implementation Playbooks]], verify →
   [[Review Checklists]], measure → [[Evaluation Harnesses]].

**Output:** Versioned, published template with full metadata header, staleness rule, and
cross-references.
