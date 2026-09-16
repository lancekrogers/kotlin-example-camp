# Task 01: Add favorites storage with idempotent writes

## Changes

- `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` — added `ArticleFavorites` table, extended `SchemaUtils.create`, idempotent `favorite`/`unfavorite`/`userAndArticle`, and `toArticles` favorite count and viewer lookups.
- `src/test/kotlin/io/realworld/app/domain/repository/ArticleFavoritesRepositoryTest.kt` — repository tests for idempotent writes, NotFoundException paths, and viewer-dependent `favorited`.

### git diff --stat

```
 .../app/domain/repository/ArticleRepository.kt     | 48 ++++++++++++++++++++--
 1 file changed, 45 insertions(+), 3 deletions(-)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
?? src/test/kotlin/io/realworld/app/domain/repository/ArticleFavoritesRepositoryTest.kt
```

## Commands

### `just test only ArticleFavoritesRepositoryTest` (first run, wrong `SqlExpressionBuilder.count` import)

Exit code: **1**

```
> Task :compileKotlin FAILED
e: file:///app/src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt:15:55 Unresolved reference: count
e: file:///app/src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt:131:46 Unresolved reference. None of the following candidates is applicable because of receiver type mismatch:
...
BUILD FAILED in 4s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### `just test only ArticleFavoritesRepositoryTest` (second run, `Follows` table missing in test DB)

Exit code: **1**

```
ArticleFavoritesRepositoryTest > favoriting twice leaves one row with favoritesCount 1 FAILED
    org.jetbrains.exposed.exceptions.ExposedSQLException: org.h2.jdbc.JdbcSQLSyntaxErrorException: Table "Follows" not found
ArticleFavoritesRepositoryTest > unfavoriting twice raises no error and favoritesCount is 0 FAILED
    (same Follows not found)
ArticleFavoritesRepositoryTest > unknown slug throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > unknown email throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > favorited depends on viewer FAILED
    (same Follows not found)

5 tests completed, 3 failed
BUILD FAILED in 7s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### `just test only ArticleFavoritesRepositoryTest` (third run, global row count violated D009)

Exit code: **1**

```
ArticleFavoritesRepositoryTest > favoriting twice leaves one row with favoritesCount 1 PASSED
ArticleFavoritesRepositoryTest > unfavoriting twice raises no error and favoritesCount is 0 FAILED
    java.lang.AssertionError: expected:<0> but was:<1>
ArticleFavoritesRepositoryTest > unknown slug throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > unknown email throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > favorited depends on viewer PASSED

5 tests completed, 1 failed
BUILD FAILED in 5s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### `just test only ArticleFavoritesRepositoryTest` (final run)

Exit code: **0**

```
ArticleFavoritesRepositoryTest > favoriting twice leaves one row with favoritesCount 1 PASSED
ArticleFavoritesRepositoryTest > unfavoriting twice raises no error and favoritesCount is 0 PASSED
ArticleFavoritesRepositoryTest > unknown slug throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > unknown email throws NotFoundException PASSED
ArticleFavoritesRepositoryTest > favorited depends on viewer PASSED

BUILD SUCCESSFUL in 5s
```

Evidence: `build/test-results/test/TEST-io.realworld.app.domain.repository.ArticleFavoritesRepositoryTest.xml` — `tests="5" skipped="0" failures="0" errors="0"`.

### `just test only ArticleSearchTest`

Exit code: **0**

```
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

BUILD SUCCESSFUL in 14s
```

Evidence: `build/test-results/test/TEST-io.realworld.app.web.controllers.ArticleSearchTest.xml` — `tests="11" skipped="0" failures="0" errors="0"`.

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 28s
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
```

## Done When

- [x] **All requirements met** — pass: `ArticleFavorites` table with composite PK, idempotent favorite/unfavorite with `NotFoundException` on unknown slug/email, `toArticles` fills `favoritesCount` and `favorited`.
- [x] **`just test only ArticleFavoritesRepositoryTest` passes** — pass: XML `tests="5" failures="0"`.
- [x] **`just test only ArticleSearchTest` still passes** — pass: XML `tests="11" failures="0"`.

## Notes

- **Count import:** `SqlExpressionBuilder.count` does not exist in Exposed 0.41.1. Used `import org.jetbrains.exposed.sql.count` so `ArticleFavorites.user.count()` resolves as `ExpressionWithColumnType.count()`.
- **Test setup:** `favorite`/`unfavorite` call `loadBySlug` → `toArticles`, which queries `Follows`. Test `@BeforeClass` instantiates `UserRepository()` so the `Follows` table exists (same in-memory DB, separate schema init).
- **D009 row isolation:** Initial row-count assertions used `ArticleFavorites.selectAll().count()`, which counted rows from prior test methods. Replaced with `favoriteRowCount(email, slug)` scoped to the test's user/article pair.
- **Anchor drift:** None in `ArticleRepository.kt` or `Article.kt:16-17`. `UserRepository.kt:38-43` / `:124` anchors match `Follows` pattern and `deleteWhere` import location.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `1011e09e-c66b-4547-88dc-e6008f8e0d7b`, 08:30:05Z to 08:35:20Z).

**Schema (D005).** `ArticleFavorites` is a plain `Table` with `PrimaryKey(user, article)` and references to `Users` and
`Articles`, the same shape as `Follows`. It joins the existing single creation call:
`SchemaUtils.create(Users, Tags, Articles, ArticleTags, ArticleFavorites)`, so creation stays idempotent and ordered by
foreign keys.

**Idempotence is structural, not incidental.** `favorite` checks for an existing row and inserts only if absent, both
inside one `transaction { }`, so a repeated favorite cannot hit the primary key. `unfavorite` uses `deleteWhere`, which
affects zero rows when nothing is favorited. Both end by reloading through `loadBySlug` in the same transaction.

**Unknown identifiers fail as 404s.** `userAndArticle` throws `NotFoundException("User not found.")` or
`NotFoundException("Article not found.")`, which `ErrorExceptionMapping.kt:34-35` maps to 404. It is private and
documented as transaction-scoped.

**The type change in `toArticles` was made correctly, and it was the risky part.** `viewerId` is now an
`EntityID<Long>` so it can be compared against `ArticleFavorites.user`; the pre-existing `Follows` check was updated to
`viewerId.value` in the same edit. Missing that would have broken `following` for every article while the favorite
fields looked right.

**H2's GROUP BY constraint respected.** The counts query slices only `ArticleFavorites.article` plus the aggregate and
groups by the same column, and the count expression is bound once (`val favCount = ArticleFavorites.user.count()`) and
reused in both `slice` and the row read, which is what makes Exposed's expression matching work.

**Tests: 5 cases, all executed fresh with the cache disabled.**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0    BUILD SUCCESSFUL in 29s
  case: favoriting twice leaves one row with favoritesCount 1 ok
  case: unfavoriting twice raises no error and favoritesCount is 0 ok
  case: unknown slug throws NotFoundException ok
  case: unknown email throws NotFoundException ok
  case: favorited depends on viewer ok
  TOTAL tests=85 ran=66 failed=0 skipped=19
```

**Search did not regress.** All 11 `ArticleSearchTest` cases still pass now that search responses carry real
`favorited` and `favoritesCount` values, which the task named as the thing to check. Reported counts (66 ran, 19
skipped) match this measurement.

**Three real failures during the work, each fixed properly:**

1. `ArticleFavorites.user.count()` needs `import org.jetbrains.exposed.sql.count`, not the `SqlExpressionBuilder`
   member. An Exposed 0.41.1 detail worth carrying into the popular-articles query in task 03.
2. The test's database setup did not construct `UserRepository()`, so `Follows` did not exist when `toArticles` ran.
   Fixed by instantiating it in `@BeforeClass`. This is the same init-order dependency the foundation slice's review
   noted: `ArticleRepository` deliberately does not create `Follows`.
3. Its own D009 violation: it first asserted a table-wide `ArticleFavorites` row count, which other tests' rows can
   change. Rescoped to the `(user, article)` pair under test.

All three belong in `AGENT_WORKLOG.md` (C8) as worked examples of subagent missteps caught by running the tests rather
than reading the report.
