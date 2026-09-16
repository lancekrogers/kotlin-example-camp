# Task 04: Test Popular and enable the author's favorite tests

## Changes

- `src/test/kotlin/io/realworld/app/web/controllers/PopularArticlesTest.kt` — new HTTP integration tests for GET /articles/feed/popular and favorite idempotency (D005, D008, D009).
- `src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt` — removed `@Ignore` on `favorite article by slug` and `unfavorite article by slug`; unique usernames/emails to avoid D009/D010 collisions.

### git diff --stat

```
 .../io/realworld/app/web/controllers/ArticleControllerTest.kt  | 10 ++++++----
 1 file changed, 6 insertions(+), 4 deletions(-)
```

(Untracked new file `PopularArticlesTest.kt` not included in diff --stat.)

### git status --short --untracked-files=all

```
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt
?? src/test/kotlin/io/realworld/app/web/controllers/PopularArticlesTest.kt
```

## Commands

### `just test only PopularArticlesTest`

Exit code: **0**

```
PopularArticlesTest > more favorites ranks first PASSED
PopularArticlesTest > no author secrets PASSED
PopularArticlesTest > offset past end PASSED
PopularArticlesTest > zero favorite included PASSED
PopularArticlesTest > pages are stable PASSED
PopularArticlesTest > double favorite PASSED
PopularArticlesTest > favorited is viewer specific PASSED
PopularArticlesTest > bad limit PASSED
PopularArticlesTest > equal counts newest first PASSED
PopularArticlesTest > anonymous is public PASSED

BUILD SUCCESSFUL in 17s
```

Test counts from `build/test-results/test/TEST-io.realworld.app.web.controllers.PopularArticlesTest.xml`: tests=10, failures=0, errors=0, skipped=0.

### `just test only ArticleControllerTest` (first run, before favorite email fix)

Exit code: **1**

```
ArticleControllerTest > favorite article by slug PASSED
ArticleControllerTest > create article FAILED
    com.mashape.unirest.http.exceptions.UnirestException: java.lang.RuntimeException: com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "errors" (class io.realworld.app.domain.UserDTO), not marked as ignorable (one known property: "user"])
     at [Source: (String)"{"errors":{"body":["Email already registered!"]}}"; line: 1, column: 50] (through reference chain: io.realworld.app.domain.UserDTO["errors"])
        at app//io.realworld.app.web.util.HttpUtil.registerUser(HttpUtil.kt:92)
        at app//io.realworld.app.web.util.HttpUtil.createUser(HttpUtil.kt:69)
        at app//io.realworld.app.web.util.HttpUtil.createArticle(HttpUtil.kt:75)
        at app//io.realworld.app.web.controllers.ArticleControllerTest.create article(ArticleControllerTest.kt:137)
ArticleControllerTest > unfavorite article by slug PASSED

14 tests completed, 1 failed, 11 skipped
BUILD FAILED
```

### `just test only ArticleControllerTest` (after giving favorite test its own registerUser)

Exit code: **0**

```
ArticleControllerTest > favorite article by slug PASSED
ArticleControllerTest > create article PASSED
ArticleControllerTest > unfavorite article by slug PASSED

BUILD SUCCESSFUL in 10s
```

