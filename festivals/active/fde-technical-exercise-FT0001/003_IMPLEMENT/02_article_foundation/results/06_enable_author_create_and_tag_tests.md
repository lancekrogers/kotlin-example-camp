# Task 06: Enable Author Create and Tag Tests — Results

## Changes

- `src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt` — changed `createArticle` POST path from `/api/articles` to `/articles` (D002).
- `src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt` — removed class-level `@Ignore`; added method-level `@Ignore` with D010 reasons on all tests except `create article`; replaced `/api/` paths with `/`.
- `src/test/kotlin/io/realworld/app/web/controllers/TagControllerTest.kt` — removed class-level `@Ignore`; unique UUID-suffixed username/email/title; assert POST 200 and both `dragons`/`training` tags present; replaced `/api/` paths.
- `src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt` — new test class: 401 without token, 422 blank title, 422 missing body, duplicate-title slug `-2`, raw-JSON author leak + ISO `createdAt`, D009 persistence probe.

```
 .../app/web/controllers/ArticleControllerTest.kt   | 48 ++++++++++++++--------
 .../app/web/controllers/TagControllerTest.kt       | 20 +++++----
 .../kotlin/io/realworld/app/web/util/HttpUtil.kt   |  2 +-
 3 files changed, 42 insertions(+), 28 deletions(-)
```

```
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/TagControllerTest.kt
 M src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt
?? src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt
```

## Commands

### `just test all` (first run, before postRaw fix) — exit code 1

Two failures in `ArticleCreateTest` because `post<ArticleDTO>` cannot deserialize error/empty bodies:

```
ArticleCreateTest > blank title returns 422 FAILED
    com.mashape.unirest.http.exceptions.UnirestException: java.lang.RuntimeException: com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "errors" (class io.realworld.app.domain.ArticleDTO), not marked as ignorable (one known property: "article"])
     at [Source: (String)"{"errors":{"body":["Article title can't be blank."]}}"; line: 1, column: 54] (through reference chain: io.realworld.app.domain.ArticleDTO["errors"])

ArticleCreateTest > create without token returns 401 FAILED
    com.mashape.unirest.http.exceptions.UnirestException: java.lang.RuntimeException: com.fasterxml.jackson.databind.exc.MismatchedInputException: No content to map due to end-of-input
     at [Source: (String)""; line: 1, column: 0]

50 tests completed, 2 failed, 19 skipped
BUILD FAILED
```

All other tests in that run passed, including persistence probe (`a_writes_row`, `b_row_survives_new_app`), `create article`, and `get all tags`.

### `just test all` (second run, after postRaw fix) — exit code 0

```
ArticleCreateTest > a_writes_row PASSED
ArticleCreateTest > b_row_survives_new_app PASSED
ArticleCreateTest > blank title returns 422 PASSED
ArticleCreateTest > create without token returns 401 PASSED
ArticleCreateTest > duplicate title slug ends with -2 PASSED
ArticleCreateTest > missing body returns 422 PASSED
ArticleCreateTest > raw response has no author secrets and ISO createdAt PASSED
ArticleControllerTest > create article PASSED
(13 other ArticleControllerTest methods SKIPPED)
TagControllerTest > get all tags PASSED
UserControllerTest > (4 tests) PASSED
(remaining unit tests) PASSED

BUILD SUCCESSFUL in 20s
```

Counts: 31 ran, 31 passed, 19 skipped, 0 failed.

### `just test census` — exit code 0

```
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=7   passed=7   failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=31 passed=31 failed=0 skipped=19
```

## Done When

- [x] **All requirements met** — pass
  - D010: class-level `@Ignore` removed from `ArticleControllerTest` and `TagControllerTest`; method-level `@Ignore` on every stubbed test; `create article` and `get all tags` run (census: `ArticleControllerTest ran=1`, `TagControllerTest ran=1`).
  - D002: all `/api/` prefixes removed from `ArticleControllerTest`, `TagControllerTest`, and `HttpUtil.kt:76`.
  - New negative/leak tests in `ArticleCreateTest` (401, 422 blank title, 422 missing body, duplicate slug `-2`, raw JSON leak check + ISO `createdAt`).
  - D009 persistence probe implemented and recorded in `results/06_persistence.md`.
- [x] **`results/06_census.md` shows expected census** — pass (`UserControllerTest ran=4`, `ArticleControllerTest ran=1 skipped=13`, `TagControllerTest ran=1`, `ArticleCreateTest ran=7`).
- [x] **`results/06_persistence.md` states persistence outcome** — pass (rows persist; probe passed).

## Notes

