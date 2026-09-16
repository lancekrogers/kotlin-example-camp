# Results: 04_article_repository_and_service_create

## Changes

| File | Description |
|------|-------------|
| `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` | Added `create`, `findBySlug`, `loadBySlug`, and `toArticles` with tag batch-load and `Profile` author mapping |
| `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt` | New service with validation, tag normalization, and slug-base delegation |
| `src/main/kotlin/io/realworld/app/config/ModulesConfig.kt` | Wired `ArticleRepository` and `ArticleService` in the ARTICLE Kodein module |
| `src/test/kotlin/io/realworld/app/domain/service/ArticleServiceTest.kt` | Direct service tests for blank-field rejection, tag normalization, and duplicate-title slug suffix |

```
 .../io/realworld/app/config/ModulesConfig.kt       |  4 ++
 .../app/domain/repository/ArticleRepository.kt     | 63 ++++++++++++++++++++++
 2 files changed, 67 insertions(+)
```

```
 M src/main/kotlin/io/realworld/app/config/ModulesConfig.kt
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
?? src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt
?? src/test/kotlin/io/realworld/app/domain/service/ArticleServiceTest.kt
```

## Commands

### `just test only ArticleServiceTest`

Exit code: 0

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

ArticleServiceTest > create normalizes tags PASSED

ArticleServiceTest > create rejects blank title PASSED

ArticleServiceTest > create rejects blank body PASSED

ArticleServiceTest > create appends -2 slug on duplicate title PASSED

ArticleServiceTest > create rejects blank description PASSED

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 11s
5 actionable tasks: 4 executed, 1 up-to-date
```

### `just test all`

Exit code: 0

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

BUILD SUCCESSFUL in 9s
5 actionable tasks: 2 executed, 3 up-to-date
```

### Grep: `password` in ArticleRepository.kt

Exit code: 1 (no matches)

```
(no output)
```

### Grep: `Users.email` in ArticleRepository.kt

Exit code: 0

```
src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
  48:        val authorRow = Users.select { Users.email eq authorEmail }.singleOrNull()
  80:        val viewerId = viewerEmail?.let { email -> Users.select { Users.email eq email }.singleOrNull()?.get(Users.id)?.value }
```

Both occurrences are inside `select { }` filters only.

## Done When

- [x] **All requirements met** — pass. `ArticleRepository` maps `Article` with tags, `Date` timestamps, and `Profile` author (`following` via `Follows` join). `ArticleService.create` validates blank fields with `require`, normalizes tags, throws `NotFoundException` for unknown author (repository). Kodein ARTICLE module binds both classes.
- [x] **`just test only ArticleServiceTest` passes** — pass. 5 tests: blank title/description/body rejection, tag normalization `["x","y"]`, duplicate title slug ends with `-2`.
- [x] **`ArticleRepository.kt` reads no password column, and `Users.email` appears only inside `select { }` filters** — pass. Grep for `password` returned no matches; both `Users.email` usages are inside `select { }` (lines 48 and 80).

## Notes

- No anchor drift encountered; task snippets matched existing `ArticleRepository`, `ModulesConfig`, and `UserService` patterns.
- `ArticleController()` left without constructor injection per task instruction (task 05).
- `NotFoundException` for unknown author is implemented in the repository but not covered by a dedicated test in this task's step 4 list; behavior is exercised indirectly via successful creates with a known author.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `39ee3dfc-53d5-4cd7-8493-8e820a7fdd67`, 07:28:34Z to 07:30:31Z).
The orchestrator captured the pre-change state of every file this task touches from `HEAD` first, then checked the
result against it:

- **Follow orientation is correct, which was the main risk.** `toArticles` tests
  `Follows.select { (Follows.user eq authorId) and (Follows.follower eq viewerId) }`. That matches
  `UserRepository.follow()` (`UserRepository.kt:112-122`), which inserts `Follows.user` = the account being followed and
  `follower` = the viewer. An inverted test would have made `following` wrong for every profile while still passing a
  naive test.
- **No author secrets reach a response (D008).** `grep -n 'password\|Users.email'` on the repository and service returns
  only two lines, both `Users.select { Users.email eq … }` filters:
  ```text
  ArticleRepository.kt:48:        val authorRow = Users.select { Users.email eq authorEmail }.singleOrNull()
  ArticleRepository.kt:80:        val viewerId = viewerEmail?.let { email -> Users.select { Users.email eq email }.singleOrNull()?.get(Users.id)?.value }
  ```
  The `Profile` is built from `username`, `bio` and `image` only.
- **Validation routes to 422.** `ArticleService.create` uses `require(...)` for blank title, description and body, and
  `ErrorExceptionMapping.kt:38` maps `IllegalArgumentException` to 422. An unknown author email throws
  `NotFoundException` (404) from the repository.
- **Kodein wiring.** The ARTICLE module gained `ArticleRepository()` and `ArticleService(instance())`;
  `ArticleController()` is untouched, as the task requires, since task 05 gives it a constructor argument.
- **One transaction for create and load.** `create` ends with `requireNotNull(loadBySlug(slug, viewerEmail = authorEmail))`
  inside the same `transaction { }`, avoiding the `NoSuchElementException` the task's error path warns about.
- **Tags are batch-loaded.** `toArticles` fetches all tags with one `inList` query rather than per row, and sorts them.
- **Test isolation (D009).** `@Before` creates a `UUID`-suffixed user per test, and the duplicate-title and tag tests use
  their own suffixed titles, so nothing depends on an empty database.
- **Independent run with the cache off:**
  ```text
  $ just build gradle "cleanTest test --tests '*ArticleServiceTest*' --no-build-cache"
  > Task :test
  ArticleServiceTest > create normalizes tags PASSED
  ArticleServiceTest > create rejects blank title PASSED
  ArticleServiceTest > create rejects blank body PASSED
  ArticleServiceTest > create appends -2 slug on duplicate title PASSED
  ArticleServiceTest > create rejects blank description PASSED
  BUILD SUCCESSFUL in 3s
  exit=0
  (XML: ArticleServiceTest tests=5 failures=0 errors=0 timestamp=2026-09-16T07:30:56.434Z)
  ```

**Observation, not a defect.** Both author lookups use `Users.select { … }`, which issues `SELECT *` and so materializes
the password hash in memory even though only `Users.id` is used. Nothing reaches a response, and D008's rule is about
responses, so this stands. Narrowing them with `.slice(Users.id)` would be a small hardening improvement; recorded here
rather than changed, to keep this task's diff to what it specified.
