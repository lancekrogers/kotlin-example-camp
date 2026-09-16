# Gate 08 results: Review Results and Iterate

## Changes

- `src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt` — tightened assertions per review findings S1–S4 and task-05 observations (dead code removal, exact counts, paging error bodies, underscore total, anonymous count).
- `src/test/kotlin/io/realworld/app/domain/PagingTest.kt` — added `parse_rejectsNonIntegerOffset` for S2 (non-numeric offset at unit layer).

### git diff --stat

```
 .../kotlin/io/realworld/app/domain/PagingTest.kt   |  8 +++++++
 .../app/web/controllers/ArticleSearchTest.kt       | 26 +++++++++++++---------
 2 files changed, 23 insertions(+), 11 deletions(-)
```

### git status --short --untracked-files=all

```
 M src/test/kotlin/io/realworld/app/domain/PagingTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt
```

## Commands

### `just test only ArticleSearchTest`

Exit code: **0**

```
> Task :test

ArticleSearchTest > no author secrets PASSED

ArticleSearchTest > bad paging PASSED

ArticleSearchTest > no match PASSED

ArticleSearchTest > case insensitive PASSED

ArticleSearchTest > anonymous request is public PASSED

ArticleSearchTest > blank q PASSED

ArticleSearchTest > limit below total PASSED

ArticleSearchTest > missing q PASSED

ArticleSearchTest > title only match PASSED

ArticleSearchTest > body only match PASSED

ArticleSearchTest > percent and underscore are literal PASSED

BUILD SUCCESSFUL in 16s
exit_code=0
```

XML: `build/test-results/test/TEST-io.realworld.app.web.controllers.ArticleSearchTest.xml` — `tests="11" skipped="0" failures="0" errors="0"`.

Full output: `results/08_search_tests.txt`

### `just test only PagingTest`

Exit code: **0**

```
> Task :test

PagingTest > parse_rejectsNonIntegerOffset PASSED

PagingTest > parse_rejectsNonIntegerLimit PASSED

PagingTest > parse_rejectsNegativeOffset PASSED

PagingTest > parse_acceptsLimitOneHundred PASSED

PagingTest > parse_acceptsLimitOne PASSED

PagingTest > parse_rejectsLimitZero PASSED

PagingTest > parse_rejectsLimitAboveMax PASSED

PagingTest > parse_defaultsToTwentyAndZero PASSED

BUILD SUCCESSFUL in 2s
exit_code=0
```

XML: `build/test-results/test/TEST-io.realworld.app.domain.PagingTest.xml` — `tests="8" skipped="0" failures="0" errors="0"`.

Full output: `results/08_paging_tests.txt`

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 29s
exit_code=0
```

Full output: `results/08_test_all.txt` (61 test methods PASSED, 19 SKIPPED in Gradle log).

### `just test census`

Exit code: **0**

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=61 passed=61 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
exit_code=0
```

Full output: `results/08_census.txt`

## Done When

### Finding-by-finding (accepted S1–S4 + task-05 observations)

| Finding | Change | Evidence |
|---|---|---|
| Task 05 obs. 1 — dead `UUID.randomUUID()` in `blank q` and `missing q` | Removed discarded expression and comment from both tests | `blank q PASSED`, `missing q PASSED` in `08_search_tests.txt`; ArticleSearchTest still `tests="11"` |
| S1 — `body only match` used `>= 1` / `any` | Assert `articlesCount == 1`, `articles.size == 1`, body contains token | `body only match PASSED` in `08_search_tests.txt` |
| S1 — `case insensitive` used `>= 1` / `any` | Assert `articlesCount == 1`, `articles.size == 1`, title match | `case insensitive PASSED` in `08_search_tests.txt` |
| Task 05 obs. 3 — underscore case lacked total assertion | Added `articlesCount == 0` alongside existing title-absence check | `percent and underscore are literal PASSED` in `08_search_tests.txt` |
| S3 — `bad paging` asserted status only | Each of five cases asserts raw body contains expected message via `getRaw` | `bad paging PASSED` in `08_search_tests.txt` |
| S2 — no non-numeric `offset` coverage | HTTP: added `offset=abc` case; unit: `parse_rejectsNonIntegerOffset` | `bad paging PASSED`; `parse_rejectsNonIntegerOffset PASSED`; PagingTest `tests="8"` |
| S4 — `anonymous request is public` could pass with empty list | Added `articlesCount == 1` | `anonymous request is public PASSED` in `08_search_tests.txt` |
| S5 — `Math.toIntExact` overflow → 500 | **Deferred** per `07_review.md` (documented, not changed) | No code change; recorded in Notes |

