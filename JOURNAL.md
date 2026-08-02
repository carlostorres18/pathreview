## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/157

**Issue title:** Relevance scorer “partial overlap” test fixture actually has full query overlap

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The relevance scorer test suite has a test named `test_query_with_partial_overlap` that is supposed to validate how the scorer handles queries where only some terms appear in a chunk, but the test fixture actually includes all of the query terms ("Python", "Django", "web", "framework") inside the chunk text. Because the scorer correctly returns 1.0 for full keyword coverage, the assertion `score < 0.9` fails even though the scorer itself is working correctly. This is a test-quality bug in the backend relevance scoring tests: the fixture description says "partial overlap" but the data represents full overlap. A successful fix would update the chunk text so that it genuinely omits one or more query terms, making the test actually exercise the partial-match code path and allowing the assertion to pass against correct scorer behavior.

**Issue-selection Reasoning:**
- **Understanding:** I can describe the problem and the correct behavior without looking at the issue again. The test fixture uses a chunk that contains every term from the query, so the scorer returns `1.0` even though the test is designed to measure partial overlap.
- **Affected code:** I found and read `test_query_with_partial_overlap` in `tests/unit/test_relevance_scorer.py` and traced the scoring logic through the `score` and `_tokenize` methods in `rag/evaluator/relevance_scorer.py`.
- **Definition of done:** Currently, all query terms are present in the chunk and the scorer returns `1.0`, which causes the `score < 0.9` assertion to fail. After the fix, the chunk should contain only a subset of the query terms, resulting in a score in the `0.3`–`0.9` range and a passing test.
- **Tier fit:** This is a Tier 1 issue and is a good first contribution because the fix is confined to a single test fixture and requires no changes to production code or other modules.
- **Codebase readiness:** I read the full test file and the scorer implementation to understand how keyword overlap is computed before selecting this issue.
- **Testing:** I ran `pytest tests/unit/test_relevance_scorer.py -q` and confirmed the reported failure: 1 failed, 18 passed, with the failing assertion showing the actual score was `1.0`.
- **Rough plan:** I will revise the test fixture so only some of the query terms appear in the chunk, rerun the unit tests to confirm they pass, and then verify that no changes are needed in the scorer itself.
- **Claims:** I reviewed the issue activity and the cohort ledger and understand that claims are non-exclusive. I am fine working on this issue alongside other contributors.
- **Time and scope:** The change is small and self-contained, so it fits comfortably within the Week 9 deadline.
- **Dependencies:** I did not find any open blockers or prerequisite issues that need to be resolved before this fix can be made.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/carlostorres18/pathreview/commit/c2369955b4fcf4d446d76cc4792006232e00ef04

**Reproduction summary:**
Ran `.venv/bin/pytest tests/unit/test_relevance_scorer.py -q` and observed `1 failed, 18 passed`. The failure is `test_query_with_partial_overlap`, which asserts `0.3 < score < 0.9` but got `score == 1.0` — the logged output shows `avg_score=1.0` because the chunk fixture text contains every token from the query, so the scorer correctly reports full overlap instead of the partial overlap the test name implies.

**PLAN.md link:** [PLAN.md](./PLAN.md) (this repo, root of branch `test/157-relevance-scorer-test-overlap`)

**Walkthrough video (recommended):** [Issue #157 Loom Video](https://www.loom.com/share/5d978a143ea84e559868a6b9d4fa0d74)

**Blockers or open questions:**
None currently — the fix is confined to rewriting one chunk-text fixture. The only thing to double-check while implementing is that the replacement text lands the overlap ratio inside `(0.3, 0.9)` and doesn't accidentally hit the zero-overlap or full-overlap paths instead, since `RelevanceScorer._tokenize` does a naive `.lower().split()` with no punctuation stripping or stemming.

**Branch name:** test/157-relevance-scorer-test-overlap

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the PLAN.md fix: rewrote the chunk text in `test_query_with_partial_overlap` (`tests/unit/test_relevance_scorer.py`) so it genuinely omits two of the four query tokens (`python`, `framework`), producing a partial-overlap score of 0.5 instead of the full-overlap 1.0 that was causing the failure. Steps 1–3 from PLAN.md are done: fixture rewritten, overlap ratio hand-verified before running, and the targeted test confirmed passing (`test_query_with_partial_overlap` PASSED, all 19 tests in the file pass).

**Next steps:**
Run the full unit suite and `make check` to confirm no regressions elsewhere, then commit and open the PR (PLAN.md steps 4–5).

**Blockers:**
None on the fix itself. Discovered along the way that `make test-unit`, `make lint`, and `make typecheck` all have pre-existing, unrelated failures repo-wide (52 failing unit tests, 182 ruff errors, 5 mypy errors from missing stubs and a numpy/mypy version mismatch) — confirmed via `git stash` that these predate this branch and aren't caused by this change, so they're out of scope for #157.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/557

**Branch:** `test/157-relevance-scorer-test-overlap`

**What you built:**
Fixed the fixture bug in `test_query_with_partial_overlap`: the chunk text previously contained every query token, so the scorer correctly returned a full-overlap score of 1.0 and the `0.3 < score < 0.9` assertion failed. The chunk text now genuinely omits some query terms, so the test exercises the intended partial-match path (score = 0.5). No production code in `rag/evaluator/relevance_scorer.py` changed — the scorer's logic was already correct.

**Tests added or updated:**
Only `tests/unit/test_relevance_scorer.py` — updated the `test_query_with_partial_overlap` fixture text and comment, and added missing type annotations across all 19 test methods (plus one `chunks: list[dict]` var annotation) so the file satisfies the mypy pre-commit hook. No new test files were added; all 19 existing tests in this file cover the scorer's behavior and now pass.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes
> Both commands exit with errors when run repo-wide, but neither failure is related to this change: `make test-unit` goes from 53 failed/375 passed (pre-fix) to 52 failed/376 passed (post-fix) — exactly the targeted test flipped, nothing else changed. `make check` fails on 182 pre-existing ruff errors and 5 pre-existing mypy errors elsewhere in the repo; `ruff check tests/unit/test_relevance_scorer.py` and the mypy pre-commit hook both pass cleanly on the file I touched.

**Draft PR feedback received from:** none