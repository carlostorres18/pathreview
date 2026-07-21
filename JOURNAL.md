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
- **Rough plan:** I will revise the test fixture so only some of the query terms appear in the chunk, rerun the unit tests to confirm they pass, and verify that no changes are needed in the scorer itself.
- **Claims:** I reviewed the issue activity and the cohort ledger and understand that claims are non-exclusive. I am fine working on this issue alongside other contributors.
- **Time and scope:** The change is small and self-contained, so it fits comfortably within the Week 9 deadline.
- **Dependencies:** I did not find any open blockers or prerequisite issues that need to be resolved before this fix can be made.


**Branch name:** test/157/relevance-scorer-test-overlap

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger