# Task 02: Add the search query to ArticleRepository

## Changes

- `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` — added `ArticlePage` data class and `search` method with case-insensitive LIKE on title/body, total count, ordering, and paging.
- `src/test/kotlin/io/realworld/app/domain/repository/ArticleSearchRepositoryTest.kt` — repository tests for title/body match, case insensitivity, literal `%`/`_`, ordering, and paging.

### git diff --stat

```
 .../app/domain/repository/ArticleRepository.kt         | 18 ++++++++++++++++++
 1 file changed, 18 insertions(+)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
?? src/test/kotlin/io/realworld/app/domain/repository/ArticleSearchRepositoryTest.kt
```

## Commands

### `just test only ArticleSearchRepositoryTest` (first run, `SqlExpressionBuilder.or` import)

Exit code: **1**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :processResources UP-TO-DATE
> Task :processTestResources NO-SOURCE

> Task :compileKotlin
e: file:///app/src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt:18:55 Unresolved reference: or
e: file:///app/src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt:78:65 Unresolved reference. None of the following candidates is applicable because of receiver type mismatch: 
public inline infix fun BigInteger.or(other: BigInteger): BigInteger defined in kotlin

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileKotlin'.
> A failure occurred while executing org.jetbrains.kotlin.compilerRunner.GradleCompilerRunnerWithWorkers$GradleKotlinCompilerWorkAction
   > Compilation error. See log for more details

> Task :compileKotlin FAILED

BUILD FAILED in 4s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### `just test only ArticleSearchRepositoryTest` (second run, underscore test asserted 1 match)

Exit code: **1**

```
> Task :test FAILED

ArticleSearchRepositoryTest > results ordered by createdAt descending PASSED
ArticleSearchRepositoryTest > body only match is found PASSED
ArticleSearchRepositoryTest > percent sign in search term is literal PASSED
ArticleSearchRepositoryTest > title only match is found PASSED
ArticleSearchRepositoryTest > results with equal createdAt ordered by id descending PASSED
ArticleSearchRepositoryTest > search is case insensitive PASSED
ArticleSearchRepositoryTest > paging returns page size and total match count PASSED
ArticleSearchRepositoryTest > underscore in search term is literal FAILED
    java.lang.AssertionError: expected:<1> but was:<0>
        at org.junit.Assert.fail(Assert.java:89)
        at org.junit.Assert.failNotEquals(Assert.java:835)
        at org.junit.Assert.assertEquals(Assert.java:120)
        at org.junit.Assert.assertEquals(Assert.java:146)
        at io.realworld.app.domain.repository.ArticleSearchRepositoryTest.underscore in search term is literal(ArticleSearchRepositoryTest.kt:109)

8 tests completed, 1 failed

BUILD FAILED in 6s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### `just test only ArticleSearchRepositoryTest` (final run)

Exit code: **0**

```
> Task :test

ArticleSearchRepositoryTest > results ordered by createdAt descending PASSED
ArticleSearchRepositoryTest > body only match is found PASSED
ArticleSearchRepositoryTest > percent sign in search term is literal PASSED
ArticleSearchRepositoryTest > title only match is found PASSED
ArticleSearchRepositoryTest > results with equal createdAt ordered by id descending PASSED
ArticleSearchRepositoryTest > search is case insensitive PASSED
ArticleSearchRepositoryTest > paging returns page size and total match count PASSED
ArticleSearchRepositoryTest > underscore in search term is literal PASSED

BUILD SUCCESSFUL in 5s
```

Evidence: `build/test-results/test/TEST-io.realworld.app.domain.repository.ArticleSearchRepositoryTest.xml` — `tests="8" skipped="0" failures="0" errors="0"`.

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 18s
```

### `just test census`

Exit code: **0**

