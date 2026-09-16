# Task 04: Expose GET /articles/search publicly — Results

## Changes

- `src/main/kotlin/io/realworld/app/web/Router.kt` — added `authenticate(optional = true) { get("search") }` as the first child of `route("articles")`, before the mandatory `authenticate` block, with an order-dependence comment (D003).
- `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt` — added `search` handler passing viewer email from optional principal to `ArticleService.search`.

### git diff --stat

```
 src/main/kotlin/io/realworld/app/web/Router.kt                      | 6 ++++++
 .../kotlin/io/realworld/app/web/controllers/ArticleController.kt    | 5 +++++
 2 files changed, 11 insertions(+)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/web/Router.kt
 M src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt
```

## Commands

### `just build compile`

Exit code: **0**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors

> Task :compileKotlin
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:15:13 Variable 'tag' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:16:13 Variable 'author' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:17:13 Variable 'favorited' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:18:13 Variable 'limit' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:19:13 Variable 'offset' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:27:13 Variable 'limit' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:28:13 Variable 'offset' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:53:13 Variable 'slug' is never used

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 4s
2 actionable tasks: 2 executed
```

### `just docker up`

Exit code: **0**

```
up at http://localhost:18080
```

### Container smoke: register, login, create article, search

Exit code: **0**

```
register:200
login:200
token obtained
{"article":{"slug":"zephyrine-searchword-article","title":"Zephyrine Searchword Article","description":"d","body":"b","tagList":["x"],"createdAt":"2026-09-16T08:12:31.238+00:00","updatedAt":"2026-09-16T08:12:31.238+00:00","favorited":false,"favoritesCount":0,"author":{"username":"search_1789546350","bio":null,"image":null,"following":false}}}
---
{"articles":[{"slug":"zephyrine-searchword-article","title":"Zephyrine Searchword Article","description":"d","body":"b","tagList":["x"],"createdAt":"2026-09-16T08:12:31.238+00:00","updatedAt":"2026-09-16T08:12:31.238+00:00","favorited":false,"favoritesCount":0,"author":{"username":"search_1789546350","bio":null,"image":null,"following":false}}],"articlesCount":1}
---
no_q:422
```

### `just docker down`

Exit code: **0**

```
stopped
```

### `just test all`

Exit code: **0**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin UP-TO-DATE
> Task :compileJava NO-SOURCE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :compileTestKotlin
> Task :compileTestJava NO-SOURCE
> Task :testClasses UP-TO-DATE

> Task :test

PagingTest > parse_rejectsNonIntegerLimit PASSED

PagingTest > parse_rejectsNegativeOffset PASSED

PagingTest > parse_acceptsLimitOneHundred PASSED

PagingTest > parse_acceptsLimitOne PASSED

PagingTest > parse_rejectsLimitZero PASSED

PagingTest > parse_rejectsLimitAboveMax PASSED

PagingTest > parse_defaultsToTwentyAndZero PASSED

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

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 19s
5 actionable tasks: 3 executed, 2 up-to-date
```

### `just test census`

Exit code: **0**

```
  PagingTest                 ran=7   passed=7   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=49 passed=49 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

## Done When

- [x] **All requirements met** — pass: `authenticate(optional = true) { get("search") }` is first child of `route("articles")` before mandatory `authenticate` block (`Router.kt:44-50`); order comment present; `ArticleController.search` passes viewer email from optional principal.
- [x] **Anonymous `GET /articles/search?q=<word>` returns 200 with `articles` array** — pass: container curl returned `{"articles":[...],"articlesCount":1}` for `q=Zephyrine`.
- [x] **`GET /articles/search` with no `q` returns 422** — pass: container curl `no_q:422`.

## Notes

- No anchor drift: task cited `Router.kt:43-45`; actual `route("articles")` at line 44, mandatory `authenticate` at line 51 after insertion.
- Task cited `ArticleController.kt` without `controllers/` path; actual file is `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt`.
- No endpoint tests added per task scope; HTTP behavior verified via container curls only. Route-order pinning tests deferred to task 05 per orchestrator guidance.
- Nothing unresolved.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `26c40a5e-ea7c-4bd1-9629-2aac34320c5c`, 08:11:12Z to 08:13:45Z).
The subagent ran its own container check; the orchestrator ran an independent one, because D003's failure mode is
invisible in the source: the route would still read as public while every anonymous request got a 401.

**Route registration (the D003 requirement).** `authenticate(optional = true) { get("search") { … } }` is the **first**
child of `route("articles")`, above the mandatory `authenticate {`, carrying the comment that explains the ordering:

```kotlin
route("articles") {
    // Public reads first. Ktor resolves equal-quality sibling routes in registration order, and
    // the authenticate block below contains {slug}, which would otherwise match "search" and
    // demand a token. Keep this block above it.
    authenticate(optional = true) {
        get("search") { articleController.search(this.context) }
    }
    authenticate {
        …
```

The mandatory block's contents are byte-identical to the pre-change snapshot the orchestrator took from `HEAD`
(feed, the `{slug}` subtree with comments and favorite, the optional-auth list route, and `post`). Nothing was moved or
reordered inside it.

**Independent container run, with no `Authorization` header at all:**

```text
$ just docker up   # then register, login, create an article containing a unique word
register=200
login=200
create=200
== ANONYMOUS search (no Authorization header at all)
anon_search=200
{"articles":[{"slug":"searchable-quokka1789546455-title","title":"Searchable quokka1789546455 title","description":"d","body":"b","tagList":["t"],"createdAt":"2026-09-16T08:14:16.070+00:00","updatedAt":"2026-09-16T08:14:16.070+00:00","favorited":false,"favoritesCount":0,"author":{"username":"orch_s_1789546455","bio":null,"image":null,"following":false}}],"articlesCount":1}
== search with token
auth_search=200
== missing q (expect 422)
missing_q=422
{"errors":{"body":["q is required."]}}
== limit out of range (expect 422)
limit_101=422
limit_abc=422
== body-only match is findable
body_word_search=200
== JSON shape of the anonymous response
top-level keys = ['articles', 'articlesCount']
articles is a list = True | count = 1
articlesCount = 1
our article present = True
author keys = ['bio', 'following', 'image', 'username']
raw contains password/token = False
following flag (anonymous) = [False]
```

What that establishes, against the task's Done When and its error paths:

- **Anonymous access works.** 200, not 401, so `search` is genuinely resolved by the optional-auth block rather than by
  `{slug}` inside the mandatory one. This is the task's first error path, ruled out empirically.
- **The response is a collection.** Top-level keys are `articles` and `articlesCount`, not a single `article` object,
  which was the task's other stated symptom of misregistration.
- **Validation reaches the wire.** No `q` → 422 with a message naming `q`; `limit=101` and `limit=abc` → 422. The
  `toIntOrNull` path from task 03 therefore produces a 422 rather than a 500 through the catch-all.
- **A viewer's token is accepted too** (200), and for an anonymous caller `following` is `false`, matching the
  requirement that anonymous callers pass `null` through to the mapping.
- **No author secrets, anonymously.** Author keys are exactly the four `Profile` fields, and the raw body contains
  neither `password` nor `token` (D008).
- **Routes stay at root** (D002): the working path is `/articles/search`, with no `/api` prefix.

The container was torn down afterwards by the run's own trap; `docker ps` shows no `ktor` container.

**Note.** The subagent's report cites "test census: ran=49", unchanged from task 03, which is correct: this task added
no tests. Task 05 adds the endpoint tests, so the suite count moves there.