### Gate Definition of Done (from `08_iterate.md`)

- [x] **All critical findings are fixed** — pass: no critical findings in `07_review.md`; accepted suggestions S1–S4 addressed in test code.
- [x] **`just test all` passes after the changes** — pass: exit 0, `TOTAL ran=61 passed=61 failed=0 skipped=19` in `08_census.txt`.
- [x] **Code review findings are addressed or explicitly deferred with a reason** — pass: S1–S4 fixed; S5 deferred with reason in `07_review.md` § Suggestions.
- [x] **Ready to commit** — pass: only test files changed, full suite green, census matches expected counts (PagingTest 7→8, total 60→61, skipped still 19).

## Notes

- **Anchor drift:** none. All cited test methods and `Paging.parse` messages matched the task description and `Paging.kt:10-14`.
- **S5 deferred:** `Math.toIntExact(page.total)` overflow remains unchanged in production; reviewer documented why in `07_review.md` (Int field constraint, unreachable in H2, clamping would lie).
- **Test counts:** PagingTest increased from 7 to 8 (`parse_rejectsNonIntegerOffset`); suite total from 60 to 61 ran; skipped unchanged at 19 per `just test census`.
- **No production files touched:** `Paging.kt`, `ArticleService.kt`, `ArticleRepository.kt`, `Router.kt` unchanged per task scope.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `dbcebb69-0000-4c38-901b-3f9b3f1934f7`, 08:27:08Z to 08:29:40Z).

**Scope: exactly two test files, no production code.** `git status --porcelain | grep 'src/main/'` returns nothing, so
every finding was addressed as a test-strength change, as intended.

**All seven fixes are present in the diff, and three landed stronger than requested:**

| Finding | Change | Note |
|---|---|---|
| Dead statements | The discarded `UUID.randomUUID().toString().take(8)` lines are gone from `blank q` and `missing q` | |
| `body only match` loose | `assertEquals(1, articlesCount)`, `assertEquals(1, articles.size)` | also swapped `any { }` for `all { }`, which is stricter than asked |
| `case insensitive` loose | same | same |
| Underscore total | `assertEquals(0, underscoreResponse.body.articlesCount)` added, keeping the second-title check | |
| 422 bodies unchecked | `bad paging` now pairs each case with its expected message and asserts `response.body.contains(expectedMessage)` | 4 cases became 5 |
| `offset=abc` uncovered | added to `bad paging`, expecting `"offset must be an integer."` | |
| `anonymous request is public` could pass empty | `assertEquals(1, response.body.articlesCount)` added | closes the hole the reviewer found |
| Non-numeric offset unit case | `PagingTest.parse_rejectsNonIntegerOffset` asserts the exact message | |

**Independent run with the cache off:**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0    BUILD SUCCESSFUL in 29s
  PagingTest: tests=8 failed=0
      parse_rejectsNonIntegerOffset, parse_rejectsNonIntegerLimit, parse_rejectsNegativeOffset,
      parse_acceptsLimitOneHundred, parse_acceptsLimitOne, parse_rejectsLimitZero,
      parse_rejectsLimitAboveMax, parse_defaultsToTwentyAndZero
  ArticleSearchTest: tests=11 failed=0
  TOTAL tests=80 ran=61 failed=0 skipped=19
```

`PagingTest` 7 → 8 and the suite 60 → 61 ran, with skips unchanged at 19, exactly as predicted. Reported counts match.

**No assertion was weakened.** Every edit either replaced an inequality with an equality, replaced `any` with `all`, or
added an assertion. The deferred item (S5, `Math.toIntExact` overflow) is untouched, as recorded in
`results/07_review.md`.

All findings from gates 06 and 07 and from the task verifications are now closed or deferred with a reason.
