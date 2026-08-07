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

**Setup confirmation:** [x] App runs locally at localhost:5173

**"Is this right for me?" checklist / scope reasoning:**
Tier 2, estimated 5–8 hours. Scope is bounded to two named files plus a new
classifier prompt, no schema or infra changes. Cross-module (safety + rag) but
each module is understandable on its own. Fits within the Weeks 8–9 build
window. Main risk: the regenerate loop needs a retry cap to avoid infinite
loops — noted for the implementation phase.

**Cohort ledger:** [ ] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Jordy-03/pathreview/commit/80b57b79ba7129906028d3fbfd1d50d93aae7dbb

**Reproduction summary:**
Traced the gap through the codebase: `core/services/review_service.py:365`
lists "Validate feedback tone and constructiveness" as a placeholder comment in
`_run_safety_checks` that is never implemented, `safety/content_filter.py` only
regex-matches a short list of harmful phrases (no constructive-vs-negative
classification), and `rag/generator/review_generator.py` has no
regenerate-on-fail path. So a dismissive or discouraging (but not "harmful")
feedback section is delivered untouched — confirming the missing step is real
and I know exactly where it lives.

**PLAN.md link:** https://github.com/Jordy-03/pathreview/blob/feat/69-feedback-tone-check/PLAN.md

**Blockers or open questions:**
Need to confirm whether the tone classifier should reuse the existing
`openai.OpenAI` client/config from `ReviewGenerator` or get its own in the
safety layer. Otherwise plan is clear going into Week 9.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the core of the fix from PLAN.md. Done so far:
- Sub-task 1 — added `TONE_CHECK_TEMPLATE` + `get_tone_check_prompt()` and a new
  `ToneClassifier` in `safety/content_filter.py` that returns
  `(is_constructive, reason)` from an LLM call.
- Sub-task 2 — wired the reject-and-regenerate loop into
  `rag/generator/review_generator.py` (`generate_section` now regenerates
  non-constructive sections up to `max_tone_retries`), extracting `_generate_once()`.
- Sub-task 4 — added structured logging (`tone_check_failed`, `tone_regenerated_ok`,
  `tone_retries_exhausted`).

**Next steps:**
Finish sub-task 5 (unit tests for the classifier and the loop), run the full
`make check` / `make test-unit` against the recorded baseline, and open a draft PR
for peer feedback.

**Blockers:**
Resolved the Week 8 open question — the classifier reuses the generator's existing
OpenAI client (injected in `ReviewGenerator.__init__`), which keeps it testable.
No other blockers.

---

### Check-in 2 (end of week)

**PR link:** <!-- PASTE the PR URL here after opening it: https://github.com/ascherj/pathreview/pull/NNN -->

**Branch:** `feat/69-feedback-tone-check`

**What you built:**
An LLM-based tone check for generated portfolio feedback. After each section is
generated, a `ToneClassifier` labels it constructive or non-constructive; the
generator rejects and regenerates non-constructive sections (with corrective
guidance) up to a bounded retry cap, then delivers the best attempt. The check
fails open on errors so it can never block or crash a review.

**Tests added or updated:**
- `tests/unit/test_content_filter.py` (new) — covers `ToneClassifier`: constructive
  feedback passes, negative feedback is flagged, the JSON verdict is extracted even
  when wrapped in prose, and all three fail-open paths (empty text, unparseable
  response, LLM error) default to constructive without blocking. Also keeps two
  regression tests for the existing `ContentFilter`.
- `tests/unit/test_review_generator.py` (new) — covers the regenerate loop: no
  regeneration when the first draft is constructive, exactly one regeneration when
  the first draft fails then passes, best-effort delivery when all retries are
  exhausted, and that disabling `tone_check_enabled` skips the classifier entirely.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
<!-- "passes" per Module 3 guidance = my changes introduce NO NEW failures.
     Baseline (branch base): 53 failing unit tests + pre-existing lint/type errors.
     After my changes: identical 53-failure set (verified by sorted diff of FAILED
     lines); passing tests 375 -> 387 (+12 new). My changed files add no new lint
     errors. See PR "Testing" section for the documented pre-existing failures. -->

**Draft PR feedback received from:** none yet (draft PR to be posted in the cohort
Slack channel for peer review)
