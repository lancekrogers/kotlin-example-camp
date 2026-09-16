# Gate 07: Review Results and Iterate

## Changes

| File | Description |
|------|-------------|
| `src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt` | Added `deleteRaw` helper for raw DELETE responses (D008 unfavorite leak test). |
| `src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt` | Added `favorite response has no author secrets`, `unfavorite response has no author secrets`, and `favorite unknown slug returns 404` tests. |
| `src/test/kotlin/io/realworld/app/web/controllers/PopularArticlesTest.kt` | Added `bad offset` test for `offset=-1` → 422. |
| `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` | Batched `Follows` lookup in `toArticles` (one query per page instead of per row). |
| `src/test/kotlin/io/realworld/app/domain/repository/ArticleFollowingMappingTest.kt` | New repository test inserting a `Follows` row directly and asserting `following` per viewer. |

### git diff --stat

```
 .../app/domain/repository/ArticleRepository.kt     |  7 ++-
 .../app/web/controllers/ArticleControllerTest.kt     | 56 ++++++++++++++++++++++
 .../app/web/controllers/PopularArticlesTest.kt     |  7 +++
 .../kotlin/io/realworld/app/web/util/HttpUtil.kt   |  3 ++
 4 files changed, 71 insertions(+), 2 deletions(-)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/PopularArticlesTest.kt
 M src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt
?? src/test/kotlin/io/realworld/app/domain/repository/ArticleFollowingMappingTest.kt
```

## Commands

### `just test only ArticleControllerTest`

Exit code: **0**

```
ArticleControllerTest > unfavorite response has no author secrets PASSED
ArticleControllerTest > favorite article by slug PASSED
ArticleControllerTest > favorite response has no author secrets PASSED
ArticleControllerTest > create article PASSED
ArticleControllerTest > unfavorite article by slug PASSED
ArticleControllerTest > favorite unknown slug returns 404 PASSED
(+ 11 SKIPPED)
BUILD SUCCESSFUL
EXIT_CODE: 0
```

Full output: `results/07_article_controller_tests.txt`

### `just test only PopularArticlesTest`

Exit code: **0**

```
PopularArticlesTest > bad offset PASSED
(+ 10 other tests PASSED)
BUILD SUCCESSFUL
EXIT_CODE: 0
```

Full output: `results/07_popular_tests.txt`

### `just test only ArticleFollowingMappingTest`

Exit code: **0**

```
ArticleFollowingMappingTest > following depends on viewer PASSED
BUILD SUCCESSFUL
EXIT_CODE: 0
```

Full output: `results/07_following_test.txt`

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 49s
EXIT_CODE: 0
```

Full output: `results/07_test_all.txt`

### `just test census`

Exit code: **0**

```
  ArticleControllerTest      ran=6   passed=6   failed=0   skipped=11
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ArticleFollowingMappingTest ran=1   passed=1   failed=0   skipped=0

  TOTAL ran=83 passed=83 failed=0 skipped=17

  WARNING: 17 test(s) skipped. A green build does not mean the application works.
