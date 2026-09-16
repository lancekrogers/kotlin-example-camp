# Task 04 census

From `just test census` after `just test all`.

Exit code: **0**

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=3   passed=3   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  PopularArticlesTest        ran=10  passed=10  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=78 passed=78 failed=0 skipped=17

  WARNING: 17 test(s) skipped. A green build does not mean the application works.
```

ArticleControllerTest enabled tests (from full suite Gradle log): `favorite article by slug` PASSED, `unfavorite article by slug` PASSED, `create article` PASSED.

PopularArticlesTest: all 10 methods PASSED (see `build/test-results/test/TEST-io.realworld.app.web.controllers.PopularArticlesTest.xml`).
