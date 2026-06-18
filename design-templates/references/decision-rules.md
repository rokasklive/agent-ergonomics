# Decision Rules (DR1–DR16)

> Operational gates, not suggestions. Each turns a design habit into a yes/no check at a specific
> stage of the procedure, and each names the failure mode (or law) it prevents — **lead with that
> reasoning when you apply it, don't just cite the number.**
>
> The decision points behind them: **D1** granularity (single vs. modular vs. comprehensive),
> **D2** evidence display (inline vs. appendix), **D3** strictness (mandatory / evidence-required /
> advisory). The failure modes they guard: **F1** Template Bloat, **F2** Template Ritualism, **F3**
> Missing Dimension Syndrome, **F4** Template–Checklist Divergence, **F5** Premature Formalization,
> **F6** Confusion with Implementation.

## Granularity (D1) — single, modular, or comprehensive

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR1** | Template spans ≥2 contracts with documented cross-dimension interactions → start with **modular templates (one per contract) + a mandatory cross-dimension interaction checklist**. Prevents monolith bloat while still forcing the interactions. | ≥2 contracts with known interactions (e.g. C3 + C8). | F1 |
| **DR2** | Greenfield system AND coordination is **mesh or closed-loop** → use a **single comprehensive document**. Modular templates under-address the interaction effects that dominate at scale here. | New-from-scratch design AND mesh/closed-loop topology. | F3, L4 |
| **DR3** | Incremental improvement of an existing system → **modular templates with progressive adoption** (pick the most critical dimension first). Comprehensive templates are too heavy to adopt incrementally. | Brownfield improvement; gradual adoption required. | F1 |

## Evidence display (D2) — inline with the decision, or in an appendix

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR4** | Adversarial reviewers OR decisions accumulated over weeks → display evidence tier and reversibility **inline with each decision field**. Appendix-only display lets evidence be fabricated retrospectively. | Audience challenges evidence quality, or decisions span time. | F2 |
| **DR5** | Audience only needs the outcome and reviews evidence separately → consolidate evidence in a **decision-register appendix**. Inline clutters the doc for readers who don't need it. | Executives / downstream implementers who trust a separate evidence review. | L1 |
| **DR6** | Any decision backed by **Tier 3 (practitioner) or Tier 4 (opinion)** evidence → evidence display **MUST be inline**. Weak evidence kept far from its decision creates false confidence; proximity forces scrutiny. | T3/T4 evidence behind an architectural decision. | F2 |

## Strictness (D3) — mandatory, evidence-required, or advisory

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR7** | Field maps to a **critical** contract obligation (C3 dimensions, C8 boundaries) → **mandatory, no skip path**. Skipping produces a design that fails at the missing dimension. | Contract-obligation field, criticality "critical". | F3 |
| **DR8** | Field maps to an obligation demanding empirical data (IV-2 scale tests, calibration) → **evidence-required**: must hold a specific measurement, or be explicitly flagged unfilled with a warning. Reject "it should work / standard / TBD / future tense." | Field needs empirical data. | F2, L2 |
| **DR9** | Field derives solely from a pattern-seed recommendation, no contract/invariant overlap → **advisory**, with a one-line skip-justification format. Preserves the recommendation without bloating the mandatory set. | Pattern-seed-only field. | F1 |
| **DR10** | Field satisfies **both** a contract obligation and a pattern-seed precondition → **higher tier wins** (contract→mandatory beats seed→advisory). A field that does both stays mandatory. | Overlap found in Stage E. | F1, F3 |
| **DR11** | Designer expertise is **below the domain floor** (can't tell "architecture choice" from "empirical evidence") → add **guidance annotations** to every mandatory and evidence-required field. Templates prevent omission, not ignorance — guidance bridges the gap. | Input 8 = low/novice. | F2 |
| **DR12** | >30% of fields rest on **Medium-or-below** evidence → add per-field confidence caveats AND mark the template **[PRELIMINARY]**, not [PRODUCTION]. Don't amplify corpus speculation. | Input 5 shows thin evidence. | F5, L5 |

## Versioning (staleness & re-audit)

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR13** | Any encoded contract/invariant/pattern seed updated since the template was generated → mark **[STALE]** and block use until re-verified. An outdated template produces designs that satisfy old requirements and miss new ones. | Corpus artifact version > template's encoded version. | F3-variant, F4 |
| **DR14** | Template increment is **MAJOR** (mandatory fields changed) → existing designs produced under the prior version must be **re-audited** against the new template. A new mandatory field means old designs are missing a dimension. | MAJOR semantic version bump. | F3 |

## Field justification (the bidirectional audit)

| # | Rule | Trigger | Guards |
|---|------|---------|--------|
| **DR15** | Field can't be traced to a contract obligation, invariant, or pattern-seed precondition → **cut it or demote to optional appendix**. Every field must earn its place. | Stage F finds no corpus artifact behind a field. | F1 |
| **DR16** | A contract obligation has **no** corresponding field → **add one, or document why it isn't templatable** (process obligation, runtime check, owned by another skill). | Stage B finds an orphan obligation. | F3 |

---

**When a rule says "cut," "reject," or "block":** you're not refusing to help. You're declining to
add a field the corpus can't justify, refusing to let a load-bearing obligation fall off the page,
or refusing to launder speculation into mandatory structure. In every case: name the obligation (or
its absence), name the tier or status, and — for HOW content crossing the ownership boundary — name
the downstream skill that owns it and the placeholder assumption you'll use until it's verified.
