---
fest_type: task
fest_id: 02_search_repository_query.md
fest_name: search_repository_query
fest_parent: 03_article_search
fest_order: 2
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.648697-06:00
fest_tracking: true
---

# Task: Add the search query to ArticleRepository

## Objective

Add `ArticleRepository.search`, returning one page of articles whose title or body contains the term, plus the total match count.

## Requirements

- [ ] Matching uses the expression proven in task 01 on both `title` and `body`, ordered by `createdAt` descending then `id` descending, and paged with `limit(n, offset)` (Exposed `AbstractQuery.kt:41`) (D004).
- [ ] The total count ignores `limit` and `offset` (D007), and the page is mapped through the existing `toArticles(rows, viewerEmail)` from `02_article_foundation`
- [ ] Repository tests cover: title-only match, body-only match, case-insensitivity, `%` and `_` as literals, ordering, and total count vs. page size

## Implementation

**Steps**

1. In `ArticleRepository.kt`, add:
   ```kotlin
   data class ArticlePage(val articles: List<Article>, val total: Long)

   fun search(term: String, limit: Int, offset: Long, viewerEmail: String?): ArticlePage = transaction {
       val pattern = LikePattern("%", '\\') + LikePattern.ofLiteral(term.lowercase()) + "%"
       val matches = (Articles.title.lowerCase() like pattern) or (Articles.body.lowerCase() like pattern)
       val total = Articles.select { matches }.count()
       val rows = (Articles innerJoin Users).select { matches }
           .orderBy(Articles.createdAt to SortOrder.DESC, Articles.id to SortOrder.DESC)
           .limit(limit, offset)
           .toList()
       ArticlePage(toArticles(rows, viewerEmail), total)
   }
   ```
   - **Case folding.** The term is lowercased in Kotlin and the columns with SQL `lowerCase()`, so both sides fold case the same way.
   - **Escaping.** `LikePattern.ofLiteral` escapes `%`, `_` and the escape character (`SQLExpressionBuilder.kt:150-178`). H2 uses the default special characters (`vendors/Default.kt:669`).
   - **Ordering.** Use one `orderBy(vararg)` call so both sort keys apply (`AbstractQuery.kt:46-48`).
   - **`ArticlePage` placement.** If the service needs it outside the repository package, move it to `src/main/kotlin/io/realworld/app/domain/Article.kt`, next to `ArticlesDTO` (`Article.kt:7`).
2. Add `src/test/kotlin/io/realworld/app/domain/repository/ArticleSearchRepositoryTest.kt`, with the same database setup as task 01.
   - **Test data.** Insert articles directly with Exposed and explicit `createdAt` values, so ordering is deterministic. Each test embeds a per-test UUID token in the title or body, and searches for that token, so other rows cannot match.
   - Title-only match: found. Body-only match: found.
   - A query in a different case from the stored text still matches.
   - **Literal characters.** With a stored title `"<token>100% pure_x"`:
     - `search("<token>100%")` matches it;
     - `search("<token>100_")` must not match a second title, `"<token>1000 purex"`.
   - **Ordering.** Three matches with increasing `createdAt` come back newest first. With equal `createdAt`, the higher id comes first.
   - **Paging.** `search(token, limit = 2, offset = 0)` returns 2 articles with `total == 3`, and `offset = 2` returns 1.
3. Run `just test only ArticleSearchRepositoryTest`.

**Error paths**

- **`total` counts unrelated rows:** either `matches` is not applied to the count query, or a test searched a term that is not unique.
- **The `%` case matches everything:** the pattern was built with `LikePattern(term)` or string concatenation instead of `ofLiteral`.

## Done When

- [ ] All requirements met
- [ ] `just test only ArticleSearchRepositoryTest` passes every listed case
