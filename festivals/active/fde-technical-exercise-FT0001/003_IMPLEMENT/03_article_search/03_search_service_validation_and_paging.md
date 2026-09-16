---
fest_type: task
fest_id: 03_search_service_validation_and_paging.md
fest_name: search_service_validation_and_paging
fest_parent: 03_article_search
fest_order: 3
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.669691-06:00
fest_updated: 2026-09-16T02:11:12.424235-06:00
fest_tracking: true
---


# Task: Validate search input and paging

## Objective

Add `ArticleService.search` and a shared `Paging` parser that turn raw query parameters into validated inputs, rejecting bad ones with 422.

## Requirements

- [ ] `q` is required and trimmed. A missing or blank `q` throws `IllegalArgumentException`, which maps to 422 (`ErrorExceptionMapping.kt:38`) (D004).
- [ ] `limit` defaults to 20 and must be an integer in 1..100. `offset` defaults to 0 and must be an integer ≥ 0. Each violation returns 422 with a message naming the parameter, and `04_popular_articles` reuses the same parser
- [ ] The service returns `ArticlesDTO(articles, articlesCount = total)` (D007)

## Implementation

**Steps**

1. Create `src/main/kotlin/io/realworld/app/domain/Paging.kt`:
   ```kotlin
   package io.realworld.app.domain

   data class Paging(val limit: Int, val offset: Long) {
       companion object {
           const val DEFAULT_LIMIT = 20
           const val MAX_LIMIT = 100

           fun parse(limit: String?, offset: String?): Paging {
               val l = if (limit == null) DEFAULT_LIMIT
                       else requireNotNull(limit.toIntOrNull()) { "limit must be an integer." }
               require(l in 1..MAX_LIMIT) { "limit must be between 1 and $MAX_LIMIT." }
               val o = if (offset == null) 0L
                       else requireNotNull(offset.toLongOrNull()) { "offset must be an integer." }
               require(o >= 0) { "offset must not be negative." }
               return Paging(l, o)
           }
       }
   }
   ```
   The defaults match the author's stub (`ArticleController.kt:15-16`). `requireNotNull` and `require` both throw `IllegalArgumentException`, which maps to 422.
2. In `ArticleService.kt`, add:
   ```kotlin
   fun search(q: String?, limit: String?, offset: String?, viewerEmail: String?): ArticlesDTO {
       val term = q?.trim()
       require(!term.isNullOrEmpty()) { "q is required." }
       val paging = Paging.parse(limit, offset)
       val page = articleRepository.search(term, paging.limit, paging.offset, viewerEmail)
       return ArticlesDTO(page.articles, Math.toIntExact(page.total))
   }
   ```
   `ArticlesDTO.articlesCount` is an `Int` (`Article.kt:7`). `Math.toIntExact` makes an impossible overflow fail loudly instead of wrapping.
3. Add `src/test/kotlin/io/realworld/app/domain/PagingTest.kt` (plain JUnit):
   - Defaults are `(20, 0)`.
   - `"1"` and `"100"` are accepted.
   - Limit `"0"`, limit `"101"`, offset `"-1"` and limit `"abc"` each throw `IllegalArgumentException` whose message names the parameter.
4. Run `just test only PagingTest`, then `just build compile`.

**Error paths**

- **`"abc"` produces a `NumberFormatException` message:** `toInt()` was used instead of `toIntOrNull()`. The message must name the parameter.
- **The service does not compile against the repository:** task 02's `search` signature changed. Keep the two in step rather than widening either.

## Done When

- [ ] All requirements met
- [ ] `just test only PagingTest` passes, and `ArticleService.search` compiles against task 02's repository method