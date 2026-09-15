---
fest_type: task
fest_id: 02_favorite_endpoints_and_real_counts.md
fest_name: favorite_endpoints_and_real_counts
fest_parent: 04_popular_articles
fest_order: 2
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.749732-06:00
fest_tracking: true
---

# Task: Wire the favorite and unfavorite endpoints

## Objective

Wire `POST` and `DELETE /articles/{slug}/favorite` to the repository through `ArticleService`, responding with the article's current state.

## Requirements

- [ ] The routes stay inside the mandatory auth block (`Router.kt:55-57`), and the handlers respond with `ctx.respond(ArticleDTO(...))` (D005).
- [ ] Unknown slug → 404; repeating a favorite or unfavorite → 200 with the state unchanged; no token → 401
- [ ] Every article response (create, search, favorite) now reports real `favorited` and `favoritesCount` values

## Implementation

**Steps**

1. **`ArticleService`.** Add two methods:
   ```kotlin
   fun favorite(email: String, slug: String): Article {
       require(slug.isNotBlank()) { "slug is required." }
       return articleRepository.favorite(email, slug)
   }

   fun unfavorite(email: String, slug: String): Article {
       require(slug.isNotBlank()) { "slug is required." }
       return articleRepository.unfavorite(email, slug)
   }
   ```
2. **`ArticleController`.** Replace the stub `favorite` (`:62`) and `unfavorite` (`:70`):
   ```kotlin
   suspend fun favorite(ctx: ApplicationCall) {
       val email = ctx.authentication.principal<User>()?.email
       require(!email.isNullOrBlank()) { "User not logged." }
       val slug = requireNotNull(ctx.parameters["slug"]) { "slug is required." }
       ctx.respond(ArticleDTO(articleService.favorite(email, slug)))
   }

   suspend fun unfavorite(ctx: ApplicationCall) {
       val email = ctx.authentication.principal<User>()?.email
       require(!email.isNullOrBlank()) { "User not logged." }
       val slug = requireNotNull(ctx.parameters["slug"]) { "slug is required." }
       ctx.respond(ArticleDTO(articleService.unfavorite(email, slug)))
   }
   ```
   No routing change is needed. `Router.kt:56` and `:57` already call these handlers inside `route("{slug}")`.
3. **Check it against a running container.** Start one with `just docker up`, create an article with a token, then:
   - `POST /articles/<slug>/favorite` twice → both 200, with `favoritesCount` 1 and `favorited` true.
   - `DELETE /articles/<slug>/favorite` twice → both 200, with `favoritesCount` 0.
   - `POST /articles/does-not-exist/favorite` → 404.
   - The same POST without a token → 401.

   Stop it with `just docker down`.

**Error paths**

- **404 on a real slug:** the request path used `/api` (D002 keeps root), or the slug was URL-encoded twice.
- **500 on an unknown slug:** a null escaped instead of `NotFoundException`; look at `userAndArticle` in task 01.

## Done When

- [ ] All requirements met
- [ ] Against a running container: repeated favorite and unfavorite return 200 with `favoritesCount` 1 and then 0; unknown slug → 404; no token → 401
