# D005: Popular Articles ordering, pagination, and favorite writes

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R1, R2, R14, R15; brief `docs/interview-exercise.md:56-58`

## Context

The brief asks for articles "ordered by number of favorites, most popular first", with `limit` and
`offset` pagination. It does not say:

- how ties are broken,
- whether unfavorited articles appear,
- what favoriting an article twice does.

Nothing can be popular until favorites exist. The favorite and unfavorite routes are wired but stubbed.

Exposed 0.41.1 provides `ExpressionWithColumnType.count()` (`SQLExpressionBuilder.kt:58`), `Query.groupBy`
(`Query.kt:164`), `orderBy(vararg Pair<Expression<*>, SortOrder>)` (`AbstractQuery.kt:46-48`), and
`limit(n: Int, offset: Long)` (`AbstractQuery.kt:41`).

## Options

### Order
- **By favorite count only.** Rejected. Articles with equal counts come back in arbitrary order, so
  offset paging can repeat or skip them between requests.
- **Count, then `createdAt`, then `id`** (chosen). A total order makes every page deterministic.
- **Time-decayed "trending".** Rejected. Nobody asked for it, and it is harder to verify.

### Which articles
- **Only articles with at least one favorite.** Rejected. A fresh database returns an empty feed, and the
  brief says "articles ordered by number of favorites", not "favorited articles".
- **Every article, zero-favorite ones last** (chosen).

### Repeated writes
- **Insert blindly.** Rejected. A second favorite violates the composite key and surfaces as a 500.
- **Idempotent writes** (chosen).

## Decision

- **Schema.** `ArticleFavorites(user → Users, article → Articles)`, primary key `(user, article)`. It
  follows the `Follows` pattern (`UserRepository.kt:43`).
- **Favorite writes.**
  - `POST /articles/{slug}/favorite` and `DELETE …/favorite` keep their existing mandatory-auth routes.
  - Favoriting an already-favorited article, or unfavoriting one that is not favorited, is a no-op that
    returns 200 with the article's current state. The check and the write run in one transaction.
  - An unknown slug → **404** via `NotFoundException` (`ErrorExceptionMapping.kt:34`).
- **Article fields, everywhere an article is returned.** `favoritesCount` is the number of favorite rows
  for that article. `favorited` is true only when the signed-in viewer has favorited it, and false for
  anonymous callers.
- **Endpoint.** `GET /articles/feed/popular?limit=<n>&offset=<n>`, public (D003).
- **Order.** Favorite count descending, then `createdAt` descending, then `id` descending.
- **Paging.** Same defaults and bounds as D004: 20 and 0; `limit` 1..100; `offset` ≥ 0; violations → 422.
- **Response.** `ArticlesDTO`, with `articlesCount` as the total number of articles (D007).

## Consequences

- **Unverified: H2's GROUP BY rules.** H2 may reject selecting article columns that are neither
  aggregated nor listed in `GROUP BY`. The popular task uses a form H2 accepts: either group by every
  selected `Articles` column, or count favorites per article id in a subquery and join to it. A test
  proves the choice.
- **Tests must cover:**
  - ordering by count;
  - equal counts ordered by recency;
  - two consecutive pages that neither overlap nor skip;
  - zero-favorite articles included;
  - an offset past the end → empty list with the correct `articlesCount`;
  - bad `limit` → 422;
  - an anonymous request → 200;
  - `favorited` true for the favoriting viewer and false anonymously;
  - favoriting twice → still 200, with `favoritesCount` unchanged.
- **Test data.** Rows persist across test methods (D009). Tests that assert order create their own
  articles with distinct favorite counts, and read their own articles' relative positions rather than
  absolute ranks.