- No anchor drift on cited line numbers (`ArticleControllerTest:18`, `TagControllerTest:14`, `HttpUtil:70`, `UserControllerTest:14`).
- `blank title returns 422` and `create without token returns 401` initially used `post<ArticleDTO>`, which throws on non-`ArticleDTO` response bodies (422 error envelope, empty 401). Fixed by switching to `postRaw` and asserting status only — same pattern as `missing body returns 422`.
- D009 persistence probe **confirmed** rows survive across test methods; no D009 amendment needed (see `results/06_persistence.md`).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `df18b433-f973-4a69-8bf9-2bf4a0d72c94`, 07:35:38Z to 07:39:03Z).
The orchestrator captured the pre-change state of all three author test files from `HEAD` before dispatch, then
compared.

**No assertion was weakened. Two were strengthened.**

- `TagControllerTest` previously asserted only `assertTrue(response.body.tags.isNotEmpty())` after an unchecked POST,
  while registering `user_name_test` — the same username `HttpUtil.createUser()` defaults to (`HttpUtil.kt:67`). With
  rows persisting across methods, that test could pass on another test's tags even if its own POST returned 401. It now
  registers `tags_user_<uuid>`, asserts `createResponse.status == 200`, and asserts the response contains both
  `dragons` and `training`, so it proves its own write.
- `ArticleControllerTest`: filtering the diff to changes that are neither `@Ignore` nor `/api` path edits leaves only
  path lines. No assertion was touched. Counts match the `HEAD` snapshot exactly: 14 `@Test`, now 13 method-level
  `@Ignore`s each naming the stubbed endpoint, and the class-level `@Ignore` gone (`class ArticleControllerTest {` is
  now at line 18).
- `HttpUtil`: the `/api/articles` post was at line 76, not the task's cited `:70` (task 01's raw helpers shifted it).
  The correct line changed; `registerUser` and `loginAndSetTokenHeader`, which already used root paths, are untouched.

**The two "fixed" tests were a harness problem, not a weakened assertion.** The first suite run failed
`blank title returns 422` and `create without token returns 401` because `post<ArticleDTO>` tries to deserialize the
response into `ArticleDTO`, which fails on an error body (`Unrecognized field "errors"`) and on an empty 401 body
(`No content to map due to end-of-input`). Switching those two to `postRaw` keeps
`assertEquals(SC_UNPROCESSABLE_ENTITY, …)` and `assertEquals(SC_UNAUTHORIZED, …)` intact. This is exactly why task 01
added the raw helpers.

**Independent full-suite run with the cache off** (the subagent's own runs would otherwise be cache hits):

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0   BUILD SUCCESSFUL in 17s
  ArticleSchemaTest          tests=1   ran=1   failed=0   skipped=0
  ArticleServiceTest         tests=5   ran=5   failed=0   skipped=0
  SlugTest                   tests=7   ran=7   failed=0   skipped=0
  ArticleControllerTest      tests=14  ran=1   failed=0   skipped=13
  ArticleCreateTest          tests=7   ran=7   failed=0   skipped=0
  CommentControllerTest      tests=3   ran=0   failed=0   skipped=3
  ProfileControllerTest      tests=3   ran=0   failed=0   skipped=3
  TagControllerTest          tests=1   ran=1   failed=0   skipped=0
  UserControllerTest         tests=4   ran=4   failed=0   skipped=0
  JsonAssertionsTest         tests=5   ran=5   failed=0   skipped=0
  TOTAL tests=50 ran=31 failed=0 skipped=19
```

**D009 probe result accepted.** `a_writes_row` then `b_row_survives_new_app` both pass, and the second asserts
`count == 1L` for the slug written by the first, through a new `AppRule`. Rows do persist across test methods, so D009's
stated reason holds and needs no amendment. Note the probe's own coupling: `b` reads a companion field set by `a` and
relies on `@FixMethodOrder(NAME_ASCENDING)`. If `a` ever fails, `b` fails too rather than passing vacuously, which is
the safe direction.

### Findings routed to the iterate gate

1. **D010, open.** `CommentControllerTest` and `ProfileControllerTest` remain entirely disabled with a bare class-level
   `@Ignore` and no reason, which `FESTIVAL_RULES.md` ("No silent skips") forbids. Out of scope for this task, which
   named only three files. The census still flags both with `<-- entire class disabled`.
2. **`/api` paths remain** in `ProfileControllerTest` (3) and `CommentControllerTest` (5), plus two commented-out lines
   in `UserControllerTest`. `05_user_activity` enables the comment test and will fix those; no planned task covers
   `ProfileControllerTest`, so its reason must say so explicitly.
