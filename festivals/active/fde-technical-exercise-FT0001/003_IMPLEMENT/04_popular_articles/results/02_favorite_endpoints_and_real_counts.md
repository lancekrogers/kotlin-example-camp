# Task 02: Wire the favorite and unfavorite endpoints

## Changes

- `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt` — added `favorite` and `unfavorite` delegating to `ArticleRepository` with blank-slug guard.
- `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt` — replaced stub `favorite`/`unfavorite` with suspend handlers that read auth principal, call service, and `ctx.respond(ArticleDTO(...))`.

### git diff --stat

```
 .../realworld/app/domain/service/ArticleService.kt | 10 ++++++++++
 .../app/web/controllers/ArticleController.kt       | 22 ++++++++++------------
 2 files changed, 20 insertions(+), 12 deletions(-)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt
 M src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt
```

## Commands

### `just docker up`

Exit code: **0**

```
up at http://localhost:18080
```

### Manual container verification (register → login → create article → favorite/unfavorite/404/401)

Exit code: **0**

```
=== register ===

HTTP_CODE:200
=== login ===

HTTP_CODE:200
TOKEN=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJBdXRoZW50aWNhdGlvbiIsImF1ZCI6Imt0b3ItYXVkaWVuY2UiLCJpc3MiOiJrdG9yLXJlYWx3b3JsZCIsImV4cCI6MTc4OTU4NDAzMywiaWF0IjoxNzg5NTQ4MDMzLCJlbWFpbCI6ImZhdl8xNzg5NTQ4MDMyQHZhbGlkX2VtYWlsLmNvbSJ9.A0Je4MvYVdF9BppnK3QhmU7LFmPxTrMNVBxGtPDEeeo
=== create article ===

HTTP_CODE:200
SLUG=favorite-test
=== POST favorite #1 ===

HTTP_CODE:200
{"article":{"slug":"favorite-test","title":"Favorite Test","description":"Test desc","body":"Test body","tagList":["fav"],"createdAt":"2026-09-16T08:40:33.355+00:00","updatedAt":"2026-09-16T08:40:33.355+00:00","favorited":true,"favoritesCount":1,"author":{"username":"favuser_1789548032","bio":null,"image":null,"following":false}}}
=== POST favorite #2 ===

HTTP_CODE:200
{"article":{"slug":"favorite-test","title":"Favorite Test","description":"Test desc","body":"Test body","tagList":["fav"],"createdAt":"2026-09-16T08:40:33.355+00:00","updatedAt":"2026-09-16T08:40:33.355+00:00","favorited":true,"favoritesCount":1,"author":{"username":"favuser_1789548032","bio":null,"image":null,"following":false}}}
=== DELETE unfavorite #1 ===

HTTP_CODE:200
{"article":{"slug":"favorite-test","title":"Favorite Test","description":"Test desc","body":"Test body","tagList":["fav"],"createdAt":"2026-09-16T08:40:33.355+00:00","updatedAt":"2026-09-16T08:40:33.355+00:00","favorited":false,"favoritesCount":0,"author":{"username":"favuser_1789548032","bio":null,"image":null,"following":false}}}
=== DELETE unfavorite #2 ===

HTTP_CODE:200
{"article":{"slug":"favorite-test","title":"Favorite Test","description":"Test desc","body":"Test body","tagList":["fav"],"createdAt":"2026-09-16T08:40:33.355+00:00","updatedAt":"2026-09-16T08:40:33.355+00:00","favorited":false,"favoritesCount":0,"author":{"username":"favuser_1789548032","bio":null,"image":null,"following":false}}}
=== POST unknown slug ===

HTTP_CODE:404
{"errors":{"body":["Article not found."]}}
=== POST no token ===

HTTP_CODE:401
```

### `just docker down`

Exit code: **0**

```
stopped
```

### `just test all`

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

ArticleFavoritesRepositoryTest > favoriting twice leaves one row with favoritesCount 1 PASSED
ArticleFavoritesRepositoryTest > unfavoriting twice raises no error and favoritesCount is 0 PASSED
ArticleFavoritesRepositoryTest > unknown slug throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > unknown email throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > favorited depends on viewer PASSED

