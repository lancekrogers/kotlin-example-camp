# Task 05: Wire POST /articles — Results

## Changes

- `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt` — inject `ArticleService`, implement `create` with principal lookup, `runCatching` for Jackson mapping errors, and `ctx.respond(ArticleDTO(...))`.
- `src/main/kotlin/io/realworld/app/config/ModulesConfig.kt` — bind `ArticleController(instance())` in the ARTICLE module.
- `src/main/kotlin/io/realworld/app/config/AppConfig.kt` — configure Jackson to serialize dates as ISO-8601 strings.

```
 src/main/kotlin/io/realworld/app/config/AppConfig.kt |  4 ++++
 .../kotlin/io/realworld/app/config/ModulesConfig.kt  |  2 +-
 .../app/web/controllers/ArticleController.kt         | 20 ++++++++++++--------
 3 files changed, 17 insertions(+), 9 deletions(-)
```

```
 M src/main/kotlin/io/realworld/app/config/AppConfig.kt
 M src/main/kotlin/io/realworld/app/config/ModulesConfig.kt
 M src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt
```

## Commands

### `just test all` — exit code 0

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :processResources UP-TO-DATE
> Task :processTestResources NO-SOURCE

> Task :compileKotlin
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:15:13 Variable 'tag' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:16:13 Variable 'author' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:17:13 Variable 'favorited' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:18:13 Variable 'limit' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:19:13 Variable 'offset' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:27:13 Variable 'limit' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:28:13 Variable 'offset' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:53:13 Variable 'slug' is never used

> Task :compileJava NO-SOURCE
> Task :classes UP-TO-DATE
> Task :compileTestKotlin
> Task :compileTestJava NO-SOURCE
> Task :testClasses UP-TO-DATE

> Task :test

ArticleSchemaTest > schema creation is idempotent and tags and slug constraints hold PASSED

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

ArticleControllerTest > create article SKIPPED

ArticleControllerTest > get all articles by author SKIPPED

ArticleControllerTest > get all articles with auth SKIPPED

ArticleControllerTest > get single article by slug SKIPPED

ArticleControllerTest > get all articles favorited by username SKIPPED

ArticleControllerTest > unfavorite article by slug SKIPPED

ArticleControllerTest > get all articles SKIPPED

CommentControllerTest > delete comment for article by slug SKIPPED

CommentControllerTest > get all comments for article by slug SKIPPED

CommentControllerTest > add comment for article by slug SKIPPED

ProfileControllerTest > unfollow profile by username SKIPPED

ProfileControllerTest > get profile by username SKIPPED

ProfileControllerTest > follow profile by username SKIPPED

TagControllerTest > get all tags SKIPPED

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

BUILD SUCCESSFUL in 20s
5 actionable tasks: 4 executed, 1 up-to-date
```

Test counts: 22 PASSED, 22 SKIPPED, 0 FAILED.

### Container verification (`just docker up` + curl + `just docker down` via EXIT trap) — exit code 0

```
=== REGISTER ===
{"user":{"email":"smoke_1789544024@valid_email.com","token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJBdXRoZW50aWNhdGlvbiIsImF1ZCI6Imt0b3ItYXVkaWVuY2UiLCJpc3MiOiJrdG9yLXJlYWx3b3JsZCIsImV4cCI6MTc4OTU4MDAyNCwiaWF0IjoxNzg5NTQ0MDI0LCJlbWFpbCI6InNtb2tlXzE3ODk1NDQwMjRAdmFsaWRfZW1haWwuY29tIn0.vgImzu0uxpthzUsgujk36Ljf1wNM_lz-5NxUiW8vYFk","username":"smoke_1789544024","bio":null,"image":null}}
HTTP_STATUS:200
=== LOGIN ===
{"user":{"email":"smoke_1789544024@valid_email.com","token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJBdXRoZW50aWNhdGlvbiIsImF1ZCI6Imt0b3ItYXVkaWVuY2UiLCJpc3MiOiJrdG9yLXJlYWx3b3JsZCIsImV4cCI6MTc4OTU4MDAyNSwiaWF0IjoxNzg5NTQ0MDI1LCJlbWFpbCI6InNtb2tlXzE3ODk1NDQwMjRAdmFsaWRfZW1haWwuY29tIn0.gzc3WP2d2vAQ2SKg__5SzPKMGXRmw-B11XibttvtpPg","username":"smoke_1789544024","bio":null,"image":null}}
HTTP_STATUS:200
=== CREATE ARTICLE (valid) ===
{"article":{"slug":"hello-world","title":"Hello World","description":"d","body":"b","tagList":["x"],"createdAt":"2026-09-16T07:33:45.429+00:00","updatedAt":"2026-09-16T07:33:45.429+00:00","favorited":false,"favoritesCount":0,"author":{"username":"smoke_1789544024","bio":null,"image":null,"following":false}}}
HTTP_STATUS:200
=== CREATE ARTICLE (missing body) ===
{"errors":{"body":["Article is invalid."]}}
HTTP_STATUS:422
=== CREATE ARTICLE (no token) ===

