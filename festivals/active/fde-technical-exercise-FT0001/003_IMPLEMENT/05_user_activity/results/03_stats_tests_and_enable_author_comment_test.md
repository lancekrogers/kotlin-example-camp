# Results: 03_stats_tests_and_enable_author_comment_test

## Changes

- `src/test/kotlin/io/realworld/app/web/controllers/ProfileStatsTest.kt` — New HTTP tests for D006 stats: zero activity, 1/1/1 after actions, favorites-given semantics (V=1, W=0), unknown username 404, anonymous 200.
- `src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt` — New comment creation error-path tests (422 blank body, 404 unknown slug, 401 no token) plus D008 raw-JSON leak check.
- `src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt` — Removed class-level `@Ignore`; method-level ignores on list/delete stubs; `/api` paths removed; enabled `add comment for article by slug` with UUID-suffixed data (D009).

```
 .../app/web/controllers/CommentControllerTest.kt   | 33 ++++++++++++++++------
 1 file changed, 25 insertions(+), 8 deletions(-)
```

```
 M src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt
?? src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt
?? src/test/kotlin/io/realworld/app/web/controllers/ProfileStatsTest.kt
```

## Commands

### `just test only ProfileStatsTest` — exit 0

```
> Task :test

ProfileStatsTest > favorites count what the user gave PASSED

ProfileStatsTest > unknown username is 404 PASSED

ProfileStatsTest > new user has zero activity PASSED

ProfileStatsTest > anonymous request is public PASSED

ProfileStatsTest > article comment and favorite each count once PASSED

BUILD SUCCESSFUL in 12s
5 actionable tasks: 3 executed, 2 up-to-date
```

### `just test only CommentCreateTest` — exit 0

```
> Task :test

CommentCreateTest > raw response has no author secrets PASSED

CommentCreateTest > blank body returns 422 PASSED

CommentCreateTest > unknown slug returns 404 PASSED

CommentCreateTest > no token returns 401 PASSED

BUILD SUCCESSFUL in 8s
5 actionable tasks: 2 executed, 3 up-to-date
```

### `just test only CommentControllerTest` — exit 0

```
> Task :test

CommentControllerTest > delete comment for article by slug SKIPPED

CommentControllerTest > get all comments for article by slug SKIPPED

CommentControllerTest > add comment for article by slug PASSED

BUILD SUCCESSFUL in 13s
5 actionable tasks: 2 executed, 3 up-to-date
```

### `just test all` — exit 0

```
BUILD SUCCESSFUL in 56s
5 actionable tasks: 2 executed, 3 up-to-date
```

Relevant Gradle log lines:

```
CommentControllerTest > delete comment for article by slug SKIPPED
CommentControllerTest > get all comments for article by slug SKIPPED
CommentControllerTest > add comment for article by slug PASSED
CommentCreateTest > raw response has no author secrets PASSED
CommentCreateTest > blank body returns 422 PASSED
CommentCreateTest > unknown slug returns 404 PASSED
CommentCreateTest > no token returns 401 PASSED
ProfileStatsTest > favorites count what the user gave PASSED
ProfileStatsTest > unknown username is 404 PASSED
ProfileStatsTest > new user has zero activity PASSED
ProfileStatsTest > anonymous request is public PASSED
ProfileStatsTest > article comment and favorite each count once PASSED
```

### `just test census` — exit 0

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleFollowingMappingTest ran=1   passed=1   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=6   passed=6   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=1   passed=1   failed=0   skipped=2
  CommentCreateTest          ran=4   passed=4   failed=0   skipped=0
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileStatsTest           ran=5   passed=5   failed=0   skipped=0
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=93 passed=93 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

Test counts from `build/test-results/test/*.xml` via `just test census`.

## Done When

- [x] **All requirements met** — pass. `ProfileStatsTest` covers D006 (zeros, each count rises, favorites-given V=1/W=0, 404 unknown, anonymous 200). `CommentCreateTest` covers comment error paths and D008 leak check. `CommentControllerTest` class-level ignore removed; list/delete skipped with D001 reasons; `/api` removed; `add comment for article by slug` enabled and passing.
- [x] **`results/03_census.md` shows `ProfileStatsTest`, `CommentCreateTest` and `add comment for article by slug` running and passing, with the remaining comment tests skipped with reasons** — pass. See `results/03_census.md`: ProfileStatsTest ran=5 passed=5; CommentCreateTest ran=4 passed=4; CommentControllerTest ran=1 passed=1 skipped=2 (list and delete with `@Ignore` reasons).

## Notes

- Branch/camp-fresh step skipped per orchestrator instruction (already on `feat/user-activity`).
- No file:line anchor drift for cited anchors (`CommentControllerTest.kt:14`, `:21`, `:35`, `:54`); class-level `@Ignore` was at line 14 as documented.
- `add comment for article by slug` uses UUID-suffixed user/title per D009 but keeps original assertions (`SC_OK` and matching comment body); no assertion weakening.
- `favorites count what the user gave` distinguishes D006 given-vs-received semantics: V favorites W's article → V `favoritesCount`=1, W `favoritesCount`=0.
- `new user has zero activity` covers zero-case repository counts (`countByAuthor`, `countFavoritesBy`) for a user with no activity.
- Suite grew from 83 ran / 17 skipped (task 02) to **93 ran / 16 skipped** (+10 new tests, -1 skip from enabling add-comment).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `b531eeec-4605-4b86-ad9c-ec68b5f886b5`, 09:20:47Z to 09:23:53Z).

**The decisive test asserts both sides.** `ProfileStatsTest.favorites count what the user gave` has V favorite W's
article, then asserts `assertEquals(1L, vStats.body.stats.favoritesCount)` **and**
`assertEquals(0L, wStats.body.stats.favoritesCount)`. A future change that reimplemented the count as favorites
*received* would fail this test rather than silently invert the feature. That was the gap the orchestrator found while
verifying task 02: the container run proved the behavior, but nothing would have caught a regression.

**The three previously untested count functions are now covered.** `ArticleRepository.countByAuthor`,
`ArticleRepository.countFavoritesBy` and `CommentRepository.countByAuthor` are exercised by
`new user has zero activity` (the zero case for all three) and `article comment and favorite each count once`
(each count rising for its own action).

**The author's test kept its assertions.** The `CommentControllerTest` diff shows `add comment for article by slug`
retaining both original assertions; the only changes are its own UUID-suffixed user, replacing the shared
`createArticle()` default that collides across tests (D009), and the `/api` path removal (D002). The class-level
`@Ignore` is gone, replaced by method-level reasons on the list and delete tests, which stay stubbed.

**Independent run with the cache off:**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0    BUILD SUCCESSFUL in 55s
  ProfileStatsTest: tests=5 ran=5 failed=0
      new user has zero activity, article comment and favorite each count once,
      favorites count what the user gave, unknown username is 404, anonymous request is public
  CommentCreateTest: tests=4 ran=4 failed=0
      blank body returns 422, unknown slug returns 404, no token returns 401,
      raw response has no author secrets
  CommentControllerTest: tests=3 ran=1 failed=0 skipped=2
      add comment for article by slug: ran
      get all comments / delete comment: SKIPPED with reasons
  TOTAL tests=109 ran=93 failed=0 skipped=16
```

83 → 93 running and skips 17 → 16, matching the reported census. `CommentCreateTest.raw response has no author secrets`
closes the D008 gap for comment responses, which no test covered before this task.

Four of the original author's disabled tests now run across the festival: `create article`, `get all tags`,
`favorite article by slug`, `unfavorite article by slug`, and now `add comment for article by slug` — five in total.
