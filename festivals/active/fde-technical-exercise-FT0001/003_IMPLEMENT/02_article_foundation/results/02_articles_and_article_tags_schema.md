# Results: 02_articles_and_article_tags_schema

## Changes

| File | Description |
|------|-------------|
| `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` | New `Articles` and `ArticleTags` tables with idempotent `SchemaUtils.create(Users, Tags, Articles, ArticleTags)` |
| `src/main/kotlin/io/realworld/app/domain/repository/TagRepository.kt` | Added `Tags.idsFor` get-or-create helper |
| `src/test/kotlin/io/realworld/app/domain/repository/ArticleSchemaTest.kt` | Schema test for idempotent creation, tag get-or-create, and duplicate-slug rejection |

```
 .../io/realworld/app/domain/repository/TagRepository.kt       | 11 +++++++++++
 1 file changed, 11 insertions(+)
```

```
 M src/main/kotlin/io/realworld/app/domain/repository/TagRepository.kt
?? src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
?? src/test/kotlin/io/realworld/app/domain/repository/ArticleSchemaTest.kt
```

## Commands

### `just build compile`

Exit code: 0

```
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin

BUILD SUCCESSFUL in 5s
2 actionable tasks: 2 executed
```

### `just test only ArticleSchemaTest`

Exit code: 0

```
> Task :test

ArticleSchemaTest > schema creation is idempotent and tags and slug constraints hold PASSED

BUILD SUCCESSFUL in 6s
5 actionable tasks: 3 executed, 2 up-to-date
```

### `just test all`

Exit code: 0

```
> Task :test

ArticleSchemaTest > schema creation is idempotent and tags and slug constraints hold PASSED

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

BUILD SUCCESSFUL in 14s
5 actionable tasks: 2 executed, 3 up-to-date
```

Test counts: 10 passed, 0 failed, 20 skipped.

### `git diff build.gradle`

Exit code: 0 (no output — dependencies block unchanged)

## Done When

- [x] **All requirements met** — pass. `Articles` has unique `slug`, `title`, `description`, text `body`, `author` FK to `Users`, epoch-ms `created_at`/`updated_at`. `ArticleTags` composite PK on `(article, tag)`. Single `SchemaUtils.create(Users, Tags, Articles, ArticleTags)` call. `Tags.idsFor` get-or-create added. No new dependencies.
- [x] **`just test only ArticleSchemaTest` passes** — pass. One test passed; idempotent creation, stable tag ids, duplicate slug rejected with `ExposedSQLException`.
- [x] **`just build compile` succeeds, and the dependencies block of `build.gradle` is unchanged** — pass. Compile exit 0; `git diff build.gradle` empty.

## Notes

- **Test compile fix:** First `just test only ArticleSchemaTest` run failed because unqualified `it[email]` / `it[username]` shadowed local variables. Qualified all column references as `Users.email`, `Articles.slug`, etc. No task-file anchor drift in main sources.
- **D009:** Test uses `UUID` suffix on email, username, slug, and tag name so rows are unique across the shared in-memory H2 database.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `36025162-a3d6-46d3-b58a-a9018600160f`, 07:21:18Z to 07:24:48Z). Checked
by the orchestrator rather than taken on trust:

- **Schema matches the requirement.** `Articles`: `slug` `varchar(255).uniqueIndex()`, `title`, `description`,
  `body` as `text`, `author` a `reference(..., Users)`, and `created_at`/`updated_at` as `long`. `ArticleTags` is a
  plain `Table` with `PrimaryKey(article, tag)`, the same shape as `Follows`.
- **One create call.** `SchemaUtils.create(Users, Tags, Articles, ArticleTags)` inside the repository's `init`, with the
  comment explaining the ordering.
- **No new dependency.** `git diff --stat build.gradle` is empty, so timestamps stay `long` columns.
- **Scope.** `git status --untracked-files=all` lists only `TagRepository.kt` modified plus the two new files.
- **Independent run with the cache off.** A plain re-run would have been served from the Gradle cache, so:
  ```text
  $ just build gradle "cleanTest test --tests '*ArticleSchemaTest*' --no-build-cache"
  > Task :test
  ArticleSchemaTest > schema creation is idempotent and tags and slug constraints hold PASSED
  BUILD SUCCESSFUL in 7s
  exit=0
  (XML: ArticleSchemaTest tests=1 failures=0 errors=0 skipped=0 timestamp=2026-09-16T07:25:17.232Z)
  ```
- **Observation, not a defect.** All three assertions live in one test method, so a failure names the method rather than
  the specific constraint. The task did not ask for separate methods and the coverage is complete, so it stands.
- **The subagent reported its own misstep** (unqualified `it[email]` shadowing a local), which matches the task's own
  error-path guidance. That belongs in `AGENT_WORKLOG.md` (C8).
