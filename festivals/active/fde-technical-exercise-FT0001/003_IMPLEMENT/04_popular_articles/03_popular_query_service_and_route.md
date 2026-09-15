---
fest_type: task
fest_id: 03_popular_query_service_and_route.md
fest_name: popular_query_service_and_route
fest_parent: 04_popular_articles
fest_order: 3
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.769687-06:00
fest_tracking: true
---

# Task: Add GET /articles/feed/popular

## Objective

Add `GET /articles/feed/popular`: every article, ordered by favorite count, then `createdAt`, then `id`, paged, public, and returning the total count.

## Requirements

- [ ] Ordering and inclusion follow D005. Zero-favorite articles are included, and ties break by `createdAt` descending and then `id` descending, so pages are stable.
- [ ] The ranking query selects only article id, `createdAt` and the favorite count, grouped by those two columns, which H2 accepts. The page's articles are then loaded and kept in ranked order
- [ ] `limit`/`offset` are validated with `Paging.parse` from `03_article_search`
- [ ] `get("feed/popular")` is added to the public `authenticate(optional = true)` block registered before the mandatory block (D003)

## Implementation

**Steps**

1. **Repository.** In `ArticleRepository`, add:
   ```kotlin
   fun popular(limit: Int, offset: Long, viewerEmail: String?): ArticlePage = transaction {
       val favCount = ArticleFavorites.user.count()
       val rankedIds = Articles.leftJoin(ArticleFavorites)
           .slice(Articles.id, Articles.createdAt, favCount)
           .selectAll()
           .groupBy(Articles.id, Articles.createdAt)
           .orderBy(favCount to SortOrder.DESC, Articles.createdAt to SortOrder.DESC, Articles.id to SortOrder.DESC)
           .limit(limit, offset)
           .map { it[Articles.id] }
       val total = Articles.selectAll().count()
       val rowsById = (Articles innerJoin Users).select { Articles.id inList rankedIds }.associateBy { it[Articles.id] }
       ArticlePage(toArticles(rankedIds.mapNotNull { rowsById[it] }, viewerEmail), total)
   }
   ```
   - **Count `ArticleFavorites.user`, not `*`.** After a LEFT JOIN, an article with no favorites still yields one row, with `user` NULL. `COUNT(user)` scores it 0, where `COUNT(*)` would score it 1.
   - `Articles.leftJoin(ArticleFavorites)` resolves through the single foreign key `ArticleFavorites.article`.
   - `toArticles` maps rows in input order, so the page keeps its ranking.
2. **Service.** In `ArticleService`, add:
   ```kotlin
   fun popular(limit: String?, offset: String?, viewerEmail: String?): ArticlesDTO {
       val paging = Paging.parse(limit, offset)
       val page = articleRepository.popular(paging.limit, paging.offset, viewerEmail)
       return ArticlesDTO(page.articles, Math.toIntExact(page.total))
   }
   ```
3. **Controller.** Add `popular` to `ArticleController`, shaped like `search`:
   ```kotlin
   suspend fun popular(ctx: ApplicationCall) {
       val viewer = ctx.authentication.principal<User>()?.email
       ctx.respond(articleService.popular(ctx.parameters["limit"], ctx.parameters["offset"], viewer))
   }
   ```
4. **Route.** In `Router.kt`, inside the public block that `03_article_search` task 04 added, add `get("feed/popular") { articleController.popular(this.context) }` next to `get("search")`.
   - The personal feed `GET /articles/feed` still resolves inside the mandatory block. In the public block, the `feed` segment has no handler of its own, so that branch fails and the mandatory block's `get("feed")` is used (Ktor 1.2.3 `RoutingResolve.kt:155-171`).
   - The personal feed is still a stub either way.
5. **Check it against a running container.**
   - Create three articles, and have two different users favorite one of them → that article comes first.
   - Anonymous `GET /articles/feed/popular` → 200.
   - `?limit=0` → 422.

**Error paths**

- **H2 reports a column "must be in the GROUP BY list":** `slice` includes a column that is not grouped, for example `title`. Keep `slice` to id, `createdAt` and the count.
- **Zero-favorite articles rank above favorited ones:** the count used `*` or the wrong column.

## Done When

- [ ] All requirements met
- [ ] Against a running container: `GET /articles/feed/popular` without a token returns 200 with the most-favorited article first and `articlesCount` equal to the total number of articles; `?limit=0` returns 422