ArticleSchemaTest > schema creation is idempotent and tags and slug constraints hold PASSED

ArticleSearchRepositoryTest > results ordered by createdAt descending PASSED
ArticleSearchRepositoryTest > body only match is found PASSED
ArticleSearchRepositoryTest > percent sign in search term is literal PASSED
ArticleSearchRepositoryTest > title only match is found PASSED
ArticleSearchRepositoryTest > results with equal createdAt ordered by id descending PASSED
ArticleSearchRepositoryTest > search is case insensitive PASSED
ArticleSearchRepositoryTest > paging returns page size and total match count PASSED
ArticleSearchRepositoryTest > underscore in search term is literal PASSED

LowerOnClobProbeTest > lower on the text body matches case-insensitively with an escaped literal PASSED

ArticleServiceTest > create normalizes tags PASSED
ArticleServiceTest > create rejects blank title PASSED
ArticleServiceTest > create rejects blank body PASSED
ArticleServiceTest > create appends -2 slug on duplicate title PASSED
ArticleServiceTest > create rejects blank description PASSED

SlugTest > uniqueSlug_reservedWords PASSED
SlugTest > toSlugBase_allSymbolsFallsBackToArticle PASSED
SlugTest > toSlugBase_punctuationRuns PASSED
SlugTest > toSlugBase_authorExpectations PASSED
SlugTest > uniqueSlug_successiveCollisions PASSED
SlugTest > toSlugBase_diacritics PASSED
SlugTest > toSlugBase_kebabCase PASSED

ArticleControllerTest > delete article by slug SKIPPED
ArticleControllerTest > update article by slug SKIPPED
ArticleControllerTest > get all articles by tag SKIPPED
ArticleControllerTest > favorite article by slug SKIPPED
ArticleControllerTest > get all articles favorited by username with auth SKIPPED
ArticleControllerTest > get all articles of feed SKIPPED
ArticleControllerTest > get all articles by author with auth SKIPPED
ArticleControllerTest > create article PASSED
ArticleControllerTest > get all articles by author SKIPPED
ArticleControllerTest > get all articles with auth SKIPPED
ArticleControllerTest > get single article by slug SKIPPED
ArticleControllerTest > get all articles favorited by username SKIPPED
ArticleControllerTest > unfavorite article by slug SKIPPED
ArticleControllerTest > get all articles SKIPPED

ArticleCreateTest > a_writes_row PASSED
ArticleCreateTest > b_row_survives_new_app PASSED
ArticleCreateTest > blank description returns 422 PASSED
ArticleCreateTest > blank title returns 422 PASSED
ArticleCreateTest > create without token returns 401 PASSED
ArticleCreateTest > duplicate title slug ends with -2 PASSED
ArticleCreateTest > missing body returns 422 PASSED
ArticleCreateTest > raw response has no author secrets and ISO createdAt PASSED
ArticleCreateTest > whitespace body returns 422 PASSED

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

CommentControllerTest > delete comment for article by slug SKIPPED
CommentControllerTest > get all comments for article by slug SKIPPED
CommentControllerTest > add comment for article by slug SKIPPED

ProfileControllerTest > unfollow profile by username SKIPPED
ProfileControllerTest > get profile by username SKIPPED
ProfileControllerTest > follow profile by username SKIPPED

TagControllerTest > get all tags PASSED

UserControllerTest > update user data PASSED
UserControllerTest > get current user by token PASSED
UserControllerTest > success login with email and password PASSED
UserControllerTest > success register user PASSED

JsonAssertionsTest > throwsWhenAuthorInArticlesArrayHasForbiddenField PASSED
JsonAssertionsTest > passesForCleanAuthorProfile PASSED
JsonAssertionsTest > throwsWhenAuthorHasPassword PASSED
JsonAssertionsTest > throwsWhenAuthorHasEmail PASSED
JsonAssertionsTest > throwsWhenAuthorHasToken PASSED