EXIT_CODE: 0
```

Full output: `results/07_census.txt`

XML totals from `build/test-results/test/*.xml`: 100 tests defined, 83 ran, 83 passed, 0 failed, 17 skipped.

## Done When

### Findings addressed

| # | Finding | Change | Evidence |
|---|---------|--------|----------|
| 1 | No raw-JSON leak test on favorite/unfavorite responses (D008) | Added `favorite response has no author secrets` and `unfavorite response has no author secrets` in `ArticleControllerTest`; added `deleteRaw` to `HttpUtil` | `07_article_controller_tests.txt`: both tests PASSED; `assertNoAuthorSecrets` called on raw bodies |
| 2 | No HTTP test for favorite on unknown slug → 404 | Added `favorite unknown slug returns 404` in `ArticleControllerTest` | `07_article_controller_tests.txt`: `favorite unknown slug returns 404 PASSED`; body contains `Article not found.` |
| 3 | No bad-`offset` case on popular | Added `bad offset` in `PopularArticlesTest` | `07_popular_tests.txt`: `bad offset PASSED`; body contains `offset must not be negative.` |
| 4 | `following` was one `Follows` query per row in `toArticles` | Batched lookup: collect author ids, single `Follows` query when viewer present | `07_test_all.txt`: BUILD SUCCESSFUL; finding 5 proves correctness |
| 5 | Nothing proved `following` at all | New `ArticleFollowingMappingTest.following depends on viewer` inserts `Follows` via Exposed | `07_following_test.txt`: `following depends on viewer PASSED` |

### Definition of Done (gate 07)

- [x] **All critical findings are fixed** — pass. All five findings from `05_testing.md` and `06_review.md` addressed (S4/S5 explicitly deferred in review, not in scope).
- [x] **`just test all` passes after the changes** — pass. `results/07_test_all.txt` `BUILD SUCCESSFUL`, `EXIT_CODE: 0`; census `TOTAL ran=83 passed=83 failed=0`.
- [x] **Code review findings are addressed or explicitly deferred with a reason** — pass. S1–S3 fixed in this gate; S4/S5 deferred in gate 06 with documented reasons (unchanged).
- [x] **Ready to commit** — pass. All targeted and full-suite tests green; no production changes beyond batched `Follows` lookup.

## Notes

- **Anchor drift:** None. `ArticleRepository.toArticles` batching applied at lines 151–160 (was 154–157 per-row query).
- **HttpUtil:** `deleteRaw` added alongside `getRaw`/`postRaw`; existing `deleteWithResponseBody` unchanged for typed responses.
- **Test counts:** Census went from 78 ran (gate 05) to 83 ran (+5 new tests: 3 ArticleControllerTest, 1 PopularArticlesTest, 1 ArticleFollowingMappingTest). Skipped count unchanged at 17.
- **D009:** All new tests use UUID-suffixed users, emails, and slugs; assert only on rows they create.
- **S4/S5 deferrals:** Unchanged from gate 06 — author's favorite tests keep original assertions; `Math.toIntExact` deferral for `articlesCount` above `Int.MAX_VALUE` remains documented in `03_article_search/results/07_review.md`.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `57a2db56-bb02-465c-8247-1128c764ed6a`, 09:01:18Z to 09:05:21Z).
Verified in deliberate order: the new guard test first, then the refactor it protects.

**1. The `following` guard is real.** `ArticleFollowingMappingTest.following depends on viewer` inserts a `Follows` row
directly (`user` = the author, `follower` = the follower's id), then asserts three distinct outcomes from that single
row: `following` **true** for the follower, **false** for an unrelated user, **false** for a null viewer. A refactor
that inverted the orientation, dropped the viewer filter, or always returned false would fail at least one of the three.

**2. The batched lookup preserves semantics.** `toArticles` now runs one query per page:

```kotlin
val followedAuthorIds = if (viewerId == null) emptySet() else
    Follows.select { (Follows.user inList authorIds) and (Follows.follower eq viewerId.value) }
        .map { it[Follows.user] }.toSet()
...
val following = authorId in followedAuthorIds
```

`Follows.user` remains the followed account and `Follows.follower` the viewer, matching `UserRepository.follow()`. An
anonymous viewer short-circuits to an empty set with no query at all. This closes the bounded N+1 the review raised and
retires the earlier deferral from the foundation slice, where the inline query had been kept for efficiency over
`findIsFollowUser`: one batched query is better than either.

**3. The author's tests were not touched.** The `ArticleControllerTest` diff contains only two new imports and three new
test methods; `favorite article by slug` and `unfavorite article by slug` keep every assertion.

**4. All five findings are closed**, each proven by a test that executes:

| Finding | Evidence |
|---|---|
| No leak test on favorite | `ArticleControllerTest.favorite response has no author secrets` — ran |
| No leak test on unfavorite | `ArticleControllerTest.unfavorite response has no author secrets` — ran |
| No HTTP 404 for an unknown slug | `ArticleControllerTest.favorite unknown slug returns 404` — ran |
| No bad-`offset` case on popular | `PopularArticlesTest.bad offset` — ran |
| `following` unbatched and untested | batched above; `ArticleFollowingMappingTest` — ran |

`HttpUtil` gained a `deleteRaw` helper alongside `getRaw`/`postRaw`, needed because typed deserialization cannot read a
raw favorite response body.

**Independent run with the cache off:**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0    BUILD SUCCESSFUL in 48s
  ArticleFollowingMappingTest: tests=1 ran=1 failed=0
  ArticleControllerTest: tests=17 ran=6 failed=0
      favorite article by slug, unfavorite article by slug, create article,
      favorite response has no author secrets, unfavorite response has no author secrets,
      favorite unknown slug returns 404
  PopularArticlesTest: tests=11 ran=11 failed=0
  ArticleFavoritesRepositoryTest: tests=5 ran=5 failed=0
  TOTAL tests=100 ran=83 failed=0 skipped=17
```

78 → 83 running, skips unchanged at 17, reported counts matching. Every finding from gates 05 and 06 is now closed or
deferred with a recorded reason.
