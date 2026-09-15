# D004: Article Search semantics

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R1, R2, R15; brief `docs/interview-exercise.md:88-89`

## Context

The brief says: "Search article titles and body content and return results using the existing
article-list response format." It does not define:

- how matching works,
- what an empty or missing `q` does,
- whether results are paged or ordered.

Exposed 0.41.1 (upstream source at tag `0.41.1`) has what a literal-safe, case-insensitive match needs:

- `lowerCase()` (`exposed-core/.../sql/SQLExpressionBuilder.kt:20`).
- `LikePattern(pattern, escapeChar)` and `LikePattern.ofLiteral(text, escapeChar = '\')`
  (`SQLExpressionBuilder.kt:135-178`). `ofLiteral` escapes the escape character and the dialect's
  special characters. On H2 those are the defaults, `%` and `_` (`vendors/Default.kt:669`; the H2
  override is commented out at `vendors/H2.kt:215`).
- `like(LikePattern)`, which builds a `LikeEscapeOp` that emits `ESCAPE` (`SQLExpressionBuilder.kt:409-410`,
  `Op.kt:479-486`).
- `LikePattern.plus` refuses to join patterns with different escape characters
  (`SQLExpressionBuilder.kt:141`). So the wildcard pieces must be built with the same escape character.

## Options

### Option A: Case-insensitive substring match with LIKE, the term escaped as a literal
- **Pros:** predictable. `%` and `_` in a query match themselves. Portable, and pagination is done by the
  database.
- **Cons:** no ranking and no word splitting.

### Option B: H2 full-text search (`FT_INIT`, Lucene)
- **Pros:** ranking and word matching.
- **Cons:** needs H2-specific setup, doesn't carry over to any other database, and is far beyond
  "a small product improvement".

### Option C: Filter in Kotlin after loading every article
- **Pros:** trivial to write.
- **Cons:** loads the whole table per request, and pagination and counts become application logic.

### Option D: Unescaped LIKE
- **Pros:** one line.
- **Cons:** `q=%` matches everything and `_` becomes a wildcard. It is a correctness bug waiting for
  input.

## Decision

**Option A**, with these rules:

- **Endpoint.** `GET /articles/search?q=<term>&limit=<n>&offset=<n>`, public (D003), mounted at root
  (D002).
- **`q`.** Required. It is trimmed. Missing or blank after trimming → **422** via `require(...)`, which
  maps through `ErrorExceptionMapping.kt:38`. The whole trimmed phrase is one substring; it is not
  split into words.
- **Match.** `lower(title) LIKE p` or `lower(body) LIKE p`, where
  `p = LikePattern("%", '\\') + LikePattern.ofLiteral(q.trim().lowercase()) + "%"`. `description` is not
  searched, because the brief names titles and body only.
- **Order.** `createdAt` descending, then `id` descending. Newest first, with a total order so pages are
  stable.
- **Paging.**
  - `limit` defaults to 20 and `offset` to 0, matching the stubbed list handler
    (`ArticleController.kt:15-16`).
  - `limit` must be 1..100, `offset` must be ≥ 0, and a non-integer is rejected. Every violation → 422
    with a message naming the parameter.
- **Response.** `ArticlesDTO(articles, articlesCount)`, where `articlesCount` is the total number of
  matches (D007), authors are `Profile`s (D008), and tags are included.
  - Before the Popular slice, `favorited` is `false` and `favoritesCount` is `0`. That is true, because
    no favorite can exist yet.
  - From the Popular slice on, both carry real values.

## Consequences

- **Unverified: LOWER on a CLOB column.** Exposed maps `text()` to `TEXT` (`vendors/Default.kt:63`),
  which H2 stores as a CLOB. Whether `LOWER` works on a CLOB column there has not been checked. The
  search task's first test is a match found **only in `body`**. If H2 rejects it, the task stops, records
  the failure under `results/`, and amends this decision before changing the column type or the query.
- **Tests must cover:**
  - a title-only match and a body-only match;
  - case-insensitivity;
  - no match, giving an empty list with `articlesCount` 0;
  - blank and missing `q` → 422;
  - `%` and `_` treated as literal characters;
  - a limit below the total, giving page size < `articlesCount`;
  - a bad `limit` or `offset` → 422;
  - an anonymous request → 200 with an `articles` list (pins D003).
- **Test data.** Rows persist across test methods in one JVM (D009), so every test uses unique titles and
  body text and asserts on its own data only.