HTTP_STATUS:401
```

Author object in created article: `{"username":"smoke_1789544024","bio":null,"image":null,"following":false}` — no `password`, `email`, or `token` fields.

`createdAt` value `"2026-09-16T07:33:45.429+00:00"` is an ISO-8601 string (not a numeric timestamp).

## Done When

- [x] **All requirements met** — pass
  - `ArticleController` takes `ArticleService` and `create` calls `ctx.respond(ArticleDTO(created))` (evidence: controller + curl 200 response).
  - Missing `body` returns 422, not 500 (evidence: curl missing-body → HTTP 422).
  - Dates serialize as ISO-8601 strings (evidence: `"createdAt":"2026-09-16T07:33:45.429+00:00"`).
  - Other stubbed methods still compile (evidence: `just test all` BUILD SUCCESSFUL; compile warnings only for unused stub variables).
- [x] **Against a running container: POST /articles with token returns 200 with slug, tags, ISO-8601 dates and secret-free author; without token 401; missing body 422** — pass (evidence: curl output above — 200 with `"slug":"hello-world"`, `"tagList":["x"]`, ISO `createdAt`, clean author; 401 no token; 422 missing body).

## Notes

- No anchor drift encountered; line numbers in the task matched the code at implementation time.
- `withColonInTimeZone` compiled without issue on the Jackson version pulled by ktor-jackson 1.2.3.
- `create` return type changed from `ArticleDTO` to `Unit` (implicit) because the controller now responds directly; `Router.kt:66` ignores the return value, so no router change was needed.
- Container torn down via `trap "just docker down" EXIT`.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `fd5cb988-5167-4de5-a4d7-17bcfe038916`, 07:31:47Z to 07:34:17Z).
The subagent ran its own container check; the orchestrator did not rely on it and ran an independent one against a
freshly started container, with a different user and a payload that also tests tag normalization end to end.

```text
$ just docker up   # then register, login, and exercise POST /articles
== register
code=200
== login
code=200
== POST /articles with token
code=200
-- body:
{"article":{"slug":"orchestrator-check-1789544091","title":"Orchestrator Check 1789544091","description":"d","body":"b","tagList":["x","y"],"createdAt":"2026-09-16T07:34:52.130+00:00","updatedAt":"2026-09-16T07:34:52.130+00:00","favorited":false,"favoritesCount":0,"author":{"username":"orch_1789544091","bio":null,"image":null,"following":false}}}
== POST /articles missing body field (expect 422)
code=422
== POST /articles blank title (expect 422)
code=422
== POST /articles without token (expect 401)
code=401
== JSON shape checks
slug          = orchestrator-check-1789544091
tagList       = ['x', 'y']
createdAt     = 2026-09-16T07:34:52.130+00:00 | ISO-8601 string: True
author keys   = ['bio', 'following', 'image', 'username']
author leaks  = none
raw has password/token anywhere: False
```

What that proves, against the task's Done When:

- **200 with a slug, tags, ISO dates and a secret-free author.** The slug came from the title through `toSlugBase`,
  `tagList` shows `[" y "]` trimmed and the duplicate `"x"` collapsed (end-to-end proof of the service's normalization,
  which the unit tests cover separately), and `createdAt` is a string starting `2026-09-16T`, so
  `WRITE_DATES_AS_TIMESTAMPS` is genuinely off.
- **401 without a token**, from the mandatory `authenticate` block at `Router.kt:45`, not from controller code.
- **422 on a body Jackson cannot map.** This is the `runCatching` path; without it the Jackson mapping exception would
  reach the catch-all and return 500.
- **422 on a blank title**, through `require(...)` → `ErrorExceptionMapping.kt:38`.
- **Teardown.** `docker ps` shows no `ktor` container after both the subagent's run and the orchestrator's.

Code checks:

- `ArticleController` now takes `ArticleService` and `create` responds itself with `ctx.respond(ArticleDTO(...))`, so
  `Router.kt:66` ignoring the return value is correct. The other stubbed methods are unchanged and still return their
  placeholder DTOs.
- `ModulesConfig` binds `ArticleController(instance())`.
- The remaining `Variable 'tag'/'limit'/'offset' is never used` compile warnings come from the untouched stubs, not from
  this change.

**Observation, not a defect.** The two new Jackson imports in `AppConfig.kt` were inserted after the `io.ktor.features.*`
group rather than in alphabetical order. The build runs no linter, so nothing fails; left as is to keep the diff to what
the task specified.