Test counts from `build/test-results/test/TEST-io.realworld.app.web.controllers.ArticleControllerTest.xml` (after full suite): tests=14, failures=0, errors=0, skipped=11 (ran=3 passed=3).

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 42s
```

All non-skipped tests passed. PopularArticlesTest: 10 PASSED. ArticleControllerTest: `favorite article by slug` PASSED, `unfavorite article by slug` PASSED, `create article` PASSED.

### `just test census`

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

## Done When

- [x] **All requirements met** — pass
  - PopularArticlesTest covers D005: ordering by count (`more favorites ranks first`), equal counts by recency (`equal counts newest first`), page stability (`pages are stable`), zero-favorite inclusion, offset past end, bad limit 422, anonymous 200, viewer-specific favorited, double favorite idempotency, raw JSON leak check (`no author secrets`).
  - ArticleControllerTest: `@Ignore` removed from `favorite article by slug` and `unfavorite article by slug`; unfavorite username changed to `user_name_test_unfavorite`.
  - Census recorded in `results/04_census.md`.
- [x] **`results/04_census.md` shows PopularArticlesTest and the author's favorite/unfavorite tests running and passing** — pass (PopularArticlesTest ran=10 passed=10; ArticleControllerTest ran=3 includes both favorite tests plus `create article`).

## Notes

- Task anchor `:190` / `:210` for `@Ignore` lines drifted to `:199` / `:220` after prior slices added lines; removed annotations at actual locations.
- Enabling `favorite article by slug` exposed an email collision: its original `createArticle()` called `createUser()` with the default `user@valid_user_mail.com`, which `create article` also uses. Fixed by registering `user_name_test_favorite` explicitly before posting the article (D009 isolation; no assertion weakened).
- `favorited is viewer specific` initially used `.single { }` on the full popular feed; changed to `.first { }` because the feed returns all articles, not just the test row.
- Skipped tests dropped from 19 to 17 (two author favorite tests enabled).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `2c9837e0-1e7f-461f-96fb-0c0661d86b87`, 08:48:37Z to 08:51:48Z).

**The author's two tests genuinely execute now.** Independent run with the build cache disabled:

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0    BUILD SUCCESSFUL in 41s
ArticleControllerTest > favorite article by slug PASSED
ArticleControllerTest > unfavorite article by slug PASSED
  ArticleControllerTest: tests=14 ran=3 failed=0 skipped=11
      favorite article by slug: ran
      create article: ran
      unfavorite article by slug: ran
  PopularArticlesTest: tests=10 ran=10 failed=0 skipped=0
  TOTAL tests=95 ran=78 failed=0 skipped=17
```

Skips fell from 19 to 17 and the suite rose to 78 running, matching the reported census. Three of the original
author's fourteen article tests now run; the other eleven still carry method-level reasons.

**No author assertion was weakened.** The diff on `ArticleControllerTest.kt` contains only the two `@Ignore` removals
plus unique user registration. Both tests keep every assertion and their slug expectations (`slug-test`,
`slug-test-2`):

- `favorite article by slug` previously called `appRule.http.createArticle(article)`, which registers
  `HttpUtil.createUser()`'s default account — the same one `create article` uses. With rows persisting across methods
  that collides, and the subagent's first run failed exactly that way (`Email already registered`). It now registers
  `favorite_slug_test@valid_email.com` / `user_name_test_favorite` and posts the article directly. This is the D009
  collision the task predicted, fixed by isolating the data rather than by touching the assertions.
- `unfavorite article by slug` kept its own registration and only gained a unique username
  (`user_name_test` → `user_name_test_unfavorite`), since the shared default is what made it fragile.

**`PopularArticlesTest` respects the absolute-rank constraint.** It pages the whole feed with `fullWalk` (`limit=100`,
increasing offset, guarded to fail after 50 pages) and asserts only the *relative* order of the articles each test
created. Every case is present and ran: ordering by count, equal counts newest first, page stability, zero-favorite
inclusion, offset past the end, `limit=0` → 422 with the parameter-named message, anonymous access, viewer-specific
`favorited`, double favorite leaving the count at 1, and the raw-JSON leak check.

Two details worth noting in the test design:

- `offset past end` reads `articlesCount` from a live request and then requests that offset, rather than assuming a
  fixed total — necessary because every earlier test's articles are still in the table.
- `favorited is viewer specific` compares the same article as seen by the favoriting viewer and anonymously, after the
  subagent corrected its own first attempt, which used `.single` on a feed that legitimately contains many articles.

**Line anchors had drifted** (the `@Ignore`s were at `:199`/`:220`, not the task's `:190`/`:210`), because earlier
slices added tests above them. The subagent used the real locations and said so.
