# PathReview — Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/69

**Issue title:** Add a "feedback tone check" that ensures all generated feedback is written constructively

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
Right now the review generator produces feedback without any check on how that
feedback is phrased, so a section can come out vague, dismissive, or
discouraging and still reach the user. This issue asks for a tone-classification
step that runs after generation: a prompt classifies each feedback section as
constructive (actionable, specific, encouraging) or negative (discouraging,
vague, dismissive). Sections that fail are rejected and regenerated. A
successful fix guarantees every delivered feedback section reads
constructively. It touches the safety layer (`safety/content_filter.py`) and
the generation path (`rag/generator/review_generator.py`), so it requires
understanding how those two modules connect.

**Branch name:** feat/69-feedback-tone-check

**Setup confirmation:** [ ] App runs locally at localhost:5173

**"Is this right for me?" checklist / scope reasoning:**
Tier 2, estimated 5–8 hours. Scope is bounded to two named files plus a new
classifier prompt, no schema or infra changes. Cross-module (safety + rag) but
each module is understandable on its own. Fits within the Weeks 8–9 build
window. Main risk: the regenerate loop needs a retry cap to avoid infinite
loops — noted for the implementation phase.

**Cohort ledger:** [ ] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [paste the pushed commit URL here after `git push`]

**Reproduction summary:**
Traced the gap through the codebase: `core/services/review_service.py:365`
lists "Validate feedback tone and constructiveness" as a placeholder comment in
`_run_safety_checks` that is never implemented, `safety/content_filter.py` only
regex-matches a short list of harmful phrases (no constructive-vs-negative
classification), and `rag/generator/review_generator.py` has no
regenerate-on-fail path. So a dismissive or discouraging (but not "harmful")
feedback section is delivered untouched — confirming the missing step is real
and I know exactly where it lives.

**PLAN.md link:** [paste PLAN.md URL, e.g. .../tree/feat/69-feedback-tone-check/PLAN.md]

**Walkthrough video (recommended):** [optional Loom link, ≤2 min — not graded]

**Blockers or open questions:**
Need to confirm whether the tone classifier should reuse the existing
`openai.OpenAI` client/config from `ReviewGenerator` or get its own in the
safety layer. Otherwise plan is clear going into Week 9.
