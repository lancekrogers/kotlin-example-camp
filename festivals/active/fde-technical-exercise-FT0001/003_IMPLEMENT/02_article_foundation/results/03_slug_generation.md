# Results: 03_slug_generation

## Changes

| File | Description |
|------|-------------|
| `src/main/kotlin/io/realworld/app/ext/Slug.kt` | Pure `toSlugBase()` and `uniqueSlug()` helpers with D003 reserved slugs |
| `src/test/kotlin/io/realworld/app/ext/SlugTest.kt` | JUnit unit tests for kebab case, punctuation, diacritics, fallback, reserved words, and collisions |

```
```

```
?? src/main/kotlin/io/realworld/app/ext/Slug.kt
?? src/test/kotlin/io/realworld/app/ext/SlugTest.kt
```

## Commands

### `just test only SlugTest`

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

SlugTest > uniqueSlug_reservedWords PASSED

SlugTest > toSlugBase_allSymbolsFallsBackToArticle PASSED

SlugTest > toSlugBase_punctuationRuns PASSED

SlugTest > toSlugBase_authorExpectations PASSED

SlugTest > uniqueSlug_successiveCollisions PASSED

SlugTest > toSlugBase_diacritics PASSED

SlugTest > toSlugBase_kebabCase PASSED

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 10s
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

BUILD SUCCESSFUL in 10s
5 actionable tasks: 2 executed, 3 up-to-date
```

## Done When

- [x] **All requirements met** — pass: `toSlugBase()` implements NFD normalization, combining-mark stripping, lowercase, non-alphanumeric collapse, trim, and `article` fallback; `uniqueSlug()` treats `search` and `feed` as taken and suffixes `-2`, `-3`, …; unit tests cover all listed cases.
- [x] **`just test only SlugTest` passes with every case listed above** — pass: exit code 0, 7/7 SlugTest methods PASSED (kebab case, author expectations, punctuation, diacritics, all-symbol fallback, both reserved words, successive collisions).

## Notes

- No anchor drift: task-specified file paths and API matched the repository layout (`ext/String.kt` neighbor directory existed as expected).
- Both new files are untracked (`??`); `git diff --stat` is empty because nothing was modified in tracked files.
- `just test all`: 18 executed tests passed (1 ArticleSchemaTest + 7 SlugTest + 4 UserControllerTest + 5 JsonAssertionsTest), 20 skipped (@Ignore controller tests), BUILD SUCCESSFUL, exit code 0.
- Nothing unresolved.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `edc5977e-3c31-4d3b-8d3f-6cc3c79710fe`, 07:25:57Z to 07:27:33Z).
Checked by the orchestrator:

- **Source matches D009's rule.** `Slug.kt` normalizes NFD, strips `\p{M}+`, lowercases, collapses `[^a-z0-9]+` to `-`,
  trims `-`, and falls back to `article`. `uniqueSlug` treats `RESERVED_SLUGS = {search, feed}` as taken (D003) and walks
  `base-2`, `base-3`, … through a lazy `generateSequence`.
- **Ordering is right.** Normalization runs before the non-slug replacement, which is what makes `Café Crème` produce
  `cafe-creme` rather than `caf-cr-me`.
- **Scope.** `git status --untracked-files=all` lists only the two new files. `git diff --stat src/main/kotlin/io/realworld/app/ext/`
  is empty, so the existing `ext/String.kt` is untouched.
- **Independent run with the cache off:**
  ```text
  $ just build gradle "cleanTest test --tests '*SlugTest*' --no-build-cache"
  > Task :test
  SlugTest > uniqueSlug_reservedWords PASSED
  SlugTest > toSlugBase_allSymbolsFallsBackToArticle PASSED
  SlugTest > toSlugBase_punctuationRuns PASSED
  SlugTest > toSlugBase_authorExpectations PASSED
  SlugTest > uniqueSlug_successiveCollisions PASSED
  SlugTest > toSlugBase_diacritics PASSED
  SlugTest > toSlugBase_kebabCase PASSED
  BUILD SUCCESSFUL in 6s
  exit=0
  (XML: SlugTest tests=7 failures=0 errors=0 timestamp=2026-09-16T07:28:01.744Z)
  ```
- **Author expectations honored.** `"slug test"` and `"slug test 2"` produce the slugs `ArticleControllerTest.kt:198,:222`
  already expect, so enabling those tests later needs no change here.
