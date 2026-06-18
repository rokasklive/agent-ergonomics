# Changelog — agent-ergonomics-inspector

## Multi-mode reviewer (Cursory / Full / Review Board)

### What changed

Introduced three **operating modes** on top of the existing single-mode reviewer,
without altering the underlying framework:

- **Cursory Review** — fast, low-token, high-signal triage. Condensed output:
  summary, condensed five-domain scorecard, top 3–5 findings, biggest risk,
  highest-ROI fix. No exhaustive coverage, no adversarial step.
- **Full Review** *(default)* — unchanged. The existing three-phase process and the
  existing required output format are exactly the Full Review, and remain the
  behavior when no mode is named.
- **Review Board** — adversarial, high-confidence mode. A Primary Inspector runs a
  Full Review; an Adversarial Reviewer tries to falsify the review itself; an
  Evidence Arbiter re-grounds against the target and issues a final report
  classifying each finding as Confirmed / Partially Confirmed / Rejected /
  Uncertain.

Specific edits:

- **`SKILL.md`**
  - Extended the frontmatter `description` with the three mode names and their
    invocation phrases (so "cursory", "review board", "adversarial review mode"
    route here). No change to the existing trigger language.
  - Added an **"Operating modes"** section (mode table, selection rules, invocation
    phrases) between the conceptual frame and the review process. Explicitly states
    Full Review is the default and the fallback for ambiguous requests.
  - Added a one-line pointer to `references/review-board.md` in the reference-files
    list, scoped to Review Board mode only.
  - Prefixed the existing "Required output format" with a sentence clarifying it is
    the Full Review (default) format. **The format body is unchanged.**
  - Added two additive output-format sections after it: **"Cursory Review output
    format"** and **"Review Board output format"**.
- **`references/review-board.md`** *(new)* — roles (Primary Inspector, Adversarial
  Reviewer, Evidence Arbiter), the adversarial falsification checklist, the
  arbitration rubric, single-context vs. subagent execution, and the cost/boundary
  tradeoff. Keeps the load-bearing output contract in `SKILL.md`; this file holds
  process detail only.
- **`evals/evals.json`** — added eval **id 3** (Cursory Review) and **id 4** (Review
  Board). Existing evals **id 0–2 are byte-for-byte unchanged.**

### Why it changed

To increase review flexibility and rigor — a cheap triage path and an expensive
high-confidence path — *without* fragmenting the framework into competing variants.
All three modes share the same five laws, five domains, scoring anchors, finding
structure, review philosophy, and causal-reasoning model. The difference is review
depth and review process, not review criteria.

Design grounding from the knowledge base:

- **Closed-loop > pipeline** (closed loops neutralize 40%+ of injected faults): the
  Review Board closes the loop a lone review leaves open — review → adversary →
  arbiter who re-reads the target.
- **Seek truth, not compromise** (integrative compromise underperforms the best
  member by 8–38%): the arbiter holds structural authority and rules on evidence
  rather than averaging the primary and adversarial positions.
- **Verification layers pay** (explicit self-review against criteria improves task
  success): the adversarial pass is that verification layer, aimed at the review.

The framework itself — the five laws, five domains, anti-pattern catalog, scorecard
methodology, finding structure, and review philosophy — was **not** redesigned.

### Compatibility impact on existing evals

- **Full Review evals (id 0–2) continue to pass unchanged.** The evals name no mode,
  the skill defaults to Full Review, and the Full Review process and output format
  are untouched. The only `SKILL.md` change touching the Full Review path is a
  single clarifying sentence above the (unchanged) output format.
- **No existing eval was rewritten or removed.** The two new evals are purely
  additive: id 3 validates the lighter Cursory behavior, id 4 validates the deeper
  Review Board behavior.
- **Risk note:** the new "Operating modes" section introduces mode-selection
  language. To protect eval compatibility it explicitly designates Full Review as
  both the default and the ambiguity fallback, and instructs the skill not to ask
  the user to choose a mode unprompted — so a mode-less prompt deterministically
  produces the same Full Review as before.
