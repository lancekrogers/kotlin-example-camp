---
fest_type: task
fest_id: 02_profile_stats_service_and_route.md
fest_name: profile_stats_service_and_route
fest_parent: 05_user_activity
fest_order: 2
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.830974-06:00
fest_updated: 2026-09-16T03:20:47.425858-06:00
fest_tracking: true
---


# Task: Add GET /profiles/{username}/stats

## Objective

Add `GET /profiles/{username}/stats`, returning how many articles the user authored, comments they wrote, and favorites they gave.

## Requirements

- [ ] `ProfileStats(articlesCount, commentsCount, favoritesCount)` and `ProfileStatsDTO(stats)` go in `Profile.kt`, and `ProfileStatsService(userRepository, articleRepository, commentRepository)` computes the counts exactly as D006 defines them.
- [ ] An unknown username → 404; a user with no activity → 200 with zeros. The route is public, inside the existing `authenticate(optional = true)` block (`Router.kt:31`) (D003)
- [ ] `ProfileController` gains only `stats`; its stubbed `get`, `follow` and `unfollow` stay stubbed

## Implementation

**Steps**

1. **DTOs.** In `src/main/kotlin/io/realworld/app/domain/Profile.kt`, after `Profile` (`:8`), add:
   ```kotlin
   data class ProfileStats(val articlesCount: Long, val commentsCount: Long, val favoritesCount: Long)
   data class ProfileStatsDTO(val stats: ProfileStats)
   ```
2. **Repository counts.** In `ArticleRepository`, add:
   ```kotlin
   fun countByAuthor(userId: Long): Long = transaction { Articles.select { Articles.author eq userId }.count() }
   fun countFavoritesBy(userId: Long): Long = transaction { ArticleFavorites.select { ArticleFavorites.user eq userId }.count() }
   ```
3. **Service.** Create `src/main/kotlin/io/realworld/app/domain/service/ProfileStatsService.kt`:
   ```kotlin
   class ProfileStatsService(
       private val users: UserRepository,
       private val articles: ArticleRepository,
       private val comments: CommentRepository
   ) {
       fun stats(username: String): ProfileStats {
           val user = users.findByUsername(username) ?: throw NotFoundException("Profile not found.")
           val id = requireNotNull(user.id)
           return ProfileStats(articles.countByAuthor(id), comments.countByAuthor(id), articles.countFavoritesBy(id))
       }
   }
   ```
   `UserRepository.findByUsername` already exists (`UserRepository.kt:62`).
4. **Controller.** In `ProfileController`, add the constructor `(private val profileStatsService: ProfileStatsService)` and this method, leaving the existing stubs untouched:
   ```kotlin
   suspend fun stats(ctx: ApplicationCall) {
       val username = requireNotNull(ctx.parameters["username"]) { "username is required." }
       ctx.respond(ProfileStatsDTO(profileStatsService.stats(username)))
   }
   ```
5. **Kodein.** In the PROFILE module (`ModulesConfig.kt:27`), bind `ProfileStatsService(instance(), instance(), instance())` and change the controller binding to `ProfileController(instance())`. The other repositories are already bound in their own modules: `UserRepository` in USER, `ArticleRepository` in ARTICLE, `CommentRepository` in COMMENT. Kodein resolves across imported modules.
6. **Route.** In `Routing.profiles`, add `get("stats") { profileController.stats(this.context) }` inside the existing optional block (`Router.kt:31`), next to `get { profileController.get(this.context) }` (`:32`). Do not put it in the mandatory `follow` block (`:34`).
7. **Check it against a running container.**
   - A new user's stats are all zero.
   - After that user creates one article, comments on another user's article, and favorites another user's article, the stats are `1/1/1`.
   - An unknown username → 404.
   - An anonymous request → 200.

**Error paths**

- **Anonymous requests get 401:** the route was placed inside the mandatory `follow` block.
- **Kodein `NotFoundException: No binding found`:** a binding is missing for `CommentRepository` or `ProfileStatsService`.

## Done When

- [ ] All requirements met
- [ ] Against a running container, a new user's stats are all zero; after one article, one comment and one favorite of another user's article, they are `1/1/1`; an unknown username returns 404; a request with no token returns 200