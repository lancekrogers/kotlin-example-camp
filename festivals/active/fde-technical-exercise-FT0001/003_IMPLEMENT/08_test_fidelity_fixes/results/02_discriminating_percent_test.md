# Evidence: the `%` literal-wildcard tests can now fail

## Changes

| File | Change |
|---|---|
| `src/test/.../repository/ArticleSearchRepositoryTest.kt` | `percent sign in search term is literal` gained a decoy article and asserts the matched title |
| `src/test/.../controllers/ArticleSearchTest.kt` | the percent half of `percent and underscore are literal` gained the same decoy and title assertion |

The underscore assertions were not touched. No assertion was deleted or loosened.

## Why a decoy rather than a different assertion

The plan first said "assert 0 matches for a term containing `%`". Reading the pattern more carefully, a stronger form
keeps the positive assertion and adds a row that only a broken implementation would match:

- **Escaped (correct):** `%…100\%%` — requires a literal `%`. Matches `…100% pure_x`, not `…100XX plain`.
- **Unescaped (broken):** `%…100%%` — `%%` collapses to `%`. Matches both.

So a correct implementation returns exactly one row and a broken one returns two. That keeps proof that the term
matches what it should, and adds proof that it does not match what it should not.

## Red: escaping deliberately removed

`ArticleRepository.search`'s pattern replaced with an unescaped one:

```kotlin
// TEMPORARY, red-path proof only. Revert.
val pattern = LikePattern("%" + term.lowercase() + "%", '\\')
```

```text
ArticleSearchRepositoryTest > percent sign in search term is literal FAILED
    java.lang.AssertionError: expected:<1> but was:<2>
        at ...ArticleSearchRepositoryTest.percent sign in search term is literal(ArticleSearchRepositoryTest.kt:101)
ArticleSearchRepositoryTest > underscore in search term is literal FAILED
    java.lang.AssertionError: expected:<0> but was:<1>
        at ...ArticleSearchRepositoryTest.underscore in search term is literal(ArticleSearchRepositoryTest.kt:113)
ArticleSearchTest > percent and underscore are literal FAILED
    java.lang.AssertionError: expected:<1> but was:<2>
        at ...ArticleSearchTest.percent and underscore are literal(ArticleSearchTest.kt:115)
> Task :test FAILED
BUILD FAILED in 30s
```

`expected:<1> but was:<2>` is the decoy being matched — the exact failure the previous version of this test could not
produce. The underscore test also failing (`expected:<0> but was:<1>`) confirms it was genuinely discriminating all
along, which is why the escaping was never actually broken in the shipped code.

## Green: reverted

```console
$ git checkout -- src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
$ grep -n 'val pattern' src/main/.../ArticleRepository.kt
101:        val pattern = LikePattern("%", '\\') + LikePattern.ofLiteral(term.lowercase()) + "%"
$ git status --short src/main | wc -l
0
```

```text
ArticleSearchRepositoryTest > percent sign in search term is literal PASSED
ArticleSearchRepositoryTest > underscore in search term is literal PASSED
ArticleSearchTest > percent and underscore are literal PASSED
BUILD SUCCESSFUL in 42s
```

Every run in this file used `--rerun-tasks --no-build-cache`, because a reverted file returns the build to a
previously cached state and Gradle will otherwise report success without executing anything.

## Relationship to the live API evidence

`003_IMPLEMENT/output_specs/requirements.md` records `GET /articles/search?q=%` returning `articlesCount: 0` while a
matching article existed — the same property, proven against a container. This task moves it into the suite, where it
guards against regression instead of living only in a transcript.
