# Results: 01_authors_as_profiles

## Changes

| File | Description |
|------|-------------|
| `src/main/kotlin/io/realworld/app/domain/Article.kt` | Changed `author` from `User?` to `Profile?` |
| `src/main/kotlin/io/realworld/app/domain/Comment.kt` | Changed `author` from `User?` to `Profile?` |
| `src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt` | Added `getRaw` and `postRaw` raw-response helpers |
| `src/test/kotlin/io/realworld/app/web/util/JsonAssertions.kt` | New reusable `assertNoAuthorSecrets(rawJson)` walker |
| `src/test/kotlin/io/realworld/app/web/util/JsonAssertionsTest.kt` | Unit tests proving the assertion passes/fails correctly |

```
 src/main/kotlin/io/realworld/app/domain/Article.kt    | 2 +-
 src/main/kotlin/io/realworld/app/domain/Comment.kt    | 2 +-
 src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt | 6 ++++++
 3 files changed, 8 insertions(+), 2 deletions(-)
```

```
 M src/main/kotlin/io/realworld/app/domain/Article.kt
 M src/main/kotlin/io/realworld/app/domain/Comment.kt
 M src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt
?? src/test/kotlin/io/realworld/app/web/util/JsonAssertions.kt
?? src/test/kotlin/io/realworld/app/web/util/JsonAssertionsTest.kt
```

## Author type check

```
src/main/kotlin/io/realworld/app/web/ErrorExceptionMapping.kt:10:import io.realworld.app.domain.exceptions.UnauthorizedException
src/main/kotlin/io/realworld/app/web/ErrorExceptionMapping.kt:31:        exception<UnauthorizedException> { cause ->
src/main/kotlin/io/realworld/app/web/ErrorExceptionMapping.kt:32:            respondError(HttpStatusCode.Unauthorized, cause.message)
src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:13:        val author = ctx.parameters["author"]
src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt:17://        articleService.findBy(tag, author, favorited, limit.toInt(), offset.toInt()).also { articles ->
src/main/kotlin/io/realworld/app/domain/Article.kt:18:                   val author: Profile? = null)
src/main/kotlin/io/realworld/app/domain/exceptions/UnauthorizedException.kt:3:class UnauthorizedException(msg: String) : Exception(msg)
src/main/kotlin/io/realworld/app/domain/Comment.kt:12:                   val author: Profile? = null)
src/main/kotlin/io/realworld/app/domain/service/UserService.kt:6:import io.realworld.app.domain.exceptions.UnauthorizedException
src/main/kotlin/io/realworld/app/domain/service/UserService.kt:34:            throw UnauthorizedException("email or password invalid!")
```

```
(no output — grep -rn 'author: User' src/main/kotlin prints nothing)
```

## Compile

Command: `just build gradle compileTestKotlin`

Exit code: 0

```
> Task :compileTestKotlin

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 6s
4 actionable tasks: 3 executed, 1 up-to-date
EXIT_CODE=0
```

## JsonAssertionsTest

Command: `just test only JsonAssertionsTest`

Exit code: 0

```
JsonAssertionsTest > throwsWhenAuthorInArticlesArrayHasForbiddenField PASSED

JsonAssertionsTest > passesForCleanAuthorProfile PASSED

JsonAssertionsTest > throwsWhenAuthorHasPassword PASSED

JsonAssertionsTest > throwsWhenAuthorHasEmail PASSED

JsonAssertionsTest > throwsWhenAuthorHasToken PASSED

BUILD SUCCESSFUL in 3s
5 actionable tasks: 2 executed, 3 up-to-date
```

## Full suite

Command: `just test all`

Exit code: 0

```
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

BUILD SUCCESSFUL in 8s
5 actionable tasks: 2 executed, 3 up-to-date
EXIT_CODE=0
```

## Orchestrator verification (2026-09-15)

A cursor-agent subagent ran this task (`composer-2.5`, session `0e70b8da-097f-4202-9c5e-433c74b9bdd6`,
21:05:23Z to 21:06:39Z). The branch `feat/article-foundation` was created from `ci/jdk-matrix`, not `master`,
because PR #3 is green but its merge is waiting on the user (see `01_ci_pipeline/results/07_fest_commit.md`).
The orchestrator checked the work instead of taking the report on trust:

- **Diff.** `Article.kt:18` and `Comment.kt:12` now read `val author: Profile? = null`. `HttpUtil.kt` gained exactly
  `getRaw` and `postRaw` after `get`. `JsonAssertions.kt` matches the task's walker line for line. No other files
  changed (`git status --untracked-files=all` lists only the five files above).
- **Type check.** `grep -rn 'author: User' src/main/kotlin` printed nothing.
- **Negative cases really test something.** Each of the four uses `assertThrows(AssertionError::class.java) { … }`
  (JUnit `4.13.2`, `gradle.properties:10`), which fails the test if no `AssertionError` is thrown. The array case puts
  the forbidden field under `articles[0].author`, so it exercises the walker's array branch.
- **Independent run.** A plain re-run of `just test only JsonAssertionsTest` came back `:test FROM-CACHE` (it reused the
  subagent's results), so it proved nothing. The suite was re-run with the cache disabled:
  ```text
  $ just build gradle "cleanTest test --tests '*JsonAssertionsTest*' --no-build-cache"
  > Task :test
  JsonAssertionsTest > throwsWhenAuthorInArticlesArrayHasForbiddenField PASSED
  JsonAssertionsTest > passesForCleanAuthorProfile PASSED
  JsonAssertionsTest > throwsWhenAuthorHasPassword PASSED
  JsonAssertionsTest > throwsWhenAuthorHasEmail PASSED
  JsonAssertionsTest > throwsWhenAuthorHasToken PASSED
  BUILD SUCCESSFUL in 3s
  exit=0
  (XML: io.realworld.app.web.util.JsonAssertionsTest tests=5 failures=0 errors=0 skipped=0 timestamp=2026-09-15T21:07:32.425Z)
  ```
- **Anchor note.** The task cited `HttpUtil.kt:37-46` for the typed helpers. On `master` they span `:31-47`. The
  subagent placed the raw helpers after `get` (`:37-38`), which is within that block. No behavior impact.

## Notes

- **Anchor drift:** None. `Article.kt:18` and `Comment.kt:12` matched task anchors. `HttpUtil.kt` typed helpers remain at lines 37–38; `getRaw`/`postRaw` inserted immediately after `get` (lines 40–44).
- **Negative tests:** JUnit 4.13.2 (`gradle.properties`) supports `Assert.assertThrows`; all four negative cases use it so each test fails if `assertNoAuthorSecrets` does not throw `AssertionError`.
- **Untracked files:** New `JsonAssertions.kt` and `JsonAssertionsTest.kt` appear in `git status --short` but not in `git diff --stat` (expected for untracked files).
- **Full suite:** 9 tests passed (4 UserController + 5 JsonAssertions); 21 article/comment/profile/tag tests remain SKIPPED (pre-existing stubs).
