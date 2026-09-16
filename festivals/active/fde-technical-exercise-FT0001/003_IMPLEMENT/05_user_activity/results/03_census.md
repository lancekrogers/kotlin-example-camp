# Census: 03_stats_tests_and_enable_author_comment_test

Recorded after `just test all` on branch `feat/user-activity`.

## `just test only ProfileStatsTest` — exit 0

```
> Task :test

ProfileStatsTest > favorites count what the user gave PASSED

ProfileStatsTest > unknown username is 404 PASSED

ProfileStatsTest > new user has zero activity PASSED

ProfileStatsTest > anonymous request is public PASSED

ProfileStatsTest > article comment and favorite each count once PASSED

BUILD SUCCESSFUL in 12s
```

## `just test only CommentCreateTest` — exit 0

```
> Task :test

CommentCreateTest > raw response has no author secrets PASSED

CommentCreateTest > blank body returns 422 PASSED

CommentCreateTest > unknown slug returns 404 PASSED

CommentCreateTest > no token returns 401 PASSED

BUILD SUCCESSFUL in 8s
```

## `just test only CommentControllerTest` — exit 0

```
> Task :test

CommentControllerTest > delete comment for article by slug SKIPPED

CommentControllerTest > get all comments for article by slug SKIPPED

CommentControllerTest > add comment for article by slug PASSED

BUILD SUCCESSFUL in 13s
```

## `just test all` — exit 0

```
BUILD SUCCESSFUL in 56s
5 actionable tasks: 2 executed, 3 up-to-date
```

Relevant lines from Gradle log:

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

## `just test census` — exit 0

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

Test counts from `build/test-results/test/*.xml` via `just test census` (same output as above).