BUILD SUCCESSFUL in 32s
```

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
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=66 passed=66 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

Evidence: `build/test-results/test/*.xml` via `just test census` — `TOTAL ran=66 passed=66 failed=0 skipped=19`.

## Done When

- [x] **All requirements met** — pass: routes unchanged inside mandatory `authenticate` block (`Router.kt:61-63`); handlers use `ctx.respond(ArticleDTO(...))`; unknown slug → 404, idempotent favorite/unfavorite → 200, no token → 401 (container curl output above); `favorited`/`favoritesCount` populated via task-01 `toArticles` on create, search, and favorite responses.
- [x] **Against a running container: repeated favorite and unfavorite return 200 with `favoritesCount` 1 and then 0; unknown slug → 404; no token → 401** — pass: manual verification section above (`POST` twice → `favoritesCount:1,favorited:true`; `DELETE` twice → `favoritesCount:0`; `does-not-exist` → 404; no token → 401).

## Notes

- **Anchor drift:** Task cited `ArticleController.kt:62` and `:70` for stub handlers; actual stubs were at lines 66 and 74. Task cited `Router.kt:56-57`; favorite routes are at lines 61-63 inside `route("favorite")` within the mandatory `authenticate` block. No routing changes were required.
- **Ignored controller tests:** `ArticleControllerTest` still has `@Ignore` on `favorite article by slug` and `unfavorite article by slug` (D001 scope); task did not name that file for changes. Container curl verification covers the endpoint behavior.
- **No schema or repository changes:** Idempotence and `toArticles` favorite fields come from task 01; this task only wires service and controller.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `0264c457-193c-4791-9b1a-40dbf0dea047`, 08:39:19Z to 08:41:51Z).
The subagent ran its own container check; the orchestrator ran an independent one against a freshly started container
with a different user.

```text
$ just docker up   # then register, login, create an article
register=200
login=200
create=200
slug=fav-target-1789548148
favorite_1=200
   favorited= True favoritesCount= 1
favorite_2=200
   favorited= True favoritesCount= 1
unfavorite_1=200
   favorited= False favoritesCount= 0
unfavorite_2=200
   favorited= False favoritesCount= 0
== unknown slug (expect 404, NOT 500)
unknown_slug=404
{"errors":{"body":["Article not found."]}}
== no token (expect 401)
no_token=401
== search now reports the favorite state
search=200
   count= 1 favorited= False favoritesCount= 0
```

What that establishes against the task's Done When:

- **Idempotence holds at the wire, not just in the repository.** A second `POST .../favorite` returns 200 with the
  count still 1, and a second `DELETE` returns 200 with the count still 0. Neither produces a primary-key error.
- **An unknown slug is a 404 carrying `"Article not found."`,** which is the task's explicit error path: a 500 there
  would mean a null escaped instead of `NotFoundException` from `userAndArticle`.
- **No token is 401,** from the mandatory `authenticate` block, before any handler runs.
- **Read this line carefully:** the closing search check shows `favorited=false, favoritesCount=0` because the run had
  just unfavorited the article twice. That is the correct state, not search failing to report favorites. The
  favorite-state-in-search path is proven by the two `favorite_*` responses above, which are themselves produced by
  `loadBySlug` → `toArticles`, the same mapping search uses, and by
  `ArticleFavoritesRepositoryTest.favorited depends on viewer` in task 01.

Code checks:

- **Routes untouched.** `git diff --stat src/main/kotlin/io/realworld/app/web/Router.kt` is empty: `Router.kt:56-57`
  already called these handlers inside the mandatory block, so wiring them needed no routing change (D005).
- **Both handlers respond themselves** with `ctx.respond(ArticleDTO(...))`, matching `UserController`'s pattern, and take
  the principal's email with `require(!email.isNullOrBlank())`.
- **The service validates the slug** with `require(slug.isNotBlank())` → 422, and delegates to the repository without
  duplicating its idempotence logic.
- **Two stubs became real.** The commented-out stub bodies for `favorite` and `unfavorite` are gone, which partly
  discharges the deferral recorded in `03_article_search/results/07_review.md` (S3 in gate 08's list): those comments
  are removed by the slice that implements each handler, as promised.
- **Suite unchanged at 66 ran / 19 skipped**, confirmed by an independent `cleanTest test --no-build-cache` run
  (85 total tests). This task added no tests by design; task 04 adds the HTTP tests and enables the author's
  favorite/unfavorite tests.
