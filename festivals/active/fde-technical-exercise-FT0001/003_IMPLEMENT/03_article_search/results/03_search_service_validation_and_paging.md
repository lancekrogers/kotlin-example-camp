# Task 03: Validate search input and paging

## Changes

- `src/main/kotlin/io/realworld/app/domain/Paging.kt` — shared `Paging.parse` for limit/offset validation with defaults 20 and 0.
- `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt` — added `search` delegating to repository and returning `ArticlesDTO` with total match count.
- `src/test/kotlin/io/realworld/app/domain/PagingTest.kt` — unit tests for defaults, accepted limits, and rejection messages.

### git diff --stat

```
 .../kotlin/io/realworld/app/domain/service/ArticleService.kt   | 10 ++++++++++
 1 file changed, 10 insertions(+)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt
?? src/main/kotlin/io/realworld/app/domain/Paging.kt
?? src/test/kotlin/io/realworld/app/domain/PagingTest.kt
```

## Commands

### `just test only PagingTest`

Exit code: **0**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :processResources UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :compileKotlin
> Task :compileJava NO-SOURCE
> Task :classes UP-TO-DATE
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

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 5s
5 actionable tasks: 4 executed, 1 up-to-date
```

PagingTest census: **ran=7 passed=7 failed=0 skipped=0**

### `just build compile`

Exit code: **0**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin UP-TO-DATE

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 2s
2 actionable tasks: 1 executed, 1 up-to-date
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
> Task :compileTestKotlin UP-TO-DATE
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
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

BUILD SUCCESSFUL in 21s
5 actionable tasks: 2 executed, 3 up-to-date
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

- [x] **All requirements met** — pass. `Paging.parse` validates limit (default 20, range 1..100) and offset (default 0, ≥ 0) with parameter-named messages; `ArticleService.search` trims/requires `q` and returns `ArticlesDTO(page.articles, Math.toIntExact(page.total))` per D004/D007.
- [x] **`just test only PagingTest` passes** — pass. 7 ran, 7 passed, 0 failed (see census above).
- [x] **`ArticleService.search` compiles against task 02's repository method** — pass. `just build compile` exit 0; `search(term, paging.limit, paging.offset, viewerEmail)` matches `ArticleRepository.search(term: String, limit: Int, offset: Long, viewerEmail: String?): ArticlePage`.

## Notes

- **Anchor drift:** Task cites `ArticleService.kt` at `src/main/kotlin/io/realworld/app/domain/ArticleService.kt`; actual path is `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt`. Implementation placed there to match existing package layout.
- **Untracked new files:** `Paging.kt` and `PagingTest.kt` are new files; `git diff --stat` only shows the modified service file until staged.
- **Test XML on host:** `build/test-results/test/*.xml` is not present on the host (Docker build); counts taken from `just test census` per D010.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `4643bfa5-f328-48b7-8eb5-424799502a17`, 08:08:21Z to 08:09:58Z).

- **Parsing cannot leak a raw platform exception.** `Paging.parse` uses `toIntOrNull` and `toLongOrNull` with
  `requireNotNull`, so a non-numeric value becomes `IllegalArgumentException("limit must be an integer.")` → 422. The
  task's error path warned specifically about `toInt()`, which would surface a `NumberFormatException` and a 500.
- **Bounds and defaults match D004:** default `(20, 0)`, `limit` in `1..100`, `offset >= 0`, and each message names its
  parameter.
- **The count is the total, not the page size (D007).** `ArticleService.search` returns
  `ArticlesDTO(page.articles, Math.toIntExact(page.total))`, taking `total` from task 02's `ArticlePage`. `toIntExact`
  makes an impossible overflow fail loudly rather than wrap.
- **`q` is required after trimming.** `require(!term.isNullOrEmpty())` on the trimmed value, so `"   "` is rejected with
  `"q is required."` → 422.
- **Messages are asserted, not assumed.** All 7 `PagingTest` cases compare `ex.message` to the exact expected string, so
  a future edit that changed a message to something unhelpful would fail the test.
- **Independent run with the cache off:**
  ```text
  $ just build gradle "cleanTest test --no-build-cache"
  suite_exit=0
    case: parse_defaultsToTwentyAndZero ok
    case: parse_acceptsLimitOne ok
    case: parse_acceptsLimitOneHundred ok
    case: parse_rejectsLimitZero ok
    case: parse_rejectsLimitAboveMax ok
    case: parse_rejectsNegativeOffset ok
    case: parse_rejectsNonIntegerLimit ok
    TOTAL tests=68 ran=49 failed=0 skipped=19
  ```
- **Reported counts matched the artifacts again** (49 ran, 19 skipped), the second task since the dispatch template
  required counts to come from the XML or census.
- **Anchor drift, correctly handled.** The task cited `ArticleService.kt` without its `service/` directory; the subagent
  edited the existing `domain/service/ArticleService.kt` rather than creating a second file, and said so.

**Observation, not a defect.** A non-integer `offset` (for example `offset=abc`) is handled by the same
`requireNotNull` and would return 422 with `"offset must be an integer."`, but no test covers that path; the task
listed four error cases and all four are tested. Worth a case in a later slice, since `04_popular_articles` reuses this
parser.