```
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

  TOTAL ran=42 passed=42 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

## Done When

- [x] **All requirements met** — pass. `ArticleRepository.search` uses `lowerCase() like` on title and body with `LikePattern.ofLiteral`, orders by `createdAt DESC, id DESC`, pages with `limit`, counts total without limit/offset, maps via `toArticles`.
- [x] **`just test only ArticleSearchRepositoryTest` passes every listed case** — pass. 8/8 from `TEST-io.realworld.app.domain.repository.ArticleSearchRepositoryTest.xml`.

## Notes

- **Import drift:** Task snippet implied `SqlExpressionBuilder.or`; Exposed 0.41.1 exposes the infix `or` from `org.jetbrains.exposed.sql.or` (`Op.kt`). Used that import instead.
- **LikePattern in transaction:** `LikePattern.ofLiteral` is built inside the `transaction` block in `search`, per task 01 constraint (needs active transaction for dialect).
- **Underscore test interpretation:** Task requires `search("<token>100_")` must **not** match `"<token>1000 purex"` (proving `_` is literal, not a single-char wildcard). Initial test wrongly expected a match on `"<token>100% pure_x"`; corrected to assert 0 matches on the wildcard-bait title only.
- **No CAST/ILIKE workaround needed:** Task 01 probe confirmed H2 accepts `lower(body) LIKE ... ESCAPE` on the CLOB body column.
- **ArticlePage placement:** Left in `ArticleRepository.kt` per task default; service layer does not need it yet.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `630de47d-11d4-46aa-9aa2-80d45960eb9a`, 08:02:59Z to 08:06:47Z).

**The query matches D004 and carries task 01's constraint into production code.**

- **Pattern built inside the transaction.** `search` opens `transaction { }` and constructs
  `LikePattern("%", '\\') + LikePattern.ofLiteral(term.lowercase()) + "%"` within it. Task 01 proved that building it
  outside throws `IllegalStateException: No transaction in context`, so this placement is load-bearing, not incidental.
- **One predicate, two queries.** `matches` is `(Articles.title.lowerCase() like pattern) or (Articles.body.lowerCase()
  like pattern)`, and the same value is passed to both the count and the page query, so `total` cannot drift from the
  filter (D007). The count query runs on `Articles` alone; the page query joins `Users` for the author.
- **Ordering in a single call.** `orderBy(Articles.createdAt to SortOrder.DESC, Articles.id to SortOrder.DESC)`, which is
  what makes both keys apply; two chained `orderBy` calls would have dropped one.
- **Paging.** `limit(limit, offset)` with `offset: Long`, matching Exposed's signature.
- **Both sides fold case identically:** the term via Kotlin `lowercase()`, the columns via SQL `lowerCase()`.

**Tests: 8 cases, all executed fresh with the build cache disabled.**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0
  case: title only match is found ok
  case: body only match is found ok
  case: search is case insensitive ok
  case: percent sign in search term is literal ok
  case: underscore in search term is literal ok
  case: results ordered by createdAt descending ok
  case: results with equal createdAt ordered by id descending ok
  case: paging returns page size and total match count ok
  TOTAL tests=61 ran=42 failed=0 skipped=19
```

The cases that matter most, checked in the source rather than taken from the summary:

- **`underscore in search term is literal`** stores `"<token>1000 purex"` and searches `"<token>100_"`, asserting
  **0** matches and `total == 0`. Without `ofLiteral` escaping, `_` would be a single-character wildcard and this would
  match, so this is the test that proves search terms cannot smuggle wildcards.
- **`percent sign in search term is literal`** stores `"<token>100% pure_x"` and finds it by searching `"<token>100%"`,
  proving `%` is escaped yet still matchable as text.
- **`paging returns page size and total match count`** asserts 2 articles with `total == 3` at offset 0, then 1 article
  with `total == 3` at offset 2, and pins the slug order of each page.
- **`results with equal createdAt ordered by id descending`** compares against the ids actually returned by the inserts
  rather than assuming which is larger, so it cannot pass by luck.
- **Isolation (D009).** Every case embeds a per-test UUID token in the text it searches for, so no case can match another
  case's rows in the shared in-memory database.

**The subagent hit two real failures and handled both correctly.** First a compile error (`SqlExpressionBuilder.or`
does not exist; the correct import is `org.jetbrains.exposed.sql.or`). Then its own underscore assertion was wrong: it
had expected `"<token>100_"` to match the `%` title. It corrected the test to the task's actual requirement rather than
loosening the escaping. Neither touched the expression under test.

**Count reporting is now accurate.** After task 01's miscount, the dispatch template requires counts from
`build/test-results/test/*.xml` or `just test census`. This report's figures (42 ran, 19 skipped) match the
orchestrator's independent measurement exactly.

**Note on gate order.** `fest next` points at `02_article_foundation/10_fest_commit.md`, which stays pending because its
only unfinished item is the merge of PR #4, and the merge was denied to the orchestrator twice (see that gate's
results). This sequence runs ahead on a branch stacked on `feat/article-foundation`; nothing here is pushed until the
parent merges.
